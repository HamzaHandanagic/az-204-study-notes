# Develop Solutions for Microsoft Azure – Day 1 (02.03.)

**Topics:** Azure App Service and Azure Function Apps

---

## Azure App Service

### When to use App Service

- **Deployment options on Azure:** AKS, Containers, Azure App Service, etc.
- **Use Azure App Service when** the workload is web-based: web apps, REST APIs, or mobile back-ends. It is a fully managed PaaS for hosting web applications.

### Creating and deploying

- App Service can be created via **Azure CLI**, **Azure Portal**, **ARM/Bicep**, or **Terraform**.
- After provisioning, you deploy your application. Common approach: **CI/CD** (e.g. **GitHub Actions**) into the App Service (deployment center, publish profile, or OAuth).
- Supported stacks: **.NET**, **Node.js**, **Python**, **Java**, **PHP**, and others.

### App Service Plans (scale up / scale out)


| Tier              | Use case                           | Notes                                                        |
| ----------------- | ---------------------------------- | ------------------------------------------------------------ |
| **Free / Shared** | Dev/test only                      | No SLA, limited scale, no custom domains (Free).             |
| **Basic**         | Dev/test, demos                    | Dedicated VM, custom domains, no deployment slots.           |
| **Standard**      | Production (minimum recommended)   | Deployment slots (e.g. staging), auto-scale, custom domains. |
| **Premium**       | High performance, VNet integration | More instances, VNet integration, no cold start.             |
| **Isolated**      | Compliance, dedicated environment  | App Service Environment (ASE), full network isolation.       |


- **Scale up:** Change plan tier (more CPU, RAM).
- **Scale out:** Add more instances (horizontal scaling). For production, **Standard** or higher is recommended so you can use **deployment slots** (e.g. staging → swap to production).

### Creating an App Service (portal steps)

1. Azure Portal → **App Services** → **Create**.
2. **Subscription** and **Resource Group**.
3. **Name**, **Publish** (Code or Container), **Runtime** (e.g. .NET 8, Node 20).
4. **Operating system:** Linux or Windows (when publishing **Code**).
5. **Region**.
6. **App Service plan:** Create new or use existing. Creating a new plan is typical for a new app.

### Deployment: Containers vs Code

- **Containers:** Build image → push to a registry (e.g. **Azure Container Registry**, Docker Hub). Use the same image across dev → test → prod. In **Deployment Center** you configure the image source (private or public registry) and tag. For production, use a **private registry** where possible.
- **Code:** Deploy via Git, ZIP, or CI/CD. **Kudu (SCM)** gives you the current state of the app (files, logs, environment) at `https://<app-name>.scm.azurewebsites.net`.

### Configuration

- **App settings:** Key-value pairs become **environment variables** in the app. Use them for config (e.g. API URLs, feature flags). They can be slot-specific (sticky to a slot on swap).
- **Connection strings:** Stored separately; also exposed as environment variables (with a prefix depending on runtime). Use for databases and other connection info.
- **General settings:** Stack (runtime version), **Always On** (keeps app loaded; not on Free/Shared), **HTTPS only**, **HTTP version**, etc.

### Custom domain and TLS

- Add a **custom domain** in the app’s **Custom domains** blade. You’ll need to add a **CNAME** (or **A** record if using root domain) in your DNS. For verification, a **TXT** record may be required.
- **SSL/TLS:** Use **Managed certificate** (Microsoft-provided, free for one cert per hostname) or **upload your own** (e.g. from your CA). SNI SSL is supported.

### Scaling (manual and automatic)

- **Manual scaling:** In the App Service plan, set **Scale out** → **Manual scale** → choose **Instance count**.
- **Autoscale:** **Scale out** → **Custom autoscale**. Define **scale conditions** (e.g. CPU > 70%), **instance limits** (min/max), and optionally **schedule** (start/end date or recurrence). Scale rules add or remove instances based on metrics (CPU, memory, HTTP queue length, custom metrics).

---

## Azure Function Apps

### Purpose

- Run **background tasks** and event-driven logic **independently** from the rest of your system.
- **Serverless:** Azure manages compute; you focus on code. Functions **scale with demand** (event-driven scaling on Consumption plan).
- **Trigger:** What invokes the function (HTTP, timer, queue, blob, Event Grid, etc.).
- A **Function App** is the unit of deployment and scaling; it can contain **multiple functions** that share the same app settings, connection strings, and hosting plan.

