---
title: "Building Freight Agent AWS: A Serverless LLM-Powered Freight Bidding Orchestrator"
layout: post
---

In the logistics industry, freight brokers act as middle-men between shippers (customers) who need cargo moved and carriers (trucking companies) who own the assets to transport it. Historically, this matching process is incredibly manual: a broker receives a free-text email from a customer, parses the shipment lanes, copies the details into multiple emails to carriers to solicit quotes, waits for bids, triggers a negotiation round, adds a margin, proposes a price back to the customer, and finally drafts a Bill of Lading (BOL).

To automate this high-friction email loop, I created **Freight Agent AWS**—an automated competitive freight bidding platform. Built using a serverless architecture on **AWS Lambda, API Gateway, and Amazon S3**, it orchestrates the entire bidding lifecycle using **Cerebras LLM (Llama-3.1)** for sub-second email intelligence, and **PostgreSQL with pgvector** to match historical shipping lane data.

In this post, we'll walk through the system's architecture, explain the bidding cycle, and break down the anatomy of the email connector that acts as the platform's nervous system.

---

## 💡 The Problem: The Manual Email Bottleneck

Freight brokers are constantly flooded with unstructured inquiry emails containing shipping requests like:
> *"Need a quote for 4 pallets of class 70 auto parts from Detroit, MI to Laredo, TX for next Tuesday. Weight is 8,500 lbs, dock-to-dock, no hazmat."*

A typical broker must:
1.  **Parse details manually** (Origin, Destination, Weight, Class, Hazmat status).
2.  **Benchmark the lane** using historical spreadsheet rates to estimate standard costs.
3.  **Email multiple carriers** to request quotes.
4.  **Manage bidding deadlines** and follow up to get the best pricing.
5.  **Trigger a Re-Bid round** to let carriers lower their rates when margins are tight.
6.  **Apply markups and propose** the final quote to the customer.

By automating this sequence, brokers can handle 10x the volume, eliminate data-entry errors, and secure the lowest carrier costs in real-time.

---

## 🏗️ 1. Core Quote State & Bid Cycle Flow

The lifecycle of a quote is managed by a state machine that transitions across 10 stages: `INTAKE` -> `OUT_TO_CARRIERS` -> `FIRST_ROUND_RECEIVED` -> `RE_BID_ROUND` -> `QUOTE_SENT` -> `AWAITING_APPROVAL` -> `APPROVED` -> `IN_TRANSIT` -> `COMPLETED` -> `LOST`.

Here is how the automated workflow operates when an inquiry arrives:

```mermaid
flowchart TD
    Customer["Customer Email"] -->|inbound email| Mailbox["Mailpit / Gmail Inbox"]
    Mailbox -->|webhook / polling| Ingestion["Intake Service"]
    
    subgraph Backend ["FastAPI Application"]
        Ingestion -->|1. Parse Email| LLM["Cerebras LLM Service"]
        LLM -->|Extract parameters| DB[("PostgreSQL + pgvector")]
        
        DB -->|2. RAG Query| RAG["Embedding & Historical RAG"]
        RAG -->|Lane benchmark| Orchestrator["Workflow Orchestrator"]
        
        Orchestrator -->|3. Status OUT_TO_CARRIERS| Outbound["SMTP Service"]
        Outbound -->|RFQ Email| Carriers["Carriers Network"]
        
        Carriers -->|4. Submit Bids| BidIngestion["Bid Parser"]
        BidIngestion -->|Record Bids| DB
        
        Orchestrator -->|5. Status RE_BID_ROUND| Rebid["Best & Final Trigger"]
        Rebid -->|Cheapest rate notification| Outbound
        
        Orchestrator -->|6. Status QUOTE_SENT| Markup["Markup Engine"]
        Markup -->|Apply Customer rules| CustProposal["Customer Quote Compose"]
        CustProposal -->|Customer Proposal Email| Customer
        
        Customer -->|7. Approved| WorkflowApprove["Booking Logic"]
        Customer -->|7. Rejected| WorkflowReject["Loss Logger"]
        
        WorkflowApprove -->|Proceed Booking| Outbound
        WorkflowApprove -->|BOL generation| BOLService["BOL & PandaDoc Service"]
        BOLService -->|Draft PandaDoc invoice| DB
    end
    
    subgraph Frontend ["React Vite Client"]
        UI["Web Dashboard"] -->|Get pipeline stats / Trigger mock actions| Backend
        UI -->|Pipeline Kanban Board| Broker["Freight Broker"]
        UI -->|Analytics Center| Broker
        UI -->|Invoice Ledger| Broker
    end
```

