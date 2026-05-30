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

# Lecture 5
Large Language Models (LLMs) possess extensive pre-trained knowledge, but they may lack information about:

- Proprietary or private documents
- Newly published information
- Domain-specific knowledge bases
- Large collections of documents that exceed the model's training scope

Two common approaches to address this limitation are:

- Fine-Tuning
- Retrieval-Augmented Generation (RAG)
## Retrival Augmented Generation (RAG)
Retrieval-Augmented Generation (RAG) is a framework in which a language model retrieves relevant information from an external knowledge source and uses that information while generating a response.
Instead of answering solely from its internal parameters ("memory"), the model answers using:

User Query + Retrieved Context + Language Model

This enables the model to access knowledge that was not part of its original training data.
### Components of RAG 
- Language Model (LLM)
Responsible for generating the final response.
Examples:GPT family,Llama,Flan-T5,Mistral
- Embedding Model
Converts text into numerical vector representations.
Examples:
OpenAI Embeddings,BGE,E5,Flan-UL2 embeddings
- Vector Database
Stores vector representations of documents and enables efficient similarity search.
Examples:
Faiss,ChromaDB,Pinecone,Weaviate
- Retrieval/Search Method
Used to identify the most relevant document chunks.
Common methods:
Similarity Search,Maximal Marginal Relevance (MMR)
- Retrieval Chain
Determines how retrieved documents are processed before being sent to the LLM.
Examples:Stuff,MapReduce,Refine,MapRerank
### RAG Architecture
Documents
    → Chunking
    → Embedding Generation
    → Vector Database Storage

User Query
    → Query Embedding
    → Similarity Search
    → Top-K Retrieval
    → Context Construction
    → LLM
    → Generated Answer

## Text → Vector Conversion

```text
Text
→ Tokens
→ Token IDs
→ Embeddings (Vectors)
```

Example:

```text
"punctuation"
→ ["punct", "uation"]
→ [1524, 931]
→ Vector
```

Semantically similar words have nearby vectors.

Example:

```text
moksha ≈ liberation ≈ spiritual freedom
```

---

## RAG Pipeline

### Indexing Phase (One-Time)

```text
Documents
→ Chunking
→ Embedding Generation
→ Vector Database
```

### Query Phase

```text
User Query
→ Query Embedding
→ Similarity Search
→ Top-K Retrieval
→ Context Construction
→ LLM
→ Generated Answer
```

---

## Similarity Search

The query and document chunks are compared in vector space.

Common metric:

- **Cosine Similarity**

```text
1.0 → Highly Similar
0.0 → Unrelated
```

---

## Top-K Retrieval

Many chunks may be relevant.

Process:

```text
Rank Chunks
→ Select Top K
→ Send to LLM
```

Typical values:

```text
K = 5, 10, 20
```

Constraint:

```text
K × Chunk Size ≤ Context Window
```

---

## Metadata Filtering

Restricts search before retrieval.

Examples:

- Chapter Number
- Document Name
- Date
- Category

Example:

```text
Search only within Chapter 18
```

---

## Retrieval Strategies

| Strategy | Description |
|-----------|------------|
| Stuff | Put all chunks directly into prompt |
| MapReduce | Process chunks separately, then combine |
| Refine | Sequentially improve answer |
| MapRerank | Generate multiple answers and choose best |

**Stuff is the most commonly used approach.**

---

## Fine-Tuning vs RAG

| Fine-Tuning | RAG |
|------------|-----|
| Updates model weights | No model changes |
| Permanent knowledge | Temporary context |
| Requires retraining | No retraining |
| Harder to update | Easy to update |

---

## Privacy Advantage

Private documents remain in the organisation's infrastructure.

Only relevant chunks are retrieved and sent during inference.

---

## Limitation

RAG can only answer using information present in the indexed documents.

**Poor documents → Poor retrieval → Poor answers**

---
# Lecture 6: Fine-Tuning, LoRA & Evaluation

## Why Fine-Tuning?

RAG can provide external knowledge, but it **cannot change model behavior**.

Fine-tuning is used to modify:

- Tone and communication style
- Persona (teacher, assistant, customer support, etc.)
- Output formatting
- Domain-specific reasoning patterns

### Real-World Architecture

```text
RAG → Retrieves factual knowledge
Fine-Tuning → Controls model behavior/personality
```

Example:

```text
Course PDFs/Notes → RAG
Teaching Style → Fine-Tuning
```

---

# Limitation of RAG

Standard RAG relies on vector similarity.

