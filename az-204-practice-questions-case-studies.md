
# AZ-204 Practice Questions – Case Studies

Exam-style case studies with scenario-based questions. Read each case study, then answer the questions that follow.

---

## Case Study 1: Contoso E-Commerce Platform

### Background

Contoso Ltd. is migrating its e-commerce platform to Azure. The platform consists of:

- **Web storefront** — An ASP.NET Core 8 web application serving the customer-facing website.
- **Order API** — A REST API that receives orders and writes them to a database.
- **Order processor** — A background service that processes orders asynchronously.
- **Product images** — Stored as files, served to browsers.

### Current architecture

- All components run on a single on-premises VM.
- Product images are stored on a local file share.
- Orders are stored in SQL Server.
- The team deploys manually via RDP.

### Requirements

| Requirement | Details |
|-------------|---------|
| **R1** | The web storefront must scale automatically during sales events (Black Friday). |
| **R2** | The Order API must be deployed independently from the storefront. |
| **R3** | Product images must be durable, globally accessible, and served with low latency. Images older than 90 days should automatically move to a lower-cost tier. |
| **R4** | The order processor must run only when new orders arrive and scale to zero when idle to minimize cost. |
| **R5** | Deployments must use CI/CD from GitHub with no stored credentials. |
| **R6** | All application secrets (connection strings, API keys) must be stored securely and not in code or config files. |
| **R7** | The team wants to monitor request performance, failures, and dependencies across all components. |

---

### Questions

**1.** Which Azure service should Contoso use to host the web storefront to meet requirement **R1**?

- A) Azure Virtual Machines with a load balancer
- B) Azure App Service on a Standard (or higher) plan with autoscale rules
- C) Azure Container Instances

---

**2.** To meet requirement **R3**, where should Contoso store product images, and how should the 90-day tier transition be configured?

- A) Azure Blob Storage (Hot tier) with a lifecycle management rule to move blobs to Cool after 90 days
- B) Azure Files with a scheduled Azure Function to delete old files
- C) Azure Cosmos DB with a TTL (time-to-live) policy of 90 days

---

**3.** Which Azure service best satisfies requirement **R4** for the order processor?

- A) Azure App Service WebJob (Continuous)
- B) Azure Functions on a Consumption plan triggered by a Storage Queue (or Service Bus Queue)
- C) Azure Container Instances with Always restart policy

---

**4.** To meet requirement **R5** (CI/CD from GitHub with no stored credentials), what should Contoso configure?

- A) A GitHub Actions workflow using a publish profile stored as a GitHub secret
- B) A GitHub Actions workflow using OpenID Connect (OIDC) workload identity federation with a service principal
- C) Manual ZIP deployment from the developer's machine

---

**5.** How should Contoso store the SQL Server connection string to satisfy requirement **R6**?

- A) In the `appsettings.json` file deployed with the application
- B) In Azure Key Vault, referenced via a Key Vault reference in App Service app settings, with the App Service's managed identity granted Key Vault Secrets User
- C) As an environment variable set via RDP on the App Service instance

---

**6.** To meet requirement **R7**, which service should Contoso enable on all components for end-to-end monitoring?

- A) Azure Activity Log
- B) Application Insights (with workspace-based Log Analytics)
- C) Azure Advisor

---

---

## Case Study 2: Woodgrove Bank Identity and API Platform

### Background

Woodgrove Bank is building a new platform with:

- **Customer portal** — A single-page application (React) where customers view accounts and transactions.
- **Banking API** — A protected ASP.NET Core Web API that serves account data.
- **Admin service** — A background daemon that synchronizes user data from the bank's Active Directory to Microsoft 365 via Microsoft Graph.
- **Partner API** — The Banking API is also consumed by external partner companies.

### Requirements

