# Onboarding Task -- Lecture Notes 

## How LLMs Generate Text

LLMs basically work by predicting the next most probable token.

Example:

Input:
```txt
The sky is
```

Possible predictions:
- blue
- clear
- beautiful

The model assigns probabilities to each token and picks one based on likelihood.

This happens repeatedly, one token at a time.

So technically, ChatGPT is just a very advanced next-word predictor.

Basically this means the model is not actually "thinking" like humans, it is predicting language patterns extremely well.

---

## Tokens

Models don't directly read words.

Text first gets broken into tokens.

Example:
```txt
"I love AI"
```

might become:
```txt
["I", "love", "AI"]
```

Sometimes even parts of words are tokens depending on the tokenizer being used.

---

## Pre-Transformer NLP 

Before transformers, people mainly used:
- RNNs
- LSTMs

Main problems:
- slow training
- hard to parallelize
- poor long-range memory

Transformers solved this using attention.

Paper:
"Attention Is All You Need" (2017)

This paper basically changed modern NLP completely.

---

## Transformer Architecture

Original transformer architecture has:
- Encoder
- Decoder

### Encoders
Mainly used for understanding context.

Example:
- BERT

They can look at both left and right context.

### Decoders
Mainly used for text generation.

Example:
- GPT models

Decoder predicts tokens sequentially.

Modern LLMs are mostly decoder-based.

---

## Attention

Attention is the core idea behind transformers.

The model learns which words are important relative to other words.

Example:
```txt
"The animal didn’t cross the road because it was tired"
```

Attention helps the model understand that "it" refers to "animal".

This was probably one of the most important concepts from the lecture.

---

## Self Attention

In self-attention, every word looks at every other word.

The model calculates relationships between tokens.

Important concepts:
- Query
- Key
- Value (QKV)

Very simplified idea:
- Query = what am I looking for?
- Key = what information do I have?
- Value = actual information passed forward

At first QKV sounded confusing, but it is basically a mechanism to measure token relevance.

---

## Word Embeddings / Vectors

Words are converted into vectors (lists of numbers).

Example:
```txt
dog -> [0.21, -0.44, ...]
```

Similar words tend to have vectors close to each other.

Examples:
- king and queen
- apple and banana

Dimensions basically represent learned features of language.

This is how models mathematically represent meaning.

---

## Positional Encoding

Attention alone doesn't understand word order.

Example:
```txt
dog bites man
```

vs

```txt
man bites dog
```

same words, completely different meaning.

So positional information is added separately.

---

## Masked Language Modeling (MLM)

Used mainly in BERT.

Some words are hidden and model predicts them.

Example:
```txt
"The cat sat on the [MASK]"
```

Prediction:
```txt
mat
```

This helps model learn context from both directions.

Unlike GPT style models, MLM models can see both left and right context during training.

---

## N-shot Learning

Model learns patterns from examples inside the prompt.

### Zero-shot
No examples are given.

Example:
```txt
Translate "Hello" to French
```

The model directly answers:
```txt
Bonjour
```

---

### One-shot
One example is provided before asking the actual question.

Example:
```txt
Dog -> Perro
Cat -> ?
```

Model understands the pattern and answers:
```txt
Gato
```

---

### Few-shot / N-shot
Multiple examples are provided.

Example:
```txt
Red -> Rouge
Blue -> Bleu
Green -> ?
```

The model infers the pattern and answers:
```txt
Vert
```

This is useful because the model can adapt to tasks without retraining.

Few-shot prompting is actually used a lot in real GenAI applications.

# RAG, Prompting and Context Windows

## RAG (Retrieval Augmented Generation)

RAG is a technique where the LLM is given external information before generating a response.

Instead of relying only on what it learned during training, the model can retrieve fresh or private data.

Basic flow:
```txt
User Query
   ↓
Retriever searches vector database
   ↓
Relevant chunks are fetched
   ↓
Context is added to prompt
   ↓
LLM generates response
```

Suppose a company has internal documentation.

Without RAG:
- model may hallucinate
- model won't know company-specific info

With RAG:
- relevant docs are fetched
- model answers using actual company data

---

## Why RAG is Useful

Main advantages:
- access to latest information
- reduces hallucinations
- works with private company data
- cheaper than retraining/fine tuning

Common use cases:
- company chatbots
- document QA systems
- customer support
- AI search systems

