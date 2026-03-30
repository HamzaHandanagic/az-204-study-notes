
# AZ-204 Practice Questions

50 exam-style questions based on the Azure Developer Associate certification topics.

---

## Questions

### Azure App Service (1–8)

**1.** You need to use deployment slots (e.g. staging → swap to production) for your web app. What is the **minimum** App Service plan tier that supports deployment slots?

- A) Basic
- B) Standard
- C) Free

---

**2.** You have an App Service running on the Free tier. A colleague reports the app goes cold after a period of inactivity. Which General setting would keep the app loaded at all times?

- A) HTTPS Only
- B) Always On
- C) ARR Affinity

---

**3.** You want app configuration values to remain with a specific slot and **not** move during a swap. What should you do?

- A) Store them in `appsettings.json` inside the deployed code
- B) Mark the App Settings as slot-specific (sticky)
- C) Use connection strings instead of app settings

---

**4.** You are deploying a containerized application to Azure App Service. Where do you configure the image source and tag?

- A) Configuration → General settings
- B) Deployment Center
- C) Custom domains blade

---

**5.** Your production web app experiences variable traffic. You want instances to scale automatically based on CPU usage. Which scaling approach should you use?

- A) Scale up to a higher tier
- B) Manual scale with a fixed instance count
- C) Custom autoscale with a CPU-based rule

---

**6.** Which tool gives you access to deployed files, logs, and environment details at `https://<app-name>.scm.azurewebsites.net`?

- A) Azure Monitor
- B) Kudu (SCM)
- C) Deployment Center

---

**7.** You need to add a custom domain to your App Service. Which DNS record type is used to map a subdomain (e.g. `www.contoso.com`) to the App Service?

- A) MX
- B) TXT
- C) CNAME

---

**8.** Your App Service needs a free, Microsoft-provided SSL certificate for a single hostname. Which option should you choose?

- A) Upload your own certificate from a CA
- B) App Service Managed Certificate
- C) Azure Key Vault certificate reference

---

### Azure Function Apps (9–16)

**9.** You have an Azure Function on the **Consumption** plan. What is the **default** timeout for a function execution?

- A) 1 minute
- B) 5 minutes
- C) 30 minutes

---

**10.** A function is triggered by an HTTP POST, reads a customer record from Cosmos DB, and writes a message to a Storage Queue. How are the Cosmos DB read and Queue write configured?

- A) Cosmos DB is an input binding; Queue is an output binding
- B) Both are triggers
- C) Both are output bindings

---

**11.** You want your Azure Function to avoid cold starts and support VNet integration. Which hosting plan should you choose?

- A) Consumption
- B) Premium
- C) Flex Consumption

---

**12.** Which file defines runtime behavior (logging, concurrency, extension settings) for the **entire** Function App and is deployed with your code?

- A) `local.settings.json`
- B) `function.json`
- C) `host.json`

---

**13.** On the Consumption plan, what happens when a Function App has no incoming events for an extended period?

- A) The app continues running on a reserved instance
- B) The app scales to zero and the next invocation may experience a cold start
- C) The app switches to the Dedicated plan automatically

---

**14.** You need multiple functions to scale independently of each other. What should you do?

- A) Place each function in its own Function App
- B) Place all functions in one Function App and configure per-function scaling
- C) Use different `function.json` scale settings per function

---

**15.** What is the **maximum** timeout you can configure for a function on the Consumption plan?

- A) 5 minutes
- B) 10 minutes
- C) 60 minutes

---

**16.** On the **Premium** plan, you want to ensure that at least 2 instances are always ready to handle requests. What should you configure?

- A) Set the minimum instance count to 2 (pre-warmed instances)
- B) Enable Always On in General settings
- C) Set the function timeout to unlimited

---

### Azure Blob Storage (17–24)

**17.** You need to store log data that is only appended to and never modified in place. Which blob type is most appropriate?

- A) Block blob
- B) Page blob
- C) Append blob

---

**18.** A blob in the **Archive** tier needs to be read immediately. What must you do first?

- A) Change the blob tier to Hot or Cool (rehydrate)
- B) Download it directly; Archive blobs are always accessible
- C) Delete and re-upload the blob to Hot tier

---

**19.** You move a blob to the **Cool** tier and delete it 10 days later. What cost implication should you expect?