Problem:

A name may appear only once in a document and later be referenced using pronouns.

Example:

```text
"Narendra Modi" appears once
Later: "he", "the Prime Minister"
```

A vector search on "Narendra Modi" may fail to retrieve relevant chunks.

---

# Keyword Augmented Retrieval (KAR)

KAR uses keyword matching instead of relying solely on vector similarity.

Process:

```text
Query
→ Extract Keywords
→ Match Against Document Keywords
→ Retrieve Relevant Chunks
```

Works well for:

- Proper nouns
- Names
- Sparse datasets
- Rare keywords

---

## RAG vs KAR

| RAG (Vector Search) | KAR (Keyword Search) |
|---------------------|----------------------|
| Semantic similarity | Exact keyword matching |
| Works on dense text | Works on sparse text |
| Handles related concepts | Handles names/entities |
| Can miss rare terms | Can miss semantic relations |

### Best Practice

```text
Hybrid Search = RAG + KAR
```

Use vector similarity and keyword matching together.

---

# Fine-Tuning Data Format

Fine-tuning requires:

```text
Question → Answer
```

Raw PDFs or articles cannot be used directly.

Example:

```text
Q: What is LoRA?
A: A parameter-efficient fine-tuning method.
```

Thousands of such examples are needed.

---

# Synthetic Data Generation

Manually creating large datasets is expensive.

Solution:

```text
Large Model (GPT-4)
→ Generate Q&A Pairs
→ Fine-Tune Smaller Model
```

Benefits:

- Faster dataset creation
- Lower cost
- Smaller models can mimic larger ones

---

# Full Fine-Tuning

Updates every parameter in the model.

```text
Data
→ Train All Weights
→ Updated Model
```

### Advantages

- Maximum adaptability
- Best possible performance

### Disadvantages

- Extremely expensive
- Large memory requirements
- Risk of catastrophic forgetting

---

# Catastrophic Forgetting

During fine-tuning, the model may overwrite previously learned knowledge.

```text
Old Knowledge + New Training
→ Old Knowledge Lost
```

More common in full fine-tuning.

---

# LoRA (Low-Rank Adaptation)

Most popular parameter-efficient fine-tuning method.

Core idea:

```text
Freeze Original Model
+
Train Small Adapter Layers
```

Instead of updating billions of parameters, LoRA updates less than 1%.

---

## LoRA vs Full Fine-Tuning

| Full Fine-Tuning | LoRA |
|------------------|-------|
| Updates all parameters | Updates adapters only |
| High memory usage | Low memory usage |
| Catastrophic forgetting risk | Much lower risk |
| Expensive | Cheap |
| Best quality | Near-equivalent quality |

---

## LoRA Rank (r)

Controls adapter size.

```text
Higher Rank
→ More trainable parameters
→ Better learning
→ More memory usage
```

Common values:

```text
r = 4, 8, 16
```

As:

```text
r → ∞
```

LoRA approaches full fine-tuning.

---

# Hardware Requirements

Memory per parameter:

```text
1 Parameter ≈ 4 Bytes
```

Example:

```text
1B Parameters ≈ 4 GB
70B Parameters ≈ 280 GB
```

Training additionally requires:

- Gradients
- Optimizer states
- Activations

Result:

```text
70B Model
≈ 1.5+ TB memory (full precision training)
```

---

# Quantization

Reduces precision of weights.

Example:

```text
32-bit → 16-bit → 8-bit → 4-bit
```

Benefits:

- Lower VRAM usage
- Faster training
- Cheaper deployment

Usually combined with LoRA.

---

# Evaluating LLMs

Evaluation is needed to verify improvements from RAG or Fine-Tuning.

## Common Metrics

| Metric | Purpose |
|----------|---------|
| BLEU | Translation quality |
| ROUGE | Summarization quality |
| Perplexity | Model confidence |
| F1 Score | QA accuracy |
| HELM | Holistic LLM evaluation |

---

## HELM (Stanford)

Holistic Evaluation of Language Models.

Measures:

- Accuracy
- Robustness
- Fairness
- Bias
- Toxicity
- Calibration

Provides a comprehensive benchmark for LLM quality.

---

# SFT vs RLHF

## Supervised Fine-Tuning (SFT)

Train directly on Q&A pairs.

```text
Question
→ Answer
→ Update Weights
```

Simple and commonly used.

---

## RLHF

Reinforcement Learning from Human Feedback.

Process:

```text
Human Preferences
→ Reward Model
→ PPO Optimization
→ Improved LLM
```

