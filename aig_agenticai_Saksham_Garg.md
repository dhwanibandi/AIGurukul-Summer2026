# Lecture 1
- The LLMs are the probalistic models working on the inputs given to them by predicting the answer based on highest probability according to different model paramters.
- Parameters- weights and coefficients of the model, and the number of parameters define the models capabilities.
- Pre-2017 we used text generation RNN basically predicting based on the previous words.
- Post-2017- It all changed with the Google- "Attention is all you need paper" - introducing Transformers Architecture.
- The sequential processing was replacced by parallel computing - leading to better context, scalability, and long term dependencies.
## How LLMs think
- Given a input the model calculates the probablities for all next words picks the next , they are trained on billions of data points.
- Take the pretrained model and train it further on specific, curated medical records, legal documents, customer support transcripts, whatever the use case demands. This is how you get specialised models like Google's Med-PaLM (fine-tuned on medical data).
## Data Representation
-Words are represented as n-dimensional vectors; similar words cluster together, opposites are farther apart.
- Each dimension captures a feature (e.g. "is it a pet?", "is it an animal?")
- Enables vector math: MAN − WOMAN + QUEEN = KING
## Attention Mechanism
- The transformer's attention mechanism lets every word look at every other word simultaneously. Not just the nearby ones — all of them. This allows the model to understand relationships across an entire passage at once.
Example 1 — Pronoun resolution:
"The girl said she is a nurse."
The model correctly learns that "she" refers to "girl" — not because of a rule we wrote, but because attention captures the dependency between those two words.
Example 2 — Word sense disambiguation:
"I went to the bank."
Financial bank or riverbank? Attention lets the model resolve this by looking at all surrounding words for context clues. Older models struggled badly with this.
Why Transformers Are So Powerful
Beyond context understanding, transformers also enable:
Parallelism — process all words at once instead of sequentially (much faster to train)
Scalability — works well as you add more data and more parameters
Long-term context — can handle thousands of tokens at once.
- "Word2Vec" was the algorithm which did this.
- LLMs are best used for reasoning, not knowledge retrieval but don't rely on them for factual accuracy, as training data may be incomplete or incorrect.
# Lecture 2
## Transformer Architecture
- Think of these like having a encoder and decoder -an encoder that reads and understands input text by converting it into rich vector representations, and a decoder that uses those representations to generate output. Simple formula: Transformer = Neural Network + Attention.
## Training techniques
Two main techniques are used:
#### Next-Token Prediction (GPT-style) 
-The model sees a partial sentence and must predict the next word. Wrong predictions are penalised via a loss function, weights get updated, and this repeats billions of times across the entire training corpus.
#### Masked Language Modelling (BERT-style)
- Random words in a sentence are hidden and the model must predict them using surrounding context - both left and right. This trains bidirectional understanding, which is why BERT is so good at comprehension tasks.

## Model Size vs. Training Quality
Generally, more parameters = better performance, but training quality and fine-tuning strategy matter just as much. A well-trained smaller model can outperform a poorly trained larger one. Measured by Likert scores, fine-tuned models show that size alone doesn't drive accuracy, how you train matters just as much.

## Prompt Engineering
The quality of the prompt defines the output drastically.
Zero-shot — No examples given. Just ask the question. The model figures it out from training alone.
One-shot — Give one labelled example before your actual query.
Few-shot — Give multiple examples before your query. Consistently the best approach — the model learns the pattern from the examples, even if they're not directly related to your actual question.
# Lecture 3
- A prompt is simply an instruction to an LLM. But there are two very different kinds, and understanding the difference matters:

#### System Prompt 
Set by the company or developer, hidden from the end user, and given higher priority than anything you type. This is how guardrails get enforced. DeepSeek refusing to answer sensitive questions? That's a system prompt at work. If you build on open-source models, you get full control over this.
#### User Prompt 
What we actually type. Lower priority than the system prompt, but this is where you control format, task instructions and tone.
## Context and RAG
Along with your prompt, you can pass external context to the model as uploaded PDFs, documents, conversation history, whatever is relevant. When you do this, the model answers from that material rather than its training memory. This is the foundation of RAG (Retrieval-Augmented Generation):

Instead of relying on what the model memorised during training, retrieve the relevant documents at query time and inject them as context.

This is how you'd build something like a course assistant that answers only from the textbook you provide. The benefit is significant — it dramatically reduces hallucinations and keeps responses grounded in real source material rather than the model making things up from statistical patterns.

# Lecture 4
## API Structure
- Every API call has two parts:

$Header$: contains the API key, sent separately from the data purely for authentication purposes

$Payload$: the actual content: system prompt, user prompt, user query, and model name
## LLMs Have No Memory — By Design

LLMs are inherently stateless. Every API call is a blank slate ,the model has no idea what you asked 30 seconds ago. For a chatbot to feel like a conversation, you have to:

Store every query and response in a backend database
Append that history as context on every new API call
 
You can get creative with this — store user preferences and inject them into every prompt, or use a second model to summarise past conversations and pass that summary as context instead of the full history. The memory is always external; the model itself never retains anything.

## RAG vs Fine-Tuning
#### RAG 
It is best Factual retrieval from fixed sources,works by pulling relevant text chunks and injects as context,PDFs, documents, knowledge bases,it is also cheap.
#### Fine-Tuning
It is best for changing model behaviour or personality,it works by actually modifies the model weights, it considers large datasets and so is comparitively has expensive compute.

# Lecture 5 - Retrieval Augmented Generation (RAG)

## Why RAG?

- LLMs cannot access:
  - Private company documents
  - Newly published information
  - Domain-specific knowledge bases
- Two major solutions:
  - RAG
  - Fine-Tuning

## What is RAG?

- RAG retrieves relevant information from an external knowledge source before generating an answer.
- Instead of relying only on model memory:

```text
User Query + Retrieved Context + LLM
```

## Core Components

### LLM
- Generates the final response.

### Embedding Model
- Converts text into vectors.
- Similar meanings produce similar vectors.

### Vector Database
- Stores:
  - Text chunks
  - Their embeddings

### Retriever
- Finds relevant chunks using similarity search.

## RAG Pipeline

### Indexing Phase

```text
Documents
→ Chunking
→ Embeddings
→ Vector Database
```

### Query Phase

```text
Query
→ Embedding
→ Similarity Search
→ Top-K Chunks
→ LLM
→ Answer
```

## Similarity Search

- Retrieval happens in vector space.
- Usually uses cosine similarity.

## Chunking

- Documents are split into chunks before indexing.

Small chunks:
- Better precision
- Less context

Large chunks:
- More context
- Lower precision

Typical sizes:
- 1024 tokens
- 2048 tokens

## Top-K Retrieval

- Retrieves the K most relevant chunks.

Constraint:

```text
Top-K × Chunk Size ≤ Context Window
```

## Retrieval Strategies

### Stuff
- Send all retrieved chunks directly.

### MapReduce
- Process chunks independently and combine results.

### Refine
- Improve answer iteratively.

### MapRerank
- Generate multiple answers and select the best.

## Fine-Tuning vs RAG

| Fine-Tuning | RAG |
|------------|-----|
| Changes model weights | No model changes |
| Permanent knowledge | Temporary context |
| Expensive | Cheap |
| Hard to update | Easy to update |
# Lecture 6 - Fine-Tuning, LoRA & Evaluation

## Why Fine-Tuning?

RAG provides knowledge but cannot change behaviour.

Fine-tuning changes:

- Tone
- Personality
- Output style
- Domain-specific reasoning

Typical production setup:

```text
Knowledge → RAG
Behaviour → Fine-Tuning
```

## Limitation of Pure RAG

Vector search struggles with:

- Names
- Proper nouns
- Rare entities
- Sparse documents

Example:

```text
Narendra Modi
↓
he
↓
the Prime Minister
```

Relevant chunks may be missed.

## Keyword Augmented Retrieval (KAR)

Uses keyword matching alongside vector search.

Best for:

- Names
- IDs
- Proper nouns
- Sparse datasets

### Best Practice

```text
Hybrid Search = RAG + KAR
```

## Fine-Tuning Data

Training requires:

```text
Question → Answer
```

not raw PDFs.

Example:

```text
Q: What is LoRA?
A: Parameter-efficient fine-tuning.
```

## Synthetic Data

Instead of writing datasets manually:

```text
Large Model
→ Generate Q&A Pairs
→ Fine-Tune Smaller Model
```

## Full Fine-Tuning

```text
Train All Parameters
```

Advantages:

- Highest flexibility

Disadvantages:

- Expensive
- Huge memory requirements
- Risk of catastrophic forgetting

## Catastrophic Forgetting

```text
New Training
→ Old Knowledge Lost
```

## LoRA

Low-Rank Adaptation.

Idea:

```text
Freeze Base Model
+
Train Small Adapters
```

Updates <1% of parameters.

