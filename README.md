# **Dynamic Agent-Proxy Gateway: The Secure Manager of Your Personal AI Ecosystem**

## **Core Concept: Connecting AI Thought with Real-World Action**

In the digital world, you have dozens of personal data sources: financial apps, shopping accounts, energy consumption data, health records, notes, WhatsApp conversations, email, and corporate databases. You also have access to powerful AI (like GPT-4 or Claude) that is excellent at **thinking, planning, and analysis**. The problem is, you cannot safely grant this AI the passwords to all your applications to let it securely **act and answer** complex questions on your behalf.

The **Dynamic Agent-Proxy Gateway** is the architecture that solves this problem. Think of it as an **Intelligent Steward** that securely bridges your intentions with real data and actions across any system you own.

## **How It Works – Three AI Roles in One Team**

Our system divides the work into specialized roles that collaborate just like a professional team:

### **1\. The Client Agent (The Planner – External AI)**

* **Role:** The brain and strategist.  
* **What it does:** It tracks your conversation, understands your intent, and decides what needs to be done (e.g., "I need to find out my spending on shopping in May").  
* *This agent is external and **never knows your passwords**.*

### **2\. The Proxy Agent (The Executor – Our Gateway)**

* **Role:** The manager, executor, and security guard.  
* **What it does:** When the Client Agent requests an action, this agent automatically performs a **triple security check**:  
  * **Authentication:** Is the calling user/AI verified?  
  * **Authorization (ACL):** Does this specific user/AI have the *actual* permission to perform this action? For example, allowing them to read purchase history but strictly forbidding money transfers.  
  * **Credential Management:** Once access is approved, our system **itself retrieves** the encrypted personal key or shared credential from an ultra-secure vault (Vault), intended only for this single transaction.

### **3\. The Connector (The Tool – Your Personal Data Source)**

* **Role:** The tool and the target of the action. This can be any API—from an energy supplier and a banking app to your personal notes database.  
* **What it does:** Our Gateway securely uses the retrieved key (which it never exposes to the AI) to perform the action on the real system and returns only the result to the AI.

## **How it Works Under the Hood? AI for AI.**

What makes our system a truly smart, secure, and extensible platform is its internal mechanism—the **Proxy Agent** (or Administrator Agent).

When you ask a complex question (e.g., "How much did I spend on footwear last year, or when was my last dentist visit?"):

1. The **External Brain (Client Agent)** sends our system the intent of the query (what you want to know). This can be an **external AI model** (like GPT-4) or a **local AI model** you self-manage.  
2. Our system activates its own, super-fast **Proxy Agent** (we utilize a low-cost LLM, e.g., Anthropic Claude 3 Haiku, or a model running directly on your hardware). The task of this internal AI is singular: to determine which function to use.  
3. This Proxy Agent does not search merely by keywords. It uses a special index that understands meaning (**embeddings model**) to find the **exact** function among thousands of connectors in milliseconds (e.g., 'Get payment history' and 'Get last preventative check-up record').  
4. Only then does it perform the verification (ACL) and securely use the keys from the secure storage (Vault) to execute the action.

In short: We use an AI that manages, discovers, and secures data access for another AI. This ensures lightning speed, maximum security, and unlimited extensibility.

## **Key Benefits for Every User**

The Agent-Proxy Gateway transforms your AI assistant into a personal analyst that securely connects your scattered data:

* **Answers from Complex Sources:** You can ask questions that require combining information from various personal systems:  
  * "**How much electricity did we consume in 2025** and what is my average monthly spending on groceries at a specific store?"  
  * "**When was my last preventative check-up** and what are my average monthly expenses for health insurance and medication combined?"  
  * "Find everything related to 'Project X' across my notes and emails and generate a summary."  
* **Absolute Data Privacy:** Your personal passwords, API keys, and tokens **are never exposed** to the external AI agent. Everything is centrally protected in the Vault, and the AI only sees the result, never the path to the data.  
* **Universal (Plug-and-Play) Integration (The Connector Protocol):** We eliminate the need for custom programming by introducing a standardized **Connector Protocol** (or "OpenAPI-X" file). Application owners (or developers) create this single, descriptive file, which defines the tool's functions, security requirements, and all necessary metadata for the Gateway's internal databases. Users can then simply **import this file** into their Gateway system and only supply the necessary personal credentials to the Vault. This instant onboarding process ensures no additional coding or configuration is required by the user, enabling immediate, secure use.  
* **Complete Control (ACL):** You, as the user, define the precise rules (ACL). You can allow the AI to **read** your bank balance but strictly forbid it from **executing** any transfers.