| Requirement | Details |
|-------------|---------|
| **R1** | Customers must sign in with their Microsoft Entra ID (work/school) accounts. MFA is required for all users. |
| **R2** | The SPA must obtain tokens to call the Banking API on behalf of the signed-in user. |
| **R3** | The Banking API must validate incoming tokens and only accept tokens issued for its own audience. |
| **R4** | The admin service runs nightly with no user interaction and needs to read all user profiles from Microsoft Graph. |
| **R5** | Partner companies are in separate Entra ID tenants and must be able to call the Partner API. |
| **R6** | The Banking API must be exposed through a gateway that enforces rate limiting, subscription keys, and JWT validation. |

---

### Questions

**7.** To meet requirement **R2**, which OAuth 2.0 flow should the React SPA use?

- A) Client credentials flow
- B) Authorization code flow with PKCE
- C) Device code flow

---

**8.** How should the Banking API validate incoming tokens to satisfy requirement **R3**?

- A) Decode the token and check only the `exp` (expiry) claim
- B) Validate the JWT signature, issuer (`iss`), and audience (`aud`) claim against the API's own application ID
- C) Call Microsoft Graph to verify the token on every request

---

**9.** For the admin service (requirement **R4**), which OAuth 2.0 flow and permission type should be used?

- A) Authorization code flow with delegated permissions
- B) Client credentials flow with application permissions (e.g. `User.Read.All`), with admin consent granted
- C) Implicit grant flow with delegated permissions

---

**10.** How should the app registration be configured to allow partner companies (requirement **R5**) to use the API?

- A) Register the app as **single-tenant** and create separate registrations in each partner tenant
- B) Register the app as **multi-tenant** so service principals are created in each partner's tenant upon consent
- C) Share the client secret with each partner company

---

**11.** To meet requirement **R1** (MFA for all users), what should Woodgrove configure?

- A) A custom MFA implementation inside the React SPA
- B) A Conditional Access policy in Entra ID requiring MFA for access to the app
- C) An APIM policy that checks for a second password header

---

**12.** Which Azure service satisfies requirement **R6** (gateway with rate limiting, subscription keys, JWT validation)?

- A) Azure Front Door
- B) Azure API Management
- C) Azure Application Gateway

---

---

## Case Study 3: Fabrikam IoT Telemetry Pipeline

### Background

Fabrikam Inc. manufactures industrial sensors. Each sensor sends telemetry data (temperature, pressure, vibration) every second. The company is building a cloud pipeline to:

- **Ingest** telemetry from 50,000 sensors worldwide.
- **Process** data in near real time to detect anomalies.
- **Store** raw telemetry for long-term analytics and compliance (must retain for 1 year).
- **Alert** operators when sensor values exceed thresholds.

### Current state

- Sensors push data over HTTPS to a custom REST API hosted on a VM.
- The VM is a bottleneck; data is lost during traffic spikes.
- Raw data is written to local disk and manually copied to an archive monthly.

### Requirements

| Requirement | Details |
|-------------|---------|
| **R1** | The ingestion layer must handle millions of events per second with at least 7 days of built-in retention. |
| **R2** | Telemetry for the same sensor must be processed in order. |
| **R3** | Multiple downstream applications (anomaly detector, dashboard, archive) must read the same stream independently. |
| **R4** | Raw telemetry must be automatically archived to Blob Storage in a structured format for batch analytics. |
| **R5** | An Azure Function must process each event in near real time to check thresholds and raise alerts. |
| **R6** | Operators must be notified via email when a threshold alert is triggered. |
| **R7** | The archived data older than 90 days must automatically move to a cheaper storage tier; data older than 365 days must be deleted. |

---

### Questions

**13.** Which Azure service should Fabrikam use for the ingestion layer (requirement **R1**)?

- A) Azure Event Grid
- B) Azure Event Hubs
- C) Azure Storage Queues

---

**14.** How should Fabrikam ensure that telemetry from the same sensor is processed **in order** (requirement **R2**)?

- A) Use a single partition in Event Hubs
- B) Send events with the sensor ID as the **partition key** so all events from the same sensor go to the same partition
- C) Use Azure Event Grid with subject filtering by sensor ID

---

**15.** Requirement **R3** states that multiple applications must read the same stream independently. How is this achieved in Event Hubs?

- A) Create a separate Event Hub for each application
- B) Create a separate **consumer group** for each application
- C) Use Azure Service Bus topics with subscriptions

