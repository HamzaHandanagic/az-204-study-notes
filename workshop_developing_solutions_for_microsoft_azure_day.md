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

The hosting plan determines how your functions are run, how they scale, and how you are billed. Choose based on whether you need scale-to-zero (pay per execution), always-on VMs, or low latency without cold start.

| Plan | Scale behavior | Typical use |
|------|----------------|-------------|
| **Consumption** | Scale to zero; scale out on events. Billing per execution and execution time. | Event-driven, short jobs, sporadic traffic. |
| **Flex Consumption** | Similar to Consumption with more control and improved cold start. | Same as Consumption with better performance needs. |
| **Premium** | Pre-warmed instances, no scale to zero by default, VNet support. | Low latency, long-running, or VNet/private endpoints. |
| **Dedicated (App Service Plan)** | Same as App Service (always-on VMs). | Long-running functions, existing App Service plan, or when Durable Functions are not suitable. |
| **Container Apps** | Scale based on HTTP or events; container-based. | When you want to run functions in your own container. |

**Plan notes:**

- **Consumption:** Instances are added and removed automatically based on trigger activity. When there is no traffic, the app can scale to zero; the next invocation may experience a **cold start**.
- **Dedicated:** Uses an App Service plan (same as web apps). Instances are always on; no scale to zero. Use for long-running functions or when you need predictable performance and no cold start.
- **Premium:** You can set a **minimum instance count** so that a number of instances stay **pre-warmed**. Incoming requests use these first, avoiding cold start. Supports VNet integration and longer execution times.

### Default timeout and limits (Consumption)

- **Timeout:** Default is **5 minutes**; maximum is **10 minutes** on Consumption. If the function runs longer, Azure terminates it. Increase timeout only if your logic legitimately needs more time (e.g. long-running API call).
- **Idle and scale to zero:** When the app is idle, the runtime may scale to zero. The next request then triggers a new instance and can see **cold start** latency (often a few seconds).
- **Premium / Dedicated:** Timeout is configurable and can be much longer than Consumption (e.g. effectively unlimited on Dedicated when not constrained by HTTP trigger defaults).

### Pre-warmed instances (Premium plan)

On the **Premium** plan you can set a **minimum number of instances** that are always ready. These **pre-warmed instances** are already loaded and waiting for requests, so the first request does not pay a cold start. Useful for production HTTP APIs or event-driven workloads where you need consistent, low latency.

### Scaling behavior

- **Unit of scale:** The **Function App** is the unit of scaling. All functions inside the same app scale together; you cannot scale individual functions separately.
- **Scale controller:** Azure’s scale controller monitors your triggers (e.g. HTTP request rate, queue message count) and adds or removes instances within the limits of your plan. On Consumption, it can scale in to zero when there is no work.
- **One app, one plan:** One Function App has one hosting plan and one set of configuration (`host.json`, app settings). All functions in that app share the same runtime and scaling behavior.

### Developing and deploying

- **Create Function App:** In the portal, go to **Function App** → **Create**. Choose **runtime** (.NET, Node, Python, etc.), **OS**, and **hosting plan** (Consumption, Premium, or Dedicated). The plan can be changed later but affects billing and scaling.
- **Develop locally** using Azure Functions tools (e.g. VS Code extension, Visual Studio, or CLI). Run and debug on your machine, then **deploy** via Visual Studio, VS Code, Azure CLI, or CI/CD (e.g. GitHub Actions) to the Function App.
- **Configuration:**
  - **host.json:** Defines runtime behavior for the *entire* Function App (logging level, extension settings, concurrency). Deployed with your code; applies to all functions in that app.
  - **local.settings.json:** Used only for local development (connection strings, app settings). Not deployed to Azure. In production, use **App Settings** (and Key Vault references if needed) in the Function App.

### Summary: App Service vs Function App

| Concept | App Service | Function App |
|--------|-------------|--------------|
| Unit of deployment | Web app (one app per plan or shared) | Function App (many functions per app) |
| Scaling | Plan-based; scale out by instance count | Event-driven (Consumption) or plan-based (Dedicated/Premium) |
| Best for | Web apps, APIs, mobile back-ends | Event-driven, serverless, background tasks |
| Configuration | App Settings, connection strings → env vars | Same; plus **host.json** for runtime behavior |


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
- **Benefits:** Global distribution (multi-region replication), **multiple consistency levels** (strong to eventual), and automatic indexing. You **choose one API per account** at creation (NoSQL/SQL, MongoDB, Cassandra, Gremlin, or Table); each API has its own data model—you cannot query the same data with different APIs in one account.

### Resource hierarchy

| Level | Description |
|-------|-------------|
| **Cosmos DB account** | Top-level Azure resource. Holds **API type** (e.g. NoSQL/SQL, MongoDB), **consistency default**, **regions**, and **keys**. Billing and connectivity boundary. |
| **Database** | Logical namespace (like a “database” in relational terms). Contains **containers**. |
| **Container** | Unit of **throughput** (RU/s) and **partitioning**. Holds **items** (documents/rows). Throughput can be **database-level** (shared across containers) or **container-level** (dedicated). |
| **Item** | Single record (e.g. JSON document in NoSQL API). Identified by **id**; partition key determines physical placement. |

So: **Account → Database(s) → Container(s) → Items**. Partition key is chosen at **container** creation and cannot be changed later.

### Supported APIs (one per account)

- **NoSQL (SQL API):** Native; JSON documents, SQL-like queries. Default choice for new apps when you want document model + SQL queries.
- **MongoDB**, **Cassandra**, **Gremlin (graph)**, **Table:** Wire-protocol compatible so you can migrate existing apps or use existing drivers. You select **one** API when creating the account; all data in that account uses that API’s model.

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

