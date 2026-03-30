
# AZ-204 Practice Questions – Set 3

50 questions covering commonly tested AZ-204 topics **not fully covered** in Sets 1–3: Azure Cache for Redis, Azure Service Bus, Durable Functions, SAS tokens, Azure App Configuration, managed identities, Cosmos DB server-side programming, and Azure CDN.

---

## Questions

### Azure Cache for Redis (1–10)

**1.** Your web app makes repeated calls to a slow SQL query that returns the same product catalog data. You want to reduce latency. Which Azure service should you use to cache the query results?

- A) Azure Blob Storage with Hot tier
- B) Azure Cache for Redis
- C) Azure Cosmos DB with Session consistency

---

**2.** You store a session token in Azure Cache for Redis and need it to expire automatically after 30 minutes of inactivity. Which Redis feature should you use?

- A) A `PERSIST` command to make the key permanent
- B) A TTL (time-to-live) / expiration set on the key (e.g. `EXPIRE key 1800`)
- C) A Redis Stream with retention

---

**3.** Your Redis cache is full and a new key must be written. You want Redis to evict the **least recently used** key to make room. Which eviction policy should you configure?

- A) `noeviction`
- B) `allkeys-lru`
- C) `volatile-random`

---

**4.** You need to cache entire .NET objects (e.g. a `Product` class) in Redis. How should you store them?

- A) Serialize the object to JSON (or binary) and store as a Redis string value
- B) Store each property as a separate Redis database
- C) Store the object's memory address as a Redis key

---

**5.** Which Azure Cache for Redis tier provides **geo-replication**, clustering, and zone redundancy for production workloads?

- A) Basic
- B) Standard
- C) Premium (or Enterprise)

---

**6.** Your application uses the **cache-aside** pattern. What is the correct sequence when reading data?

- A) Read from cache → if miss, read from database → write result to cache → return data
- B) Write to cache first → then write to database → return data
- C) Read from database only → never use cache for reads

---

**7.** You update a product's price in the database. The old price is still served from Redis cache. What should you do?

- A) Wait for the key to expire naturally; no action needed
- B) Invalidate (delete) the cache key when the database is updated, so the next read fetches fresh data
- C) Restart the Redis instance to clear all keys

---

**8.** In .NET, which NuGet package is commonly used to connect to Azure Cache for Redis?

- A) `Microsoft.Extensions.Caching.Cosmos`
- B) `StackExchange.Redis`
- C) `Azure.Storage.Blobs`

---

**9.** You configure Redis with `maxmemory-policy = volatile-lru`. Which keys are eligible for eviction?

- A) All keys in the cache
- B) Only keys that have an **expiration (TTL)** set
- C) Only keys that were written in the last hour

---

**10.** Your Redis cache connection string contains the hostname, port, password, and `ssl=True`. Why is `ssl=True` important?

- A) It enables Redis clustering
- B) It encrypts the connection between the client and Azure Cache for Redis (TLS)
- C) It enables Redis persistence to disk

---

### Azure Service Bus (11–20)

**11.** You need a message broker where multiple subscribers each receive their **own copy** of every message. Which Service Bus entity should you use?

- A) Service Bus **Queue**
- B) Service Bus **Topic** with multiple **Subscriptions**
- C) Storage Queue

---

**12.** A consumer receives a message from a Service Bus Queue using **peek-lock** mode. The consumer crashes before completing processing. What happens to the message?

- A) The message is permanently lost
- B) The message lock expires and the message becomes available for another consumer to receive
- C) The message is moved immediately to the dead-letter queue

---

**13.** A message fails processing repeatedly and exceeds the **max delivery count** on a Service Bus Queue. Where does the message go?

- A) It is deleted permanently
- B) It is moved to the **dead-letter queue** (sub-queue of the main queue)
- C) It is sent back to the producer

---

**14.** You need guaranteed **FIFO (first-in, first-out)** ordering of messages in Service Bus. Which feature should you enable?

- A) Message deferral
- B) **Sessions** (session-enabled queue or subscription)
- C) Duplicate detection

---

**15.** What is the difference between **receive-and-delete** and **peek-lock** receive modes in Service Bus?

- A) Receive-and-delete is slower but safer; peek-lock is faster
- B) **Receive-and-delete** removes the message immediately on receive (at-most-once); **peek-lock** holds the message invisibly until the consumer completes or abandons it (at-least-once)
- C) They are identical in behavior

---

**16.** Your Service Bus topic has 3 subscriptions. Subscription A needs only messages with `priority = "high"`. How do you configure this?

