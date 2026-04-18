# multi-agent-support-system
An intelligent multi-agent customer support system that classifies user intent, retrieves contextual knowledge using RAG, and executes real actions like refunds and order tracking via APIs


# MVP model Architecture.
```text
    ┌────────────────────────────────────────────────────┐
    │                 User / Frontend                    │
    │          (React.js UI + WebSockets)                │
    └─────────────────────────┬──────────────────────────┘
                              │ 1. User Message ("Where is my order?")
                              ▼
    ┌────────────────────────────────────────────────────┐
    │             Backend Gateway (Node.js)              │
    │         (Auth, Session ID, Payload Prep)           │
    └─────────────────────────┬──────────────────────────┘
                              │ 2. HTTP POST
                              ▼
╔════════════════════════════════════════════════════════════╗
║               n8n ORCHESTRATOR (The Supervisor)            ║
║                                                            ║
║  ┌─────────────────┐      ┌───────────────────────────┐    ║
║  │ Webhook Trigger │ ───► │ Intent Classifier (LLM)   │    ║
║  └─────────────────┘      │ (Outputs intent + score)  │    ║
║                           └─────────────┬─────────────┘    ║
║                                         │ 3. JSON Output   ║
║                                         ▼                  ║
║                           ┌───────────────────────────┐    ║
║                           │   Routing Layer (Switch)  │    ║
║                           └──────┬─────────────┬──────┘    ║
╚══════════════════════════════════│═════════════│═══════════╝
                                   │             │
        4a. [Intent: order, refund]│             │ 4b. [Intent: faq]
                                   │             │
             ┌─────────────────────┘             └──────────────────────┐
             ▼                                                          ▼
┌──────────────────────────┐                               ┌──────────────────────────┐
│     ACTION PATH          │                               │     KNOWLEDGE PATH       │
│  (Deterministic Tasks)   │                               │       (RAG Pipeline)     │
├──────────────────────────┤                               ├──────────────────────────┤
│ 1. HTTP Call to Node.js  │                               │ 1. Embed Query (LLM)     │
│ 2. Trigger Action Agent  │                               │ 2. Search Vector DB      │
│    (orderAgent API)      │                               │    (Chroma / FAISS)      │
│ 3. Query Business DB/API │                               │ 3. Retrieve Context Docs │
│ 4. Return Raw Data       │                               │ 4. Synthesize Answer     │
└────────────┬─────────────┘                               └────────────┬─────────────┘
             │                                                          │
             └─────────────────────┐             ┌──────────────────────┘
                                   ▼             ▼
                           ┌───────────────────────────┐
                           │ Response Formatter (LLM)  │
                           │  (Formats text for UI)    │
                           └─────────────┬─────────────┘
                                         │ 5. Final Output
                                         ▼
    ┌────────────────────────────────────────────────────┐
    │             Backend Gateway (Node.js)              │
    └─────────────────────────┬──────────────────────────┘
                              │ 6. WebSocket Push
                              ▼
    ┌────────────────────────────────────────────────────┐
    │                 User / Frontend                    │
    └────────────────────────────────────────────────────┘
```