**Topics:** Implement containerized solutions; implement authentication and authorization.

---

## Implement containerized solutions

### Key concepts

| Term | Meaning |
|------|--------|
| **Container** | A runnable instance of an **image**. Isolated process (and filesystem) on a host. You build once from an image, run anywhere (local, Azure, Kubernetes, etc.). |
| **Image** | Read-only template made of **layers**. Built from a **Dockerfile**. Stored in a **registry** (e.g. Azure Container Registry, Docker Hub) and pulled to run as containers. |
| **Layer** | Each instruction in a Dockerfile (e.g. `RUN`, `COPY`) typically creates one **layer**. Layers are cached; changing one line rebuilds only that layer and below. |
| **Dockerfile** | Text file with instructions to build an image: base image (`FROM`), installs (`RUN`), files (`COPY`), working directory (`WORKDIR`), default command (`CMD`/`ENTRYPOINT`), exposed ports (`EXPOSE`). |
| **Registry** | Storage and distribution for images. **Azure Container Registry (ACR)** is a private registry in Azure; you push/pull images by name and **tag** (e.g. `myacr.azurecr.io/myapp:v1`). |
| **Tag** | Label for a specific version of an image (e.g. `v1`, `latest`). Same image ID can have multiple tags (e.g. local `demo:v1` and registry `myacr.azurecr.io/demo:v1`). |

### Azure Container Registry (ACR)

- **Purpose:** Private Docker registry in Azure. Store and manage container **images**; integrate with Azure services (App Service, AKS, Azure DevOps, etc.) and existing container workflows (Docker CLI, Kubernetes).
- **Use cases:** Build and store images for your apps; deploy to App Service, AKS, or other orchestrators (e.g. Kubernetes, Docker Swarm); use with CI/CD to push images after build.
- **Tiers:**

| Tier | Use case | Notes |
|------|----------|--------|
| **Basic** | Learning, low traffic | Lower cost; fewer storage and throughput limits. No geo-replication. |
| **Standard** | Production, moderate load | More storage and throughput; recommended default for most apps. |
| **Premium** | High throughput, geo-replication | Advanced storage features (geo-replication, zone redundancy), higher limits, content trust. |

### ACR storage capabilities (Standard / Premium)

Higher tiers benefit from Azure storage features:

- **Encryption at rest:** All images and layers encrypted by default (Microsoft-managed keys; CMK optional on Premium).
- **Geo-redundant storage (GRS):** Replication to another region for durability (Premium supports **geo-replication** to multiple regions for pull from nearby locations).
- **Zone redundancy (ZRS):** Replication across availability zones in one region (Premium) for higher availability.
- **Scalable storage:** Storage grows with usage; you pay for what you use.

### Dockerfile: elements and layers

- A **Dockerfile** defines how to build an image. Each instruction usually adds one **layer**. Order matters: put rarely changed steps first so cache is reused.
- **Common instructions:**

| Instruction | Purpose |
|-------------|---------|
| **FROM** | Base (parent) image. Must be the first non-comment line. e.g. `FROM nginx:alpine`. |
| **RUN** | Run a command in the image (e.g. install packages: `RUN apt-get update && apt-get install -y nginx`). |
| **WORKDIR** | Set working directory for following instructions and for the running container. |
| **COPY** | Copy files from build context into the image (e.g. `COPY . /app`). Use **ADD** only when you need URL or archive extraction. |
| **EXPOSE** | Document which port the container listens on (e.g. `EXPOSE 80`). Does not publish the port; use `docker run -p` or orchestrator to map host port. |
| **CMD** | Default command when the container starts. One per Dockerfile; can be overridden at `docker run`. |
| **ENTRYPOINT** | Main executable; arguments from `docker run` are appended. Often used with **CMD** as default arguments. |

**Example Dockerfile (simple web app):**

