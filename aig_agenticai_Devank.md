# lecture notes

---

## traditional NLP → transformers

early NLP = hand-written grammar rules + statistical methods. worked for simple stuff but real language is ambiguous, context-heavy, constantly changing — these systems couldn't handle it

2017 — google drops "Attention Is All You Need" — everything changes
- old models: one word at a time (sequential)
- transformers: many words in parallel → faster training, better at long-range relationships in text

after transformers:
- BERT → language understanding
- GPT → text generation

modern chatbots mostly decoder-style transformers — best at generating natural conversational text

---

## attention mechanism

core idea behind transformers

instead of left to right, every word gets compared with every other word — calculates how strongly they relate

eg: "The girl said she is a nurse" → model links "she" to "girl"

also handles ambiguous words:
- bank = financial institution OR side of a river
- attention figures out which one fits from context

---

## word embeddings / vector space

LLMs don't read words symbolically — convert them to mathematical vectors (points in high-dimensional space)

similar words cluster close together:
- king and queen
- doctor and nurse
- man and woman

model captures meaning + analogies mathematically. better embeddings = better reasoning

common embedding dimensions: 512 / 1024 / 4096 / 16000+  
higher dimensions = more precise clustering but more memory, GPU, storage, training complexity

---

## how LLMs learn — 2 phases

**phase 1: pretraining**
- massive internet-scale text
- learns grammar, sentence structure, reasoning patterns, general world knowledge, token prediction
- general purpose at this stage, no specialization
- basically passive observation

**phase 2: fine-tuning**
- further training on focused datasets
- eg: medical docs, legal records, customer support chats, company knowledge bases
- teaches domain-specific behavior

**catastrophic forgetting**
- big risk with full fine-tuning
- if you update all weights aggressively → model can lose previously learned knowledge while adapting to new data
- learns new domain, forgets old capabilities
- major engineering challenge in production

---

## transformer architecture types

**encoder-only** (eg: BERT)
- focused on understanding text
- good for: classification, sentiment analysis, semantic search, information extraction
- converts text into contextual vector representations

**decoder-only** (eg: GPT, Llama)
- predicts next token
- dominates conversational AI
- most chatbots use this

**encoder-decoder** (eg: BART)
- combines both
- maps one sequence to another
- ideal for: translation, summarization, sequence transformation

---

## why transformers beat RNNs and LSTMs

RNNs/LSTMs were sequential → two problems:
1. slow training
2. weak long-term memory — forgot info after just a few words

transformers fixed both — attention + parallel processing, handles entire token groups at once, maintains long-range context across massive text windows

---

## GPU and compute constraints

LLMs need serious hardware. even relatively small research fine-tuning can require multiple enterprise GPUs

eg from lecture: fine-tuning setup for recent-event data needed:
- 2x H100 GPUs
- 160 GB combined VRAM

that VRAM handles model weights, gradients, optimizer states, batch processing

full fine-tuning of large models → multiple A100s, massive VRAM, heavy optimizer memory. very expensive

---

## prompt engineering

small prompt changes = very different outputs. major skill in GenAI

**zero-shot** — no examples, just ask  
eg: "summarize this article"

**one-shot** — one example before the real task, teaches expected structure

**few-shot / n-shot** — multiple examples, model aligns to demonstrated pattern. examples don't even need to match the topic — structure itself teaches behavior

---

## prompt structure / pipeline

modern AI systems stack multiple layers:

**system prompt** — hidden developer instructions. controls safety rules, behavior limits, platform restrictions, moderation. highest priority

**developer prompt** → **user prompt** — priority flows downward, higher overrides lower

**user prompt** — direct user input, tone, formatting, roleplay, output language etc

**context** — supporting info injected into prompt: PDF text, chat history, retrieved docs, database results

**final question** — actual task to solve

model processes all layers together in one combined prompt

---

## context window limits

every model has a hard token limit — must fit system prompt + user prompt + context + generated response all together

exceed the limit → model starts dropping/truncating info → lost memory, broken reasoning, incomplete responses

context management is critical in production

eg context windows from lecture:

| model | context length |
|---|---|
| sonar-deep-research | 128k |
| sonar-reasoning-pro | 128k |
| sonar-pro | 200k |

larger windows need more compute, memory, cost

even with huge context windows — attention quality can degrade, important info gets lost, inference slows down. why chunking + RAG still matters

---

## why responses change every time

LLMs are probabilistic — not a fixed database. calculate probability distributions for next token at every step. token sampling changes between runs → same question, different wording each time. normal behavior

---

## API reality

most developers never run models locally. app sends prompt + model name + API key → actual model runs on centralized GPU servers elsewhere. app only communicates over network

---

## RAG (retrieval-augmented generation)

LLMs hallucinate when they lack reliable knowledge. static training memory isn't enough for private or constantly changing info. RAG fixes this without retraining

**RAG pipeline:**
```
user query
   ↓
retriever searches vector DB
   ↓
relevant chunks fetched
   ↓
context injected into prompt
   ↓
LLM generates response
```

full pipeline breakdown:
1. ingestion + chunking — large files split into paragraphs/sections/sliding windows, keeps token usage manageable
2. tokenization + embeddings — chunks converted to vectors via embedding model
3. vector DB storage — fast similarity search across massive datasets
4. query embedding — user query converted to vector using same embedding model
5. similarity search — query vector vs stored vectors, scores closer to 1 = stronger similarity
6. top-K retrieval — pick highest ranked chunks (eg top 5 or top 10)
7. context injection — retrieved chunks inserted into final prompt, LLM answers from that

**context vs RAG** — people confuse these:
- context = plain text inserted into prompt
- RAG = the entire retrieval pipeline that finds and injects that context  
not the same thing