- A) Create a **SQL filter** on subscription A: `priority = 'high'`
- B) Create separate topics for each priority level
- C) Use a timer to poll and discard low-priority messages

---

**17.** You need to process related messages as a group (e.g. all items in one order) and ensure they are processed by the same consumer in order. Which Service Bus feature supports this?

- A) Partitioned queues
- B) **Message sessions** with a shared `SessionId`
- C) Auto-forwarding

---

**18.** Which Service Bus tier supports **topics and subscriptions**, message sessions, and duplicate detection?

- A) Basic
- B) Standard
- C) Both Standard and Premium

---

**19.** You want to schedule a Service Bus message to become visible at a specific future time. Which property do you set?

- A) `TimeToLive`
- B) `ScheduledEnqueueTimeUtc`
- C) `SessionId`

---

**20.** Your team debates using Azure Storage Queues vs Azure Service Bus Queues. The scenario requires dead-letter support, sessions, and transactions. Which should you choose?

- A) Azure Storage Queues (simpler and cheaper)
- B) Azure Service Bus Queues (supports dead-letter, sessions, and transactions)
- C) Azure Event Hubs

---

### Durable Functions (21–28)

**21.** In Durable Functions, what is the role of the **orchestrator function**?

- A) It performs the actual work (e.g. API calls, database writes)
- B) It coordinates and sequences calls to **activity functions**, managing state and retries
- C) It receives the external HTTP request from the client

---

**22.** You need to call 5 activity functions in parallel and wait for all to complete before continuing. Which Durable Functions pattern is this?

- A) Function chaining
- B) Fan-out / fan-in
- C) Monitor

---

**23.** An orchestrator needs to pause and wait for a human to approve a request (e.g. via email link). Which Durable Functions pattern should you use?

- A) Eternal orchestration
- B) Human interaction (wait for external event)
- C) Function chaining

---

**24.** The **monitor** pattern in Durable Functions is used for which scenario?

- A) Calling activity functions in sequence
- B) Polling an external resource at intervals until a condition is met (e.g. job status becomes "complete")
- C) Running a function on a CRON schedule

---

**25.** An orchestrator function must be **deterministic**. Which of the following is NOT allowed inside an orchestrator?

- A) Calling `context.CallActivityAsync()`
- B) Calling `DateTime.UtcNow` directly (use `context.CurrentUtcDateTime` instead)
- C) Awaiting `context.WaitForExternalEvent()`

---

**26.** In Durable Functions, where is the orchestration state (history) persisted by default?

- A) In-memory on the function host only
- B) In Azure Storage (tables and queues in the task hub)
- C) In Azure Cosmos DB

---

**27.** You want an orchestrator that runs continuously — processing work, sleeping for a period, and repeating forever (e.g. a cleanup loop). Which pattern is this?

- A) Fan-out / fan-in
- B) Eternal orchestration (the orchestrator calls `ContinueAsNew` to loop)
- C) Sub-orchestration

---

**28.** How do you start (trigger) a Durable Functions orchestration from an HTTP request?

- A) The orchestrator function itself has an HTTP trigger
- B) A separate **client (starter) function** with an HTTP trigger calls `starter.StartNewAsync()` to begin the orchestration
- C) You deploy the orchestrator and it starts automatically

---

### SAS Tokens and Stored Access Policies (29–34)

**29.** You need to grant a third-party application read-only access to a single blob for 1 hour without sharing your account key. What should you generate?

- A) A **service SAS** scoped to the blob with read permission and 1-hour expiry
- B) An **account SAS** with full account access
- C) A new storage account access key for the third party

---

**30.** What is a **user delegation SAS** and why is it preferred over a service SAS?

- A) A SAS signed with the storage account key; preferred because it's simpler
- B) A SAS signed with **Azure AD credentials** (via a user delegation key); preferred because it does not require storing or sharing the account key
- C) A SAS that never expires; preferred for long-lived access

---

**31.** You create a **stored access policy** on a Blob container and associate a SAS token with it. What is the primary benefit?

- A) The SAS token becomes irrevocable
- B) You can **revoke or modify** the SAS by changing or deleting the stored access policy, without regenerating the account key
- C) The SAS token can access all containers in the account

---

**32.** An **account SAS** differs from a **service SAS** in which way?

- A) Account SAS can grant access to **multiple services** (Blob, Queue, Table, File) and service-level operations; service SAS is scoped to one service
- B) Account SAS is scoped to a single blob; service SAS covers the entire account
- C) There is no difference; they are interchangeable

---

**33.** You suspect a SAS token has been compromised. The SAS was created as an ad-hoc service SAS (no stored access policy). How can you immediately revoke it?