```dockerfile
FROM nginx:alpine
WORKDIR /usr/share/nginx/html
RUN apk add --no-cache curl
COPY index.html .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

- **Layers:** `FROM` → base layer; `WORKDIR` → new layer; `RUN` → layer with curl; `COPY` → layer with your file; `EXPOSE`/`CMD` → metadata. Rebuilding only changes to `index.html` reuses cache up to `COPY` and rebuilds from there.

### Building an image from a Dockerfile

- **Command:** `docker build -t <name>:<tag> <context>`  
  Example: `docker build -t demo:v1 .`  
  - `-t` assigns the image name and tag (e.g. `demo:v1`).  
  - `.` is the build context (current directory); the Dockerfile in that directory is used by default (`Dockerfile`), or specify with `-f Dockerfile.prod`.
- The result is a **local** image. To use it in Azure, you **tag** it with your ACR login server and **push** (see below).

### Prerequisites and workflow: push image to ACR

1. **Prerequisites:** **Docker** installed locally; **Azure CLI** installed and logged in (`az login`); an **Azure Container Registry** created (Portal: **Containers** → **Container registries** → Create; choose **Pricing tier** Basic/Standard/Premium).
2. **Permissions:** Assign **RBAC** so your user or CI identity can push. Common role: **AcrPush** (push and pull). In Portal: ACR → **Access control (IAM)** → Add role assignment → **AcrPush** → assign to yourself or a service principal.
3. **Log in to ACR from Docker:**  
   `az acr login --name <acr-name>`  
   (Uses your Azure identity to log Docker into the registry.)
4. **Tag the image for ACR:**  
   Image names pushed to ACR must be prefixed with the registry login server: `<acr-name>.azurecr.io/<repo>:<tag>`.  
   Example: `docker tag demo:v1 myacr.azurecr.io/demo:v1`  
   This creates a **new tag** pointing to the **same image ID** (you can confirm with `docker images`; both `demo:v1` and `myacr.azurecr.io/demo:v1` show the same image ID).
5. **Push:**  
   `docker push myacr.azurecr.io/demo:v1`  
   Layers are uploaded to ACR; the image is then available for pull from Azure (App Service, AKS, etc.) or other clients with pull permissions.

### Azure Container Instances (ACI) – benefits and concepts

**Benefits of ACI:**

- **Fast startup:** Containers start in seconds without managing VMs or orchestrators.
- **Public IP and DNS:** Optional public IP with a configurable DNS name label for direct access.
- **Hypervisor-level security:** Containers run in isolated environments with strong isolation.
- **Custom sizes:** Request specific CPU and memory per container (within regional limits).
- **Persistent storage:** Mount Azure File Shares (or other volumes) so data survives container restarts.
- **Linux and Windows containers:** Both are supported (choose at container group creation).
- **Co-scheduled groups:** Multiple containers in one group share lifecycle, network, and storage.
- **VNet deployment:** Deploy into an Azure virtual network for private connectivity.

**Container group vs container instance:**

- A **container instance** is a single container (one image) with its own CPU/memory request.
- A **container group** is the deployable unit in ACI: it can contain one or more container instances that share the same host, local network, and optionally the same volumes. Resources (CPU, memory, optionally GPUs) are allocated by summing the resource requests of all instances in the group.
- A **sidecar container** is a secondary container in the same group that supports the main container (e.g. logging, proxy, file sync). All containers in the group share the same lifecycle and can communicate via `localhost`.

**Storage and file shares:**

- **Mounted file share:** You can mount an **Azure File Share** (or other volume) into one or more containers in a group. The same file share can be mounted at **different mount paths** in different containers (or in the same container under different paths) — each mount is independent.
- Azure File Share is one common way to share data between containers in a group and to persist data across restarts.

**Ways to deploy ACI:**

- **ARM/Bicep template:** Good for automation and for creating an ACI plus supporting resources (e.g. storage account and file share) in one deployment.
- **YAML:** Define the container group in a YAML file and deploy with `az container create --file deploy.yaml`. Often used for single or multi-container groups.
- **Azure CLI:** `az container create` with parameters (name, image, resource group, CPU/memory, env vars, etc.).
- **Azure Portal:** You can create a **single-container** group via the portal. **Multi-container groups** are not fully supported in the portal UI and are typically created via YAML or ARM.

**Resource allocation and limits:**

- CPU and memory are specified per container; the group’s total is the sum of all containers. Optionally, GPUs can be requested where available.
- **Networking:** Container groups can have a shared IP and optional FQDN; they can also be deployed into a VNet.
- **Typical per-container limits** (region-dependent): previously 4 vCPU and 16 GB RAM; larger sizes (e.g. up to 31 vCPU and 240 GB memory) are available in many regions. Check [ACI quotas](https://learn.microsoft.com/en-us/azure/container-instances/container-instances-resource-and-quota-limits) for your region.

**Example multi-container group (YAML):** a web app plus a sidecar that runs a helper process; both share a file share mounted at different paths.

**Restart policies:**

- **Always:** Restart the container whenever it stops (default). Used for long-running services.
- **On-failure:** Restart only when the container exits with a **non-zero exit code** (e.g. unhandled exception or process failure). Zero exit code = no restart.
- **Never:** Do not restart. Used for one-off or batch jobs.

An unhandled exception that terminates the process typically results in a non-zero exit code, so with **On-failure** the container will be restarted.

**Secrets and environment variables:**

- ACI supports passing secrets and configuration as **environment variables** for both Windows and Linux containers. Avoid putting secrets in the image; use env vars or Azure Key Vault integration.
- **Ways to set environment variables:**
  1. **YAML:** In the container group definition, under `containers[*].properties.environmentVariables` (or `secureEnvironmentVariables` for secrets).
  2. **Azure CLI:** `az container create ... --environment-variables "KEY1=value1" "KEY2=value2"` or `--secure-environment-variables "SECRET1=val1"`.
  3. **Portal:** When creating a container (single-container group), use the **Containers** → **Environment variables** section to add key-value pairs.

**Summary:** ACI is a serverless container runtime: you specify the image(s), CPU/memory, restart policy, env vars, and optional storage/network. No cluster to manage; billing is per second of use. Use container groups for co-scheduled multi-container workloads and Azure File Share for shared or persistent storage.

---

## Implement authentication and authorization

**Authentication** verifies identity (who you are); **authorization** verifies what you are allowed to do. In Azure and in apps, common building blocks include:

- **Microsoft Entra ID (formerly Azure AD):** Identity provider for work/school accounts, B2B guests, and (via Entra External ID) B2C/customer identities.
- **OAuth 2.0 / OpenID Connect (OIDC):** Standards for obtaining tokens and signing in users.
- **RBAC (Role-Based Access Control):** Authorization model in Azure (roles assigned to users, groups, or service principals).
- **Managed identities:** Azure-managed identities for apps to authenticate to Azure services without storing credentials.

**Microsoft Identity Platform** is the set of services and tools (authentication service, open-source libraries like MSAL, app registration and management) that let you build apps where users sign in with work/school accounts (Entra ID), personal Microsoft accounts, or external identities (B2B, B2C). Flow at a high level: your app uses **MSAL** to talk to **Microsoft Entra endpoints**; the user signs in and the app receives tokens for **work/school accounts**, **personal Microsoft accounts**, or **social/local accounts** (in B2C scenarios).

**MSAL (Microsoft Authentication Library)** is the supported way to integrate with the identity platform. Use MSAL to get tokens; do not implement OAuth/OIDC from scratch. **Entra.microsoft.com** (and the **Azure Portal** → **Microsoft Entra ID** → **App registrations**) is where you register an app. **Microsoft Graph** is the API to access Microsoft 365 and Azure AD data; you call Graph (REST or SDK) with an access token obtained via MSAL (see “Implement Solutions with Microsoft Graph” below).

**Application (app registration) vs service principal:**

- When you register an app in the portal, two main objects are created:
  1. **Application object** (in the home tenant): the “template” of the app (identifier, redirect URIs, credentials, API permissions). There is one per app registration.
  2. **Service principal** (in the tenant where the app is used): the “instance” of the app in that tenant, used for sign-in, RBAC, and consent. In the home tenant there is usually one service principal per app registration; in other tenants, a service principal is created when the app is consented or assigned (e.g. multi-tenant or enterprise app).

**Tenancy:**

- **Single-tenant:** Only identities in your tenant can use the app. Typical for internal LOB apps.
- **Multi-tenant:** Users from other tenants can sign in. The app appears as an **Enterprise application** in each tenant where it is used; each tenant has its own service principal.

**Permission types:**

- **Delegated permissions:** Used when a **signed-in user** is present. The app acts on behalf of the user; effective permissions are the intersection of app permissions and the user’s rights. Example: “Sign in and read user profile.”
- **Application permissions (app-only):** Used when the app runs **without a user** (e.g. background jobs, daemons). The app uses its own identity; admin consent is required. Example: “Read all users” for a sync job.

**Consent types:**

- **Static (admin) consent:** Admin pre-approves the permissions the app needs. Users do not see consent prompts for those permissions.
- **Incremental and dynamic user consent:** The app requests additional permissions at runtime (e.g. via `scope` or `extraScopesToConsent`). Only **delegated** permissions can be consented by users; high-privilege or application permissions require admin consent.
- **Admin consent:** Required for application permissions and for high-privilege delegated permissions. Granted per tenant (e.g. “Grant admin consent for this organization”).

In OAuth 2.0 / OpenID Connect, the app requests the permissions it needs using the **scope** query parameter in the authorization request (e.g. `scope=openid User.Read`).

**Concepts in brief:**

- **OpenID Connect (OIDC):** Identity layer on top of OAuth 2.0; provides ID tokens and user-info for “who is the user.”
- **OAuth 2.0:** Framework for obtaining **access tokens** to call APIs on behalf of a user or the app.
- **Token:** A signed artifact (e.g. access token, ID token, refresh token) issued by Entra ID after authentication.
- **Claims:** Name-value pairs inside a token (e.g. `oid`, `sub`, `scp`) used for identity and authorization.
- **Flows:** The sequence of steps (e.g. authorization code, client credentials) used to obtain tokens.

**Conditional Access:**

- **Conditional Access** policies enforce extra checks before granting access: e.g. require **multi-factor authentication (MFA)**, allow only **Intune-enrolled or compliant devices**, restrict by **location or IP range**, or require approved client apps. Policies are evaluated after successful authentication; if conditions are not met, access is blocked or limited.
- **Impact on the app:** Your app must support modern authentication (e.g. OAuth 2.0 with MSAL). If MFA or device compliance is required, the user completes that during sign-in; the app receives tokens only if Conditional Access allows it. Some policies may require **claims** or **device tokens** that your app may need to pass or validate.

**MSAL – application types and flows:**

- MSAL supports **web apps**, **web APIs**, **single-page apps (SPA)**, **mobile and native apps**, and **daemons / server-side apps**. You choose the right **flow** and **client type** (public vs confidential).
- **Authentication flows:**
  - **Authorization code:** Web and native apps; user signs in in a browser, app receives an authorization code and exchanges it for tokens. Recommended for apps with a user.
  - **Client credentials:** App-only; no user. App uses client ID and secret (or certificate) to get tokens for its own identity. Used for background services calling Graph or other APIs.
  - **On-behalf-of (OBO):** An upstream web API receives a token from the client and exchanges it for a new token to call a downstream API on behalf of the same user.
  - **Implicit grant:** Deprecated; avoid for new apps.
  - **Device code:** For devices without a browser (e.g. CLI, IoT). User enters a code in a browser on another device to sign in.
  - **Integrated Windows Authentication (IWA):** On Windows domain-joined machines; can acquire token silently with no UI when the user is already signed in.
  - **Username/password (ROPC):** High risk; only use in controlled scenarios (e.g. testing). Not recommended for production.

**Initializing client applications:**

- **PublicClientApplicationBuilder:** For public clients (desktop, mobile) that cannot keep a secret. Use for interactive sign-in (e.g. `AcquireTokenInteractive`).
- **ConfidentialClientApplicationBuilder:** For web apps and web APIs that can keep a secret (client secret or certificate). Use for server-side flows (authorization code, client credentials).
- Common builder methods:
  - **WithAuthority(tenant)** or **WithAuthority(authorityUri):** Identity provider (e.g. `https://login.microsoftonline.com/{tenant}`). Use `common` or `organizations` for multi-tenant.
  - **WithTenantId(tenantId):** Pin to a specific tenant.
  - **WithRedirectUri(uri):** Callback URL for authorization code flow.
  - **WithClientId(clientId)** / **WithClientSecret(secret)** or **WithCertificate(cert):** App identity for confidential clients.