- A) No extra cost; Cool tier has no minimum retention
- B) An early deletion fee because Cool has a 30-day minimum retention
- C) The blob is automatically moved to Archive before deletion

---

**20.** Which storage account type is recommended as the default for new deployments supporting Blob, File, Queue, and Table?

- A) Blob Storage (legacy)
- B) General Purpose v2 (GPv2)
- C) Block Blob Storage

---

**21.** You want to automate moving blobs from Hot to Cool after 30 days and delete them after 365 days. Which feature should you use?

- A) Azure Policy
- B) Blob lifecycle management rules
- C) Azure Advisor recommendations

---

**22.** For production workloads, what is the **recommended** authentication method to access Blob Storage?

- A) Storage account access key
- B) Microsoft Entra ID (Azure AD) with RBAC
- C) Anonymous public access

---

**23.** In Blob Storage, you see a path `images/2024/photo.jpg`. What does this path represent?

- A) A real folder hierarchy inside the container
- B) A flat blob name that simulates folders using delimiters
- C) A separate container named `images` with a sub-container `2024`

---

**24.** Which redundancy option provides the **highest** durability by combining geo-replication and zone redundancy?

- A) LRS
- B) GRS
- C) GZRS

---

### Azure Cosmos DB (25–30)

**25.** You are creating a new Cosmos DB account for a document-based application that needs SQL-like queries. Which API should you select?

- A) MongoDB API
- B) NoSQL (SQL) API
- C) Table API

---

**26.** In Cosmos DB, what is the unit used to measure the cost of a database operation?

- A) DTU (Database Transaction Unit)
- B) RU (Request Unit)
- C) CU (Compute Unit)

---

**27.** Your application has highly variable traffic with long idle periods. Which Cosmos DB throughput mode is best suited?

- A) Provisioned throughput (manual)
- B) Serverless
- C) Autoscale provisioned

---

**28.** At which level in the Cosmos DB hierarchy do you define the **partition key**?

- A) Account level
- B) Database level
- C) Container level

---

**29.** Which consistency level is the **default** for many applications and provides per-session read-your-own-writes guarantees?

- A) Strong
- B) Session
- C) Eventual

---

**30.** You need the lowest possible read latency and can tolerate occasionally stale reads. Which consistency level should you choose?

- A) Bounded staleness
- B) Strong
- C) Eventual

---

### Containerized Solutions (31–36)

**31.** Which Dockerfile instruction sets the default command that runs when a container starts?

- A) `RUN`
- B) `CMD`
- C) `COPY`

---

**32.** You have a local image `myapp:v1`. To push it to Azure Container Registry `myacr.azurecr.io`, what must you do first?

- A) Tag it as `myacr.azurecr.io/myapp:v1`
- B) Rename the Dockerfile to match the registry name
- C) Convert the image to a `.tar` file and upload manually

---

**33.** Which ACR tier supports **geo-replication** to multiple regions?

- A) Basic
- B) Standard
- C) Premium

---

**34.** In Azure Container Instances, you want a container to restart only when it exits with a non-zero exit code. Which restart policy should you use?

- A) Always
- B) On-failure
- C) Never

---

**35.** You need to deploy a multi-container group in ACI with a web app and a sidecar. Which deployment method is recommended?

- A) Azure Portal UI
- B) YAML file with `az container create --file`
- C) Docker Compose on the local machine

---

**36.** In an ACI container group, how do sidecar containers communicate with the main container?

- A) Via a public endpoint assigned to each container
- B) Via `localhost` (they share the same network namespace)
- C) Via an Azure Service Bus queue

---

### Authentication & Authorization (37–42)

**37.** You are building a daemon service (no user sign-in) that calls Microsoft Graph. Which OAuth 2.0 flow should you use?

- A) Authorization code flow
- B) Client credentials flow
- C) Device code flow

---

**38.** What are the **two objects** created in Azure AD when you register an application?

- A) Application object and Service principal
- B) Managed identity and Access policy
- C) Client secret and Client certificate

---

**39.** Your web API receives a token from a client and needs to call a downstream API on behalf of the same user. Which flow should you use?

- A) Implicit grant
- B) On-behalf-of (OBO)
- C) Client credentials

---

**40.** Which MSAL builder should you use for a server-side web app that can securely store a client secret?