- A) Delete the stored access policy
- B) **Regenerate the storage account key** used to sign the SAS (this invalidates all SAS tokens signed with that key)
- C) Set the SAS expiry to a past time retroactively

---

**34.** When constructing a SAS token, which parameter defines what operations are allowed (e.g. read, write, delete, list)?

- A) `sig` (signature)
- B) `sp` (signed permissions)
- C) `se` (signed expiry)

---

### Azure App Configuration (35–38)

**35.** Your application needs **feature flags** (e.g. enable/disable a "Beta UI" for a percentage of users) with centralized management. Which Azure service provides built-in feature flag support?

- A) Azure Key Vault
- B) Azure App Configuration
- C) Azure Cosmos DB

---

**36.** You use Azure App Configuration with a **sentinel key** (e.g. `Settings:Sentinel`). What is the purpose of the sentinel key?

- A) It stores the most critical secret for the application
- B) It acts as a **signal** — the app monitors only this key, and when its value changes, the app knows to refresh all configuration values
- C) It encrypts all other configuration keys

---

**37.** How does Azure App Configuration differ from Azure Key Vault in terms of what you should store?

- A) App Configuration stores **non-secret configuration** (feature flags, app settings, endpoint URLs); Key Vault stores **secrets, keys, and certificates**
- B) They are identical; use either one for all configuration and secrets
- C) App Configuration replaces Key Vault entirely

---

**38.** Your .NET app uses `Microsoft.Extensions.Configuration.AzureAppConfiguration`. You want configuration to refresh automatically when values change. Which method do you call in the configuration builder?

- A) `.ConfigureRefresh(refresh => refresh.Register("Settings:Sentinel", refreshAll: true))`
- B) `.AddJsonFile("appsettings.json")`
- C) `.AddEnvironmentVariables()`

---

### Managed Identities (39–42)

**39.** What is the key difference between a **system-assigned** and **user-assigned** managed identity?

- A) System-assigned is tied to a single Azure resource and deleted when the resource is deleted; user-assigned is a standalone resource that can be shared across multiple resources
- B) System-assigned can access any Azure resource without RBAC; user-assigned requires RBAC
- C) User-assigned identities are less secure than system-assigned

---

**40.** You have 5 Azure Functions that all need the same permissions on a Key Vault. You want to manage access as a single identity. Which identity type should you use?

- A) A separate system-assigned managed identity on each Function App (5 identities, 5 role assignments)
- B) A single **user-assigned managed identity** assigned to all 5 Function Apps (1 identity, 1 role assignment)
- C) A service principal with a shared client secret

---

**41.** In code, how does your application acquire an access token using its managed identity to call Azure Blob Storage?

- A) Use `DefaultAzureCredential` (or `ManagedIdentityCredential`) to get a token for the audience `https://storage.azure.com/`; pass the credential to `BlobServiceClient`
- B) Read the storage account key from an environment variable and use it directly
- C) Call the Azure Portal REST API to retrieve the key at runtime

---

**42.** You enable a system-assigned managed identity on an App Service. Where do you grant this identity permission to read Blob Storage data?

- A) On the App Service's Configuration blade
- B) On the **Storage account's IAM (Access Control)** blade — assign the **Storage Blob Data Reader** role to the App Service's managed identity
- C) In the App Service's Managed Identity blade by typing the storage account name

---

### Cosmos DB Server-Side Programming (43–46)

**43.** You need to execute multiple Cosmos DB operations (read + write) as an **atomic transaction** (all succeed or all fail). Which Cosmos DB feature provides ACID transactions within a single partition?

- A) Change Feed processor
- B) **Stored procedures** (JavaScript, executed server-side within a partition)
- C) Input bindings in Azure Functions

---

**44.** What is a key constraint of Cosmos DB stored procedures regarding partitioning?

- A) Stored procedures can run across all partitions simultaneously
- B) Stored procedures execute against a **single logical partition**; you must specify the partition key value when calling them
- C) Stored procedures are not supported in the NoSQL API

---

**45.** You want logic to run automatically **before** an item is written to Cosmos DB (e.g. to validate or enrich the document). Which Cosmos DB feature should you use?

- A) A **pre-trigger** (server-side JavaScript that runs before the operation)
- B) A post-trigger
- C) An Azure Function with an HTTP trigger

---

**46.** Cosmos DB **user-defined functions (UDFs)** are used for which purpose?

- A) To replace stored procedures for write operations
- B) To define **custom logic** that can be called within a SQL query (e.g. a formatting or calculation function)
- C) To trigger external webhooks when a document changes

