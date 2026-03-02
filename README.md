# Memory and Learning
**Session Time:** 120 minutes  
**Format:** Instructor-Led  

---

## Prerequisites

- Completion of Sessions 1–7  
- Understanding of short-term and long-term memory as agent components (Session 2)  
- Familiarity with context management strategies, including summarization and selective retrieval (Session 6)  
- Understanding of the feedback loop as a mechanism for agent improvement (Session 2)  

---

## Learning Objectives

By the end of this session, students will be able to:

1. **Differentiate** between short-term (contextual) memory and long-term (persistent) memory in agentic systems.
2. **Apply** a simple memory setup that allows the agent to remember information from previous interactions.
3. **Evaluate** how memory enhances the agent's ability to adapt and improve over time, while taking privacy considerations for stored user data into account.

> Memory is what transforms an agent from a stateless tool into a persistent, adaptive collaborator. This session moves from the conceptual (introduced in Session 2) to the practical — how memory is actually built, stored, retrieved, and governed.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Short-Term vs. Long-Term Memory | 15 min |
| 2 | Building a Memory System: From Concept to Configuration | 20 min |
| 3 | Memory, Adaptation, and Privacy | 10 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Short-Term vs. Long-Term Memory (15 min)

### The Statelessness Problem

By default, every LLM call is stateless. The model receives a prompt, generates a response, and forgets everything. The next call starts from zero — no knowledge of who the user is, what was discussed, or what actions were taken.

For a one-shot task, statelessness is fine. For anything that requires continuity — a multi-turn conversation, a workflow that spans sessions, a user relationship that develops over time — statelessness is a fundamental architectural limitation.

Memory is the architectural solution to statelessness. But not all memory is the same, and choosing the wrong type for a given use case produces systems that are either unnecessarily complex or functionally inadequate.

### Short-Term Memory (Contextual)

Short-term memory is everything the agent can currently "see" — the active contents of the LLM's context window during a single session.

**How it works:**  
The conversation history is passed as part of the prompt on every LLM call. Each new turn appends the previous exchange, so the model has access to the full session transcript when generating its next response.

```
Turn 1:
  User:  "My order hasn't arrived."
  Agent: "I can help. What's your order number?"

Turn 2 — what the LLM receives:
  [System prompt]
  User:  "My order hasn't arrived."
  Agent: "I can help. What's your order number?"
  User:  "It's 48291."
```

The model "remembers" turn 1 not because it stored anything — but because turn 1 was re-injected into the prompt for turn 2.

**Properties:**

| Property | Value |
|---|---|
| Scope | Current session only |
| Storage location | LLM context window (in-memory, not persisted) |
| Retrieval | Automatic — all content is present |
| Persistence | Destroyed when the session ends |
| Capacity limit | Model's context window (e.g., 128K tokens for GPT-4o) |
| Implementation complexity | Minimal — most platforms handle this automatically |

**Where it works well:**  
Single-session tasks, conversations where the full history fits in the context window, workflows where there is no need to remember anything between sessions.

**Where it breaks down:**  
Long conversations (history exceeds context window), multi-session workflows (user returns the next day), personalization across users (agent needs to know who this user is before they say anything).

### Long-Term Memory (Persistent)

Long-term memory stores information outside the LLM, in external systems that survive session end. It is retrieved selectively — not everything is injected into every prompt, only what is relevant to the current interaction.

**How it works:**  
At defined points in the conversation, the agent writes information to an external store (a vector database, a key-value store, or a relational database). At the start of subsequent sessions, the agent queries that store with the current context and retrieves the most relevant information before reasoning begins.

```
Session 1 (ends):
  Agent writes to memory:
    → "User prefers email over phone for follow-ups"
    → "Order #48291 was resolved with expedited shipping on 2024-03-12"
    → "User has expressed frustration with delivery delays twice"

Session 2 (starts):
  Agent queries memory with: "Customer C10482 — support interaction"
  Agent retrieves:
    → Previous resolution summary
    → Communication preference
    → Frustration history
  Agent injects retrieved context into prompt before reasoning
```

**Properties:**

| Property | Value |
|---|---|
| Scope | Cross-session, cross-interaction |
| Storage location | External database (vector, key-value, or relational) |
| Retrieval | Query-based (semantic search or exact lookup) |
| Persistence | Survives session end; configurable retention policy |
| Capacity limit | Effectively unbounded |
| Implementation complexity | Significant — requires database setup, embedding model, retrieval logic |