### Triggers and bindings

- **Trigger:** Defines *when* the function runs (e.g. HTTP request, new queue message, blob uploaded, timer).
- **Binding:** Declarative connection to a resource. Types:
  - **Input binding:** Read data from a service (e.g. read from Blob, Cosmos DB, Table).
  - **Output binding:** Write data without writing SDK code (e.g. write to Queue, Blob, Service Bus).
- Bindings are configured in the function signature (and optionally in `function.json` for non-.NET). Example: *“When an HTTP request comes in (trigger), read a blob (input binding) and put a message on a queue (output binding).”*

**Example scenario:** Order processing

- **Trigger:** HTTP POST with order payload (or a message from a queue).
- **Input binding:** Read customer profile from Cosmos DB or Blob.
- **Output binding:** Send confirmation to a Queue or Service Bus for downstream processing.
- The function runs only when the trigger fires; bindings handle read/write to other services.

### Hosting plans


| Plan                             | Scale behavior                                                                | Typical use                                                                                    |
| -------------------------------- | ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Consumption**                  | Scale to zero; scale out on events. Billing per execution and execution time. | Event-driven, short jobs, sporadic traffic.                                                    |
| **Flex Consumption**             | Similar to Consumption with more control and improved cold start.             | Same as Consumption with better performance needs.                                             |
| **Premium**                      | Pre-warmed instances, no scale to zero by default, VNet support.              | Low latency, long-running, or VNet/private endpoints.                                          |
| **Dedicated (App Service Plan)** | Same as App Service (always-on VMs).                                          | Long-running functions, existing App Service plan, or when Durable Functions are not suitable. |
| **Container Apps**               | Scale based on HTTP or events; container-based.                               | When you want to run functions in your own container.                                          |


- **Consumption:** Event-driven; instances are added/removed automatically. **Cold start** possible when scaling from zero.
- **Dedicated:** Uses an App Service plan (same as web apps). Good for functions that run a long time or need always-on; no scale-to-zero.
- **Premium:** Keeps instances **pre-warmed** so requests avoid cold start; supports VNet and longer execution.

### Default timeout and limits (Consumption)

- **Timeout:** **5 minutes** (default; max 10 minutes on Consumption). Function is terminated after the timeout.
- If the app is idle, the instance can be **scaled to zero**; the next request may see **cold start** latency.
- **Premium / Dedicated:** Timeout is configurable and can be much longer than Consumption (e.g. effectively unlimited on Dedicated when not using HTTP trigger default limits).

### Pre-warmed instances (Premium plan)

- **Pre-warmed instances:** Minimum number of instances that are always **ready** (already loaded). Requests hit these first, reducing **cold start**.
- **Minimum instance count** > 0 keeps the app warm. Useful for production HTTP APIs built with Functions.

### Scaling behavior

- **Unit of scale:** The **Function App** (all functions in the app scale together).
- **Scale controller** watches trigger events (e.g. queue length, HTTP request rate) and adds or removes instances.
- Instances can **scale in to zero** (Consumption) when no work is running. Next invocation may trigger a **cold start** (delay while a new instance starts).
- One Function App = one deployment unit; all functions inside share configuration (`host.json`, app settings) and the same plan.

### Developing and deploying

- **Create Function App:** Portal → **Function App** → Create. Choose runtime (.NET, Node, Python, etc.), OS, and **hosting plan**.
- **Develop locally** (e.g. in .NET with Azure Functions tools), run and debug, then **deploy** (Visual Studio, VS Code, CLI, or CI/CD) to the Function App.
- **Configuration:**
  - `**host.json`:** Settings for the entire Function App (logging, extensions, concurrency). Deploy with the app; applies to all functions in that app.
  - `**local.settings.json`:** Local-only settings (connection strings, app settings). Not deployed; use App Settings in Azure for production.

### Summary


| Concept            | App Service                                 | Function App                                                 |
| ------------------ | ------------------------------------------- | ------------------------------------------------------------ |
| Unit of deployment | Web app (one app per plan or shared)        | Function App (many functions per app)                        |
| Scaling            | Plan-based; scale out by instances          | Event-driven (Consumption) or plan-based (Dedicated/Premium) |
| Best for           | Web apps, APIs, mobile back-ends            | Event-driven, serverless, background tasks                   |
| Configuration      | App Settings, Connection strings → env vars | Same; plus `host.json` for runtime behavior                  |