---

### Azure CDN and Caching (47–50)

**47.** Your Blob Storage–hosted static website is accessed globally. Users in Asia experience high latency. Which service should you put in front of the storage account?

- A) Azure API Management
- B) Azure CDN (Content Delivery Network)
- C) Azure Traffic Manager

---

**48.** You update an image in Blob Storage, but the CDN edge servers still serve the old version. What should you do to force the edge to fetch the updated file?

- A) **Purge** the CDN endpoint (or the specific asset path)
- B) Delete and recreate the CDN profile
- C) Change the Blob Storage redundancy from LRS to GRS

---

**49.** You want the CDN to cache responses for 1 hour regardless of what the origin's `Cache-Control` header says. How should you configure this?

- A) Set a **CDN caching rule** that overrides the origin header with a custom TTL of 3600 seconds
- B) Change the Blob Storage access tier to Hot
- C) Enable HTTPS on the CDN endpoint

---

**50.** Which caching behavior option tells the CDN to **always respect** the `Cache-Control` and `Expires` headers sent by the origin, without overriding them?

- A) Override
- B) **Honor origin** (pass-through / respect origin cache directives)
- C) Bypass cache

---

---

## Answer Key

| # | Answer | Explanation |
|---|--------|-------------|
| **Azure Cache for Redis** | | |
| 1 | **B** | **Azure Cache for Redis** is a managed in-memory cache for low-latency reads. It's the standard choice for caching database query results. |
| 2 | **B** | Setting a **TTL (EXPIRE)** on the key causes Redis to automatically delete it after the specified time (1800 seconds = 30 minutes). |
| 3 | **B** | `allkeys-lru` evicts the **least recently used** key from all keys when memory is full. `noeviction` returns errors instead. |
| 4 | **A** | Objects must be **serialized** (JSON, MessagePack, or binary) to store in Redis. Redis stores string values; it cannot store in-memory .NET objects directly. |
| 5 | **C** | **Premium** (and Enterprise) tiers support geo-replication, clustering, persistence, zone redundancy, and VNet integration. |
| 6 | **A** | **Cache-aside**: check cache first → on miss, read from DB → populate cache → return. This is the most common caching pattern. |
| 7 | **B** | **Invalidate the cache key** when the underlying data changes. The next read will miss, fetch fresh data from DB, and repopulate the cache. |
| 8 | **B** | **StackExchange.Redis** is the standard .NET client for Redis. `Microsoft.Extensions.Caching.StackExchangeRedis` wraps it for `IDistributedCache`. |
| 9 | **B** | `volatile-lru` evicts least recently used keys **only among keys that have a TTL set**. Keys without expiration are never evicted. |
| 10 | **B** | `ssl=True` enables **TLS encryption** on the connection. Azure Cache for Redis requires TLS by default to protect data in transit. |
| **Azure Service Bus** | | |
| 11 | **B** | A **Topic + Subscriptions** pattern delivers a copy of each message to every subscription (pub/sub). A Queue delivers each message to one consumer only. |
| 12 | **B** | In **peek-lock**, the message is locked but not deleted. If the lock expires (consumer crashed), the message becomes visible again for redelivery. |
| 13 | **B** | After exceeding **max delivery count**, Service Bus moves the message to the **dead-letter queue** where it can be inspected and reprocessed. |
| 14 | **B** | **Sessions** guarantee FIFO ordering per session ID. Messages with the same `SessionId` are delivered in order to a single consumer. |
| 15 | **B** | **Receive-and-delete** removes immediately (at-most-once, no retry). **Peek-lock** holds the message; the consumer must explicitly complete or abandon it (at-least-once). |
| 16 | **A** | A **SQL filter** on the subscription evaluates message properties. `priority = 'high'` ensures only matching messages are delivered to subscription A. |
| 17 | **B** | **Message sessions** group related messages by `SessionId`. The session is locked to one consumer, ensuring ordered and exclusive processing of the group. |
| 18 | **C** | **Both Standard and Premium** support topics, subscriptions, sessions, and duplicate detection. Basic tier supports queues only (no topics). |
| 19 | **B** | `ScheduledEnqueueTimeUtc` schedules a message to become visible at a future time. `TimeToLive` controls when the message expires and is discarded. |
| 20 | **B** | **Service Bus** supports dead-letter queues, sessions, and transactions. Storage Queues are simpler but lack these advanced features. |
| **Durable Functions** | | |
| 21 | **B** | The **orchestrator** coordinates the workflow: it calls activity functions, manages state, handles retries, and defines the execution order. |
| 22 | **B** | **Fan-out / fan-in**: launch multiple activities in parallel (`Task.WhenAll`) and aggregate results when all complete. |
| 23 | **B** | **Human interaction**: the orchestrator calls `WaitForExternalEvent` to pause until an external signal (e.g. approval webhook) is received. |
| 24 | **B** | The **monitor** pattern polls an external condition at intervals using `CreateTimer`, checking until the condition is met or a timeout occurs. |
| 25 | **B** | Orchestrators must be **deterministic**. `DateTime.UtcNow` is non-deterministic (returns different values on replay). Use `context.CurrentUtcDateTime` instead. |
| 26 | **B** | Durable Functions stores orchestration state in **Azure Storage** (tables for history, queues for messages) in the configured task hub. |
| 27 | **B** | **Eternal orchestration**: the orchestrator calls `ContinueAsNew()` at the end to restart itself, creating an infinite loop without unbounded history growth. |
| 28 | **B** | A **client (starter) function** (e.g. HTTP-triggered) calls `StartNewAsync()` on the orchestration client to begin the orchestration. The orchestrator itself has no trigger annotation. |
| **SAS Tokens** | | |
| 29 | **A** | A **service SAS** scoped to a single blob with read-only permission and a short expiry is the least-privilege option for this scenario. |
| 30 | **B** | A **user delegation SAS** is signed with Azure AD credentials (not the account key). It's preferred because it eliminates the risk of account key exposure. |
| 31 | **B** | A **stored access policy** lets you revoke or modify SAS tokens associated with it without regenerating the account key. It provides server-side control over SAS lifetime. |
| 32 | **A** | **Account SAS** can grant access across multiple storage services and service-level operations; **service SAS** is scoped to a single service (e.g. Blob only). |
| 33 | **B** | Without a stored access policy, the only way to revoke an ad-hoc SAS is to **regenerate the account key** that signed it (this revokes all SAS tokens signed with that key). |
| 34 | **B** | `sp` (signed permissions) defines the allowed operations (e.g. `r` = read, `w` = write, `d` = delete, `l` = list). |
| **Azure App Configuration** | | |
| 35 | **B** | **Azure App Configuration** has built-in feature flag management with targeting, percentage rollout, and on/off toggling. |
| 36 | **B** | A **sentinel key** is a lightweight signal. The app watches only this key; when it changes, the app triggers a full configuration refresh, reducing polling overhead. |
| 37 | **A** | **App Configuration** is for non-secret settings (URLs, feature flags, connection strings without passwords). **Key Vault** is for secrets, keys, and certificates. They complement each other. |
| 38 | **A** | `.ConfigureRefresh()` with `.Register("Settings:Sentinel", refreshAll: true)` tells the provider to watch the sentinel key and reload all values when it changes. |
| **Managed Identities** | | |
| 39 | **A** | **System-assigned** is 1:1 with the resource (created and deleted with it). **User-assigned** is a standalone resource that can be attached to multiple resources. |
| 40 | **B** | A **user-assigned managed identity** shared across all 5 functions requires only 1 role assignment on Key Vault, simplifying management. |
| 41 | **A** | `DefaultAzureCredential` (or `ManagedIdentityCredential`) acquires a token from the local IMDS endpoint. The credential is passed to the SDK client (e.g. `BlobServiceClient`). |
| 42 | **B** | RBAC is granted on the **target resource** (Storage account IAM blade). Assign **Storage Blob Data Reader** to the App Service's managed identity principal. |
| **Cosmos DB Server-Side** | | |
| 43 | **B** | **Stored procedures** run server-side within a single partition and support ACID transactions across multiple operations. |
| 44 | **B** | Stored procedures are scoped to a **single logical partition**. You pass the partition key value when invoking them; they cannot span partitions. |
| 45 | **A** | A **pre-trigger** runs before the operation (create, replace, delete) and can validate, modify, or reject the document before it is committed. |
| 46 | **B** | **UDFs** are JavaScript functions registered in Cosmos DB that can be used inside SQL queries for custom computations or formatting. |
| **Azure CDN** | | |
| 47 | **B** | **Azure CDN** caches content at edge locations worldwide, reducing latency for users far from the origin (Blob Storage). |
| 48 | **A** | **Purging** the CDN endpoint (or specific path) forces edge servers to discard cached content and fetch the latest version from the origin. |
| 49 | **A** | A **CDN caching rule** with the Override behavior sets a custom TTL, ignoring whatever the origin sends in `Cache-Control`. |
| 50 | **B** | **Honor origin** tells the CDN to respect the origin's `Cache-Control` and `Expires` headers without any CDN-side override. |