---

### Implement solutions with Microsoft Graph

**Microsoft Graph** is the unified API for Microsoft 365 and related services: a single endpoint to access users, mail, calendar, teams, devices, and more. You can call it via **REST** or via the **Microsoft Graph SDK** (.NET: `Microsoft.Graph`, `Microsoft.Graph.Core`; authentication is handled by an auth provider you pass in). Graph requires an **OAuth 2.0 access token** issued by the Microsoft identity platform (Entra ID). The token must include the appropriate **scopes** for the operations you perform (e.g. `User.Read`, `Mail.Read`, or for app-only `https://graph.microsoft.com/.default`).

**Which token does Graph require?**

- An **access token** (OAuth 2.0 bearer token) from the Microsoft identity platform.
- **Delegated access:** Request scopes like `User.Read`, `Mail.Read`; the token represents the signed-in user.
- **Application access (app-only):** Request the scope `https://graph.microsoft.com/.default`; the token represents the app (admin consent required for the app permissions).

**How to query Microsoft Graph by using REST**

1. Obtain an access token (e.g. via MSAL) with the required scope (e.g. `https://graph.microsoft.com/.default` or `User.Read`).
2. Send an HTTP request to `https://graph.microsoft.com/v1.0/<resource>` (or `beta` for beta APIs) with header: `Authorization: Bearer <access_token>`.
3. Use standard HTTP methods: GET (read), POST (create), PATCH (update), DELETE.