**Where it works well:**  
Customer service (agent knows the user's history), personal assistants (agent remembers preferences and past tasks), knowledge agents (agent accumulates domain knowledge over time), multi-session workflows.

**Where it adds unnecessary complexity:**  
Single-use tools, anonymous interactions with no continuity value, tasks where every session is genuinely independent.

### The Four Types of Long-Term Memory

Long-term memory is not monolithic. Different types of information should be stored and retrieved differently:

<img width="934" height="421" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/0897c8c0-375a-49a4-89f9-c13575c92993" />

A well-designed memory system often combines multiple types. A customer service agent might use semantic memory for product knowledge, episodic memory for past interactions, and entity memory for customer profiles.

### Side-by-Side Comparison

| Dimension | Short-Term (Contextual) | Long-Term (Persistent) |
|---|---|---|
| Scope | Current session | All sessions |
| Persistence | In-memory only | External database |
| Retrieval | Automatic (full history present) | Query-based (selective injection) |
| Capacity | Context window size | Effectively unbounded |
| Relevance control | None (everything is present) | High (only relevant content retrieved) |
| Implementation effort | Minimal | Significant |
| Privacy risk | Low (no data stored externally) | Higher (data persists; requires governance) |
| Best for | Single-session tasks | Continuity, personalization, adaptation |

---

## 2. Building a Memory System: From Concept to Configuration (20 min)

### The Memory Pipeline

A functional long-term memory system has four stages that operate on every interaction:

**Interaction occurs**

<img width="693" height="306" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/06370da0-4a58-420e-80e0-9695f94d5616" />


Each stage requires explicit design decisions. The following sections examine each stage in detail.

### Stage 1: Write — What to Store and When

Not everything that happens in a conversation is worth storing. Indiscriminate storage produces a memory system filled with noise, which degrades retrieval quality and increases storage costs.

**Decision 1: What to store**

Apply a relevance filter: store information that will be useful in future interactions and that the agent cannot derive from other sources at the time of retrieval.

| Store this | Not this |
|---|---|
| User preferences stated explicitly ("I prefer email") | Boilerplate greetings and acknowledgments |
| Resolution outcomes ("Refund of $120 processed on 2024-03-12") | Intermediate reasoning steps within a session |
| Recurring patterns ("User has reported late delivery three times") | Tool call parameters that are order-specific and non-recurring |
| Entity facts ("Account tier: Premium; primary contact: Maria Silva") | Information already available in the live CRM at query time |

**Decision 2: When to store**

Memory writes should happen at defined trigger points, not continuously:

- **End of session:** Write a structured summary of the interaction
- **On resolution:** Write the outcome and any commitments made
- **On explicit user statement:** When a user states a preference, constraint, or identity fact
- **On pattern detection:** When the agent detects a recurring issue worth flagging

**Decision 3: Storage format**

How information is stored determines how it can be retrieved. Two formats are most common:

*Unstructured text chunks* — natural language summaries stored as text, embedded for semantic search:
```
"Customer C10482 contacted support on 2024-03-12 regarding a delayed order (#48291). 
Issue was resolved by upgrading to expedited shipping at no charge. Customer expressed 
satisfaction with the resolution. Communication preference: email."
```

*Structured records* — key-value or relational format, retrieved by exact lookup:
```json
{
  "customer_id": "C10482",
  "name": "Maria Silva",
  "account_tier": "Premium",
  "communication_preference": "email",
  "open_issues": [],
  "last_interaction": "2024-03-12",
  "resolution_history": ["order_48291_expedited_shipping_waived"]
}
```

Most production systems use both: structured records for entity facts (fast, exact lookup) and text chunks for interaction history (flexible, semantic search).

### Stage 2: Embed — Converting Text to Vectors

For semantic search to work, stored text must be converted to vector representations — numerical embeddings that capture the semantic meaning of the text. When a query arrives, it is also embedded, and the system finds stored vectors that are most similar to the query vector.

**How embeddings enable semantic retrieval:**

```
Stored memory:  "Customer prefers to be contacted by email for all follow-ups"
                → Embedded as vector [0.23, -0.87, 0.41, ...]

New query:      "What communication channel does this customer prefer?"
                → Embedded as vector [0.21, -0.84, 0.39, ...]

Similarity:     High — these vectors are close in the embedding space
Result:         Memory chunk retrieved and injected into the prompt
```

The embedding model is a critical configuration choice. The same model must be used for both storing and querying — if the model is changed, all stored embeddings must be regenerated.

**Common embedding models used in agentic systems:**

| Model | Provider | Key Characteristic |
|---|---|---|
| text-embedding-3-small | OpenAI | Fast, cost-efficient, good general purpose |
| text-embedding-3-large | OpenAI | Higher accuracy, higher cost |
| embed-english-v3.0 | Cohere | Strong multilingual support |
| all-MiniLM-L6-v2 | HuggingFace (open-source) | Self-hostable, no API cost |

### Stage 3: Retrieve — Querying the Memory Store

Retrieval is the most architecturally nuanced stage of the memory pipeline. It determines what information the agent has access to when it reasons — and therefore directly determines the quality of its decisions.

**Retrieval parameters:**

**k (number of results):** How many memory chunks to retrieve per query.
- Too few: the agent misses relevant context
- Too many: the context window fills with marginally relevant information, degrading reasoning quality
- Typical starting value: k = 3–5; adjust based on observed performance

**Similarity threshold:** The minimum similarity score required for a chunk to be returned.
- Without a threshold, the system always returns k results even when none are relevant
- Setting a threshold prevents irrelevant memories from being injected
- Typical starting value: 0.75–0.80 cosine similarity; adjust based on embedding model

**Recency weighting:** Should recent memories be prioritized over older but semantically similar ones?
- For customer service: yes — a recent interaction is more relevant than one from two years ago
- For knowledge bases: no — a well-written policy document from last year is as relevant as one from this week

**Hybrid retrieval:** Combining semantic search with metadata filters for more precise results.
```
Query: "Has this customer reported delivery issues before?"
Semantic search: finds interaction summaries mentioning delivery
Metadata filter: customer_id = "C10482" AND category = "delivery"
Result: only delivery-related memories for this specific customer
```

### Stage 4: Inject — Structuring Retrieved Context in the Prompt

Retrieved memory must be structured carefully in the prompt to ensure the LLM treats it as contextual background rather than as new instructions.

**Recommended injection structure:**

```
[System Prompt]
You are a customer support agent for Meridian Commerce...

[Retrieved Memory — injected before current input]
Customer history (retrieved from memory):
- Last interaction: 2024-03-12 — Delayed order resolved with expedited shipping
- Communication preference: email
- Account tier: Premium
- Note: Customer has reported delivery delays on two previous occasions

[Current Input]
Customer: "Hi, I'm having another issue with a delivery."
```

By labeling the retrieved content as "Customer history" and positioning it after the system prompt but before the current input, the LLM receives it as background context — relevant to the current interaction but clearly distinct from the user's current message.

---

## 3. Memory, Adaptation, and Privacy (10 min)

### How Memory Enables Adaptation

An agent with long-term memory does not just remember — it adapts. Over time, accumulated memory enables three forms of adaptation that are not possible in a stateless system:

<img width="930" height="402" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/86739971-3b3a-44dd-a352-f94ba07e496c" />


### Privacy: What Memory Costs

Long-term memory is powerful, but it comes with obligations. When an agent stores information about users across sessions, it is not just building a better product — it is creating a data asset that carries legal, ethical, and security responsibilities.


<img width="926" height="387" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/a96d34f6-55c1-424e-8fe3-30aea922a3b7" />


### The Memory Governance Checklist

Before deploying a long-term memory system, verify:

<img width="536" height="387" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/9cd6436e-1e41-4e64-a581-6fda77392ec9" />


A memory system that fails more than two of these checks should not be deployed with real user data until the gaps are addressed.

---

## 4. Break (10 min)

---

## 5. Lab (15 min)

*Lab details in the Lab Guide document.*

---

## 6. Wrap-Up & Reflection (5 min)




> Next session: **Multi-Agent Coordination** 

---

## Resources

- [LangChain — Memory Types and Configuration](https://docs.langchain.com/docs/components/memory)
- [OpenAI — Assistants API: Thread and Message Management](https://platform.openai.com/docs/assistants/overview)
- [Pinecone — Vector Database Documentation](https://docs.pinecone.io/)
- [Weaviate — Getting Started Guide](https://weaviate.io/developers/weaviate)
- [ChromaDB — Open-Source Embedding Database](https://docs.trychroma.com/)
- European Commission — *General Data Protection Regulation (GDPR), Article 17: Right to Erasure*
- Lilian Weng — *LLM-powered Autonomous Agents: Memory* (2023) — [lilianweng.github.io](https://lilianweng.github.io/posts/2023-06-23-agent/#memory)