### Flow Highlights
*   **Intake Parsing:** A Python ingestion service watches the broker's mailbox. When an email is fetched, **Cerebras Llama-3.1-70B** extracts key logistics attributes in sub-second inference speed.
*   **Historical Lane RAG:** The backend vectorizes the shipping lane (e.g. Detroit to Laredo) and queries the PostgreSQL database using **pgvector** similarity search (`<=>`) to fetch historical prices, helping the broker benchmark the bids.
*   **Automated Carrier RFQs:** The engine automatically emails a selected network of carriers with an RFQ. As carriers reply with pricing and transit times, the email system parses the values and inserts them into the DB.
*   **Re-Bid Trigger:** If a carrier submits a bid, the orchestrator triggers a time-bound "Re-Bid Round", alerting other carriers of the current lowest price (without revealing identity) to drive down costs.
*   **Approval & BOL Drafting:** Once the bid window closes, the broker approves the best price on their React Kanban board. The system automatically creates a Bill of Lading (BOL), syncs details to the invoice ledger, and drafts booking confirmations.

---

## ☁️ 2. AWS Serverless Architecture

To make this application highly scalable, cost-efficient, and easy to maintain, it is built entirely using serverless components on AWS. The FastAPI backend is adapted to AWS Lambda via Mangum, and S3 is utilized for both static web hosting and attachment storage.

```mermaid
graph TD
    subgraph Client ["Client Tier"]
        Browser["React Frontend - Vite"]
    end

    subgraph AWS ["AWS Cloud Infrastructure"]
        subgraph StaticHosting ["S3 Web Hosting"]
            FrontendBucket["S3 Frontend Bucket - freight-agent-frontend-dev-*"]
        end

        subgraph APIGateway ["API Gateway"]
            HttpApi["HTTP API Gateway - proxy path"]
        end

        subgraph Compute ["Serverless Compute"]
            ApiLambda["FastAPI App Lambda - app.main.handler"]
            CronLambda["Cron Processor Lambda - app.main.cron_handler"]
        end

        subgraph EventBridge ["Scheduling"]
            CronRule["EventBridge Rule - rate 1 minute"]
        end

        subgraph Storage ["Storage and Database"]
            Postgres["Amazon RDS PostgreSQL - with pgvector"]
            PrivateS3["Private S3 Bucket - dispatch-private"]
        end
    end

    subgraph External ["External Services"]
        Cerebras["Cerebras AI API - Llama-3-70B/8B"]
        EmailServer["Mailpit / SMTP and IMAP Server"]
        GoogleAuth["Google OAuth 2.0"]
    end

    %% Interactions
    Browser -->|Fetches Static Assets| FrontendBucket
    Browser -->|API Requests| HttpApi
    HttpApi -->|Invokes| ApiLambda

    CronRule -->|Triggers Every Minute| CronLambda

    %% Lambda Operations
    ApiLambda -->|SQL Queries and pgvector RAG| Postgres
    ApiLambda -->|Read/Write Attachments| PrivateS3
    ApiLambda -->|Google SSO / Authentication| GoogleAuth

    CronLambda -->|Read/Write Bids and Quotes| Postgres
    CronLambda -->|Check Workflow Timers| Postgres
    CronLambda -->|Poll IMAP and Send SMTP| EmailServer
    CronLambda -->|LLM Parse and Extract Bids| Cerebras
    
    %% Shared DB access
    ApiLambda -.->|Internal Service Calls| Cerebras
```

