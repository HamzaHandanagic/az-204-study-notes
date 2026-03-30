
# AZ-204 Practice Questions – Set 2

50 high-likelihood exam questions sourced from commonly tested AZ-204 objectives.

---

## Questions

### Implement Azure App Service Web Apps (1–10)

**1.** You deploy an ASP.NET Core app to Azure App Service. The app reads a database connection string from an environment variable. Where should you store this value so it is **not** checked into source control and is available at runtime?

- A) In the App Service **Connection strings** configuration
- B) In the `launchSettings.json` file in the repository
- C) Hard-coded in `Program.cs`

---

**2.** You use deployment slots to test a new release. After swapping the staging slot to production, users report issues. What is the **fastest** way to roll back?

- A) Redeploy the previous version from source control
- B) Swap the slots again (production ↔ staging)
- C) Delete the production slot and recreate it

---

**3.** Your App Service needs to call an Azure SQL database using its managed identity. Which authentication token audience should you request?

- A) `https://management.azure.com/`
- B) `https://database.windows.net/`
- C) `https://vault.azure.net/`

---

**4.** You configure autoscale on an App Service plan with a rule: "When average CPU > 70% for 10 minutes, increase instance count by 1." What also must you define to prevent runaway scaling?

- A) A cool-down period and a maximum instance limit
- B) A minimum App Service plan tier
- C) A deployment slot for overflow traffic

---

**5.** An App Service deployment using ZIP Deploy succeeds, but the app shows the default landing page. What is the most likely cause?

- A) The ZIP file does not contain the app at its root; files are nested in a subfolder
- B) The App Service plan is on the Free tier
- C) The custom domain is not configured

---

**6.** You need your App Service to run a background task (WebJob) that processes queue messages continuously. Which WebJob type should you use?

- A) Triggered WebJob
- B) Continuous WebJob
- C) Scheduled WebJob (CRON)

---

**7.** Your team uses GitHub Actions for CI/CD to App Service. Which authentication method is recommended by Microsoft for the deployment action?

- A) Publish profile downloaded from the portal
- B) OpenID Connect (OIDC) federated credential with a service principal
- C) Storage account SAS token

---

**8.** You want to route 10% of production traffic to a staging slot to test a new feature. Which App Service feature enables this?

- A) Traffic Manager
- B) Testing in production (slot traffic routing)
- C) Azure Front Door

---

**9.** An app deployed to App Service for Linux returns a 502 error. Logs show the container failed to start within the expected time. What setting should you increase?

- A) `WEBSITES_CONTAINER_START_TIME_LIMIT`
- B) `WEBSITE_TIME_ZONE`
- C) `WEBSITES_PORT`

---

**10.** You want your App Service to accept traffic **only** over HTTPS. Where do you configure this?

- A) Custom domains → Add binding
- B) Configuration → General settings → HTTPS Only = On
- C) Networking → Access Restrictions

---

### Implement Azure Functions (11–18)

**11.** You write an Azure Function triggered by an Azure Storage Queue. The message processing fails repeatedly. After 5 dequeue attempts the message is moved to a special queue. What is that queue called?

- A) Dead-letter queue
- B) Poison queue
- C) Retry queue

---

**12.** Your timer-triggered function must run every weekday at 8:00 AM UTC. Which CRON expression is correct?

- A) `0 0 8 * * 1-5`
- B) `0 8 * * 1-5`
- C) `8 0 * * * *`

---

**13.** You are developing a Durable Function orchestration that calls three activity functions in sequence. Which Durable Functions pattern is this?

- A) Fan-out / fan-in
- B) Function chaining
- C) Monitor

---

**14.** A Durable Function orchestrator must call an external API and wait for a callback (webhook). Which pattern should you use?

- A) Function chaining
- B) Human interaction / external events
- C) Eternal orchestration

---

**15.** You need an Azure Function to run inside a virtual network and access private resources. The Consumption plan does not support VNet integration. Which plan should you choose?

- A) Dedicated (App Service) plan with VNet integration
- B) Premium plan with VNet integration
- C) Both A and B support VNet integration

---

**16.** In a .NET isolated process Azure Function, where do you configure the function's trigger binding?

