# multi-agent-support-system
An intelligent multi-agent customer support system that classifies user intent, retrieves contextual knowledge using RAG, and executes real actions like refunds and order tracking via APIs


# MVP model Architecture.
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