**Example – REST (GET users):**

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

**How to query Microsoft Graph by using SDK (.NET)**

1. Install packages: `Microsoft.Graph` and an auth package (e.g. `Microsoft.Identity.Client` for MSAL).
2. Build an authentication provider that gets the access token (e.g. `ClientSecretCredential` for app-only or a delegate provider that uses MSAL).
3. Create a `GraphServiceClient` with that provider.
4. Call the SDK methods (e.g. `graphClient.Users.GetAsync()`); the SDK sends REST requests and adds the token automatically.

**Example – create a Graph client and retrieve a list of users (.NET):**

```csharp
using Azure.Identity;
using Microsoft.Graph;

// 1. Build credential (app-only example; for user-delegated use MSAL and a delegate provider)
var credential = new ClientSecretCredential(
    tenantId: "<tenant-id>",
    clientId: "<client-id>",
    clientSecret: "<client-secret>");

// 2. Create auth provider and Graph client
var graphClient = new GraphServiceClient(credential);

// 3. Retrieve list of users
var users = await graphClient.Users
    .GetAsync(requestConfiguration => requestConfiguration.QueryParameters.Select = "displayName,mail,userPrincipalName");

foreach (var user in users?.Value ?? Enumerable.Empty<Microsoft.Graph.Models.User>())
{
    Console.WriteLine($"{user.DisplayName} – {user.Mail ?? user.UserPrincipalName}");
}
```

**Example – retrieve other entities (e.g. groups):**

```csharp
var groups = await graphClient.Groups.GetAsync();
foreach (var group in groups?.Value ?? Enumerable.Empty<Microsoft.Graph.Models.Group>())
{
    Console.WriteLine(group.DisplayName);
}
```

The same pattern applies: obtain a token (via MSAL or `ClientSecretCredential`), create a `GraphServiceClient`, then call the appropriate collection (e.g. `Users`, `Groups`, `Me.MailFolders`) and handle the result.


# Develop Solutions for Microsoft Azure – Day 4 (05.03)

**Topics:** Implement secure cloud solutions (Azure Key Vault), Implement API Management, Develop event-based solutions (Event Grid, Event Hubs)

---

## Implement secure cloud solutions: Azure Key Vault

### What is Azure Key Vault?

**Azure Key Vault** is a cloud service for **securely storing and accessing secrets**. It centralizes application secrets, keys, and certificates so you avoid storing them in code, config files, or environment variables. Key Vault provides:

- **Hardware Security Module (HSM)–backed storage** for keys (when using Premium tier).
- **Audit logging** of all access (who accessed what and when).
- **Access control** via Azure RBAC or Key Vault access policies.
- **Integration** with Azure services (App Service, Functions, AKS, VMs) and Azure AD.

### What Key Vault stores (three object types)

| Type | Purpose | Typical use |
|------|---------|-------------|
| **Secrets** | Arbitrary key–value pairs (e.g. connection strings, API keys, passwords). Stored encrypted; retrieved via REST or SDK. | `SqlConnectionString`, `ApiKey`, `StorageAccountKey` |
| **Keys** | Cryptographic keys (symmetric or asymmetric). Used for encrypt/decrypt, sign/verify. Can be HSM-backed. | Encryption at rest, JWT signing, key wrap |
| **Certificates** | X.509 certificates (public + private key). Supports renewal and lifecycle. | TLS/SSL, client auth, code signing |

Secrets have a **max size** (e.g. 25 KB for secret value). For larger data, store a reference (e.g. storage SAS URL) in Key Vault, not the data itself.

### Key benefits (exam-relevant)

- **Monitoring:** All access is logged to Log Analytics or Event Hub. You can detect anomalous access and meet compliance.
- **Rotation:** You can rotate secrets without redeploying the app (e.g. update the secret in the vault; app reads the new value on next fetch). Optional integration with **Azure App Configuration** and **Event Grid** for secret rotation events.
- **No secrets in code:** Apps authenticate to Azure AD (e.g. managed identity) and then read secrets from Key Vault at runtime.
- **Unified access control:** One place to grant/revoke access per app or environment.

### Authentication to Key Vault (three approaches)

To read/write Key Vault, the **caller** (app, script, user) must authenticate. Key Vault uses **Azure AD** for identity. Common approaches:

| Method | Description | When to use |
|--------|-------------|-------------|
| **1. Managed identity for Azure resources** | The app runs on an Azure resource (App Service, VM, Function App, AKS). You enable a **system-assigned** or **user-assigned** managed identity and grant that identity access to the vault. No credentials in code. | **Recommended** for apps hosted on Azure. No secrets to manage. |
| **2. Service principal + certificate** | You register an app in Azure AD and authenticate with a **client ID + certificate** (private key). The certificate is installed on the machine or stored securely. | Automation, CI/CD, or on-premises apps where managed identity is not available. |
| **3. Service principal + client secret** | Same as above but using a **client secret** instead of a certificate. Simpler but **less secure** (secret can be leaked). | Dev/test only; **not recommended** for production. |