- A) In `function.json` only
- B) Using attributes on the method parameters in C# code
- C) In `host.json` under the `bindings` section

---

**17.** Your Azure Function writes output to both a Storage Queue and a Cosmos DB container from a single function execution. How is this achieved?

- A) Multiple output bindings on the function
- B) Two separate triggers in the same function
- C) You must split it into two separate functions

---

**18.** You enable Application Insights for your Function App. Which `host.json` section controls the sampling rate for telemetry?

- A) `logging.applicationInsights.samplingSettings`
- B) `extensions.http.routePrefix`
- C) `functionTimeout`

---

### Develop Solutions That Use Blob Storage (19–24)

**19.** You need to generate a URL that gives a third-party read access to a specific blob for 24 hours without sharing your storage account key. What should you create?

- A) A Shared Access Signature (SAS) with read permission and 24-hour expiry
- B) A public container access level
- C) An Azure AD guest invitation

---

**20.** Using the .NET SDK, which class do you use to interact with a **specific container** in Blob Storage?

- A) `BlobServiceClient`
- B) `BlobContainerClient`
- C) `BlobClient`

---

**21.** You upload a 500 GB file to Block Blob storage. The SDK splits it into blocks. What is the **maximum size** of a single block blob (approximately)?

- A) 4.75 TB
- B) 256 MB
- C) 8 TB

---

**22.** You set the **default access tier** of a storage account to Cool. You upload a blob without specifying a tier. What tier is the blob placed in?

- A) Hot (always the default for individual blobs)
- B) Cool (inherits the account default)
- C) Archive

---

**23.** Your application uses `BlobServiceClient` authenticated with `DefaultAzureCredential`. In production on App Service, which identity does it use?

- A) The storage account access key
- B) The App Service's managed identity
- C) The Azure CLI credential of the developer who deployed

---

**24.** You need to receive a notification every time a blob is created in a specific container. Which Azure service should you use?

- A) Azure Event Grid with a Blob Storage event subscription
- B) Azure Monitor metric alert
- C) Blob lifecycle management policy

---

### Develop Solutions That Use Azure Cosmos DB (25–30)

**25.** You create a Cosmos DB container with partition key `/customerId`. A query uses `WHERE customerId = 'C001'`. Is this a cross-partition query?

- A) Yes, it always scans all partitions
- B) No, it targets a single partition because the filter matches the partition key
- C) It depends on the consistency level

---

**26.** You perform a point read in Cosmos DB: read a single 1 KB document by its `id` and partition key. Approximately how many RUs does this cost?

- A) 1 RU
- B) 5 RU
- C) 10 RU

---

**27.** Your application uses the Cosmos DB .NET SDK. You want to create a document if it does not exist or replace it if it does. Which method should you use?

- A) `container.CreateItemAsync()`
- B) `container.UpsertItemAsync()`
- C) `container.ReadItemAsync()`

---

**28.** You configure Cosmos DB with **Strong** consistency in a multi-region setup. What is the trade-off?

- A) Higher write latency because writes must be confirmed in all regions
- B) No impact; Strong consistency is free in multi-region
- C) Data is only written to the primary region and never replicated

---

**29.** You want to react to every insert and update in a Cosmos DB container in near real time. Which feature should you use?

- A) Cosmos DB Change Feed
- B) Cosmos DB Stored Procedures
- C) Azure Event Grid system topic for Cosmos DB

---

**30.** Your Cosmos DB account uses Serverless mode. You try to enable **geo-replication** to a second region. What happens?

- A) It succeeds; Serverless supports multi-region
- B) It fails; Serverless mode does not support multi-region writes or replication
- C) It succeeds but only for reads, not writes

---

### Implement Containerized Solutions (31–36)

**31.** You want to build a container image in the cloud without installing Docker locally. Which Azure service can build an image from a Dockerfile directly?

- A) Azure Container Instances
- B) Azure Container Registry Tasks (`az acr build`)
- C) Azure Kubernetes Service

---

**32.** Which RBAC role should you assign to a CI/CD pipeline identity that needs to **push** images to ACR but not manage the registry?