# Develop Solutions for Microsoft Azure – Day 2 (03.03)

**Topics:** Azure Blob Storage and Azure Cosmos DB

---

## Azure Blob Storage

### Purpose and use cases

- **Unstructured data at scale:** Blob storage holds massive amounts of **unstructured** data (no schema). You don’t manage disks or VM storage; Azure handles capacity and durability.
- **Typical uses (AZ-204):** Serving images or documents to browsers, storing files for distributed access, streaming video/audio, writing logs, backup/restore, disaster recovery, archiving, and as a target for app data (e.g. Parquet files for analytics).
- **SDK:** Use the **Azure Storage Blobs** client library (e.g. `Azure.Storage.Blobs` for .NET). Additional packages: `Azure.Storage.Common` (shared types, auth), and optionally **Azure.Identity** for Microsoft Entra ID (Azure AD) authentication.

### Blob types

| Type | Use case | Notes |
|------|----------|--------|
| **Block blob** | Most common: files, images, logs, streaming | Data split into blocks (up to ~4.75 TB per blob). Optimized for upload/download and tiering (Hot/Cool/Cold/Archive). |
| **Append blob** | Logging, append-only | Append blocks only; no random write. Good for logs and telemetry. |
| **Page blob** | Random read/write, VHDs | Used for Azure IaaS VM disks. 512-byte pages; up to 8 TB. |

For typical app scenarios (files, documents, backups), **block blobs** are the default.

### Storage account hierarchy and structure

- **Storage account** is the top-level Azure resource. It has a **globally unique** name that becomes part of the endpoint URL (e.g. `https://<accountname>.blob.core.windows.net`). You are **billed at the storage account level** (capacity, transactions, data transfer). Everything below lives inside one subscription and resource group.
- **Container** lives inside a storage account. It’s like a **root folder** for blobs. You create containers to organize data (e.g. one container per app or per environment). There is **no real folder hierarchy** inside the account; the hierarchy is **account → containers**.
- **Blob** is the object inside a container. You can **simulate folders** by using names with delimiters (e.g. `parquet/2024/01/file.parquet`). The service treats this as a flat name; “folders” are a naming convention only (e.g. in Portal/Explorer they are shown as a virtual hierarchy).
- **Comparison:** Resource group = Azure management boundary; storage account = billing and endpoint boundary; container = logical grouping of blobs; blob = single object (file).

### Storage account types and redundancy

- **Account types:** **General Purpose v2 (GPv2)** is the default and recommended; it supports Blob, File, Queue, and Table. **Block Blob Storage** and **Blob Storage** (legacy) are blob-only. **File Storage** is for Azure Files only.
- **Redundancy (durability):** LRS (single region, 3 copies), ZRS (zone-redundant in one region), GRS/RA-GRS (geo-redundant, optional read-access to secondary), GZRS/RA-GZRS (geo + zone). **GZRS/RA-GZRS** is the highest durability and cost.

### Access tiers (block blobs only)

| Tier | Cost | Access | Use case |
|------|------|--------|----------|
| **Hot** | Higher storage cost, lower access cost | Immediate | Frequently accessed data. |
| **Cool** | Lower storage, higher access | Immediate | 30-day minimum; infrequent access. **Early deletion fee** if moved or deleted before 30 days. |
| **Cold** | Lower than Cool | Immediate | 90-day minimum; similar early deletion if removed earlier. Can be set as **account** default (containers inherit); individual blobs can override when uploading. |
| **Archive** | Lowest storage, highest access cost | **Not immediately accessible** | 180-day minimum. Blob must be **rehydrated** (copy to Hot/Cool) before read. Rehydration can take **hours**; **high-priority** rehydration is faster but more expensive. **Archive tier is set at blob level only** (not account/container default). |

Choose tiers carefully: moving to Cool/Cold/Archive can have minimum retention and early-deletion penalties; moving to Archive means the blob is offline until rehydration.

### Creating a storage account (conceptual steps)

