# Memory and Learning
**Session Time:** 120 minutes  
**Format:** Instructor-Led  

---

## Prerequisites

- Completion of Sessions 1–7  
- Understanding of short-term and long-term memory as agent components (Session 2)  
- Familiarity with context management strategies introduced in Session 6  
- Understanding of the feedback loop as a mechanism for agent improvement (Session 2)  

---

## Learning Objectives

By the end of this session, students will be able to:

1. **Differentiate** between short-term (contextual) memory and long-term (persistent) memory in agentic systems.
2. **Apply** a simple memory setup that allows the agent to remember information from previous interactions.
3. **Evaluate** how memory enhances the agent's ability to adapt and improve over time, while taking privacy considerations for stored user data into account.

> Memory is what transforms an agent from a stateless tool into a persistent, adaptive collaborator. This session moves from the conceptual (introduced in Session 2) to the practical — how memory works, how to set it up in a no-code platform, and what responsibilities come with storing user data.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Short-Term vs. Long-Term Memory | 15 min |
| 2 | How Memory Works in Practice | 20 min |
| 3 | Memory, Adaptation, and Privacy | 10 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Short-Term vs. Long-Term Memory (15 min)

### The Statelessness Problem

By default, every LLM call is stateless. The model receives a prompt, generates a response, and forgets everything. The next call starts from zero — no knowledge of who the user is, what was discussed, or what actions were taken.

Imagine a customer service agent that, every time a customer comes back, greets them as if they've never spoken before. No record of the last issue. No memory of what was promised. Starting from scratch, every time.

That is a stateless agent — and it is the default behavior of any LLM without memory built in.

**Memory is the architectural solution to statelessness.** But there are two fundamentally different types of memory, and they solve different problems.

---

### Short-Term Memory (Contextual)

Short-term memory is everything the agent can currently "see" — the active contents of the conversation during a single session.

**How it works in plain language:**  
The conversation history is passed along with every new message. Each time the user sends something new, the agent receives the full conversation so far — not just the latest message. That's why it can refer back to what was said earlier in the same chat.

```
Turn 1:
  User:  "My order hasn't arrived."
  Agent: "I can help. What's your order number?"

Turn 2 — what the agent receives:
  [Everything from Turn 1, plus:]
  User: "It's 48291."
```

The agent "remembers" Turn 1 not because it stored anything — but because Turn 1 was included in the new message it received.

**What it's good for:**  
Single conversations. Tasks that start and finish in one session. Simple workflows where nothing needs to carry over to a future interaction.

**Where it breaks down:**  
The session ends — and everything is gone. If the user comes back tomorrow, the agent has no idea who they are or what happened. It starts over.

---

### Long-Term Memory (Persistent)

Long-term memory stores information outside the agent, in an external system that survives when the session ends. That stored information can be retrieved and used in future interactions.

**How it works in plain language:**  
Think of it like a notebook the agent keeps between sessions. At the end of a conversation, it writes down what's important. The next time the same user comes back, the agent checks the notebook before responding — and already knows their name, their history, their preferences.

```
Session 1 ends:
  Agent saves to memory:
    → "User prefers email for follow-ups"
    → "Order #48291 — resolved March 12 with expedited shipping"
    → "User has reported delivery delays twice before"

Session 2 begins:
  Agent checks memory before responding:
    → Retrieves the notes above
    → Knows who the user is and what happened last time
    → Can personalize the conversation from the first message
```

**What it's good for:**  
Returning users. Multi-session workflows. Personalization. Any situation where continuity across time is valuable.

**What it requires:**  
A place to store the information (a database or memory system built into your platform) and a decision about what to save and what to leave out.

---

### Side-by-Side Comparison

<img width="935" height="428" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/7e09c983-ab82-4494-aec8-97b946c293c2" />

---

### Quick Check (2 min)

> A user contacts a travel booking agent to plan a trip to Japan. They return three weeks later wanting to add flights. What type of memory would the agent need to remember the original trip plan? What type would be enough to handle the current conversation?

*(Expected answer: Long-term memory to recognize the user and recall the original plan. Short-term memory is sufficient within each individual session.)*

---

## 2. How Memory Works in Practice (20 min)

### What Happens When Memory Is Enabled

Whether you're using GPT Builder, Flowise, Relevance AI, or another no-code platform, enabling memory follows the same basic pattern — even if the interface looks different. At its core, the memory system does four things:

<img width="917" height="227" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/f34770ab-29f1-4074-8658-6aef7e3cea17" />


You don't need to understand how databases work to use this. What you do need to understand is: **what to save, and when**.

---

### What Should Be Saved?

Not everything is worth remembering. Saving too much creates noise — the agent retrieves irrelevant information and gets confused. Saving too little means the memory is useless.

A good rule of thumb: **save what would be useful to know at the start of the next conversation.**

<img width="937" height="292" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/0380dd86-02da-40de-bcaf-a381bbd2bf1f" />