### AWS Infrastructure breakdown:
*   **FastAPI API Lambda:** Standard endpoints (e.g. quotes tracking, carrier listings, analytics summaries) are served via API Gateway routing HTTP traffic directly into a FastAPI Lambda function.
*   **Cron Processor Lambda:** A decoupled cron Lambda is invoked every minute by **Amazon EventBridge** to poll the IMAP mail server, process new emails, check active bidding timers, and transition quotes across workflow statuses.
*   **Amazon RDS PostgreSQL:** Host database storing structured carrier data, billing logs, and vector embeddings using the `pgvector` extension.
*   **Amazon S3:** The React UI is hosted statically in an S3 Bucket. A private S3 bucket is also set up to host generated files like Bill of Ladings, invoices, and email attachments.

---

## 🧬 3. Modular Application Architecture

The application's backend codebase is structured modularly. This makes it simple to add new router paths, plug in custom markup logic, or swapping LLM models.

```mermaid
graph TB
    subgraph Frontend ["Frontend UI - React/Vite"]
        UI["React Pages and State"]
        APIClient["API Client - Axios"]
        UI --> APIClient
    end

    subgraph Backend ["FastAPI Backend Application"]
        subgraph Routers ["FastAPI APIRouter Endpoints"]
            AuthRoute["auth.py - User Auth and OAuth"]
            QuotesRoute["quotes.py - Quote Management"]
            CarriersRoute["carriers.py - Carrier Management"]
            CustomersRoute["customers.py - Customer Management"]
            SimRoute["simulator.py - Round Simulation"]
            AnalyticsRoute["analytics.py - Metrics and Charts"]
            CredsRoute["email_credentials.py - SMTP settings"]
        end

        subgraph CoreServices ["Business Logic and Services"]
            Workflow["workflow.py - State Machine and Timers"]
            EmailSvc["email_service.py - SMTP and IMAP Engine"]
            AISvc["cerebras_service.py - Llama-3 Parsing and Extraction"]
            EmbedSvc["embedding_service.py - pgvector Embeddings"]
            BillingSvc["billing.py - Invoice and BOL Generation"]
            S3Storage["s3_storage.py - S3 Attachment Helper"]
        end

        subgraph DataAccess ["Database and ORM"]
            Models["models.py - SQLAlchemy Models"]
            Schemas["schemas.py - Pydantic Validation"]
            DBConnection["database.py - Database Connections"]
        end
    end

    subgraph Database ["PostgreSQL Database"]
        PGVector["pgvector Extension"]
        Tables["Relational Tables"]
    end

    %% Flow connections
    APIClient --> Routers
    
    %% Router connections to Services & Database
    QuotesRoute --> Workflow
    QuotesRoute --> EmbedSvc
    CredsRoute --> DBConnection
    AuthRoute --> DBConnection
    
    %% Services orchestration
    Workflow --> EmailSvc
    Workflow --> BillingSvc
    Workflow --> S3Storage
    
    EmailSvc --> AISvc
    EmailSvc --> S3Storage
    EmailSvc --> Workflow
    
    %% Data / Database Connections
    Workflow --> Models
    EmailSvc --> Models
    Models --> DBConnection
    DBConnection --> Tables
    EmbedSvc -.-> PGVector
```

---

## 🔌 4. The Email Connector Anatomy

The **Email Connector** is the gateway connecting our system to the external email servers. It implements a bi-directional integration:
1.  **Inbound Email Processing (IMAP):** Connects to the inbox, fetches unread emails, extracts attachments, extracts the Quote ID (e.g. `Q-1002`) using regex, parses payloads using Cerebras AI, and triggers state transitions.
2.  **Outbound Email Processing (SMTP):** composes styled HTML templates, injects workflow-driven parameters (like margins and bidding instructions), and dispatches emails.

Here is the architectural anatomy of how the email connector interacts with external mailboxes, AI systems, and storage buckets:

```mermaid
graph TD
    subgraph EmailServer ["Email Server / Sandbox (Gmail / Mailpit)"]
        Mailbox["Broker's Mailbox"]
        SMTPHost["SMTP Server (Port 587/1025)"]
        IMAPHost["IMAP Server (Port 993/143)"]
    end

    subgraph ConnectorAnatomy ["Email Connector Anatomy"]
        subgraph InboundFlow ["Inbound Connector (IMAP Client)"]
            IMAPPoller["IMAP Client Poller"]
            EmailParser["MIME parser (email.message)"]
            RegexRouter["Regex ID Extractor (Q-XXXX)"]
            LLMParser["Cerebras AI Parser (Llama 3.1)"]
            WorkflowRouter["Workflow Transition Engine"]
        end

        subgraph OutboundFlow ["Outbound Connector (SMTP Client)"]
            SMTPSender["SMTP Client Sender"]
            HTMLComposer["HTML Email Composer"]
        end
    end

    subgraph DataStore ["Database & Storage"]
        DB[("Amazon RDS Postgres")]
        S3Bucket[("S3 Attachments Bucket")]
    end

    %% Interactions
    IMAPHost -->|Fetches raw MIME| IMAPPoller
    IMAPPoller --> EmailParser
    EmailParser --> RegexRouter
    RegexRouter -->|Quote ID matched| WorkflowRouter
    RegexRouter -->|No Quote ID (New Inquiry)| LLMParser
    LLMParser -->|Extracts lane metadata| WorkflowRouter
    
    WorkflowRouter -->|Save quotes/bids| DB
    EmailParser -->|Extract PDF BOL/Attachments| S3Bucket

    HTMLComposer -->|Generates HTML payload| SMTPSender
    SMTPSender -->|Sends SMTP traffic| SMTPHost
    
    DB -->|Trigger RFQ/Proposal| HTMLComposer
```

### Connector Operations:
*   **The Mailbox Poller (`IMAP`):** Loops through the inbox. For every email fetched, it generates a hash key. It uses a `ProcessedEmail` table in the database to prevent duplicate processing.
*   **The Regex Router:** A fast check matches string tags like `Q-\d{4}` in the email subject or body.
    *   **If a match is found:** The email is classified as a reply. The system checks if it is a customer approval/rejection or a carrier bid, and updates the quote.
    *   **If no match is found:** The email is treated as a new inquiry. The raw text is passed to Cerebras LLM to parse out location coordinates, carrier requirements, and weight.
*   **The HTML Template Composer:** Composes highly structured logistics documents. For carriers, it templates a bid solicitation. For customers, it templates a markup proposal showing historical lane competitiveness scores to validate the rates.

---

## 🚀 How to Deploy the Serverless Stack

The stack is configured to deploy directly to AWS using the **Serverless Framework (v3)**.

### 1. Build and Deploy API Gateway + Lambda
Initialize your AWS credentials locally, install the NPM plugins, and run:
```bash
npm install
npx serverless deploy --stage dev
```
This automatically packs the Python code into a Lambda layer, provisions the API Gateway HTTP routes, registers the EventBridge scheduler rules, and sets up execution roles.

### 2. Deploy Frontend to S3
The React Vite application can be built and synced to its hosting bucket using the deploy script:
```bash
./deploy_frontend.sh dev
```
The script resolves your AWS Account ID, compiles the frontend bundle using the correct API Gateway URL, syncs the assets to S3, and prints the public HTTP endpoint of the broker UI.

---

## 🛠️ Tech Stack Recap

*   **Backend Core:** Python 3.11, FastAPI, Mangum, SQLAlchemy, PostgreSQL (pgvector).
*   **Serverless Infrastructure:** Serverless Framework, AWS Lambda, API Gateway, Amazon S3, EventBridge.
*   **AI Engine:** Cerebras Cloud SDK (Llama-3.1-70b-preview) for instant completions and parsing.
*   **Frontend Client:** React 18, Vite, Tailwind CSS, Axios, Lucide Icons, Recharts.

Check out the full repository and set up your serverless bidding broker agent:
👉 **[freight-agent-aws GitHub Repository](https://github.com/vishwakarma09/freight-agent-aws)**