**Conclusion:** The Agent-Proxy Gateway transforms your chaotic digital life into a secure, intelligent, and fully automated AI ecosystem, available directly to you as an individual.

<br /><br />

# **Technical Architecture: Dynamic Agent-Proxy Gateway**

**Version: 1.0** **Date: November 9, 2025**

## **1\. Introduction and System Objectives**

### **1.1 Problem Statement**

Modern Large Language Models (LLMs) and AI agents lack secure, reliable, and standardized access to external, private, or real-time data sources (APIs, databases, internal systems). Direct integration of these sources into client applications is insecure, poorly scalable, and requires constant refactoring of business logic.

### **1.2 Solution**

This document describes the architecture of a **Dynamic Agent-Proxy Gateway** (hereafter "the Gateway")—a robust, multi-tenant system that serves as a secure intermediary between external AI agents and internal data sources.

### **1.3 Key Objectives**

* **Security:** To provide granular control over *who* (Authentication), *what* (Authorization), and *with what credentials* (Credential Scoping) can access data.  
* **Standardization:** To create a unified protocol ("Connector Protocol") for defining and onboarding any data source.  
* **Flexibility:** To allow external AI agents (from any provider) to securely utilize tools without knowledge of their internal logic.  
* **Abstraction:** To separate the semantic *discovery* of a tool from its technical *execution*.

## **2\. Architecture Overview**

The system is built on a hierarchical dual-agent model, which decouples conversation management from tool execution.

1. **Client Agent (External):** The "brain" of the operation. It is **stateful**, maintains the conversation context, and plans execution steps. It communicates with our Gateway via a secure API.  
2. **Agent-Proxy Gateway (Our System):** The "heart" and "executor." It contains the Proxy Agent and the Data Hub. It is **stateless** and processes atomic transactions.  
3. **Data Hub (Internal):** The storage layer for metadata, semantics, and credentials.  
4. **Connectors (Sources):** External APIs and third-party services.

## **3\. Key Components**

### **3.1 Client Agent (External)**

* **Role:** Planner, Context Manager.  
* **Technology:** Any LLM system (OpenAI, Anthropic, custom) capable of making API calls and maintaining state.  
* **Responsibility:** Maintaining conversation history, storing API call results, and formulating complete, contextual queries for the Gateway.

### **3.2 Agent-Proxy Gateway (Core System)**

* **Role:** Secure Intermediary, Orchestrator, Executor.  
* **Technology:** Python, FastAPI, LangGraph.  
* **Responsibility:** Receiving queries, verifying identity (Authentication), checking permissions (Authorization), executing tools, and returning results.

### **3.3 Proxy Agent (Internal)**

* **Role:** Stateless transactional executor.  
* **Technology:** A fast, low-cost LLM (e.g., **Anthropic Claude 3 Haiku**).  
* **Responsibility:** Within a single transaction:  
  1. Understand the API query (user\_query).  
  2. Find the best semantic tool (by querying the Vector DB).  
  3. Prepare the tool for execution.  
  4. Return the result.

### **3.4 Data Hub (Storage)**

The Data Hub is divided into three specialized databases:

#### **3.4.1 Relational Database (PostgreSQL)**

* **Purpose:** "Connector Catalog," technical schemata, ACLs.  
* **Tables:**  
  * connectors: Stores connector definitions, including full\_schema\_json (JSONB) and credential\_scope ('SHARED'/'PER\_USER').  
  * users: List of system users.  
  * groups: List of permission groups.  
  * user\_groups\_membership: Join table (M:N) for users and groups.  
  * **function\_permissions**: The core ACL table (operation\_id, principal\_type \['USER'/'GROUP'\], principal\_id, access \['ALLOW'\]).

#### **3.4.2 Vector Database (ChromaDB)**