1. **Subscription** and **Resource group**.
2. **Storage account name** (globally unique), **Region**, **Performance** (Standard or Premium for block blobs).
3. **Redundancy:** LRS, GRS, ZRS, GZRS, etc.
4. **Security:** Enable **Secure transfer (HTTPS)** required (recommended for production). Optionally enable **Azure AD Kerberos** for Azure Files (see below).
5. **Data protection:** **Soft delete** for blobs (e.g. 7-day retention)—deleted blobs can be restored within the retention period.
6. **Encryption:** By default, **Microsoft-managed keys (MMK)** encrypt data at rest. For stricter compliance, use **customer-managed keys (CMK)** in Azure Key Vault—you control key lifecycle and revocation.

### Azure Files and Azure AD Kerberos (Azure Files only)

- **Azure Files** is the file-share service in the same storage account (SMB/NFS). For **Azure AD Kerberos** authentication:
  1. **Enable Azure AD Kerberos** on the storage account (Azure Files identity-based auth).
  2. **Assign RBAC** (e.g. Storage File Data SMB Share Reader/Contributor) so users or identities can access the share.
  3. Users **sign in with Microsoft Entra ID**; no domain-joined machine required.
  4. Access can be refined with **NTFS ACLs** on the share.

### Other storage services in the account (overview)

| Service | Purpose |
|---------|--------|
| **Blob** | Unstructured object storage (block, append, page blobs). |
| **Files** | SMB/NFS file shares (lift-and-shift, shared drives). |
| **Queues** | Message queue for async work (e.g. between app and worker; 64 KB messages). |
| **Tables** | NoSQL key-value store (part of same account; distinct from Cosmos DB Table API). |

All share the same storage account endpoint namespace (e.g. `*.blob.`, `*.file.`, `*.queue.`, `*.table.`).

### Security (important for AZ-204)

- **Authentication:** Prefer **Microsoft Entra ID (Azure AD)** over account key. Use a **service principal** or managed identity with **RBAC** (e.g. **Storage Blob Data Contributor**, **Storage Blob Data Reader**) scoped to the storage account or a **specific container** for least privilege.
- **Authorization:** **RBAC** roles (e.g. at resource group or storage account) control who can manage the account and who can read/write data. **SAS (shared access signature)** gives time-limited, scoped access (container/blob level) when you can’t use Entra ID (e.g. temporary client access).
- **Encryption at rest:** Always on. **MMK (Microsoft-managed keys):** Azure manages keys. **CMK (customer-managed keys):** Keys in your Key Vault; required for certain compliance and for granular control (rotate, revoke). Use **HTTPS** in production for encryption in transit.

### Data lifecycle management (policies and rules)

- **Lifecycle management** automates **tier transitions** (e.g. Hot → Cool after 30 days, Cool → Archive after 90 days) and **deletion** at end of lifecycle. Available for **General Purpose v2** accounts. Rules run **once per day**; you target **containers** or **blob name prefixes** (subset of blobs).
- **Policy structure (REST/ARM):** A policy is a JSON object; each **rule** has a name (e.g. `rule1`), `type` (e.g. Lifecycle), and `definition` with filters (prefix, container) and actions (transition to Cool/Cold/Archive, or delete). Example shape: `{ "rules": [ { "name": "rule1", "type": "Lifecycle", "definition": { "filters": { ... }, "actions": { ... } } } ] }`.
- **Portal:** Storage account → **Data management** → **Lifecycle management** → Add rule → define filters (containers/prefixes) and actions (transitions, delete).
- **Rehydration from Archive:** Copy the blob to Hot (or Cool) tier. **Standard** rehydration can take hours; **high-priority** is faster and more expensive.

### Working with Azure Blobs in code (AZ-204)

- **NuGet:** `Azure.Storage.Blobs` (and `Azure.Identity` for Entra ID). Use **BlobServiceClient** (connection to account), **BlobContainerClient** (container), **BlobClient** (single blob) or **BlockBlobClient** for block blobs.
- **Authentication:** Connection string (dev/test) or **DefaultAzureCredential** (production: managed identity, service principal, or Azure CLI). Avoid hardcoding keys.
- **Operations:** Create container, upload/download blocks or full blob, list blobs with prefix (virtual “folders”), set tier (Hot/Cool/Archive), delete (respect soft-delete retention).

---

## Azure Cosmos DB

