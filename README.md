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