**Best practice:** Prefer **managed identity** for any workload running in Azure (App Service, Functions, VM, AKS). Use **certificate-based** service principal for automation or hybrid scenarios. Avoid client secrets in production.

### Two ways to control access: Access policy vs RBAC

Key Vault supports two **access models** (you choose one per vault; they are mutually exclusive in practice):

**1. Vault access policy (classic)**  
- You attach **access policies** to the vault. Each policy is a **principal** (user, app, or managed identity) + **permissions** (e.g. Get, List for secrets; Get, List, Create for keys).  
- Permissions are **Key Vault–specific** (Secrets: Get/List; Keys: Get/List/Create/Sign; Certificates: Get/List/Update).  
- **Use when:** You want simple, vault-scoped “who can read secrets” rules. Legacy model; still widely used.

**2. Azure RBAC (role-based access control)**  
- You assign **Azure roles** (e.g. **Key Vault Secrets User**, **Key Vault Crypto User**, **Key Vault Administrator**) to users, groups, or managed identities at the vault (or higher) scope.  
- Roles are aligned with Azure’s RBAC model and work with PIM, conditionals, and management groups.  
- **Use when:** You want to standardize on RBAC, use built-in roles (Secrets User, Crypto User, Administrator), or integrate with Azure AD groups and PIM.

**Exam tip:** Know that both exist. “Key Vault Secrets User” allows **read** (Get, List) of secrets; “Key Vault Administrator” allows full access. For an app that only reads secrets, assign **Key Vault Secrets User** to its managed identity.

### Using Key Vault in code: SDK and DefaultAzureCredential

**NuGet packages (typical):**

- **Azure.Security.KeyVault.Secrets** – get/list/create/delete secrets.
- **Azure.Identity** – authentication (credentials that work with Key Vault).

Other packages: **Azure.Security.KeyVault.Keys**, **Azure.Security.KeyVault.Certificates** for keys and certs.

**DefaultAzureCredential** (from `Azure.Identity`):

- A **credential chain** that tries several authentication methods in order: environment variables → managed identity → Visual Studio / Azure CLI (for local dev) → etc.  
- **In Azure (e.g. App Service, VM, Function):** When you enable **managed identity**, `DefaultAzureCredential` will use it automatically—no code change when moving from dev to prod.  
- **Locally:** It uses Azure CLI or Visual Studio login so you don’t need a client secret in dev.  
- Use it when creating the **SecretClient:**

```csharp
var vaultUri = new Uri("https://<your-vault-name>.vault.azure.net/");
var client = new SecretClient(vaultUri, new DefaultAzureCredential());
KeyVaultSecret secret = await client.GetSecretAsync("MySecretName");
string value = secret.Value;
```

**Best practice:** Store the **Key Vault URI** (or vault name) in app settings; never store secrets in app settings. The app uses managed identity + URI to read secrets at runtime.

### Soft-delete and purge

- **Soft-delete** is **always on** for Key Vault (you cannot disable it). When you delete a secret, key, or certificate, it is marked deleted but retained for a **configurable retention period** (default 90 days).  
- During retention you can **recover** the object. After retention, it can be **purged** (permanent delete).  
- **Purge protection** (optional): Prevents permanent deletion until the retention period has passed; useful for compliance.  
- **Exam:** Know that soft-delete is mandatory, and that recovery and purge are part of the lifecycle.

### Best practices (summary)

1. **One vault per app (or per environment):** Isolate dev/prod; limit blast radius.  
2. **Control access:** Use RBAC or access policies; grant minimum permissions (e.g. Secrets User for read-only).  
3. **Backup:** For keys and certificates, use Key Vault backup/restore or your own process; secrets can be re-created from app config if needed.  
4. **Logging:** Enable **diagnostic settings** and send logs to Log Analytics or Storage for audit.  
5. **Recovery:** Use soft-delete and optionally purge protection; document recovery steps.  
6. **Network:** Use **private endpoints** or **firewall rules** to restrict access to Key Vault from selected VNets and IPs (exam may mention private link / firewall).

---

## Implement API Management

### Why API Management?

You have backend APIs (your own or third-party) and want to **expose them in a controlled way**: authentication, rate limiting, transformation, logging, and a clear contract for consumers. **Azure API Management (APIM)** is a managed **API gateway** that sits between consumers and your APIs and provides:

- **Gateway:** Single entry point; routing, policies, caching.  
- **Management plane:** Create and manage APIs, products, subscriptions in Azure Portal or via ARM/APIM REST API.  
- **Developer portal:** Self-service portal where developers discover APIs, get subscription keys, and try operations.

### Main components (exam-relevant)

| Component | Role |
|-----------|------|
| **API gateway** | Receives client requests, applies policies (auth, rate limit, transform), forwards to backend, returns response. Can cache responses. |
| **Azure portal (management)** | Admin configures APIs, products, groups, policies, analytics. Used by API owners/ops. |
| **Developer portal** | Developers browse APIs, subscribe to products, get keys, run try-it-now. Can be customized and self-hosted. |

**APIs and operations:**  
- An **API** in APIM is a logical grouping (e.g. “Orders API”) with a **base URL** and optional **version** (e.g. v1, v2).  
- Each API contains **operations** (e.g. `GET /orders`, `POST /orders`). Each operation maps to a **backend** HTTP method and URL (and can override the backend URL).  
- **Products** group one or more APIs. Consumers **subscribe** to a product and get a **subscription key** (or use OAuth/other auth) to call APIs in that product.

