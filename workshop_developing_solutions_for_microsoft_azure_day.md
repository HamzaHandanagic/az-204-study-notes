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
- **Premium / Dedicated:** Timeout can be much longer (e.g. unbounded on Dedicated, configurable on Premium).

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