- A) AcrPull
- B) AcrPush
- C) Owner

---

**33.** You deploy a container to ACI and need the data to persist across container restarts. What should you mount?

- A) A local Docker volume
- B) An Azure File Share as a volume
- C) A temporary RAM disk

---

**34.** In a multi-stage Dockerfile, what is the primary benefit of using a separate build stage and a final runtime stage?

- A) The final image is smaller because build tools and intermediate files are not included
- B) The build runs faster because stages run in parallel
- C) It allows running multiple containers from one Dockerfile

---

**35.** You create an ACI container group with 2 containers: one requesting 1 vCPU / 1 GB RAM and another requesting 0.5 vCPU / 0.5 GB RAM. What are the group's total allocated resources?

- A) 1.5 vCPU and 1.5 GB RAM (sum of all containers)
- B) 1 vCPU and 1 GB RAM (maximum of any single container)
- C) 2 vCPU and 2 GB RAM (rounded up)

---

**36.** You want to pull an image from ACR into an Azure App Service for Containers. The ACR has **admin account disabled**. What is the recommended way to authenticate?

- A) Enable the ACR admin account and use the username/password
- B) Use a managed identity on the App Service with AcrPull role
- C) Make the ACR repository public

---

### Implement User Authentication and Authorization (37–42)

**37.** Your single-page application (SPA) running in the browser needs to sign in users with Microsoft Entra ID. Which OAuth 2.0 flow is recommended?

- A) Client credentials flow
- B) Authorization code flow with PKCE
- C) Resource owner password credentials (ROPC) flow

---

**38.** You register a multi-tenant application. A user from a different tenant signs in for the first time. What must happen before the app can access their data?

- A) The user (or tenant admin) must grant **consent** to the requested permissions
- B) The user must be added as a guest in your home tenant
- C) You must create a separate app registration in their tenant

---

**39.** A Conditional Access policy requires MFA for all users accessing your web app. Your app uses MSAL for authentication. What additional code change is needed in your app?

- A) No code change; Entra ID handles MFA during the sign-in flow, and MSAL receives tokens only after MFA succeeds
- B) You must implement a custom MFA challenge in your app
- C) You must disable token caching in MSAL

---

**40.** Your web API validates incoming JWT access tokens. Which claim should you check to ensure the token was issued for **your** API and not a different resource?

- A) `aud` (audience)
- B) `iss` (issuer)
- C) `sub` (subject)

---

**41.** You call Microsoft Graph to read the signed-in user's profile. Which minimal scope should you request?

- A) `User.ReadWrite.All`
- B) `User.Read`
- C) `Directory.Read.All`

---

**42.** Your app uses `ConfidentialClientApplicationBuilder` with a client certificate instead of a client secret. Why is a certificate preferred for production?

- A) Certificates are easier to create than secrets
- B) Certificates cannot be accidentally leaked in logs or config files as easily as secrets, and support hardware storage (HSM)
- C) Certificates do not expire and never need rotation

---

### Implement Secure Cloud Solutions (43–46)

**43.** You store a database connection string in Azure Key Vault. Your App Service references it using a **Key Vault reference** in app settings (e.g. `@Microsoft.KeyVault(...)`). What happens when the secret is updated in Key Vault?

- A) The App Service automatically picks up the new value (within the cache refresh interval) without redeployment
- B) You must redeploy the App Service for the new value to take effect
- C) The App Service crashes because the reference becomes invalid

---

**44.** You need to grant a Function App access to Key Vault secrets. You enable a system-assigned managed identity on the Function App. What is the next step?

- A) Store the managed identity's password in the Function App settings
- B) Assign the **Key Vault Secrets User** role (or add an access policy) for the managed identity on the Key Vault
- C) Create a client secret for the managed identity in Entra ID

---

**45.** Your company policy requires that encryption keys for storage are managed in your own Key Vault. What is this called?

- A) Microsoft-managed keys (MMK)
- B) Customer-managed keys (CMK)
- C) Platform-managed keys (PMK)

---

**46.** You accidentally deleted a Key Vault itself (not just a secret). The vault had **purge protection** enabled with a 90-day retention. Can you recover it?