In most real-world systems today, RAG is extremely common.

---

## Vector Databases in RAG

In most RAG systems:
- documents are converted into embeddings/vectors
- stored inside vector databases/vector stores

Basic pipeline:
```txt
Documents
   ↓
Convert to embeddings
   ↓
Store vectors in vector DB
   ↓
User query converted to embedding
   ↓
Similarity search performed
   ↓
Most relevant chunks retrieved
```

The vector database retrieves documents that are semantically similar instead of exact keyword matches.

Example:

A search for:
```txt
"How do transformers work?"
```

may also retrieve:
```txt
"Explanation of attention mechanism in LLMs"
```

even if the wording is different.

This is basically semantic similarity search using embeddings.

Popular vector DBs:
- Pinecone
- Weaviate
- ChromaDB
- FAISS

---

## Search QA Pipeline

The lecture also showed how RAG-based search QA systems work.

### Step 1: Build Vector Store
- collect documents
- generate embeddings
- store embeddings in FAISS/vector DB

### Step 2: Query Search
- user query is converted into embedding
- similarity search retrieves closest documents

### Step 3: Pass Retrieved Context to LLM
- retrieved documents are attached to prompt
- LLM generates final response using that context

This is why RAG systems can answer questions from PDFs or internal company docs.

---

## Context Length

Context length = amount of text/tokens a model can remember in one interaction.

Usually measured in tokens.

Example:
```txt
128k context window
```

means the model can process around 128k tokens at once.

Larger context windows help with:
- long conversations
- large PDFs
- codebases
- RAG systems

---

## Context Windows

Some models support different context windows.

Example from lecture:

| Model | Context Length |
|---|---|
| sonar-deep-research | 128k |
| sonar-reasoning-pro | 128k |
| sonar-pro | 200k |

Larger context windows usually require:
- more compute
- more memory
- higher cost

---

## Problem with Long Contexts

Even with huge context windows:
- model attention quality may degrade
- important info can get lost
- inference becomes slower

This is why chunking + RAG is still important.

---

## Hardcoded Prompts

Hardcoded prompts are prompts written directly inside the code.

Example:
```python
prompt = "You are a helpful AI assistant"
```

or

```python
system_prompt = """
You are an expert coding assistant.
Answer only technical questions.
"""
```

These prompts define:
- AI personality
- tone
- response style
- restrictions
- task behavior

---

## Why Prompt Engineering Matters

Small changes in prompts can heavily affect outputs.

Prompt decides:
- response format
- creativity
- conciseness
- reasoning style
- safety behavior

Example:

Prompt 1:
```txt
Explain transformers
```

Prompt 2:
```txt
Explain transformers like I am a beginner with examples
```

Same model, very different outputs.

This is why prompt engineering became a major skill in GenAI.

---

## Writing Better Prompts

Good prompts are usually:
- clear
- specific
- constraint based
- format aware

Bad prompt:
```txt
Tell me about AI
```

Better prompt:
```txt
Explain transformer architecture in simple terms with examples and keep it under 300 words
```

More context generally gives better outputs.

---

## Prompt Components

Prompts often contain:
- role
- instructions
- context
- examples
- output format

Example:
```txt
You are a senior software engineer.
Explain APIs to a beginner.
Use simple language and examples.
```

---

## System Prompt vs User Prompt

Modern chat models usually work with multiple prompt layers.

Mainly:
- system prompt
- user prompt

### System Prompt

Defines overall model behavior.

Usually hidden from user.

Controls:
- personality
- rules
- restrictions
- response style

Example:
```txt
You are a professional coding assistant.
Always answer concisely.
Do not generate harmful code.
```

System prompt has higher priority.

### User Prompt

Actual input from the user.

Example:
```txt
Explain REST APIs
```

The model combines:
- system instructions
- user request
- conversation history

before generating output.

Usually priority works like this:

```txt
System Prompt
   ↓
Developer Prompt
   ↓
User Prompt
```

Higher level instructions generally override lower ones.

---

## Example from Lecture

The notebook example shown in lecture used hardcoded prompts like:

```python
prompt = """
Assume the persona of Lord Krishna...
Reply in English only...
"""
```

This is an example of:
- persona prompting
- instruction prompting
- behavioral conditioning

The prompt strongly influences:
- tone
- vocabulary
- speaking style
- response structure