---

**16.** Which Event Hubs feature automatically writes incoming events to Blob Storage or Data Lake in Avro format to meet requirement **R4**?

- A) Event Hubs **Capture**
- B) Event Hubs **Auto-inflate**
- C) Event Hubs **Dedicated cluster**

---

**17.** For requirement **R5**, the Azure Function processes events from Event Hubs. Which trigger type should the function use?

- A) HTTP trigger
- B) Event Hub trigger
- C) Timer trigger

---

**18.** When the Azure Function detects a threshold breach, it must notify operators by email (requirement **R6**). Which combination of Azure services can achieve this?

- A) The function sends an event to **Azure Event Grid**, which triggers an **Azure Monitor action group** configured to send email
- B) The function writes a log entry to Application Insights and relies on the operator to check the dashboard
- C) The function calls the SMTP server directly using hard-coded credentials

---

**19.** To meet requirement **R7** (move archived data to a cheaper tier after 90 days, delete after 365 days), which feature should Fabrikam configure on the storage account?

- A) Azure Policy with a deny effect on old blobs
- B) Blob lifecycle management rules with tier transition (Hot → Cool at 90 days) and delete action at 365 days
- C) An Azure Function on a daily timer that scans and deletes old blobs

---

---

## Case Study 4: Adventure Works Microservices Migration

### Background

Adventure Works is decomposing a monolithic .NET application into microservices:

- **Product catalog service** — Reads product data from Azure Cosmos DB.
- **Inventory service** — Manages stock levels; uses Azure SQL.
- **Web gateway** — A public-facing API that routes requests to internal services.
- **Image resizer** — A service that resizes uploaded product images and stores thumbnails.

### Requirements

| Requirement | Details |
|-------------|---------|
| **R1** | The product catalog service must handle variable read traffic with low latency; reads can tolerate slightly stale data. |
| **R2** | The Cosmos DB container's partition key must support efficient queries that filter by product category. |
| **R3** | The image resizer runs on-demand when a new image is uploaded and should not consume resources when idle. |
| **R4** | All services are containerized. Container images must be stored in a private Azure registry. The registry must support geo-replication. |
| **R5** | The web gateway must enforce rate limiting, require subscription keys, and transform responses (remove internal headers). |
| **R6** | Developers need a portal to discover APIs, view documentation, and get subscription keys. |
| **R7** | All services must authenticate to Azure resources (Cosmos DB, SQL, Key Vault) without storing credentials. |

---

### Questions

**20.** For the product catalog Cosmos DB (requirement **R1**), which throughput mode and consistency level combination is best?

- A) Serverless with Strong consistency
- B) Autoscale provisioned throughput with Eventual (or Session) consistency
- C) Manual provisioned throughput with Bounded staleness

---

**21.** To meet requirement **R2**, what should Adventure Works choose as the partition key for the product catalog container?

- A) `/id` (the document's unique identifier)
- B) `/categoryId` (the product category)
- C) `/createdDate` (the creation timestamp)

---

**22.** Which Azure service should run the image resizer to meet requirement **R3** (on-demand, scale-to-zero)?

- A) Azure App Service on a Dedicated plan
- B) Azure Functions on a Consumption plan with a Blob trigger
- C) Azure Virtual Machine with a scheduled task

---

**23.** Which ACR tier must Adventure Works use to meet requirement **R4** (private registry with geo-replication)?

- A) Basic
- B) Standard
- C) Premium

---

**24.** Which Azure service should serve as the web gateway (requirements **R5** and **R6**)?

- A) Azure Application Gateway
- B) Azure API Management
- C) Azure Front Door

---

**25.** How should all services authenticate to Azure resources without storing credentials (requirement **R7**)?

- A) Use a shared storage account key stored in each service's environment variables
- B) Enable **managed identities** on each service and grant RBAC roles to those identities on the target resources
- C) Create a single service principal with a client secret and share it across all services

---

---

## Case Study 5: Relecloud Secure Application with Key Vault and Monitoring

### Background

Relecloud is a SaaS company running a multi-tier application on Azure:

- **Frontend** — An ASP.NET Core web app on Azure App Service.
- **Backend API** — An ASP.NET Core Web API on a separate App Service.
- **Worker** — An Azure Function that processes messages from a Service Bus Queue.
- **Data layer** — Azure SQL Database and Azure Blob Storage.

### Current issues

- Connection strings are stored in `appsettings.json` and checked into Git.
- When a secret (e.g. SQL password) changes, the team must redeploy all services.
- There is no centralized monitoring; developers check logs on each service individually.
- A recent outage was caused by an expired SSL certificate that no one noticed.

### Requirements

| Requirement | Details |
|-------------|---------|
| **R1** | All secrets must be removed from source code and config files. |
| **R2** | When a secret rotates in Key Vault, services must pick up the new value without redeployment. |
| **R3** | Each service should have the **minimum** Key Vault permissions needed (frontend and backend read secrets only; the worker reads secrets and one encryption key). |
| **R4** | SSL certificates must be stored in Key Vault with expiration alerts. |
| **R5** | All services must send telemetry (requests, dependencies, exceptions) to a single monitoring workspace. |
| **R6** | An alert must fire when the error rate exceeds 5% over a 5-minute window. |
| **R7** | The team needs a real-time view of live requests and performance during deployments. |

---

### Questions

**26.** How should Relecloud store and access secrets to meet requirements **R1** and **R2**?

- A) Store secrets in Azure Key Vault; configure App Service apps to use **Key Vault references** in app settings with managed identity authentication
- B) Store secrets in Azure App Configuration; hard-code the App Configuration connection string in each service
- C) Store secrets in environment variables set via the Azure Portal; redeploy when they change

---

**27.** To satisfy requirement **R3**, which RBAC roles should be assigned?

- A) **Key Vault Administrator** for all services
- B) **Key Vault Secrets User** for frontend and backend; **Key Vault Secrets User** + **Key Vault Crypto User** for the worker
- C) **Key Vault Secrets Officer** for all services

---

**28.** How should Relecloud manage SSL certificates and receive expiration alerts (requirement **R4**)?

- A) Store certificates in Key Vault; configure Key Vault to send **certificate near-expiry events** to Event Grid, which triggers an action group to email the team
- B) Store certificates on the App Service custom domain blade and manually check expiry monthly
- C) Use App Service Managed Certificates; they auto-renew and never expire

---

**29.** Which service should Relecloud enable on all components to meet requirement **R5** (centralized monitoring)?

- A) Azure Activity Log forwarded to a Storage Account
- B) Application Insights connected to a shared **workspace-based** Log Analytics workspace
- C) Azure Advisor with weekly digest emails

---

**30.** To create the error rate alert described in requirement **R6**, what should Relecloud configure?

- A) A **log alert** in Azure Monitor using a KQL query on the `requests` table (e.g. where `success == false`) with a 5-minute evaluation window, linked to an **action group** that sends email
- B) A Blob lifecycle rule that deletes error logs older than 5 minutes
- C) A custom health-check endpoint that the team polls manually

---

**31.** Which Application Insights feature provides a real-time stream of requests and performance metrics during deployments (requirement **R7**)?

- A) Application Map
- B) Live Metrics
- C) Availability Tests

---

---

## Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| **Case Study 1 – Contoso E-Commerce** | | |
| 1 | **B** | **App Service on Standard or higher** supports autoscale rules to handle traffic spikes. VMs require manual setup; ACI is not designed for web app hosting at scale. |
| 2 | **A** | **Blob Storage (Hot tier)** is ideal for serving images. A **lifecycle management rule** automates the tier transition to Cool after 90 days without code. |
| 3 | **B** | **Azure Functions on Consumption** scales to zero when no orders are pending and triggers on queue messages, paying only per execution. |
| 4 | **B** | **OIDC workload identity federation** eliminates stored credentials. The GitHub Actions workflow authenticates via a federated token from Entra ID. |
| 5 | **B** | Storing the connection string in **Key Vault** and referencing it via a Key Vault reference with managed identity keeps secrets out of code and config. |
| 6 | **B** | **Application Insights** (workspace-based) provides request tracing, dependency tracking, failure analysis, and end-to-end correlation across services. |
| **Case Study 2 – Woodgrove Bank** | | |
| 7 | **B** | **Authorization code flow with PKCE** is the recommended flow for SPAs (public clients). Client credentials is for daemons; device code is for browserless devices. |
| 8 | **B** | The API must validate the JWT **signature**, **issuer**, and **audience** to ensure the token is genuine and intended for this API. |
| 9 | **B** | A daemon with no user uses **client credentials flow** with **application permissions** (e.g. `User.Read.All`). Admin consent is required because there is no user to consent. |
| 10 | **B** | A **multi-tenant** registration allows other tenants to consent and creates a service principal in each partner tenant automatically. |
| 11 | **B** | **Conditional Access** policies in Entra ID enforce MFA at the identity provider level, transparent to the app. No custom MFA code is needed. |
| 12 | **B** | **Azure API Management** provides rate limiting (`rate-limit` policy), subscription keys, and `validate-jwt` policy—all requirements of the gateway. |
| **Case Study 3 – Fabrikam IoT** | | |
| 13 | **B** | **Azure Event Hubs** is designed for high-throughput ingestion (millions of events/sec) with configurable retention (up to 7+ days). |
| 14 | **B** | Using the **sensor ID as the partition key** ensures all events from the same sensor land in the same partition, preserving order per sensor. |
| 15 | **B** | Each **consumer group** maintains its own read position (offset) per partition, allowing independent processing by multiple applications on the same stream. |
| 16 | **A** | **Event Hubs Capture** automatically writes events to Blob Storage or Data Lake in Avro format at configurable intervals. |
| 17 | **B** | An **Event Hub trigger** binds the function to an Event Hubs consumer group, invoking it as events arrive. |
| 18 | **A** | The function publishes an event to **Event Grid** (or directly to an action group / Logic App). An **Azure Monitor action group** sends the email notification to operators. |
| 19 | **B** | **Blob lifecycle management rules** automate tier transitions (Hot → Cool at 90 days) and deletion (at 365 days) without code. |
| **Case Study 4 – Adventure Works** | | |
| 20 | **B** | **Autoscale provisioned** handles variable read traffic by scaling RU/s within a range. **Eventual or Session** consistency provides low latency for reads that can tolerate slight staleness. |
| 21 | **B** | `/categoryId` matches the query filter pattern, making queries single-partition (efficient). `/id` would spread reads across all partitions; `/createdDate` leads to hot partitions. |
| 22 | **B** | **Azure Functions on Consumption with a Blob trigger** runs only when a new image appears and scales to zero when idle. |
| 23 | **C** | **Premium** tier is required for geo-replication, zone redundancy, and content trust in ACR. |
| 24 | **B** | **Azure API Management** provides rate limiting, subscription key enforcement, response transformation policies, and a developer portal for API discovery. |
| 25 | **B** | **Managed identities** with RBAC on target resources provide credential-free authentication. Each service gets its own identity with least-privilege roles. |
| **Case Study 5 – Relecloud** | | |
| 26 | **A** | **Key Vault references** in App Service app settings resolve secrets at runtime via managed identity. When a secret is updated, the app picks up the new value without redeployment. |
| 27 | **B** | **Key Vault Secrets User** (read-only secrets) for frontend/backend; the worker additionally needs **Key Vault Crypto User** for encryption key operations. This follows least-privilege. |
| 28 | **A** | Key Vault can emit **certificate near-expiry events** via Event Grid. An action group subscribed to those events sends email alerts, preventing surprise expirations. |
| 29 | **B** | **Application Insights** connected to a shared **workspace-based Log Analytics workspace** centralizes telemetry from all services for unified querying and alerting. |
| 30 | **A** | A **log alert** with a KQL query on the `requests` table evaluates error rate over a window. An **action group** delivers email notifications when the threshold is breached. |
| 31 | **B** | **Live Metrics** streams real-time request rates, failures, and performance counters—ideal for monitoring during deployments. |