### Creating an API Management instance

1. **Portal:** Create a resource → **API Management** → choose **Consumption** or **Developer / Standard / Premium** tier.  
2. **Pricing tiers:**  
   - **Consumption:** Serverless; pay per call; no SLA; good for dev/spiky workloads.  
   - **Developer:** Dev/test; no SLA.  
   - **Standard / Premium:** Production; SLA; VNet injection, multi-region (Premium).  
3. **Deployment:** After creation you get a gateway URL (e.g. `https://<name>.azure-api.net`). You then **add APIs** and **operations** and attach **policies**.

### Creating and documenting APIs

- **Add API:** Define name, URL suffix, base URL (optional), possibly version (e.g. “v1”).  
- **Add operations:** For each endpoint (e.g. `GET /orders`), define display name, URL (path + method), and **backend** (full URL or “use base + path”).  
- **Documentation:** In the **Developer portal**, you add descriptions, request/response examples, and terms. You can also import **OpenAPI (Swagger)** so the API and operations are created from the spec; APIM can expose the spec and a “try it” experience.  
- **Revisions and versions:** Use **revisions** for in-place iteration (same URL); use **versions** (e.g. v1, v2) when you want a distinct path or product for backward compatibility.

### Configuring access to the API

- **Subscription keys:** APIM can require a **subscription key** (header `Ocp-Apim-Subscription-Key` or query). Key is tied to a **subscription** (scope: all APIs, or a product). Create subscriptions in the portal; give the key to developers.  
- **OAuth 2.0 / OpenID Connect:** Configure a **developer console** OAuth2 server or validate **JWT** in policy so that only valid tokens can call the API.  
- **Client certificates:** Require a client cert for TLS mutual auth; validate in policy.  
- **IP / VNet:** Restrict caller IP or allow only traffic from a VNet (Premium; use **virtual network integration**).

**Exam:** Know subscription key vs OAuth; know that access can be at **product** level (subscription to product) or **API** level.

### Policies: what they are and where they apply

**Policies** are XML fragments that run at defined **stages** of the request/response pipeline. They execute in order and can **set variables**, **change requests/responses**, **call external services**, or **short-circuit** (e.g. return mock or error).

**Scope (order of application):**  
Global → Product → API → Operation. More specific scope overrides or adds to broader scope.

**Stages:**

- **Inbound:** After request is received, before forwarding to backend. Use for: validate JWT, check subscription, rate limit, set headers, transform URL.  
- **Backend:** Before request is sent to backend. Use for: change backend URL, set backend headers.  
- **Outbound:** After backend response, before sending to client. Use for: remove headers, transform response, cache.  
- **On-error:** When an error occurs in any stage. Use for: custom error response, logging.

### Important policy examples (exam-style)

| Policy | Scope / stage | Purpose |
|--------|----------------|---------|
| **validate-jwt** | Inbound | Validate token (OpenID Connect); reject if invalid. |
| **check-subscription** | Inbound | Ensure valid subscription key (often automatic when subscriptions are enabled). |
| **rate-limit** / **quota** | Inbound | Throttle by key or IP; enforce call quota. |
| **set-backend-service** | Backend | Route to a different backend URL. |
| **cache-store** / **cache-lookup** | Inbound/Outbound | Cache responses (e.g. cache-lookup inbound; on cache miss, forward; outbound cache-store). |
| **forward-request** | Inbound | Send request to backend (default if no policy short-circuits). |
| **return-response** | Inbound/Outbound | Return a fixed response (e.g. mock) without calling backend. |
| **retry** | Inbound | Retry backend call with condition (e.g. on 5xx). |
| **log-to-eventhub** | Inbound/Outbound | Send message to Event Hub for analytics or audit. |
| **limit-concurrency** | Inbound | Cap concurrent requests to backend to avoid overload. |

**Base policy:** Every scope has a base policy (e.g. `<inbound><base/><forward-request/></inbound>`). `<base/>` includes the policy from the parent scope.

### Caching

- **Response caching:** Use **cache-lookup** (inbound) and **cache-store** (outbound) with a cache duration and key (e.g. by subscription + URL). APIM has an **internal cache** (per gateway node) or you can use **external Redis** (Standard/Premium) for distributed cache.  
- **Vary by:** Cache key can include headers (e.g. `Accept-Language`) so different responses are cached per key.

---

## Develop event-based solutions

Azure provides two main event-driven services that appear on AZ-204: **Event Grid** (event routing and reaction) and **Event Hubs** (high-throughput event ingestion and streaming). They are used for different scenarios.

---

### Azure Event Grid

**What it is:**  
Event Grid is a **serverless event routing service**. It connects **event sources** (publishers) to **event handlers** (subscribers) via **topics** and **subscriptions**. Events are **delivered** (push) to handlers; Event Grid does not store events for long-term replay (unlike Event Hubs).

Example: Blob uploaded → Event Grid → Azure Function runs

**Concepts (exam-relevant):**

| Concept | Description |
|---------|-------------|
| **Event** | A small JSON message describing “something that happened” (past tense). Contains: id, subject, data, eventType, eventTime, dataVersion. Schema can be **Event Grid schema** or **Cloud Events 1.0**. |
| **Publisher / event source** | Service or app that **sends** events (e.g. Storage Account, Resource Group, Custom Topic, IoT Hub, Media Services). |
| **Topic** | Endpoint where publishers send events. **System topics** (e.g. for Blob created, Resource Group) are built-in; **custom topics** are user-created for app events. |
| **Event subscription** | Binding from a topic to a **handler** (webhook, Function, Queue, Event Hubs, etc.). You define filters (event type, subject prefix) and optional **dead-letter** and **retry** settings. |
| **Handler** | Destination that receives events: Azure Function, Webhook (HTTP), Storage Queue, Service Bus Queue/Topic, Event Hubs, Hybrid Connection, etc. |