LLMs are actually extremely sensitive to prompting.

---

## Fine Tuning

Fine tuning means training the model further on domain-specific data.

The model is additionally trained on specialized data so that it behaves better for specific tasks/domains.

Can improve:
- conversational quality
- response formatting
- tone/style consistency
- task specialization

Examples:
- medical assistant
- legal chatbot
- coding assistant

Fine tuning is especially useful when:
- behavior consistency matters
- response style matters
- repeated task patterns exist

Example:
A customer support AI can be fine tuned to:
- sound professional
- give shorter replies
- follow company tone guidelines

---

## Comparison from Lecture

The lecture compared:
- a fine tuned LLaMA 3.3 70B model
- a RAG augmented model

Results showed:

| Task Type | Better Approach |
|---|---|
| Knowledge-heavy queries | RAG |
| Conversational quality | Fine tuning |
| Concise responses | Fine tuning |

---

## Observation

RAG performed much better for factual/knowledge retrieval tasks because:
- it can access external documents
- information stays updated
- model does not rely only on training data

Fine tuned models performed better in:
- conversational flow
- response style
- concise answers

because the behavior itself was trained into the model.

---

## Key Difference

### Fine Tuning
Changes the model itself.

### RAG
Changes the information provided to the model.

---

## Simple Analogy

Fine tuning:
```txt
Teaching the student new behavior permanently
```

RAG:
```txt
Giving the student an open book during the exam
```

---

## Important Practical Insight

In real-world AI systems, both are often combined.

Example:
- Fine tune model for company tone/style
- Use RAG for company knowledge base

This gives:
- better conversational quality
- updated information
- lower hallucinations
- more reliable outputs

## Prompts With and Without Context

The lecture also compared prompts with context vs prompts without context.

This is a very important idea in RAG systems and modern AI applications.

---

## Prompt Without Context

In this approach, the model only receives:
- system instructions
- user query
- conversation history

Example:
```python
system_prompt = f"""
These are conversations between user1 and user2.

Assume the persona of Lord Krishna...
"""
```

Here, the model relies only on:
- pretrained knowledge
- prompt instructions
- memory from conversation

Problem:
- model may hallucinate
- responses may become generic
- model may not know domain-specific information

---

## Prompt With Context

In this approach, external information is added into the prompt before generation.

Example:
```python
system_prompt = f"""
These are the relevant chunks of text for this query:
{context}

Assume the persona of Lord Krishna...
"""
```

The variable:
```python
{context}
```

usually comes from:
- RAG pipeline
- vector database retrieval
- document search

This gives the model additional information while answering.

---

## Why Context Helps

Adding context improves:
- factual accuracy
- relevance
- domain-specific responses
- grounding

The model now answers using retrieved information instead of relying purely on memory.

This is one of the core ideas behind RAG.

---

## Simple Example

### Without Context

Prompt:
```txt
Explain company leave policy
```

Possible issue:
- model may generate generic HR policy

---

### With Context

Prompt becomes:
```txt
Relevant company policy:
Employees receive 20 paid leaves annually...

Now explain company leave policy.
```

Now the answer becomes:
- more accurate
- company specific
- less hallucinated

---

## Important Observation

The lecture examples showed that even with the same system prompt/persona:
- adding context changes output quality significantly

This is because the model now has access to external grounded information.

So in most production AI systems today:
- prompts are usually combined with retrieved context
- especially in RAG based applications

# KAR (Knowledge Augmented Response)

KAR is an approach where LLMs are improved using external knowledge and retrieval systems.

Main idea:
```txt
LLM + External Knowledge = Better Responses
```

Instead of relying only on pretrained knowledge, the model retrieves additional information before generating responses.

This is closely related to RAG systems.

---

## Personalized Conversations using Fine Tuning

The lecture discussed fine tuning models for different personality styles.

Examples:
- Creative Nurturer
- Visionary Idealist
- Traditional Caregiver
- Rigid Instructor

These personalities were based on Big5 personality traits:
- openness
- agreeableness
- extraversion
- conscientiousness
- neuroticism

The goal was to make responses feel stylistically different depending on personality type.

---

## Big5 Chat Dataset

A personality-based dataset was used for fine tuning.

Dataset contained:
- around 100k records
- personality traits with high/low levels
- instruction + input + output format