### Purpose and characteristics

- **Fully managed NoSQL** database: low latency, **elastic scalability**, and **well-defined consistency semantics**. Suited for globally distributed apps that need single-digit-millisecond reads/writes and flexible schema.
- **Billing:** You pay for **throughput** (request units, RU/s) and **storage**. No upfront capacity; scale throughput and storage independently.
- **SLA:** Up to **99.999%** availability (multi-region write), **<10 ms** latency at 99th percentile for read and write when deployed in the same region as the app.
- **Benefits:** Global distribution (multi-region replication), **multiple consistency levels** (strong to eventual), automatic indexing, and support for **multiple APIs** (SQL, MongoDB, Cassandra, Gremlin, Table) over the same data model.

### Resource hierarchy

| Level | Description |
|-------|-------------|
| **Cosmos DB account** | Top-level Azure resource. Holds **API type** (e.g. NoSQL/SQL, MongoDB), **consistency default**, **regions**, and **keys**. Billing and connectivity boundary. |
| **Database** | Logical namespace (like a “database” in relational terms). Contains **containers**. |
| **Container** | Unit of **throughput** (RU/s) and **partitioning**. Holds **items** (documents/rows). Throughput can be **database-level** (shared across containers) or **container-level** (dedicated). |
| **Item** | Single record (e.g. JSON document in NoSQL API). Identified by **id**; partition key determines physical placement. |

So: **Account → Database(s) → Container(s) → Items**. Partition key is chosen at **container** creation and cannot be changed later.

### Supported API(one per account)

- **NoSQL (SQL API):** Native; JSON documents, SQL-like queries. Default choice for new apps when you want document model + SQL queries.
- **MongoDB**, **Cassandra**, **Gremlin (graph)**, **Table:** Wire-protocol compatible so you can migrate existing apps or use existing drivers. Under the hood, data is stored in the same Cosmos DB model; the API layer translates.

### Request Units (RUs) and throughput modes

- **Request Unit (RU):** Measure of cost for a single request (e.g. 1 KB read = 1 RU; writes cost more). Throughput is provisioned in **RU/s** (RUs per second).
- **Three modes (AZ-204):**
  - **Provisioned throughput:** You set **RU/s** (and optionally storage) per container or database. Predictable cost; good for steady workload. **Autoscale** is provisioned with a max RU/s; it scales between 10% and 100% of that max based on load.
  - **Serverless:** No provisioned RU/s; you pay per **request** (RU consumption) and storage. Scale to zero; good for spiky or low-volume workloads.
  - **Autoscale (provisioned):** Same as provisioned but with a **range** (e.g. 400–4000 RU/s); Azure scales within the range. Billing based on the highest RU/s used in the hour.

Choose **provisioned** for steady production load, **serverless** for variable or low traffic, **autoscale** for variable load with a cap.

### Consistency levels

- **Strong:** Linearizable; highest latency, highest consistency.
- **Bounded staleness:** Lag in reads (time or K updates).
- **Session:** Per-session guarantees (default for many apps).
- **Consistent prefix:** No out-of-order reads.
- **Eventual:** Lowest latency; reads may be stale.

Trade-off: stronger consistency → higher latency and possibly more RU cost; eventual → lowest latency, possible staleness.

### Summary (Day 2)

| Topic | Blob Storage | Cosmos DB |
|-------|--------------|-----------|
| Data model | Unstructured (blobs); flat namespace with virtual folders | Structured items (e.g. JSON); partition key per container |
| Scaling | Automatic (capacity); you manage tiers and lifecycle | Elastic; you set RU/s (provisioned/serverless/autoscale) |
| Security | RBAC + Entra ID; MMK/CMK; HTTPS | Keys or Entra ID; encryption at rest; network/firewall rules |
| AZ-204 focus | SDK (BlobServiceClient, containers, blobs), tiers, lifecycle, security | Hierarchy, RU/s modes, APIs, consistency, partition key |

# Develop Solutions for Microsoft Azure – Day 3 (04.03)

**Topics:** Implement containerized solutions and implementing Authorization and Authentification

---

## Azure Cosmos DB

Topics: Azure Container Registry. Scalable orchestrator systems like Docker Swarm etc. ACR is used with existing container development. Use cases. Tiers: Basic, Premium, Standard.