**How it works:**  
1. A source (e.g. Blob Storage) emits an event when something happens (e.g. blob created).  
2. The event is sent to a **topic** (system topic for Blob, or a custom topic).  
3. **Event Grid** matches the event to **subscriptions** (filters) and **delivers** to each subscriber’s endpoint (POST with event payload).  
4. Handler returns 200 OK; otherwise Event Grid **retries** (exponential backoff). After max retries, event can be sent to **dead-letter** (Storage) if configured.

**Delivery and retries:**  
- **Retry policy:** Configurable (e.g. max delivery attempts, time-to-live). Events are retried on failure (e.g. 5xx, timeout).  
- **Dead-letter:** If delivery fails after all retries, the event can be stored in a Storage container for later inspection.  
- **Idempotency:** Handlers should be idempotent (same event may be delivered more than once in edge cases).

**Use cases (exam):**  
- React to Azure resource events (VM created, Blob uploaded, Resource deleted).  
- Custom app events (order placed, user registered) via **custom topic**.  
- Decouple publishers and subscribers (many subscribers to one topic).  
- Trigger serverless (Function, Logic App) from events.

**Pricing model:** Pay per **operations** (event published, delivery attempt, etc.); no fixed cost for “idle” topics.

---

### Azure Event Hubs

**What it is:**  
Event Hubs is a **managed event streaming** and **ingestion** service. It is designed for **high throughput** and **retention**: producers send events to a **namespace** and **event hub**; consumers read from **partitions** in a **consumer group**. Events are stored for a **retention period** (e.g. 1–7 days) so multiple consumers can read the same stream.

Example: 10,000 IoT devices → Event Hub → Stream Analytics → Data Lake

**Concepts (exam-relevant):**

| Concept | Description |
|---------|-------------|
| **Namespace** | Container for one or more event hubs; has a FQDN and shared networking/throughput settings. |
| **Event hub** | Named stream of events. Has **partitions** (ordered sequence per partition). |
| **Partition** | Ordered event stream. Producers can send with **partition key** (same key → same partition) or let the service assign. **Partition count** is set at creation (cannot change later for standard tier). |
| **Consumer group** | View of the full event hub. Each consumer group has its own offset per partition so multiple apps can read the same stream independently. Default: `$Default`. |
| **Throughput units (TU)** / **Processing units (PU)** | Standard tier: **throughput units** (ingress + egress). Premium: **processing units**. Auto-inflate can scale TUs. |
| **Capture** | Optional: automatically send event hub data to **Blob Storage** or **Data Lake** in Avro format for batch/analytics. |

**Producers:** Send events via **AMQP** or **HTTPS** (REST). Can use **partition key** for ordering or **partition id** for direct send. **Event Hubs SDK** (e.g. .NET) handles batching and connection management.

**Consumers:** Read via **Event Processor Client** (recommended) or **consumer client** per partition. Event Processor uses **checkpointing** (store offset in Blob) so processing can resume and scale across instances.

**Event Grid vs Event Hubs (exam):**

| Aspect | Event Grid | Event Hubs |
|--------|------------|------------|
| **Purpose** | Route events to handlers (push); react to events. | Ingest and stream events; store for retention; pull/process. |
| **Retention** | No retention (deliver and done). | Configurable retention (e.g. 1–7 days). |
| **Throughput** | High but tuned for routing. | Very high (millions events/sec) with partitions. |
| **Consumers** | Each subscription gets a copy (push). | Multiple consumer groups read same stream (pull). |
| **Typical use** | React to Blob upload, resource changes, custom app events. | Telemetry, logs, stream processing (e.g. with Stream Analytics, Spark). |

**Use cases for Event Hubs:**  
- Telemetry and logging from devices or apps.  
- Stream processing (e.g. Azure Stream Analytics, Spark).  
- **Capture** to Data Lake or Blob for batch analytics.  
- Building a **event-sourced** or replayable pipeline.

**Security:**  
- **Shared Access Signature (SAS)** or **Azure AD** (recommended) for auth.  
- **Private endpoints** and **firewall rules** for network isolation (Premium).


There is also Azure Service Bus - message broker. Similar to RabbitMQ. Supports: queues, topics, subscriptions, dead-letter queues, transactions, message-ordering. 

Example flow: Order Service → Service Bus Queue → Payment Service

| Feature           | Event Grid      | Service Bus            | Event Hubs            |
| ----------------- | --------------- | ---------------------- | --------------------- |
| Purpose           | Event routing   | Reliable messaging     | Event streaming       |
| Message volume    | Low–medium      | Medium                 | Extremely high        |
| Persistence       | Short           | Long                   | Stream retention      |
| Ordering          | No              | Yes                    | Partition-based       |
| Complex workflows | No              | Yes                    | No                    |
| Typical use       | React to events | Microservice messaging | Telemetry / analytics |


---

## Quick reference for AZ-204

- **Key Vault:** Secrets/keys/certificates; auth via managed identity (preferred) or service principal; access via RBAC or access policy; use `DefaultAzureCredential` + SecretClient; soft-delete always on.  
- **API Management:** Gateway + management + developer portal; APIs → operations → backend; products and subscriptions for access; policies (inbound/backend/outbound/on-error) for auth, rate limit, cache, retry, mock, log to Event Hub.  
- **Event Grid:** Event routing; topics and subscriptions; push to handlers; retry and dead-letter; system and custom topics.  
- **Event Hubs:** High-throughput ingestion; partitions and consumer groups; retention; capture to Storage; use for streaming and replay, not just one-off delivery.