Example:
```json
{
  "instruction": "You are a teacher with openness low",
  "input": "Should genetic engineering enhance intelligence?",
  "output": "We should be cautious about altering genetics..."
}
```

This helps train different conversational styles and tones.

---

## LoRA (Low Rank Adaptation)

LoRA is a parameter-efficient fine tuning method.

Instead of retraining the full LLM:
- only smaller adapter layers are trained

Advantages:
- lower GPU usage
- lower memory usage
- cheaper training
- faster fine tuning

Higher LoRA rank:
- better adaptation
- but more compute required

---

## Hyperparameter Tuning

Some important LLM/RAG hyperparameters:

### Chunk Size
Documents are split into chunks before storing in vector DBs.

Too small:
- loses context

Too large:
- wastes tokens

---

### Chunk Overlap

Consecutive chunks share some text.

Example:
```txt
Chunk 1 -> lines 1-100
Chunk 2 -> lines 90-190
```

Helps avoid missing information near chunk boundaries.

---

### Temperature

Controls randomness in generation.

Lower temperature:
- more factual/predictable

Higher temperature:
- more creative/random

---



## Quantization

Quantization reduces model precision to make models smaller and faster.

Advantages:
- smaller model size
- faster inference
- easier deployment
- lower memory usage

Very useful for:
- local LLMs
- edge devices
- cheaper deployment

---

## Practical Observation

Modern AI systems usually combine:
- RAG
- LoRA fine tuning
- prompt engineering
- vector databases
- quantization

because each technique improves different parts of the system.

---
# Agentic AI and Multi-Agent Systems

## What is Agentic AI?

Agentic AI refers to AI systems that can:
- reason
- plan
- use tools
- perform tasks autonomously
- interact with external systems

Instead of a single prompt-response interaction, agentic systems can execute multiple steps to complete tasks.

Example:
```txt
User asks for restaurant recommendations
   ↓
Agent searches web
   ↓
Collects reviews/location data
   ↓
Summarizes results
   ↓
Returns final answer
```

---

## Single LLM vs Multi-Agent Systems

### Single LLM Workflow

Traditional LLM interaction:
```txt
User Prompt → LLM → Response
```

Good for:
- simple Q&A
- summarization
- code generation

But not ideal for:
- complex workflows
- multi-step reasoning
- tool usage
- task delegation

---

### Multi-Agent Systems

In multi-agent systems:
- multiple specialized agents work together
- each agent handles a specific responsibility

Example:
- Research Agent
- Writer Agent
- Planner Agent
- Reviewer Agent

This makes the workflow more modular and scalable.

---

## CrewAI

CrewAI is a framework used to build multi-agent AI systems.

It helps define:
- agents
- tasks
- tools
- execution flow

Main idea:
```txt
Multiple AI agents collaborating together
```

The lecture mainly demonstrated:
- agent creation
- task assignment
- tool integration
- sequential execution

---

## Setting Up Agents

Each agent usually contains:
- role
- goal
- backstory
- tools
- LLM configuration

Example from lecture:

```python
news_researcher = Agent(
    role='Situational Expert',
    goal='Provide accurate information',
    tools=[custom_google_search_tool]
)
```

Different agents can have different personalities and responsibilities.

---





## Agent Backstory

Agents can also have backstories/personas.

Example:
```python
backstory = """
You are a resourceful expert skilled at gathering information.
"""
```

Backstories influence:
- response style
- reasoning behavior
- communication tone

Very similar to advanced prompt engineering.

---

## Tools in Agentic AI

Agents become more powerful when connected to tools.

Example tools:
- Google Search
- APIs
- databases
- calculators
- SQL execution
- web scraping

Without tools:
- LLM only relies on training data

With tools:
- system can interact with real-world information

---


## Tasks in CrewAI

Tasks define what each agent should do.

Example:
```python
research_task = Task(
    description="Research latest information about topic"
)
```

Tasks usually contain:
- instructions
- expected output
- assigned agent

---

## Sequential Task Execution

Tasks can run in sequence.

Example:
```txt
Research Agent
      ↓
Writer Agent
      ↓
Final Output
```

One agent’s output becomes another agent’s input.

This creates workflow automation.



---

## Important Observation

Agentic AI is not just:
```txt
Prompt → Response
```

It combines:
- reasoning
- memory
- tools
- planning
- multiple agents
- task orchestration

to solve more complex problems.

---


