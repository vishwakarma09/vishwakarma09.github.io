---
title: "Building Freight Agent AWS in Java: Spring Boot 3 & Java 21 Serverless Orchestrator"
layout: post
---

In the logistics industry, freight brokers act as middle-men between shippers (customers) who need cargo moved and carriers (trucking companies) who own the assets to transport it. Historically, this matching process is incredibly manual: a broker receives a free-text email from a customer, parses the shipment lanes, copies the details into multiple emails to carriers to solicit quotes, waits for bids, triggers a negotiation round, adds a margin, proposes a price back to the customer, and finally drafts a Bill of Lading (BOL).

To automate this high-friction email loop, I previously built the **[Freight Agent AWS](https://github.com/vishwakarma09/freight-agent-aws)** project in Python. As a port of that original repository to Java, the new **[freight-agent-aws-java](https://github.com/vishwakarma09/freight-agent-aws-java)** project implements the exact same competitive bidding orchestrator using **Spring Boot 3** and **Java 21**, bringing enterprise-grade type safety, robust design patterns, and high-performance serverless execution on AWS Lambda.

In this post, we'll explore why Java and Spring Boot are excellent choices for serverless logistics engines, break down the system architecture, and walk through how to run and deploy it.

---

## 🚀 Why Java & Spring Boot? The Enterprise Serverless Advantage

While Python is popular for AI prototyping and Go for raw compilation speed, migrating the backend to Java 21 and Spring Boot 3 provides unique benefits for complex logistics workflows:

1. **Strong Domain Modeling**: Java's static typing and object-oriented structure make it natural to model complex state transitions, quotes, carriers, and user accounts.
2. **Modern Java 21 Features**: Leverages modern features like records, pattern matching, and enhanced switch expressions, resulting in clean, readable, and highly maintainable business logic.
3. **Enterprise-Grade Ecosystem**: Integration with **Spring Data JPA** and **Hibernate** simplifies database orchestration, automatic schema migrations, and relational mapping.
4. **Serverless Adapters**: By using `aws-serverless-java-container-springboot3`, we can wrap a standard Spring Boot application context into a Lambda handler, exposing MVC REST endpoints directly through API Gateway proxy requests.
5. **Compile-Time Safety**: Strong compile-time validation eliminates runtime type-coercion errors, making automated cron executions and mail processing highly predictable.

---

## 🏗️ 1. AWS Serverless Architecture

The backend is deployed serverless using the Serverless Framework. It maps an Amazon API Gateway to a Spring Boot-driven AWS Lambda function, runs scheduled IMAP polling via an EventBridge Scheduler, connects to PostgreSQL with `pgvector` on Amazon RDS, and manages document storage on S3.

```mermaid
graph TD
    User([User Browser]) -->|HTTP/HTTPS| S3Frontend["S3 Static Website Bucket<br/>freight-agent-frontend-java-dev"]
    User -->|API Calls| APIGateway["Amazon API Gateway<br/>HTTP API v2.0"]
    APIGateway -->|Proxy Stream Request| LambdaAPI["AWS Lambda: API Function<br/>Spring Boot 3 / Java 21"]
    
    EventBridge["Amazon EventBridge<br/>Rate: 10 mins"] -->|Trigger| LambdaCron["AWS Lambda: Cron Function<br/>Java 21"]
    
    LambdaAPI -->|Read/Write| RDSPostgres[("Amazon RDS PostgreSQL<br/>with pgvector")]
    LambdaCron -->|Read/Write| RDSPostgres
    
    LambdaAPI -->|Presigned URLs / Upload| S3Private["S3 Private Bucket<br/>dispatch-private-java"]
    LambdaAPI -->|Send Notifications| SMTPServer["Secure SMTP Server"]
    LambdaCron -->|Poll Emails| SMTPIMAP["IMAP Email Ingestion"]
```

### AWS Infrastructure Breakdown:
* **Spring Boot API Lambda**: Handles incoming HTTP traffic. The application is loaded inside an AWS Lambda environment using custom stream handlers, directing API requests to appropriate Spring Controllers.
* **Cron Lambda**: Triggered every 10 minutes by EventBridge to poll the IMAP email inbox, parse bids, monitor active bidding timers, and transition quotes through statuses.
* **Amazon RDS PostgreSQL**: Stores relational database models and supports similarity searches (`pgvector`) to benchmark lanes against historical shipments.
* **Amazon S3**: Hosts the compiled React frontend application publicly, and hosts private documents (generated BOLs and Invoices).

---

## 🧬 2. Application Layer Architecture

The Java codebase is organized into entry handlers, web controllers, business services, infrastructure wrappers, and JPA repositories:

```mermaid
graph TD
    IncomingEvent[AWS Lambda Event / Request] --> LambdaHandler["Lambda Handlers<br/>StreamLambdaHandler / CronLambdaHandler"]
    LambdaHandler -->|HttpApiV2ProxyRequest| SpringDispatcher[Spring Boot DispatcherServlet]
    SpringDispatcher --> Controllers["REST Controllers<br/>AuthController / QuoteController / CarrierController / etc."]
    
    Controllers --> BusinessServices["Business Services<br/>WorkflowService / EmailService"]
    BusinessServices --> HelperServices["Helper / Infrastructure Services<br/>CerebrasService / EmbeddingService / S3 / Crypt"]
    
    BusinessServices --> Repositories["JPA Repositories<br/>UserRepository / QuoteRepository / etc."]
    Repositories --> Database[(PostgreSQL Database)]
```

### Layer Details:
* **Lambda Handlers**: Custom entry points that boot the Spring Application context and map requests to the `DispatcherServlet` (for API calls) or execute workflow evaluations (for Cron tasks).
* **REST Controllers**: Mapped to standard Spring MVC endpoints (`@RestController`), handling authentication, quote searches, and manual action requests.
* **Business Services**: Reusable domain layers like `WorkflowService` containing state logic, or `EmailService` for sending/polling.
* **JPA Repositories**: Interfaces extending `JpaRepository` to abstract DB queries.

---

## ⚙️ 3. Bidding State Machine Workflow

The bidding process follows a strict state-driven pipeline, ensuring quotes transition seamlessly from intake to booking or loss:

```mermaid
stateDiagram-v2
    [*] --> INTAKE : Customer Email Ingested / Manual Create
    INTAKE --> OUT_TO_CARRIERS : Broadcast RFQs to Carrier Network
    OUT_TO_CARRIERS --> RE_BID_ROUND : 1st Round Ends (Under-bid Check / Re-bid)
    OUT_TO_CARRIERS --> QUOTE_SENT : 1st Round Ends (Good Bids / Select Winner)
    RE_BID_ROUND --> QUOTE_SENT : 2nd Round Ends (Select Winner & Send Quote to Customer)
    
    QUOTE_SENT --> AWAITING_APPROVAL : Quote Emailed to Customer
    AWAITING_APPROVAL --> APPROVED : Customer Approves (via UI / Email)
    AWAITING_APPROVAL --> LOST : Customer Rejects / Quote Expires
    
    APPROVED --> [*] : Generate BOL & Invoice
    LOST --> [*] : Record Lost Reason
```

### State Flow Highlights:
1. **Intake Processing**: Raw customer emails are fetched, and **Cerebras LLM (Llama-3)** parses logistical criteria (origin, destination, weight, class).
2. **Lane Benchmarking (RAG)**: The system vectorizes the origin and destination coordinates and queries the PostgreSQL database using `pgvector` to identify similar historical shipments and estimate competitive prices.
3. **RFQ Broadcast**: The broker broadcasts the RFQ via email to carrier contacts.
4. **Automated Re-Bid Trigger**: The system waits for carrier bids. If multiple carriers submit rates, a re-bid round is triggered. The system emails other carriers, notifying them of the current low-bid (without revealing names) to drive margins down.
5. **Approval and Booking**: The broker views the bids on their React-based Kanban board. Once they click "Approve", the Go backend automatically compiles a Bill of Lading (BOL), creates an invoice PDF, uploads them to the private S3 bucket, and sends booking confirmations.

---

## 🌐 Live Deployments

You can test the frontend dashboard, inspect live API responses, and run simulated bidding rounds using the active deployment URLs below:

* 👉 **Frontend Application URL**: [http://freight-agent-frontend-java-dev-865122443732.s3-website.us-east-2.amazonaws.com/](http://freight-agent-frontend-java-dev-865122443732.s3-website.us-east-2.amazonaws.com/)
* 👉 **Backend API Gateway URL**: [https://1gf0myv6k5.execute-api.us-east-2.amazonaws.com/](https://1gf0myv6k5.execute-api.us-east-2.amazonaws.com/)

---

## 🛠️ Local Development Setup

To run the application locally on your machine:

### 1. Prerequisites
* **Java Development Kit (JDK) 21**
* **Node.js** (v18 or higher)
* **AWS CLI** (configured with access to S3/RDS if testing integrations)
* **Python 3** (only for running integration tests)

### 2. Environment Configuration (`.env`)
Create a `.env` file in the project root:
```env
ACCESS_KEY=your_aws_access_key
SECRET_ACCESS_KEY=your_aws_secret_key
DATABASE_URL=postgresql://user:password@host:port/database_name
CEREBRAS_API_KEY=your_cerebras_api_key
SMTP_HOST=mail.example.com
SMTP_PORT=587
SMTP_USER=dispatch-notification@example.com
SMTP_PASSWORD=your_smtp_password
EMAILS_FROM_EMAIL=dispatch-notification@example.com
SMTP_TLS=True
SMTP_AUTH=True
HIBERNATE_DDL_AUTO=update
```

### 3. Running the App
* **Start API Server**:
  ```bash
  export $(grep -v '^#' .env | xargs) && ./mvnw spring-boot:run
  ```
  On startup, the system automatically runs database migrations and seeds initial data.

* **Start Frontend Dashboard**:
  ```bash
  cd frontend
  npm install --legacy-peer-deps
  npm run dev
  ```
  The dashboard will open at `http://localhost:5173`.

* **Run Integration Tests**:
  ```bash
  API_URL=http://localhost:8000 python3 test_api.py
  ```

---

## 🚀 Deploying to AWS

Deploying the Spring Boot 3 serverless backend and S3 static website is handled via Maven and the Serverless Framework:

1. **Build and Shaded JAR Generation**:
   ```bash
   ./mvnw clean package -DskipTests
   ```
   This compiles the Java code and creates a shaded JAR in the `target/` directory containing all dependencies optimized for Lambda invocation.

2. **Deploy Serverless Backend**:
   ```bash
   npx serverless deploy
   ```

3. **Deploy React Frontend**:
   ```bash
   ./deploy_frontend.sh dev
   ```

---

Check out the full Java implementation and fork the repository on GitHub:
👉 **[freight-agent-aws-java Repository](https://github.com/vishwakarma09/freight-agent-aws-java)**