# Tech Stack.
```text
-----------------------------------------------------------------------------------------------------------------------
 ARCHITECTURE COMPONENT | APPLICABLE TECH STACK                        | PURPOSE / APPLICABILITY
-----------------------------------------------------------------------------------------------------------------------
 Frontend UI            | React.js                                     | Builds the conversational Chat UI and 
                        |                                              | manages client-side state.
------------------------|----------------------------------------------|-----------------------------------------------
 Client-Server Comm.    | WebSockets (Socket.io) or HTTP (Axios/Fetch) | Handles real-time, bi-directional messaging 
                        |                                              | or standard request/response cycles.
------------------------|----------------------------------------------|-----------------------------------------------
 Backend Gateway        | Node.js, Express.js                          | Acts as the primary server. Handles auth, 
                        |                                              | session IDs, and forwards payloads to n8n.
------------------------|----------------------------------------------|-----------------------------------------------
 Orchestrator           | n8n                                          | The workflow engine. Manages webhooks, 
                        |                                              | routes data between LLMs, APIs, & DBs.
------------------------|----------------------------------------------|-----------------------------------------------
 Intent Classifier      | OpenAI (GPT-3.5/4) or Ollama (Local)         | Analyzes user messages to output a 
                        |                                              | structured JSON intent and confidence score.
------------------------|----------------------------------------------|-----------------------------------------------
 Routing Layer          | n8n (Switch Node)                            | Reads the JSON intent and directs the flow 
                        |                                              | to the Action Path or Knowledge Path.
------------------------|----------------------------------------------|-----------------------------------------------
 Action Agents          | Node.js, Express.js, SQL/NoSQL Drivers       | Custom endpoints in your backend that 
                        |                                              | execute deterministic business tasks.
------------------------|----------------------------------------------|-----------------------------------------------
 Embedding Model        | OpenAI Text-Embedding-3 or Local equivalent  | Converts the user's text query into 
                        |                                              | high-dimensional vectors for search.
------------------------|----------------------------------------------|-----------------------------------------------
 Vector Database        | FAISS (Local) or Chroma                      | Stores embedded documents and retrieves 
                        |                                              | relevant chunks based on query vector.
------------------------|----------------------------------------------|-----------------------------------------------
 Knowledge Agent (LLM)  | OpenAI, Anthropic, or Ollama                 | Takes retrieved context from Vector DB and 
                        |                                              | synthesizes a grounded, natural answer.
------------------------|----------------------------------------------|-----------------------------------------------
 Response Formatter     | OpenAI (GPT-3.5/4) or Ollama                 | A final LLM step to ensure output is 
                        |                                              | conversationally polished for the UI.
-----------------------------------------------------------------------------------------------------------------------
```

# FAQs about this Project.
```text
-----------------------------------------------------------------------------------------------------------------------
 CLIENT FAQ QUESTION                    | GENERATED ANSWER
-----------------------------------------------------------------------------------------------------------------------
 How is this different from a basic     | Unlike basic chatbots that just guess answers, this uses a multi-agent system. 
 ChatGPT bot?                           | It strictly separates factual search (RAG) from real business actions (APIs).
----------------------------------------|------------------------------------------------------------------------------
 Will the AI hallucinate or make up     | No. For business tasks (like refunds), it uses deterministic APIs, not LLMs. 
 wrong information about our rules?     | For questions, the RAG system strictly answers only from your uploaded docs.
----------------------------------------|------------------------------------------------------------------------------
 Can it connect to our existing SQL     | Yes. The Node.js action agents act as a bridge and can connect to any REST 
 database, ERP, or CRM?                 | API, SQL/NoSQL database, or CRM (like Salesforce) your company already uses.
----------------------------------------|------------------------------------------------------------------------------
 How do we teach the bot new policies   | You don't need to retrain the AI. You simply upload your new PDF or FAQ into 
 or update the knowledge base?          | the Vector DB, and the Knowledge Agent instantly uses the updated information.
----------------------------------------|------------------------------------------------------------------------------
 Is our company data secure? Can we     | Yes. Your Node.js gateway secures all data. If privacy is a strict concern, 
 host this locally?                     | we can swap OpenAI for Ollama, running the LLM entirely on your own servers.
----------------------------------------|------------------------------------------------------------------------------
 What happens if the bot is confused    | The system's Intent Classifier returns a "confidence score." If the score is 
 by a customer's question?              | too low, it triggers a fallback agent to seamlessly route to a human agent.
----------------------------------------|------------------------------------------------------------------------------
 Will API costs for the LLM be too      | We optimize costs. Deterministic queries (like checking an order) bypass 
 expensive with high user traffic?      | heavy LLM generation. You only pay for classification and complex FAQ answers.
----------------------------------------|------------------------------------------------------------------------------
 Can we add more features, like booking | Absolutely. Because it is orchestrated by n8n, we just add a new intent to 
 appointments, later on?                | the classifier and connect a new backend API without rewriting the system.
----------------------------------------|------------------------------------------------------------------------------
 How fast are the response times for    | API actions (like checking order status) are nearly instantaneous. Knowledge 
 the end user?                          | questions (RAG) take 1-3 seconds, delivered smoothly via WebSocket streams.
----------------------------------------|------------------------------------------------------------------------------
 Do we need a developer every time we   | Not for workflow changes. n8n provides a visual interface where you can easily
 want to change the routing logic?      | drag and drop nodes to change how intents are routed without touching code.
-----------------------------------------------------------------------------------------------------------------------
```