- A) `PublicClientApplicationBuilder`
- B) `ConfidentialClientApplicationBuilder`
- C) `ManagedIdentityApplicationBuilder`

---

**41.** An app needs the permission "Read all users" and runs without any signed-in user. What type of permission is this?

- A) Delegated permission
- B) Application permission (app-only)
- C) Incremental consent permission

---

**42.** You want to retrieve a list of users from Microsoft 365 using the Graph SDK in .NET. Which class do you instantiate to make the call?

- A) `HttpClient`
- B) `GraphServiceClient`
- C) `SecretClient`

---

### Azure Key Vault (43–46)

**43.** Your App Service needs to read secrets from Key Vault in production. What is the **recommended** authentication method?

- A) Service principal with client secret
- B) Managed identity for the App Service
- C) Storing the Key Vault access key in app settings

---

**44.** You delete a secret from Key Vault. Can you recover it?

- A) No; deletion is permanent
- B) Yes; soft-delete is always enabled and the secret can be recovered within the retention period
- C) Only if you enabled soft-delete manually before deletion

---

**45.** Which Azure RBAC role should you assign to an app's managed identity if it only needs to **read** secrets?

- A) Key Vault Administrator
- B) Key Vault Secrets User
- C) Key Vault Crypto Officer

---

**46.** Which credential class from `Azure.Identity` automatically works with managed identity in Azure **and** Azure CLI locally?

- A) `ClientSecretCredential`
- B) `DefaultAzureCredential`
- C) `ManagedIdentityCredential`

---

### API Management (47–48)

**47.** In API Management, you want to validate a JWT token on every incoming request. In which **policy section** should you place the `validate-jwt` policy?

- A) Outbound
- B) Backend
- C) Inbound

---

**48.** A developer needs to discover your APIs, subscribe to a product, and get a subscription key. Which APIM component provides this self-service experience?

- A) API gateway
- B) Developer portal
- C) Azure Portal management blade

---

### Event-Based Solutions (49–50)

**49.** A blob is uploaded to Azure Storage and you want an Azure Function to react to this event. Which service routes the event from Storage to the Function?

- A) Azure Event Hubs
- B) Azure Event Grid
- C) Azure Service Bus

---

**50.** You need to ingest millions of telemetry events per second from IoT devices and retain them for 7 days for stream processing. Which service should you use?

- A) Azure Event Grid
- B) Azure Event Hubs
- C) Azure Storage Queues

---

---

## Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **B** | **Standard** is the minimum tier that supports deployment slots. Basic supports custom domains but not slots. |
| 2 | **B** | **Always On** keeps the app loaded and prevents it from going idle. It is not available on Free/Shared tiers. |
| 3 | **B** | Marking app settings as **slot-specific (sticky)** ensures they stay with the slot and do not move during a swap. |
| 4 | **B** | The **Deployment Center** is where you configure the container image source (registry, image, and tag) for a containerized App Service. |
| 5 | **C** | **Custom autoscale** with a CPU-based scale rule automatically adds or removes instances based on the metric threshold. |
| 6 | **B** | **Kudu (SCM)** is the debugging/management console accessible at the `.scm.azurewebsites.net` URL. |
| 7 | **C** | A **CNAME** record maps a subdomain (e.g. `www`) to the App Service default hostname. An A record is used for root domains. |
| 8 | **B** | **App Service Managed Certificate** is free and Microsoft-managed, providing one certificate per hostname. |
| 9 | **B** | The default timeout on the Consumption plan is **5 minutes**; the maximum is 10 minutes. |
| 10 | **A** | Cosmos DB is read via an **input binding**; the Queue is written via an **output binding**. Only the HTTP POST is the trigger. |
| 11 | **B** | The **Premium** plan provides pre-warmed instances (no cold start) and VNet integration. |
| 12 | **C** | **`host.json`** configures runtime-level behavior for the entire Function App (logging, concurrency, extensions). |
| 13 | **B** | On Consumption, the app **scales to zero** when idle. The next invocation triggers a new instance with potential cold start. |
| 14 | **A** | The Function App is the unit of scaling. All functions in one app scale together, so separate apps are needed for independent scaling. |
| 15 | **B** | The maximum configurable timeout on the Consumption plan is **10 minutes**. |
| 16 | **A** | On the Premium plan, setting the **minimum instance count** keeps pre-warmed instances always ready. |
| 17 | **C** | **Append blobs** are optimized for append-only operations like logging. Block blobs support random write; page blobs are for VHDs. |
| 18 | **A** | Archive blobs are offline. You must **rehydrate** (change tier to Hot or Cool) before reading. This can take hours. |
| 19 | **B** | Cool tier has a **30-day minimum retention**. Deleting before 30 days incurs an early deletion fee. |
| 20 | **B** | **General Purpose v2 (GPv2)** is the recommended default; it supports all storage services (Blob, File, Queue, Table). |
| 21 | **B** | **Blob lifecycle management rules** automate tier transitions and deletion based on age or other filters. |
| 22 | **B** | **Microsoft Entra ID with RBAC** (e.g. Storage Blob Data Contributor) is the recommended production auth method over access keys. |
| 23 | **B** | Blob Storage has a flat namespace. The `/` delimiter in the name simulates a folder hierarchy but it is a single blob name. |
| 24 | **C** | **GZRS** (Geo-Zone-Redundant Storage) combines zone redundancy within a region and geo-replication for the highest durability. |
| 25 | **B** | The **NoSQL (SQL) API** is the native Cosmos DB API for JSON documents with SQL-like query support. |
| 26 | **B** | **Request Units (RU)** measure the cost of database operations in Cosmos DB (reads, writes, queries). |
| 27 | **B** | **Serverless** mode charges per request with no provisioned throughput, ideal for variable traffic with idle periods and scale-to-zero. |
| 28 | **C** | The partition key is defined at the **container** level and cannot be changed after creation. |
| 29 | **B** | **Session** consistency is the default for many apps, providing read-your-own-writes guarantees within a session. |
| 30 | **C** | **Eventual** consistency provides the lowest latency but reads may return stale data. |
| 31 | **B** | **`CMD`** sets the default command to run when the container starts. `RUN` executes during build; `COPY` copies files. |
| 32 | **A** | Images must be tagged with the ACR login server prefix (`myacr.azurecr.io/myapp:v1`) before pushing. |
| 33 | **C** | **Premium** tier supports geo-replication, zone redundancy, and content trust. |
| 34 | **B** | **On-failure** restarts the container only on non-zero exit codes (errors). Zero exit = no restart. |
| 35 | **B** | Multi-container groups are not fully supported in the portal. Use a **YAML file** or ARM/Bicep template. |
| 36 | **B** | Containers in the same ACI group share the network namespace and communicate via **`localhost`**. |
| 37 | **B** | **Client credentials** flow is for app-only (daemon) scenarios with no signed-in user. |
| 38 | **A** | Registering an app creates an **Application object** (template) and a **Service principal** (instance in the tenant). |
| 39 | **B** | **On-behalf-of (OBO)** lets a middle-tier API exchange the user's token for a new token to call a downstream API. |
| 40 | **B** | **`ConfidentialClientApplicationBuilder`** is for apps that can securely store credentials (web apps, APIs). |
| 41 | **B** | **Application permissions** are used when the app acts as itself without a user; admin consent is required. |
| 42 | **B** | **`GraphServiceClient`** is the main class in the Microsoft Graph SDK for making API calls. |
| 43 | **B** | **Managed identity** is the recommended approach; no credentials to manage. The identity authenticates to Key Vault via Azure AD. |
| 44 | **B** | **Soft-delete is always enabled** in Key Vault. Deleted secrets are retained and recoverable within the retention period (default 90 days). |
| 45 | **B** | **Key Vault Secrets User** grants read access (Get, List) to secrets. Administrator grants full control. |
| 46 | **B** | **`DefaultAzureCredential`** chains multiple credential types: managed identity in Azure, Azure CLI/VS locally. |
| 47 | **C** | Token validation should happen in the **Inbound** section, before the request reaches the backend. |
| 48 | **B** | The **Developer portal** is the self-service site where developers discover APIs, subscribe, and get keys. |
| 49 | **B** | **Azure Event Grid** routes resource events (e.g. blob created) to handlers like Azure Functions. |
| 50 | **B** | **Azure Event Hubs** is designed for high-throughput event ingestion (millions/sec) with configurable retention for stream processing. |