## LoRA vs Full Fine-Tuning

| Full FT | LoRA |
|----------|------|
| All parameters | Adapters only |
| Expensive | Cheap |
| High memory | Low memory |
| Forgetting risk | Lower risk |

## Quantization

Reduce precision:

```text
32-bit
→ 16-bit
→ 8-bit
→ 4-bit
```

Benefits:

- Lower VRAM
- Faster training
- Cheaper deployment

Usually combined with LoRA.

## Evaluating Models

### BLEU
- Translation quality

### ROUGE
- Summarization quality

### Perplexity
- Model confidence

### F1
- QA accuracy

### HELM
- Overall LLM evaluation

Measures:

- Accuracy
- Robustness
- Fairness
- Bias
- Toxicity

## Training Approaches

### SFT

```text
Question
→ Answer
→ Train
```

### RLHF

```text
Human Preferences
→ Reward Model
→ Optimisation
```

# Lecture 8 - RAG Implementation & Hybrid Retrieval

## Building a Production RAG System

A practical RAG system contains:

- Embedding Model
- VectorStoreIndex
- KeywordTableIndex
- Hybrid Retriever
- Response Synthesizer
- Query Engine

## Hybrid Search

Combines:

### Vector Retrieval

- Semantic similarity
- Handles paraphrases

Example:

```text
car ≈ automobile
```

### Keyword Retrieval

- Exact matching
- Handles:
  - Names
  - Locations
  - IDs
  - Rare terms

### Merge Modes

#### AND

- Must appear in both searches.
- Higher precision.

#### OR

- Can appear in either search.
- Higher recall.

## Why the Same Embedding Model Matters

Documents and queries must live in the same vector space.

```text
Document Embeddings
=
Query Embeddings
```

Otherwise similarity search becomes meaningless.

## Encoder vs Decoder Models

### Encoder

- Creates embeddings.
- Produces fixed-length vectors.

Examples:

- BERT
- Sentence Transformers
- embedding-001

### Decoder

- Generates text.

Examples:

- GPT
- Gemini
- Claude
- LLaMA

## LlamaIndex Components

### VectorStoreIndex
- Stores embeddings.

### KeywordTableIndex
- Stores keywords.

### Custom Retriever
- Combines vector and keyword retrieval.

### Response Synthesizer
- Generates answers from retrieved chunks.

### Query Engine
- Connects everything together.

## RAG as an Agent

A RAG system already behaves like a simple agent:

```text
Query
→ Retrieve Information
→ Generate Answer
```

More advanced agents extend this idea using:

- Search APIs
- Databases
- Email APIs
- External services
# Lecture 9 - Agentic AI & CrewAI

## What is an Agent?

- An autonomous workflow that performs actions towards a goal.
- LLMs may be used in some steps, but not necessarily all.

Possible actions:

- API Calls
- Database Queries
- Python Functions
- External Services

## CrewAI

Framework for building multi-agent systems.

Main Components:

### Agent

Defined by:

- Role
- Goal
- Backstory
- Tools
- LLM

### Task

Defines:

- What to do
- Expected output
- Assigned agent

### Crew

Combines agents and tasks.

## Execution Modes

### Sequential

```text
Task A
→ Task B
→ Task C
```

### Parallel

```text
Task A
↘
  Merge
↗
Task B
```

## Example Pipeline

| Step | Agent | Tool |
|--------|--------|--------|
| Research | Researcher | Search API |
| Writing | Writer | LLM |
| Review | Reviewer | LLM |
| Rewrite | Rewriter | LLM |
| Send | Sender | Email API |

## Tools

Agents can use:

- Google Search
- Brave Search
- SQL Databases
- Vector Databases
- Email APIs
- Custom Python Functions

## RAG vs Agentic AI

| RAG | Agentic AI |
|------|-----------|
| Fixed knowledge | Live data |
| Single workflow | Multi-step workflow |
| Retrieval + generation | Planning + tools + generation |
| Lower complexity | Higher complexity |

## Caching

Store results from previous runs.

Benefits:

- Lower latency
- Lower API costs
- Faster execution

## Limitations

### No Memory

- Agents do not remember previous runs.

### Probabilistic Outputs

- Same task may produce different outputs.

### Automation Risks

Avoid autonomous control over:

- Payments
- Financial transactions
- Irreversible actions

Human approval should remain in the loop.

## Key Idea

Modern AI systems often combine:

```text
RAG
+
Agents
+
Tools
+
Fine-Tuning
```

to build production-grade applications.
