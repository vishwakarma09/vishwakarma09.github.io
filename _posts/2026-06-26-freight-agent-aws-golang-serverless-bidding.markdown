---
title: "Building Freight Agent AWS in Go: High-Performance Serverless Bidding Orchestrator"
layout: post
---

In the logistics industry, freight brokers act as middlemen between shippers (customers) who need cargo moved and carriers (trucking companies) who own the assets to transport it. Historically, this matching process is incredibly manual: a broker receives a free-text email from a customer, parses the shipment lanes, copies the details into multiple emails to carriers to solicit quotes, waits for bids, triggers a negotiation round, adds a margin, proposes a price back to the customer, and finally drafts a Bill of Lading (BOL).

To automate this high-friction email loop, I previously built the **[Freight Agent AWS](https://github.com/vishwakarma09/freight-agent-aws)** project in Python. As a port of that original repository to Golang, the new **[freight-agent-aws-golang](https://github.com/vishwakarma09/freight-agent-aws-golang)** project is optimized to maximize execution speed, minimize memory usage, and slash AWS Lambda cold starts to near-zero. 

In this post, we’ll explore the details of the **Freight Agent AWS - Go Migration** (`freight-agent-aws-golang`), why Go is an ideal choice for serverless logistics engines, its system architecture, and how you can run and deploy it.

---

## 🚀 Why Go? The Serverless Performance Edge

While Python is excellent for rapid prototyping and AI experimentation, Go provides significant performance advantages when deployed on AWS Lambda:

1. **Sub-Millisecond Cold Starts**: Python serverless functions (especially those loading heavy libraries like FastAPI, Pydantic, and SQLAlchemy) can suffer from cold starts of 1–3 seconds. A compiled Go binary starts execution in milliseconds.
2. **Minimal Memory Footprint**: The Go API function typically runs in less than 20MB of RAM, compared to 60MB+ for Python equivalents. This allows us to provision Lambda functions with lower memory settings (e.g., 128MB or 256MB), directly translating to lower AWS bills.
3. **Cross-Compilation for ARM64**: Go makes it trivial to cross-compile binary targets. We build for the high-efficiency `provided.al2023` ARM64 runtime, taking advantage of Graviton processors' pricing and speed.
4. **Type-Safe Reliability**: Complex state transitions benefit from Go's compile-time safety, preventing unexpected runtime null pointer reference crashes during cron evaluations.

---

## 🏗️ 1. AWS Serverless Architecture

The application is deployed using the Serverless Framework. It adapts a **Gin HTTP Router** to AWS API Gateway HTTP APIs, runs background polling via an EventBridge Scheduler, connects to PostgreSQL with `pgvector` on Amazon RDS, and handles document uploads to S3.

```mermaid
graph TD
    Client[Web Browser Client] -->|HTTPS Web App| S3Web[S3 Public Web Hosting Bucket]
    Client -->|REST API Requests| APIGateway[AWS API Gateway HTTP API v2]
    
    APIGateway -->|Proxy ANY| LambdaAPI[Lambda: API Function - Go]
    EventBridge[EventBridge Scheduler] -->|Rate: 10m| LambdaCron[Lambda: Cron Function - Go]
    
    LambdaAPI -->|Connect| RDS[Amazon RDS PostgreSQL + pgvector]
    LambdaCron -->|Connect| RDS
    
    LambdaAPI -->|Presign / Upload BOL & Invoice| S3Private[S3 Private Storage Bucket]
    
    LambdaAPI -->|Send Notifications| SMTP[SMTP Server / Mailpit]
    LambdaCron -->|Ingest Carrier Bids| IMAP[IMAP Polling / Mailpit API]
```

### AWS Infrastructure Breakdown:
* **Go API Lambda**: Serves all REST endpoints. The Gin router is wrapped using `aws-lambda-go-api-proxy/gin` to seamlessly map API Gateway payloads to standard Go HTTP handlers.
* **Go Cron Lambda**: Invoked every 10 minutes by Amazon EventBridge. It polls the IMAP mailbox, processes incoming emails, evaluates active bidding timers, and transitions quotes across statuses.
* **Amazon RDS PostgreSQL**: Stores application data and matches shipping lanes using `pgvector` similarity search (`<=>`) for historical RAG.
* **Amazon S3**: Hosts the React/Vite web application bundle publicly, and acts as private storage for generated freight documents (BOLs and Invoices).

---

## 🧬 2. Modular Application Architecture

The Go codebase is cleanly divided into entrypoints (`cmd/`) and modular business logic (`internal/`):

```mermaid
graph TD
    subgraph CMD [Entrypoints]
        API[cmd/api/main.go]
        CRON[cmd/cron/main.go]
    end

    subgraph Internal [Internal Router & Middleware]
        Setup[internal/routes/router.go]
        Auth[internal/routes/middleware.go]
    end

    subgraph Services [Business Logic Layer]
        WorkflowService[internal/services/workflow.go]
        AIService[internal/services/ai.go]
        EmailService[internal/services/email.go]
        S3Service[internal/services/s3_storage.go]
        EmbeddingService[internal/services/embedding.go]
        BillingService[internal/services/billing.go]
    end

    subgraph DB [Data Layer]
        GORM[GORM ORM Engine]
        Models[internal/models/models.go]
        RDS_PG[(PostgreSQL Database)]
    end

    API --> Setup
    Setup --> Auth
    Auth --> WorkflowService
    CRON --> WorkflowService

    WorkflowService --> AIService
    WorkflowService --> EmailService
    WorkflowService --> S3Service
    WorkflowService --> EmbeddingService
    WorkflowService --> BillingService

    WorkflowService --> GORM
    GORM --> Models
    Models --> RDS_PG
```

### Key Components:
* **Entrypoints**: `cmd/api/main.go` runs the web server locally or boots the Lambda handler if `AWS_LAMBDA_FUNCTION_NAME` is detected. `cmd/cron/main.go` runs the scheduled processing tasks.
* **Services**: Reusable modules implementing business operations. For example, `embedding.go` interfaces with OpenAI/Cerebras embeddings, while `billing.go` generates PDFs and uploads them to S3.
* **ORM Engine**: We utilize **GORM** for relational database mappings, schema migrations, and clean relational updates.

---

## ⚙️ 3. State Machine Workflow

The bidding process follows a strict state-driven pipeline, ensuring quotes transition seamlessly from intake to booking or loss:

```mermaid
stateDiagram-v2
    [*] --> INTAKE : Customer Email / API Quote Request
    INTAKE --> OUT_TO_CARRIERS : Broadcast RFQs to Carriers & Set 2-Min Timer
    
    state OUT_TO_CARRIERS {
        [*] --> AwaitFirstRoundBids
        AwaitFirstRoundBids --> EvaluateFirstRoundBids : 2-Min Timer Expires
    }
    
    EvaluateFirstRoundBids --> RE_BID_ROUND : Multiple Bids (Send current low-bid to others)
    EvaluateFirstRoundBids --> AWAITING_APPROVAL : Single Bid or No Re-bids
    
    state RE_BID_ROUND {
        [*] --> AwaitRebids
        AwaitRebids --> EvaluateFinalBids : 30-Min Timer Expires
    }
    
    EvaluateFinalBids --> AWAITING_APPROVAL
    
    AWAITING_APPROVAL --> BOOKED : Broker Approves Bid (Gen Invoice/BOL, upload to S3)
    AWAITING_APPROVAL --> LOST : Broker Rejects / Lost to Competitor
    AWAITING_APPROVAL --> CANCELLED : Manual Cancellation
    
    BOOKED --> [*]
    LOST --> [*]
    CANCELLED --> [*]
```

### State Flow Highlights:
1. **Intake Ingestion**: An incoming email inquiry is processed. The raw text is passed to **Cerebras LLM (Llama-3)** to extract structured attributes (origin, destination, weight, class).
2. **Lane Benchmarking (RAG)**: The system vectorizes the origin and destination coordinates and queries the PostgreSQL database using `pgvector` to identify similar historical shipments and estimate competitive prices.
3. **Carrier Solicitation**: The broker broadcasts the RFQ via email to a network of carrier contacts.
4. **Automated Re-Bid Trigger**: The system waits for carrier bids. If multiple carriers submit rates, a re-bid round is triggered. The system emails other carriers, notifying them of the current low-bid (without revealing names) to drive margins down.
5. **Approval and Booking**: The broker views the bids on their React-based Kanban board. Once they click "Approve", the Go backend automatically compiles a Bill of Lading (BOL), creates an invoice PDF, uploads them to the private S3 bucket, and sends booking confirmations.

---

## 🌐 Live Demo URL

You can check out the live broker dashboard to test the Go-rebuild API endpoints and watch simulated bidding rounds:
👉 **[Freight Agent Go Live Demo](http://freight-agent-frontend-go-dev-865122443732.s3-website.us-east-2.amazonaws.com/)**

---

## 🛠️ Local Development Setup

To configure and run the application locally on your machine:

### 1. Prerequisites
* **Go**: Version 1.21 or higher.
* **Node.js**: Version 18 or higher (for the React frontend).
* **PostgreSQL**: A local instance with the `pgvector` extension enabled:
  ```sql
  CREATE EXTENSION IF NOT EXISTS vector;
  ```

### 2. Configuration (`.env`)
Create a `.env` file in the project root:
```ini
ACCESS_KEY=your-aws-access-key-id
SECRET_ACCESS_KEY=your-aws-secret-access-key
API_URL=http://localhost:8000
FRONTEND_URL=http://localhost:5173
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
CEREBRAS_API_KEY=your-cerebras-key
GOOGLE_CLIENT_ID=your-google-client-id
GOOGLE_CLIENT_SECRET=your-google-client-secret

# SMTP/IMAP Email Server (e.g., Mailpit)
SMTP_HOST=localhost
SMTP_PORT=1025
SMTP_USER=
SMTP_PASSWORD=
EMAILS_FROM_EMAIL=broker@yourdomain.com
SMTP_TLS=False
```

### 3. Running the App
* **Start API Server** ( Gin HTTP router running on `:8000`):
  ```bash
  go run cmd/api/main.go
  ```
* **Trigger Cron Tasks** (Run once locally to process mail/timers):
  ```bash
  go run cmd/cron/main.go
  ```
* **Start Frontend Dashboard**:
  ```bash
  cd frontend
  npm install
  npm run dev
  ```
* **Run Integration Tests**:
  ```bash
  python3 test_api.py
  ```

---

## 🚀 Deploying to AWS

Deploying the compiled Go binaries and serverless stack is handled using the Serverless Framework:

1. **Compile Binaries**:
   ```bash
   make build
   ```
   *This compiles the Go binaries targeted for Linux ARM64 (`GOOS=linux GOARCH=arm64`) and packages them into `bin/api/api.zip` and `bin/cron/cron.zip`.*

2. **Deploy Serverless Backend**:
   ```bash
   npx serverless@3 deploy --stage dev --region us-east-2
   ```

3. **Deploy React Frontend to S3**:
   ```bash
   ./deploy_frontend.sh dev
   ```

Check out the full Go codebase and build your high-performance serverless logistics broker:
👉 **[freight-agent-aws-golang Repository](https://github.com/vishwakarma09/freight-agent-aws-golang)**
