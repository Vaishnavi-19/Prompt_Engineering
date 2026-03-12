# Prompt Engineering Guide

A comprehensive guide to understanding Large Language Models, prompt design patterns, and context engineering.

---

## Table of Contents

1. [What is an LLM?](#1-what-is-an-llm)
2. [What is a Prompt?](#2-what-is-a-prompt)
3. [Prompting Techniques](#3-prompting-techniques)
   - [Zero-Shot Prompting](#31-zero-shot-prompting)
   - [Few-Shot Prompting](#32-few-shot-prompting)
   - [Chain of Thought (CoT)](#33-chain-of-thought-cot)
   - [ReAct Prompting](#34-react-prompting)
4. [Prompting Tips](#4-prompting-tips)
5. [Context Engineering with System Prompts](#5-context-engineering-with-system-prompts)

---

## 1. What is an LLM?

A **Large Language Model (LLM)** is a type of artificial intelligence model trained on massive amounts of text data to understand and generate human language. LLMs are built on the **Transformer architecture** and learn statistical patterns, grammar, facts, and reasoning abilities from billions of text tokens.

### Key Characteristics

| Characteristic | Description |
|---|---|
| **Scale** | Trained on hundreds of billions to trillions of tokens |
| **Parameters** | Contain billions of learnable weights (e.g., GPT-4 ~1.7T, Llama 3 ~70B) |
| **Capabilities** | Text generation, summarization, translation, coding, reasoning, Q&A |
| **Probabilistic** | Predicts the next most likely token given the input context |

### How LLMs Work (Simplified)

```
Input Text (Prompt)
       ↓
  Tokenization  →  [Token IDs]
       ↓
 Transformer Layers  →  Attention + Feed-Forward
       ↓
  Output Probabilities  →  Token Sampling
       ↓
Generated Text (Completion)
```

Popular LLMs include **GPT-4o**, **Claude 3**, **Gemini 1.5**, **Llama 3**, and **Mistral**.

---

## 2. What is a Prompt?

A **prompt** is the input you provide to an LLM to guide it toward producing a desired output. Think of it as an instruction, question, or context block that shapes what the model generates.

### Anatomy of a Prompt

```
┌─────────────────────────────────────────────────────────┐
│  SYSTEM CONTEXT (optional)                              │
│  "You are a helpful assistant specialized in finance."  │
├─────────────────────────────────────────────────────────┤
│  INSTRUCTION                                            │
│  "Summarize the following earnings report."             │
├─────────────────────────────────────────────────────────┤
│  INPUT DATA / CONTEXT                                   │
│  "Q3 revenue was $4.2B, up 18% YoY..."                  │
├─────────────────────────────────────────────────────────┤
│  OUTPUT INDICATOR (optional)                            │
│  "Summary:"                                             │
└─────────────────────────────────────────────────────────┘
```

A well-crafted prompt is the difference between a vague, unreliable response and a precise, actionable one.

---

## 3. Prompting Techniques

---

### 3.1 Zero-Shot Prompting

**Zero-Shot Prompting** asks the model to perform a task without providing any examples. You rely entirely on the model's pre-trained knowledge.

**When to use:** Simple, well-defined tasks; quick prototyping; when the task is within the model's general knowledge.

#### Example — Sentiment Classification

**Prompt:**
```
Classify the sentiment of the following review as Positive, Negative, or Neutral.

Review: "The battery life is amazing but the screen resolution is disappointing."

Sentiment:
```

**Output:**
```
Negative
```

> **Note:** Zero-shot works well for straightforward tasks but can fail on ambiguous or domain-specific problems.

---

#### Example — Language Translation

**Prompt:**
```
Translate the following sentence to French.

Sentence: "Artificial Intelligence is transforming the world."

Translation:
```

**Output:**
```
L'Intelligence Artificielle transforme le monde.
```

---

### 3.2 Few-Shot Prompting

**Few-Shot Prompting** provides the model with a handful of input-output examples before asking it to complete a new task. This guides the model on the expected format, tone, and reasoning pattern.

**When to use:** Tasks with a specific output format; domain-specific classification; when zero-shot produces inconsistent results.

#### Example — Product Category Classification

**Prompt:**
```
Classify the product into one of: Electronics, Clothing, Food, or Furniture.

Product: "Sony WH-1000XM5 Noise Cancelling Headphones"
Category: Electronics

Product: "Levi's 501 Original Fit Jeans"
Category: Clothing

Product: "Organic Rolled Oats 500g"
Category: Food

Product: "IKEA MALM 6-drawer Dresser"
Category: Furniture

Product: "Apple MacBook Pro 16-inch"
Category:
```

**Output:**
```
Electronics
```

---

#### Example — Email Tone Rewriting

**Prompt:**
```
Rewrite the email in a professional tone.

Casual: "Hey, just checking if you got my invoice? Need the cash ASAP."
Professional: "I wanted to follow up on the invoice I sent earlier. Could you please confirm receipt and advise on the expected payment timeline?"

Casual: "Can we push the meeting? Something came up."
Professional: "Would it be possible to reschedule our meeting? An urgent matter has arisen on my end."

Casual: "The report you sent is all messed up, fix it."
Professional:
```

**Output:**
```
The report you sent contains some discrepancies. Could you please review and send a corrected version at your earliest convenience?
```

---

### 3.3 Chain of Thought (CoT)

**Chain of Thought (CoT) Prompting** encourages the model to reason through a problem step by step before arriving at a final answer. This significantly improves performance on multi-step reasoning, math, and logic tasks.

**When to use:** Math problems, logical reasoning, multi-step decisions, debugging code, or any task requiring intermediate reasoning.

#### Example — Math Word Problem

**Prompt:**
```
Solve the following problem step by step.

A train travels from City A to City B at 90 km/h. The distance is 360 km.
It stops at a midpoint for 30 minutes. What is the total travel time?

Let's think step by step:
```

**Output:**
```
Step 1: Calculate total travel time without the stop.
   Distance = 360 km, Speed = 90 km/h
   Time = Distance / Speed = 360 / 90 = 4 hours

Step 2: Add the stop duration.
   Stop = 30 minutes = 0.5 hours

Step 3: Total time = 4 + 0.5 = 4.5 hours

Answer: The total travel time is 4 hours and 30 minutes.
```

---

#### Example — Logical Deduction

**Prompt:**
```
Answer the question by reasoning step by step.

Alice is taller than Bob. Bob is taller than Carol. Is Alice taller than Carol?

Let's think step by step:
```

**Output:**
```
Step 1: Alice > Bob (Alice is taller than Bob)
Step 2: Bob > Carol (Bob is taller than Carol)
Step 3: By transitivity: Alice > Bob > Carol, so Alice > Carol.

Answer: Yes, Alice is taller than Carol.
```

> **Tip:** Adding the phrase **"Let's think step by step"** is a simple and effective way to trigger chain-of-thought reasoning in most LLMs.

---

### 3.4 ReAct Prompting

**ReAct (Reasoning + Acting)** is a prompting framework that interleaves **thinking** (reasoning traces) with **acting** (tool calls or external actions). The model cycles through: `Thought → Action → Observation → Thought → ...` until it reaches a final answer.

**When to use:** Agentic tasks; tool-augmented LLMs; tasks requiring information retrieval, code execution, or API calls.

#### ReAct Loop Structure

```
Thought:   What do I know and what do I need to find out?
Action:    Call a tool or perform an action (e.g., Search, Calculator, API)
Observation: Result returned from the action
Thought:   What does this result tell me? Do I need more information?
Action:    (next action if needed)
...
Answer:    Final conclusion based on all observations
```

---

#### Example — Answering a Factual Question with Search

**Prompt:**
```
Answer the following question using the Thought/Action/Observation format.
You have access to a Search tool.

Question: What is the population of Japan as of 2024?

Thought: I need to find the current population of Japan. I should search for this.
Action: Search("Japan population 2024")
Observation: Japan's population in 2024 is approximately 123.7 million, continuing a gradual decline.

Thought: I now have the answer directly from the search result.
Answer: Japan's population as of 2024 is approximately 123.7 million.
```

---

#### Example — Multi-Step Calculation with Tool Use

**Prompt:**
```
Use Thought/Action/Observation steps to solve the problem.
You have access to a Calculator tool.

Question: If a laptop costs $1,200 and is discounted by 15%, then taxed at 8%, what is the final price?

Thought: First I need to calculate the discounted price.
Action: Calculator("1200 * (1 - 0.15)")
Observation: 1020

Thought: Now I need to apply 8% tax on $1,020.
Action: Calculator("1020 * (1 + 0.08)")
Observation: 1101.6

Thought: I have the final price after discount and tax.
Answer: The final price of the laptop is $1,101.60.
```

---

## 4. Prompting Tips

Apply these best practices to get consistently better results from any LLM.

### Be Specific and Clear

| Instead of... | Use... |
|---|---|
| "Write about AI" | "Write a 200-word blog intro about how AI is used in healthcare diagnostics" |
| "Summarize this" | "Summarize this article in 3 bullet points, each under 20 words" |
| "Fix my code" | "Fix the Python KeyError on line 12 and explain why it occurred" |

### Use Role Assignment

Give the model a persona to activate relevant knowledge and tone:

```
You are a senior software engineer with expertise in distributed systems.
Review the following architecture design and identify potential bottlenecks.
```

### Specify Output Format

```
Return your answer as a JSON object with keys: "summary", "sentiment", and "keywords".
```

### Use Delimiters to Separate Sections

Use `###`, `"""`, `<tag>`, or `---` to clearly separate instructions from data:

```
Summarize the text below:

"""
Quantum computing leverages principles of superposition and entanglement...
"""
```

### Iterate and Refine

LLM prompting is iterative. If the first output is off:
- Add more context
- Provide a counter-example of what you *don't* want
- Break the task into smaller sub-prompts

### Control Output Length

```
Answer in exactly 3 sentences.
Answer in under 50 words.
Provide a detailed explanation with at least 5 points.
```

### Avoid Negative Instructions Where Possible

| Less Effective | More Effective |
|---|---|
| "Don't be vague" | "Be specific and use concrete numbers or examples" |
| "Don't use jargon" | "Use plain English suitable for a non-technical audience" |

### Temperature and Sampling (API-level Tips)

| Parameter | Low Value | High Value |
|---|---|---|
| `temperature` | Deterministic, focused | Creative, varied |
| `top_p` | Narrow vocabulary | Broader word choice |
| `max_tokens` | Short responses | Longer responses |

---

## 5. Context Engineering with System Prompts

**Context Engineering** is the practice of strategically designing and managing the full context window — including the system prompt, conversation history, retrieved documents, and tool outputs — to maximally guide LLM behavior.

### The Context Window

```
┌────────────────────────────────────────────────────────┐
│                    CONTEXT WINDOW                      │
│  ┌──────────────────────────────────────────────────┐  │
│  │  SYSTEM PROMPT                                   │  │
│  │  Role, rules, persona, output format, tone       │  │
│  ├──────────────────────────────────────────────────┤  │
│  │  RETRIEVED CONTEXT (RAG)                         │  │
│  │  Relevant documents, policies, knowledge base    │  │
│  ├──────────────────────────────────────────────────┤  │
│  │  CONVERSATION HISTORY                            │  │
│  │  Previous turns (user + assistant)               │  │
│  ├──────────────────────────────────────────────────┤  │
│  │  TOOL OUTPUTS                                    │  │
│  │  Results from function calls, APIs, search       │  │
│  ├──────────────────────────────────────────────────┤  │
│  │  CURRENT USER MESSAGE                            │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘
```

### The System Prompt

The **system prompt** is a special instruction block set before any user interaction. It defines the model's **identity**, **behavior**, **constraints**, and **output style** for the entire session.

#### System Prompt Template

```
You are [ROLE/PERSONA].

## Objective
[What this assistant is designed to do]

## Capabilities
- [Capability 1]
- [Capability 2]

## Constraints
- [What the assistant must NOT do]
- [Boundaries, e.g., "Do not discuss competitor products"]

## Output Format
[Specify structure: bullet points, JSON, markdown, etc.]

## Tone
[Formal / Friendly / Technical / Concise]
```

---

#### Example — Customer Support Agent

```
You are Aria, a customer support specialist for TechNova Electronics.

## Objective
Help customers resolve product issues, process returns, and answer product questions.

## Capabilities
- Troubleshoot device issues using the knowledge base
- Initiate return and refund requests
- Escalate complex cases to a human agent

## Constraints
- Do not discuss competitor products
- Do not make promises about refund timelines not listed in policy
- Do not share internal system information

## Output Format
- Use clear, numbered steps for troubleshooting
- Keep responses under 150 words unless a detailed explanation is required

## Tone
Friendly, patient, and professional
```

---

#### Example — Code Review Assistant

```
You are an expert software engineer performing code reviews.

## Objective
Review submitted code for correctness, security vulnerabilities, performance issues,
and adherence to clean code principles.

## Review Criteria
1. Correctness — Does the code do what is intended?
2. Security — Are there any OWASP Top 10 vulnerabilities?
3. Performance — Are there unnecessary loops, N+1 queries, or memory leaks?
4. Readability — Is the code self-documenting and well-structured?

## Output Format
Return your review as:
- **Summary**: One sentence overall assessment
- **Issues**: Bulleted list with severity (Critical / Major / Minor) and line references
- **Suggestions**: Concrete improvements with code snippets where helpful

## Tone
Technical and precise. Do not praise; focus on issues and improvements.
```

---

### Context Engineering Best Practices

| Practice | Description |
|---|---|
| **Front-load critical instructions** | Place the most important rules at the top of the system prompt — models weight earlier context more heavily |
| **Be explicit about persona** | Name, role, and expertise level all influence output style |
| **Set hard constraints** | Explicitly state what the model must never do |
| **Define output schema** | Pre-defining format (JSON, markdown, table) improves consistency |
| **Manage context length** | Compress or summarize old history to stay within the context window |
| **Inject relevant context dynamically** | Use RAG to fill the context with documents relevant to the user's query |
| **Separate concerns with delimiters** | Use XML-like tags (`<policy>`, `<examples>`, `<user_query>`) to structure multi-part prompts |

### Context Engineering vs. Prompt Engineering

| Aspect | Prompt Engineering | Context Engineering |
|---|---|---|
| **Scope** | A single prompt or message | The full context window across a session |
| **Focus** | Instruction wording and structure | What information to include, where, and when |
| **Techniques** | Zero-shot, few-shot, CoT, ReAct | RAG, memory management, system prompt design |
| **Goal** | Get the right output for one query | Shape model behavior reliably across all queries |

---

## Summary

```
LLM  →  Trained on vast text data; generates tokens based on context

Prompt  →  Your instruction to the LLM; shapes every output

Zero-Shot   →  No examples; rely on model knowledge
Few-Shot    →  Provide examples; guide format and reasoning
CoT         →  Step-by-step reasoning; improves complex tasks
ReAct       →  Thought + Action + Observation loops; enables tool use

Tips        →  Be specific, assign roles, define format, iterate

Context Engineering  →  Manage the full context window strategically
System Prompt        →  Define persona, rules, constraints, and format
```

---

*Happy Prompting!*