- A) No; deleting a vault is permanent regardless of settings
- B) Yes; the vault is soft-deleted and can be recovered within the retention period
- C) Only if you contact Microsoft Support within 24 hours

---

### Implement API Management (47–48)

**47.** You want to limit a consumer to 100 calls per minute in APIM. Which policy should you apply in the **inbound** section?

- A) `<cache-lookup>`
- B) `<rate-limit calls="100" renewal-period="60" />`
- C) `<set-backend-service>`

---

**48.** Your APIM instance receives a request. The policy execution order across scopes is applied in which order?

- A) Operation → API → Product → Global
- B) Global → Product → API → Operation
- C) API → Global → Operation → Product

---

### Develop Event-Based Solutions (49–50)

**49.** You create an Event Grid subscription for blob created events. Delivery to your webhook fails repeatedly. After all retry attempts, where can the undelivered events be stored?

- A) In an Azure Storage **dead-letter** container configured on the subscription
- B) They are permanently lost after retry exhaustion
- C) They are sent back to the publisher (Storage Account)

---

**50.** You are using Azure Event Hubs with the **Event Processor Client** in .NET. Where does the processor store its **checkpoint** (offset per partition)?

- A) In the Event Hub namespace itself
- B) In an Azure Blob Storage container
- C) In an Azure SQL Database

---

---

## Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **A** | App Service **Connection strings** (and App Settings) are stored securely in Azure and exposed as environment variables at runtime, separate from source control. |
| 2 | **B** | Swapping slots is instant and reversible. Swapping production back to staging restores the previous version immediately. |
| 3 | **B** | `https://database.windows.net/` is the token audience for Azure SQL. The app requests this audience when acquiring a token via managed identity. |
| 4 | **A** | Autoscale rules need a **cool-down period** (to avoid flapping) and a **maximum instance limit** to cap costs and prevent runaway scaling. |
| 5 | **A** | ZIP Deploy expects the app content at the root of the archive. A nested subfolder means the runtime finds no startup file and shows the default page. |
| 6 | **B** | **Continuous WebJobs** run in a loop and are ideal for always-on queue processing. Triggered WebJobs run on a schedule or manual trigger. |
| 7 | **B** | Microsoft recommends **OIDC federated credentials** (workload identity federation) for GitHub Actions, eliminating the need to store secrets. |
| 8 | **B** | App Service **Testing in production** lets you route a percentage of live traffic to a different slot without DNS changes. |
| 9 | **A** | `WEBSITES_CONTAINER_START_TIME_LIMIT` (default 230 seconds) controls how long the platform waits for the container to start before returning 502. |
| 10 | **B** | **HTTPS Only** in General settings redirects all HTTP traffic to HTTPS, enforcing secure connections. |
| 11 | **B** | After 5 failed dequeue attempts, the Azure Functions runtime moves the message to the **poison queue** (named `<original-queue>-poison`). |
| 12 | **A** | Azure Functions uses 6-field CRON: `{second} {minute} {hour} {day} {month} {day-of-week}`. `0 0 8 * * 1-5` = every weekday at 08:00:00 UTC. |
| 13 | **B** | **Function chaining** calls activity functions one after another, passing output of one as input to the next. |
| 14 | **B** | The **human interaction / external events** pattern uses `WaitForExternalEvent` so the orchestration pauses until a callback or event arrives. |
| 15 | **C** | Both the **Premium** plan and the **Dedicated (App Service)** plan support VNet integration. Premium also offers event-driven scaling. |
| 16 | **B** | In .NET (both in-process and isolated), trigger and binding configuration is done via **C# attributes** on method parameters. |
| 17 | **A** | A single function can have **multiple output bindings** (e.g. one for Queue, one for Cosmos DB) declared as attributes or in `function.json`. |
| 18 | **A** | `logging.applicationInsights.samplingSettings` in `host.json` controls adaptive sampling for Application Insights telemetry. |
| 19 | **A** | A **SAS (Shared Access Signature)** provides time-limited, scoped access to a blob without exposing the account key. |
| 20 | **B** | `BlobContainerClient` provides operations on a specific container (list blobs, upload, delete). `BlobServiceClient` is account-level; `BlobClient` is blob-level. |
| 21 | **A** | A single block blob can be up to approximately **4.75 TB** (with blocks up to 4000 MiB each and up to 50,000 blocks). |
| 22 | **B** | If no tier is specified on upload, the blob **inherits the account default** tier, which is Cool in this case. |
| 23 | **B** | On App Service, `DefaultAzureCredential` resolves to the **managed identity** of the App Service in production. |
| 24 | **A** | **Azure Event Grid** has a system topic for Blob Storage that emits `BlobCreated` events, which you can subscribe to and route to handlers. |
| 25 | **B** | When the `WHERE` clause filters on the partition key value, Cosmos DB routes the query to a **single partition**, making it efficient. |
| 26 | **A** | A point read of a 1 KB item by `id` and partition key costs approximately **1 RU**. |
| 27 | **B** | `UpsertItemAsync()` creates the document if it doesn't exist or replaces it if it does, in a single operation. |
| 28 | **A** | **Strong** consistency in multi-region requires synchronous replication, resulting in **higher write latency** and more RU cost. |
| 29 | **A** | The **Change Feed** provides an ordered stream of changes (inserts and updates) for a container, enabling near-real-time processing. |
| 30 | **B** | **Serverless** mode does not support multi-region replication. You must use provisioned or autoscale throughput for geo-replication. |
| 31 | **B** | **ACR Tasks** (`az acr build`) sends the Dockerfile and context to ACR, which builds and stores the image in the cloud—no local Docker needed. |
| 32 | **B** | **AcrPush** grants permission to push and pull images. AcrPull is read-only; Owner is too broad. |
| 33 | **B** | Mounting an **Azure File Share** provides persistent storage that survives container restarts and can be shared across containers. |
| 34 | **A** | Multi-stage builds copy only the final artifacts into the runtime image, excluding compilers and intermediate files, producing a **smaller image**. |
| 35 | **A** | ACI allocates resources as the **sum** of all container requests in the group: 1.5 vCPU and 1.5 GB RAM total. |
| 36 | **B** | Using a **managed identity with AcrPull** is the recommended, secretless way to authenticate App Service to ACR. |
| 37 | **B** | **Authorization code flow with PKCE** is recommended for SPAs and public clients; it avoids exposing tokens in the URL (unlike implicit). |
| 38 | **A** | Multi-tenant apps require **consent** (user or admin) in the external tenant before the app can access that tenant's data. |
| 39 | **A** | MFA is handled by the identity provider during the sign-in redirect. MSAL receives tokens only after all Conditional Access requirements are satisfied; **no app code change** is needed. |
| 40 | **A** | The `aud` (audience) claim identifies the intended recipient of the token. Your API should validate that `aud` matches its own application ID. |
| 41 | **B** | `User.Read` is the minimal scope to read the signed-in user's own profile from Microsoft Graph. |
| 42 | **B** | Certificates are harder to leak than plain-text secrets and can be stored in HSMs; they provide stronger credential security for production. |
| 43 | **A** | Key Vault references in App Service are periodically refreshed. When the secret value changes, the App Service picks up the new value **without redeployment**. |
| 44 | **B** | After enabling the managed identity, you must **grant it access** on the Key Vault (via RBAC role or access policy) so it can read secrets. |
| 45 | **B** | **Customer-managed keys (CMK)** means you control the encryption key lifecycle in your own Key Vault. |
| 46 | **B** | Key Vaults themselves support **soft-delete** (mandatory) and **purge protection**. A deleted vault can be recovered within the retention period. |
| 47 | **B** | The `<rate-limit>` policy restricts call rate. `calls="100"` with `renewal-period="60"` (seconds) limits to 100 calls/minute. |
| 48 | **B** | Policy scopes execute from broadest to narrowest: **Global → Product → API → Operation**. `<base/>` controls where parent policies are inserted. |
| 49 | **A** | Event Grid supports a **dead-letter destination** (a Storage container) where undelivered events are stored after all retries fail. |
| 50 | **B** | The Event Processor Client stores checkpoints (partition offsets) in **Azure Blob Storage**, enabling distributed and resumable processing. |