---

### Setting Up Memory in a No-Code Platform

Every platform handles memory slightly differently, but the configuration decisions are the same across all of them. Here's what you'll typically configure:

<img width="896" height="416" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/a71a5018-739b-4df7-9243-1d78da6d9bf9" />

---

### What Memory Looks Like in the Prompt

When the agent retrieves a memory and uses it, that retrieved information is inserted into the prompt before the agent responds. In practice, this looks something like:

```
[System Prompt]
You are a customer support agent for Meridian Commerce...

[Memory — retrieved from previous sessions]
Customer history:
- Last interaction: March 12 — Delayed order resolved with expedited shipping
- Communication preference: email
- Account tier: Premium
- Note: Has reported delivery issues twice before

[Current message from user]
"Hi, I'm having another issue with a delivery."
```

The agent sees the memory as background context. It knows this user before they've said more than one sentence. That's what makes the interaction feel personalized and continuous — not magic, just well-structured information flow.

---

### Memory Types in No-Code Platforms

Most no-code platforms simplify memory into two or three practical options. Here's how to think about them:

<img width="920" height="358" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/1c4e7067-5eda-4882-9ddc-9feb63fc4233" />


In most entry-level agent projects, you will use conversation memory by default and add user memory when continuity becomes important.

---

### Discussion Prompt (3 min)

> Think about a real situation — at work or in your personal life — where an AI tool completely forgot who you were and what you needed, even though you'd used it before. What would have been different if it had memory? What would you want it to remember?

---

## 3. Memory, Adaptation, and Privacy (10 min)

### How Memory Enables Better Agent Behavior

An agent with memory doesn't just remember — it gets better at serving the user over time. Three things happen when memory is working well:

**1. Personalization**  
The agent adjusts its behavior based on what it knows about the specific user.

```
Without memory:
  Agent: "Would you prefer a refund or a replacement?"

With memory (user has previously declined replacements twice):
  Agent: "I'll go ahead and process a refund — I can see that's been your 
          preference in past interactions. Shall I proceed?"
```

This isn't just a nicer experience. It reduces back-and-forth, speeds up resolution, and signals to the user that their history is valued.

**2. Pattern recognition**  
When the agent accumulates memory across many users, patterns emerge that would be invisible in any single conversation.

```
Memory analysis reveals:
  → 23% of users in a specific region have reported delivery delays this month
  → That's 4x higher than usual

This surfaces a systemic issue — a carrier problem in that region — 
that no individual conversation would have revealed.
```

This is not something the agent does alone — it requires someone to review the memory data. But the data is only available because memory was being collected.

**3. Operational improvement**  
Memory of past failures helps identify where the agent needs to improve — and makes those improvements targeted and measurable.

```
Memory analysis reveals:
  → The agent escalates to a human for 34% of refund requests 
    between $150–$200
  → On review, 80% of those escalations were unnecessary

Fix:
  → System prompt updated to clarify the policy boundary
  → Escalation rate for that range drops from 34% to 8%
```

This cycle — observe, identify, improve — is only possible when the agent's behavior is being recorded and reviewed.

---

### Privacy: What Memory Costs

Long-term memory is powerful, but it comes with real responsibilities. When an agent stores information about users across sessions, it creates a data asset that must be managed carefully.

**Four things to get right:**

<img width="923" height="284" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/1d44a2ef-1bb6-4f80-8cc0-b76855f8ca16" />


### The Memory Governance Checklist

Before deploying a long-term memory system with real users, verify:

<img width="614" height="354" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/466cb947-4f09-4d44-9e02-5adaac91ffbe" />


A memory system that fails more than two of these checks should not be used with real user data until the gaps are addressed.

---

## 4. Break (10 min)

---

## 5. Lab (60 min)

https://ga-curriculum.github.io/memory-and-learning/

---

## 6. Wrap-Up & Reflection (5 min)

### Key Takeaways
- Memory Is What Makes Agents Feel Continuous
- Short-Term Memory Is Automatic. Long-Term Memory Is a Design Decision.
- Memory Comes With Responsibility


> Next session: **Multi-Agent Coordination**

---

## Resources

- [OpenAI — Memory in ChatGPT and Assistants API](https://platform.openai.com/docs/assistants/overview)
- [Flowise — Memory Node Documentation](https://docs.flowiseai.com/integrations/memory)
- [Relevance AI — Agent Memory Configuration](https://relevanceai.com/docs)
- [LangChain — Memory Types Overview](https://docs.langchain.com/docs/components/memory)
- European Commission — *General Data Protection Regulation (GDPR), Article 17: Right to Erasure*
- Lilian Weng — *LLM-powered Autonomous Agents: Memory* (2023) — [lilianweng.github.io](https://lilianweng.github.io/posts/2023-06-23-agent/#memory)


---

*General Assembly · Build AI Agents · BAA 8.0 · © 2026*