* **Purpose:** "Semantic Function Index" for fast tool discovery.  
* **Structure:** Each document represents *one function* (endpoint).  
* **Metadata (for each vector):**  
  * connector\_id (UUID): Links to PostgreSQL and Vault.  
  * operation\_id (String): The exact function name to be executed.  
  * user\_id / group\_ids (Filtering): Ensures the agent only searches for functions it is theoretically allowed to see.

#### **3.4.3 Secrets Manager (HashiCorp Vault)**

* **Purpose:** Secure storage of API keys, tokens, certificates.  
* **Structure (Paths):**  
  * **Shared Credentials:** secret/data/shared/\<connector\_id\>/\<vault\_key\_name\>  
  * **Per-User Credentials:** secret/data/user/\<user\_id\>/\<connector\_id\>/\<vault\_key\_name\>

## **4\. Connector Protocol ("OpenAPI-X")**

To ensure standardization and "plug-and-play" onboarding, third-party providers must supply a definition file in the following format:

### **4.1 Format**

A standard **OpenAPI 3.1 Specification** (JSON or YAML), extended with two custom (mandatory) x- sections.

### **4.2 Mandatory Extensions (x-)**

**1\. x-auth-type (String, Mandatory)**

* **Description:** Defines the authentication type required by the connector.  
* **Values:** api-key, oauth2-client-credentials, client-certificate, username-password, etc.

**2\. x-required-secrets (Array, Mandatory)**

* **Description:** A list of secrets that the user (or admin) must provide to the Vault.  
* **Object Structure:**  
  * secret\_id (String): Internal ID (e.g., "apiKey").  
  * description (String): Help text for the user (e.g., "Find this in your account Settings \> API Keys").  
  * vault\_key\_name (String): The key name used for storage in the Vault.

### **4.3 Schema Example (YAML)**

`openapi: 3.1.0`  
`info:`  
  `title: "Email Service Connector"`  
  `description: "Allows sending and reading emails on behalf of the user."`  
  `version: "1.0.0"`  
`servers:`  
  `- url: "[https://api.emailservice.com/v1](https://api.emailservice.com/v1)"`

`# --- Our Extensions ---`  
`x-auth-type: "oauth2-client-credentials"`  
`x-required-secrets:`  
  `- secret_id: "client_id"`  
    `description: "Your Client ID from the developer portal."`  
    `vault_key_name: "CLIENT_ID"`  
  `- secret_id: "client_secret"`  
    `description: "Your Client Secret."`  
    `vault_key_name: "CLIENT_SECRET"`  
`# --- End of Extensions ---`

`paths:`  
  `/send:`  
    `post:`  
      `summary: "Sends a single email. Suitable for sending invoices or notifications."`  
      `operationId: "send_email"`  
      `requestBody:`  
        `content:`  
          `application/json:`  
            `schema:`  
              `type: object`  
              `properties:`  
                `to: { type: string, format: email }`  
                `subject: { type: string }`  
                `body: { type: string }`  
      `responses:`  
        `'200':`  
          `description: "Email sent."`

## **5\. Processes and Data Flows**

### **5.1 Ingestion Pipeline (Connector Onboarding)**

An automated process triggered by the upload of an OpenAPI-X file:

1. **Validation:** Check syntax (JSON/YAML), structure (OpenAPI 3.1), and our custom x- extensions.  
2. **Schema Storage:** A new record is created in the connectors table (PostgreSQL) with status PENDING\_SECRETS. The credential\_scope (SHARED/PER\_USER) is defined.  
3. **Credential Handling (UI \+ Vault):** The system displays a conditional form based on x-required-secrets. The user (or admin) submits the credentials, which are stored directly in the Vault at the correct path (SHARED or PER\_USER).  
4. **Vectorization:** An asynchronous process. The system extracts info.description and paths.\*.summary from the connector. These texts are sent to the **System Embedding Model**, and the resulting vectors are stored in **ChromaDB** with metadata (connector\_id, operation\_id).  
5. **Activation:** Once credentials are in the Vault (Step 3\) and vectors are in ChromaDB (Step 4), the connector's status in PostgreSQL is updated to ACTIVE.

### **5.2 Transactional Flow (Query Processing)**

Detailed steps the Gateway performs upon receiving a POST /api/v1/agent/query:

1. **Authentication (Who are you?):** The system verifies the Authorization: Bearer \<JWT\> token from the header. It extracts user\_id and group\_ids. (Failure \-\> 401 Unauthorized).  
2. **Semantic Discovery:** The Proxy Agent (LLM) takes the user\_query and user\_context\_data, creates a query, and sends it to ChromaDB.  
3. **Tool Discovery:** ChromaDB returns the most relevant function (e.g., operation\_id \= "send\_email" and connector\_id \= "email-uuid"). (Not Found \-\> 404 Not Found).  
4. **Authorization (ACL \- What are you allowed?):** The system checks the function\_permissions table (PostgreSQL) to see if the user\_id or its group\_ids have 'ALLOW' permission for operation\_id \= "send\_email". (Failure \-\> 403 Forbidden).  
5. **Credential Scope Discovery:** The system checks the credential\_scope in PostgreSQL for connector\_id \= "email-uuid".  
6. **Credential Retrieval (Vault):**  
   * If SHARED, it fetches the key from secret/shared/email-uuid/.  
   * If PER\_USER, it fetches the key from secret/user/\<user\_id\>/email-uuid/. (Failure \-\> 400 Bad Request \- Missing personal credentials).  
7. **Execution:** The Proxy Agent (LLM) uses the full\_schema\_json (from PostgreSQL) and the fetched credentials (from Vault) to execute the real API call to the connector.  
8. **Response:** The result of the call is wrapped in the standardized JSON response and sent back to the Client Agent.

## **6\. Technology Stack**

| Component | Technology | Rationale |
| :---- | :---- | :---- |
| **Backend API** | Python 3.11+, FastAPI | Speed, modernity, async support, AI ecosystem. |
| **Orchestration** | LangChain / LangGraph | Standard "glue" for AI apps, supports stateful/stateless agents. |
| **Relational DB** | PostgreSQL 16+ | Robustness, reliability, JSONB support. |
| **Vector DB** | ChromaDB | Open-source, API-first, easy integration with LangChain. |
| **Secrets Manager** | HashiCorp Vault | Industry standard for secrets management, API-driven. |
| **Proxy Agent** | Anthropic Claude 3 Haiku | Low latency, low cost, excellent tool use capability. |
| **Client Agent (External)** | OpenAI GPT-4o / Claude 3 Opus | (Recommended) Top-tier performance for planning and context. |
| **Embedding Model** | (System-configurable) | See Section 7\. |

## **7\. Embedding Model Management**

The system uses a **single, system-configurable embedding model** to guarantee vector space consistency.

* **Configuration:** The user selects and configures the model in the system settings (not per-connector) (e.g., OpenAI text-embedding-3-small or default all-MiniLM-L6-v2).  
* **Credentials:** API keys for embedding models (e.g., OPENAI\_API\_KEY) are stored in the Vault.  
* **Re-Indexing:** If the user changes the system embedding model, the system **must** trigger a full re-indexing process for all vectors in ChromaDB.

## **8\. Communication API (Gateway Endpoint)**

### **8.1 Endpoint**

POST /api/v1/agent/query

* **Authentication:** Authorization: Bearer \<JWT\_TOKEN\>

### **8.2 Request Body Schema**

`{`  
  `"user_query": "Send that invoice we talked about to accounting@company.com",`  
  `"conversation_id": "conv-uuid-12345",`  
  `"user_context_data": {`  
    `"invoice_details": {`  
      `"id": "FA-987",`  
      `"amount": 15000,`  
      `"currency": "USD"`  
    `}`  
  `}`  
`}`

### **8.3 Response Schema (Success, 200 OK)**

`{`  
  `"status": "SUCCESS",`  
  `"final_data": {`  
    `"email_id": "msg-xyz-789",`  
    `"status": "Sent"`  
  `},`  
  `"internal_log": "Tool 'send_email' found. Execution successful."`  
`}`

### **8.4 Response Schema (Failure, 4xx/5xx)**

`{`  
  `"status": "FAILURE",`  
  `"error_code": "AUTH_FAILED",`  
  `"error_message": "Authentication with the target connector failed. Please check your personal credentials.",`  
  `"technical_details": {`  
    `"http_status": 401,`  
    `"api_message": "Invalid Credentials"`  
  `}`  
`}`  