Humans rank responses instead of providing direct labels.

---

# Key Takeaways

```text
RAG
→ Adds external knowledge

KAR
→ Fixes sparse keyword retrieval problems

Fine-Tuning
→ Changes model behavior

LoRA
→ Efficient fine-tuning (<1% parameters)

Quantization
→ Reduces memory usage

HELM
→ Evaluates overall model quality

Best Real-World System
→ RAG + KAR + LoRA Fine-Tuning
```


# Lecture 8

## Why Generic LLMs Are Not Enough

- A pretrained LLM only knows what it has seen during training.
- It cannot answer questions about proprietary company data, internal documents, or domain-specific information that was never part of its training corpus.
- Two major solutions:
  - **RAG (Retrieval-Augmented Generation)**
  - **Fine-Tuning**
- RAG is usually the preferred starting point because it is much cheaper and easier to implement.

## RAG vs Fine-Tuning

### RAG

- Does not modify model weights.
- Retrieves relevant information during query time.
- Injects retrieved information into the prompt.
- Low computational cost.
- Best for factual knowledge retrieval.

### Fine-Tuning

- Updates model weights through additional training.
- Knowledge becomes part of the model.
- Requires significant computational resources.
- Best for changing behaviour, style, or domain expertise.

---

## Vector Database

- Stores information in two forms:
  - Original text chunk
  - Vector embedding
- Queries are also converted into vectors.
- Similarity search finds the closest chunk vectors.
- Retrieved text chunks are passed to the LLM.

---

## RAG Pipeline

### Indexing Phase (One-Time Process)

1. Load source documents.
2. Extract text.
3. Split text into chunks.
4. Generate embeddings for each chunk.
5. Store chunk text and embeddings in a vector database.
6. Extract and store keywords separately.

### Query Phase

1. Convert user query into an embedding.
2. Perform vector similarity search.
3. Perform keyword search.
4. Merge both result sets.
5. Pass retrieved chunks and query to the LLM.
6. Generate the final response.

---

## Hybrid Search

Hybrid search combines:

### Vector Retrieval

- Uses semantic similarity.
- Understands meaning and paraphrases.
- Example:
  - "car" ≈ "automobile"

### Keyword Retrieval

- Uses exact keyword matching.
- Useful for:
  - Names
  - Locations
  - IDs
  - Rare terms

### Merge Strategies

#### AND Merge

- Chunk must appear in both searches.
- Higher precision.
- Smaller result set.

#### OR Merge

- Chunk can appear in either search.
- Broader coverage.
- Larger result set.

---

## Why the Same Embedding Model Must Be Used

- Document chunks and user queries must exist in the same vector space.
- Using different embedding models makes similarity scores meaningless.
- The same embedding model must be used during:
  - Indexing
  - Retrieval

---

## Encoder vs Decoder Models

### Encoder Models

- Produce fixed-length vector representations.
- Used for embeddings.
- Examples:
  - BERT
  - Sentence Transformers
  - embedding-001

### Decoder Models

- Generate text token by token.
- Used for response generation.
- Examples:
  - GPT
  - Gemini
  - Claude
  - LLaMA

### In RAG

- Encoder → Creates embeddings.
- Decoder → Generates the final answer.

---

## Chunk Size

Chunk size determines how much text is stored in each chunk.

### Small Chunks

- Better retrieval precision.
- Less surrounding context.

### Large Chunks

- More context.
- Lower retrieval precision.

Typical chunk sizes:

- 1024 tokens
- 2048 tokens

---

## Top-K Retrieval

- Top-K = Number of chunks retrieved.
- Larger K provides more context.
- Too large a value introduces noise and may exceed the context window.

Constraint:

Top-K × Chunk Size ≤ Context Window

---

## LlamaIndex Components

### Embedding Model

- Converts text into vectors.

### VectorStoreIndex

- Stores chunk embeddings.

### KeywordTableIndex

- Stores chunk keywords.

### Custom Retriever

- Combines vector and keyword retrieval.

### Response Synthesizer

- Sends retrieved chunks and query to the LLM.

### Query Engine

- Connects retriever and synthesizer.

---

## RAG as a Simple Agent

- A RAG system already behaves like a basic AI agent.
- It interacts with an external knowledge source.
- Retrieves information.
- Uses that information to answer questions.

More advanced agents extend the same idea to:

- Web Search
- Databases
- Email APIs
- Calendar APIs
- External Services

RAG is the foundation of modern AI agent systems.