**enterprise security benefit** — companies don't need to upload full confidential datasets to public models. host internal vector DBs, retrieve only relevant chunks, send minimal info to external API. reduces data exposure

popular vector DBs: Pinecone, Weaviate, ChromaDB, FAISS

---

## prompts with vs without context

**without context** — model relies only on pretrained knowledge + prompt instructions + conversation history. may hallucinate, responses can be generic

**with context:**
```python
system_prompt = f"""
relevant chunks for this query:
{context}

[rest of instructions]
"""
```
context variable comes from RAG pipeline / vector DB retrieval. improves factual accuracy, relevance, domain-specific responses, grounding

eg from lecture — same persona prompt, completely different output quality once context was added

---

## RAG vs fine-tuning — not competitors

both solve different problems, most production systems combine both

**use RAG for changing knowledge** — facts, reports, schedules, project updates, research papers. updating vector DB needs no retraining, instant

**use fine-tuning for stable behavior** — writing style, response structure, tone, formatting, behavioral personas. permanently modifies model behavior

lecture comparison (LLaMA 3.3 70B fine-tuned vs RAG-augmented):

| task type | better approach |
|---|---|
| knowledge-heavy queries | RAG |
| conversational quality | fine-tuning |
| concise responses | fine-tuning |

analogy from lecture:
- fine-tuning = teaching the student new behavior permanently
- RAG = giving the student an open book during the exam

---

## sparse data problem in vector search

pure vector similarity has a weakness — rare identifiers can disappear inside semantic averaging:
- employee IDs
- serial numbers
- unique names
- location codes

vector search may retrieve wrong chunks

**hybrid search** fixes this — combine semantic vector search + keyword matching. keyword retrieval ensures exact matches for critical identifiers. much more reliable

---

## synthetic data generation

building large training datasets manually = unrealistic. instead use powerful proprietary models to generate synthetic QA pairs automatically, then train smaller open-source models on that. heavily reduces data collection costs

---

## LoRA (low-rank adaptation)

full fine-tuning updates every parameter — needs enormous GPU + memory

LoRA instead:
- freezes original model weights
- inserts tiny trainable matrices into the network
- trains less than 1% of parameters

massively reduces hardware costs, preserves strong performance

higher LoRA rank = better adaptation but more compute

---

## quantization

compresses model weights to lower precision:
- 32-bit → 16-bit
- 16-bit → 4-bit

reduces VRAM, storage, inference costs. small accuracy tradeoff

useful for local LLMs, edge devices, cheaper deployment

---

## KAR (knowledge augmented response)

closely related to RAG

```
LLM + external knowledge = better responses
```

instead of only pretrained knowledge, model retrieves additional info before generating

---

## personalized conversations via fine-tuning

lecture discussed fine-tuning for different personality styles based on Big5 traits:
- openness
- agreeableness
- extraversion
- conscientiousness
- neuroticism

personality types: Creative Nurturer, Visionary Idealist, Traditional Caregiver, Rigid Instructor

dataset used: Big5 Chat dataset — ~100k records, personality traits at high/low levels, instruction + input + output format

```json
{
  "instruction": "You are a teacher with openness low",
  "input": "Should genetic engineering enhance intelligence?",
  "output": "We should be cautious about altering genetics..."
}
```

---

## hyperparameter tuning

**chunk size** — too small = loses context. too large = wastes tokens

**chunk overlap** — consecutive chunks share some text to avoid missing info near boundaries  
eg: chunk 1 → lines 1-100, chunk 2 → lines 90-190

**temperature** — controls randomness. lower = factual/predictable. higher = creative/random

---

## agentic AI / multi-agent systems

not magical autonomous intelligence — workflow systems

instead of one giant prompt, work gets split into multiple specialized AI workers

agent can: reason, plan, use tools, execute multiple steps, interact with external systems

eg:
```
user asks for restaurant recs
   ↓
agent searches web
   ↓
collects reviews + location data
   ↓
summarizes
   ↓
final answer
```

**single LLM** fine for simple Q&A, summarization, code gen — not great for complex workflows, multi-step reasoning, tool usage, task delegation

**multi-agent** — modular, scalable, each agent owns one responsibility

---

## CrewAI

framework for building multi-agent systems. defines agents, tasks, tools, execution flow

each agent has: role, goal, backstory, tools, LLM config

```python
news_researcher = Agent(
    role='Situational Expert',
    goal='Provide accurate information',
    tools=[custom_google_search_tool]
)
```

backstory influences response style, reasoning, tone — basically advanced prompt engineering

---

## agent roles (example workflow)

**research agent** — gathers raw info using search APIs, databases, retrieval systems. LLM doesn't browse automatically, it triggers external tools

**writer agent** — converts raw research into summaries, bullet points, reports

**reviewer agent** — critiques output, checks clarity, formatting, accuracy, rule compliance

**action agent** — interacts with external APIs: sending emails, updating databases, triggering workflows

---

## tools in agentic AI

without tools → only training data  
with tools → can interact with real world

examples: google search, APIs, databases, calculators, SQL execution, web scraping

---

## sequential vs parallel execution

sequential:
```
research agent → writer agent → final output
```
one agent's output = next agent's input

parallel — some tasks run simultaneously eg two research agents scraping different sources, merge downstream. improves speed

---

## caching

reuse stored outputs instead of repeating expensive API calls. reduces costs

---

## safety / guardrails

autonomous systems need strict restrictions. never give unrestricted access to:
- financial systems
- payment methods
- sensitive credentials
- auth tokens

bad prompt loop can trigger repeated transactions or runaway automation fast. production systems always need monitoring + safety controls

---

## practical observation

modern AI systems usually combine all of this:
- RAG
- LoRA fine-tuning
- prompt engineering
- vector databases
- quantization

each technique improves a different part of the system
