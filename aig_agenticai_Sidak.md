# Generative AI — Lecture Notes (Batch 1)

---

### Lecture 1 · Introduction to Generative AI & Large Language Models

#### What is Generative AI?

Generative AI refers to a class of AI systems that can produce new content — text, images, audio, video, and code — rather than just classifying or analysing existing data. LLMs like GPT-4, Claude, and LLaMA are at the centre of this space. Their use cases go well beyond chatbots: document summarisation, code generation, language translation, image synthesis (e.g., Stable Diffusion), music generation, and even protein sequence modelling in biology. 

#### A Brief History of LLMs

Before 2017, NLP relied on rule-based and statistical methods — RNNs, LSTMs, N-gram models — which processed language sequentially, struggled with long-range dependencies, and were expensive to scale. The turning point was the 2017 paper *"Attention Is All You Need"* (Vaswani et al., Google Brain), which introduced the Transformer architecture and replaced sequential processing with parallel, attention-based computation.

What followed was rapid growth:
- 2018: BERT (340M parameters) — Google's bidirectional encoder, open-sourced.
- 2019: GPT-2 (1.5B parameters) — OpenAI.
- 2020: GPT-3 (175B parameters), demonstrating emergent capabilities at scale.
- 2022: ChatGPT launches — the mainstream inflection point.
- 2023 onwards: GPT-4, Claude, Gemini, and a major open-source expansion (LLaMA, Mistral, Falcon).

#### How LLMs Actually Work — Next-Token Prediction

LLMs are probabilistic, not deterministic. Given an input sequence, the model assigns a probability distribution across its entire vocabulary (which can be 50,000–100,000+ tokens) and outputs the most likely next token. This repeats iteratively until a full response is built, one token at a time. The model is not searching the internet — every response comes from patterns baked into its weights during training. This is exactly why LLMs can produce confident, fluently written wrong answers: they are always picking the statistically likely next word, not checking facts.

Mathematically, this is similar to regression — fitting a probability distribution over a discrete vocabulary rather than predicting a continuous value.

Three key training objectives used across model families:
- *Next Token Prediction (Causal LM)* — predict the next word using only left context. Used in GPT-style models.
- *Masked Language Modeling (MLM)* — a word is masked and predicted using both left and right context. Used in BERT-style bidirectional models. BERT's performance measurably drops when MLM is removed.
- *Next Sentence Prediction (NSP)* — predict whether one sentence logically follows another. Used in BERT to learn cross-sentence relationships.

#### The Transformer Architecture

The Transformer drops sequential processing entirely in favour of a parallel, attention-based design. Its two main parts are the **encoder** (reads the input and produces contextual vectors for each token) and the **decoder** (generates output tokens by attending to those vectors and previously generated tokens). The **attention mechanism** is the core innovation — it lets every token attend to every other token in the sequence simultaneously, computing a weighted relevance score for each pair.

#### Pre-training and Fine-tuning

*Pre-training* exposes the model to enormous volumes of generic text — web pages, books, code — using self-supervised objectives. No task-specific labels are needed. The model learns grammar, world knowledge, and reasoning patterns broadly, producing a **foundation model**. Think of it like a child absorbing everything in their environment before formal schooling.

*Fine-tuning* continues training on a smaller, curated, domain-specific dataset to teach the model specialised knowledge or align it to a specific task. Example: PaLM-Med (Google) is a 540B parameter model fine-tuned on medical literature. One important risk is *catastrophic forgetting* — if fine-tuning data conflicts with pre-training data, the model can unlearn previously acquired knowledge.


#### Prompting and In-Context Learning

A prompt is the natural language instruction given to an LLM at inference time. Writing prompts that reliably produce the desired output is called *prompt engineering*. A key capability of LLMs is *in-context learning* — the ability to follow instructions, adopt personas, and perform structured tasks without any retraining, purely through the prompt. This distinguishes LLMs from traditional ML models, which needed task-specific training for every new task.

Prompting by example count:
- *Zero-shot* — instruction only, no examples. The model relies entirely on pre-trained knowledge.
- *One-shot* — one example provided alongside the instruction to guide format or style.
- *Few-shot* — a small set of examples. Significantly improves consistency and quality on structured tasks.

---

### Lecture 2 · Word Embeddings, Attention & Transformer Architecture

#### Fine-tuning in Practice

Fine-tuning starts from pre-trained weights and uses only a new domain-specific dataset — it does not involve mixing in the original pre-training data. The goal is to teach the model things it hasn't encountered, either because of a knowledge cutoff or because the domain is too specialised to have appeared in general training data.

#### The Attention Mechanism

Words don't carry fixed meanings — context determines what a word means. "Bank" means something completely different in "river bank" versus "bank robbery." A model processing words one at a time cannot reliably resolve this. Attention solves it by letting every token in a sequence attend to every other token simultaneously, computing a relevance weight for each pair. The model learns during training which contextual connections matter.

This is what sets Transformers apart from RNNs. RNNs compress all prior context into a single hidden state vector — a bottleneck that causes information about distant tokens to fade. Attention maintains direct connections across all token pairs, regardless of distance.

#### Word Embeddings

Computers can't process raw text directly. Words need to be converted into numerical vectors called **embeddings** — each word becomes a point in a high-dimensional space, where its position encodes meaning.

Key properties:
- Words with similar meanings cluster together in this space.
- Semantic relationships survive as vector arithmetic — the classic example: `king − man + woman ≈ queen`.
- The dimensions are not labelled human concepts. They're abstract coordinates whose meaning emerges from training.
- Typical sizes in modern models: 512, 4096, 8192 — almost always multiples of 512 for hardware efficiency.
- Higher dimensions capture richer representations but cost more in memory and compute.

#### Transformer Structure

The original 2017 Transformer uses 6 stacked encoder layers and 6 stacked decoder layers, each refining the representations from the layer before.

Each **encoder layer** has two parts: a multi-head self-attention layer (each token incorporates information from all others) and a feed-forward neural network (applies a non-linear transformation per token independently).

Each **decoder layer** has three parts: self-attention (attends to previously generated tokens), cross-attention (attends to the encoder's output, linking generated output back to input), and a feed-forward network.

A critical point for understanding modern LLMs: nearly all production models today — GPT, Claude, LLaMA, Gemini — are **decoder-only**. There is no encoder. The decoder handles both the input and the generation entirely through self-attention.

---

### Lecture 3 - API Setup & Prompt Engineering

#### Setting Up the Environment

The session used the **Perplexity Sonar Reasoning** model. Getting API access involved signing up at perplexity.ai, navigating to Settings → API, loading $5 in credits (enough for roughly six months of light use), and generating an API key. One useful property of Sonar Reasoning: it automatically applies chain-of-thought reasoning on every query without needing to be asked. Most LLMs require explicit prompting ("think step by step") to trigger this behaviour.

#### The Bhagwan Speaks Demo

The first notebook introduced prompt engineering through a persona-based application called "Bhagwan Speaks" — an LLM that responds as Lord Krishna, drawing from the Bhagavad Gita. 

Two API endpoints were set up in the notebook:
- *Cell 1*: A pre-hosted endpoint with a fixed system prompt (hard-coded on the server). Good for a quick demo but not editable.
- *Cell 2*: A second endpoint that accepted a custom `prompt` variable.

Both endpoints forwarded requests to the Sonar Reasoning model. Key input parameters included the user's question, the persona to adopt, the user's name, model name, API key, and gender.

#### What Makes a Prompt Work

The system prompt in Cell 2 was a good case study in prompt engineering. Breaking it down:

- *Persona definition* — tells the model not just who to be, but the qualities associated with that persona: "supremely wise, infinitely compassionate, and delightfully playful."
- *Tone descriptors* — guides the emotional register: "serenity of a cosmic being," "light and playful like the flute."
- *Content grounding* — instructs the model to draw from actual Sanskrit phrases in the Gita and Ramayana, keeping responses culturally accurate.
- *Hard guardrails* — respond in 5–6 sentences only; English only; correct Sanskrit spellings; quote the Gita when relevant to reduce hallucination.

The difference between Cell 1 and Cell 2 outputs was noticeable — Cell 2 was more scripturally grounded, more precise, and better formatted. Same model, different prompt.

---

### Lecture 4 · API Internals, System vs User Prompts, Temperature & RAG vs Fine-Tuning

#### Inside the API Code

The code has three main parts:

1. **Library imports** — `requests` for HTTP, `pydantic` for input validation, `pandas`, `json`, logging.
2. **Input class** — a Pydantic `BaseModel` that defines all expected fields: the user's question (`rephrased_memories`), persona (`user1`), recipient name (`user2`), gender, model name, API key, and an optional custom system prompt. This is basic OOP — the class acts as a structured container, values accessed via `data.user1`, `data.gender`, etc.
3. **Response function** — reads all values from the input object, determines the salutation via simple conditional logic (no LLM involved), builds the system and user prompts, assembles the HTTP header (with API key) and payload (model + messages), and sends a POST request to the Perplexity endpoint. Status 200 = success; 400/401 = bad request or authentication error.

#### System Prompt vs User Prompt

This is one of the most practically important concepts in building LLM applications.

| | System Prompt | User Prompt |
|---|---|---|
| Purpose | Sets the model's persona, rules, tone, constraints | The user's actual question or instruction |
| Controlled by | Developer | End user |
| In ChatGPT | Hidden — hard-coded by OpenAI | Whatever you type |
| In a custom API app | Fully under developer control | Developer or end user |

In any consumer-facing LLM product, there is always a system prompt running invisibly in the background shaping model behaviour. When building on an API directly, developers control both. In the above notebook, the system prompt held the persona, tone, salutation rules, citation instructions, and length constraints — the user prompt held the actual question, the user's name, and conversation history.

The two prompts are sent as separate entries in a `messages` array with a `role` field:
```python
payload = {
    "model": data.modelname,
    "messages": [
        {"role": "system", "content": system_prompt},
        {"role": "user",   "content": user_prompt}
    ]
}
```

#### Conversation Memory

LLMs are stateless at the API level — every call is independent, and the model remembers nothing from previous turns. To maintain a coherent conversation, the full message history has to be passed in with every new request:

```python
messages = [
    {"role": "system",    "content": system_prompt},
    {"role": "user",      "content": "First question"},
    {"role": "assistant", "content": "First response"},
    {"role": "user",      "content": "New question"},
]
```

#### Temperature and Top-P

These parameters control how the model samples its next token from the probability distribution.

*Temperature* scales the distribution before sampling. At temperature = 0, the model always picks the single most likely token — output is near-deterministic, consistent, but can be rigid. At higher values (0.8–1.2), probability mass spreads toward lower-probability tokens, making output more varied and creative but also more prone to hallucination. Use low temperature for factual tasks, higher for creative ones.

*Top-P* (nucleus sampling) restricts the model's pool to the smallest set of tokens that collectively account for the top-P fraction of probability mass. With top-p = 0.9, the model only considers tokens from the top 90% of probability — cutting off the long tail of unlikely tokens. Temperature and top-p work together and are typically tuned as a pair.

#### RAG vs Fine-Tuning

Both strategies solve the same problem: an LLM has broad general knowledge but lacks the specific domain knowledge your application needs.

| Dimension | RAG | Fine-Tuning |
|-----------|-----|-------------|
| Factual accuracy (limited data) | Higher | Lower |
| Conversational style | Less natural | More natural |
| Compute cost | Low | High (GPU-intensive) |
| Data format needed | Raw text | Structured Q&A pairs |
| Changes model weights | No | Yes |
| Time to deploy | Fast | Slower |

RAG leaves the model unchanged — at query time, relevant document chunks are retrieved and injected into the prompt. Quick to deploy, cheap to run, but the model's conversational tone stays as-is.

Fine-tuning updates the model's weights on domain data, internalising new knowledge and producing more natural stylistic responses. But it needs significant compute, large volumes of Q&A pairs (ideally 50,000+), and risks catastrophic forgetting.

---

### Lecture 5 · RAG — Retrieval Augmented Generation

#### Why RAG Exists

When you send a prompt to an LLM, it responds based entirely on what it learned during training. It has no awareness of events after its cutoff date, and no knowledge of specialised or private information that was never in its training data.

RAG solves both problems without retraining the model. Instead, it retrieves relevant content from a document store at query time and injects it directly into the prompt.

#### From Text to Vectors — Tokenization and Embeddings

Before documents can be stored or searched, text must become something mathematically comparable. This involves three steps that are often conflated but are distinct:

1. *Tokenization* — raw text is split into tokens from the model's fixed vocabulary. Common words map to a single token. Rare or misspelled words get split into sub-word tokens — e.g., "punctuation" might become ["punct", "uation"].
2. *Indexing* — each token is assigned its numerical position in the vocabulary. The sentence becomes a list of integers.
3. *Embedding* — each integer is projected into an n-dimensional vector by the embeddings model. This vector encodes semantic meaning. Words used in similar contexts cluster together; opposite concepts appear along the same directional axis.

All three steps are handled by a dedicated **embeddings model** — completely separate from the LLM. The embeddings model only converts text to vectors; it doesn't generate language. In a RAG pipeline, both are used: the embeddings model handles vectorisation, the LLM handles response generation.

#### The RAG Pipeline

*Phase 1 — Indexing (one-time setup):*
Source documents (PDF, Word, HTML, plain text) are text-extracted, split into fixed-length chunks, passed through the embeddings model to generate a vector per chunk, and stored in a vector database alongside the original text. Done once, repeated only when documents are updated.

*Phase 2 — Query time:*
The user's query is passed through the same embeddings model to produce a query vector. The database runs a similarity search — comparing the query vector against all stored chunk vectors, scoring each 0 (irrelevant) to 1 (identical). A score threshold filters out low-relevance chunks. The top-K highest-scoring chunks are selected and injected into the prompt alongside the user's question. The LLM then generates a response grounded in that retrieved context.

#### Key Design Decisions

- *Similarity metric* — cosine similarity is standard (measures angle between vectors, invariant to magnitude). Maximum Marginal Relevance (MMR) penalises redundancy, selecting diverse chunks rather than multiple chunks covering the same content.
- *Score threshold* — too high and nothing gets retrieved; too low and irrelevant content gets in.
- *Top-K* — number of chunks to retrieve. Must be balanced: `K × chunk_size ≤ context_window_limit`.
- *Metadata filtering* — chunks can be tagged at indexing time (chapter number, document name, date). At query time, filters can pre-restrict the search space before similarity scoring — e.g., "only search within Chapter 18."

#### What to Do with Retrieved Chunks

| Strategy | How It Works |
|----------|-------------|
| Stuff | All K chunks injected directly into the prompt. Simplest; works well for small K. |
| Map Reduce | Each chunk processed independently; partial answers combined. Good for large document sets. |
| Refine | Chunks processed sequentially; each refines the previous answer. |
| Map Rerank | One answer generated per chunk; answers ranked; best selected. |

For most cases, Stuff is sufficient. The others come into play when the corpus is large or chunks are too many to fit in one prompt.

#### RAG and Data Privacy

In a RAG setup, source documents stay in the organisation's own infrastructure. Only the most relevant text chunks are temporarily sent to the LLM for a specific query. Data is never permanently baked into a shared model's weights and can't leak to other users.

#### Limitations

RAG is only as good as the knowledge source behind it. If the indexed documents are incomplete, poorly written, or simply don't contain the answer, retrieval will either return irrelevant chunks or nothing — and the LLM won't be able to help.

---

### Lecture 6 · Fine-Tuning Deep Dive — LoRA, Synthetic Data & Evaluation

#### Where RAG Falls Short — Keyword Augmented Retrieval (KAR)

Vector similarity retrieval breaks down on sparse data. If a key term appears only once or twice in a large document and is later referred to only through pronouns — "he," "they," "the Prime Minister" — the vector search on that name may fail entirely. 

Keyword Augmented Retrieval (KAR) handles this by shifting from vector comparison to keyword overlap. KAR extracts meaningful keywords (nouns, named entities — filtering stop words) from both the query and each document chunk. Chunks that share keywords with the query are selected as context. It's precise for proper nouns and rare terms.

KAR is not a replacement for RAG — it's complementary. Vector similarity handles conceptually related queries; keyword matching handles precise names and sparse references. Combining both (hybrid search) is the most robust approach.

| | RAG (Vector) | KAR (Keyword) |
|---|---|---|
| Works well on | Dense, concept-rich text | Sparse data, proper nouns, names |
| Fails on | Sparse / pronoun-heavy text | Semantically related but lexically different queries |

#### Fine-Tuning — What the Data Needs to Look Like

Fine-tuning needs structured question-answer pairs, not raw text. Articles, PDFs, and web pages have to be converted into instruction-response pairs first.

#### Generating Synthetic Training Data

Creating tens of thousands of Q&A pairs manually is not practical. A workable alternative: use a large capable model (GPT-4 or a large open-source model) to generate synthetic Q&A pairs from raw source documents. Supply it with the source text, prompt it to generate question-answer pairs, collect the outputs, and use those to fine-tune a smaller target model. 

#### Full Fine-Tuning vs LoRA

*Full fine-tuning* updates every parameter in the model. Thorough, but extremely memory-intensive and risks catastrophic forgetting — all weights are in play, so new training data can overwrite patterns from pre-training.

*LoRA (Low-Rank Adaptation)* is a parameter-efficient alternative. Instead of modifying the original weight matrices, LoRA inserts small adapter matrices at specific layers and trains only those. The original model weights are frozen throughout.

The key hyperparameter is **LoRA rank (r)** — higher rank means more expressive adaptation but more compute. In practice, small ranks (4, 8, or 16) are sufficient for most tasks.

| | Full Fine-Tuning | LoRA |
|---|---|---|
| Parameters updated | All (~100%) | < 1% (adapter matrices) |
| Base weights | Modified | Frozen |
| Catastrophic forgetting | High risk | Very low risk |
| Compute and memory | Very high | ~80% less |
| Quality | Best (with enough data) | Near-equivalent for most tasks |

#### Compute Requirements

Each parameter is stored as a 32-bit float (4 bytes). A 70B parameter model needs roughly 280 GB just to store. During training, you also need memory for gradients (4 bytes/param), optimizer states with Adam (8 bytes/param), and activations (~8 bytes/param) — totalling about 24 bytes per parameter, or ~1,680 GB at full precision for 70B. In practice, LoRA and quantization bring this down significantly.

#### Quantization

Quantization reduces numerical precision of weights to cut memory use. Quantization is routinely combined with LoRA to make 7B–70B parameter fine-tuning feasible on a small number of GPUs.

#### Evaluating LLM Output

| Metric | What It Measures |
|--------|-----------------|
| BLEU | N-gram overlap between output and reference. Higher is better. Common for translation. |
| ROUGE | Recall-oriented n-gram matching. Common for summarisation tasks. |
| Perplexity | Model confidence when predicting tokens. Lower = more confident, consistent predictions. |
| F1 Score | Harmonic mean of precision and recall. Used for Q&A accuracy. |
| HELM (Stanford) | Holistic evaluation covering accuracy, calibration, and robustness across varied phrasings. |


#### Supervised Fine-Tuning vs RLHF

Supervised Fine-Tuning (SFT) directly updates weights using labelled Q&A examples — straightforward to implement and reason about.

Reinforcement Learning from Human Feedback (RLHF) works differently. Human raters compare pairs of model responses, indicating which is better. These preference annotations train a separate **reward model**, which scores responses according to human preferences. The base LLM is then updated using PPO to maximise the reward model's score. Both SFT and RLHF modify the LLM's weights, but RLHF does so indirectly through a learned reward signal rather than direct supervision.

---

### Lecture 8 · RAG Implementation — Vector DB, Hybrid Search & Chunking

#### Building a RAG Pipeline in Code

*Indexing phase:* Source documents are loaded, text extracted, split into chunks of approximately 1024–2048 tokens. Each chunk is passed through the embeddings model to generate a vector. The text and its vector are stored together in a `VectorStoreIndex`. At the same time, keywords are extracted from each chunk and stored in a `KeywordTableIndex`. 

*Query phase:* The user's query is vectorised by the same embeddings model. A hybrid retriever runs two searches in parallel: a vector similarity search and a keyword search. The custom retriever merges both result sets — configurable as AND (chunk must appear in both) or OR (chunk from either qualifies). The merged text chunks go to the Response Synthesizer, which injects them into a prompt and sends it to Gemini for response generation.

#### Hybrid Search — Why Both Methods Are Needed

Vector-only retrieval fails on sparse data. A name like "Mahatma Gandhi" mentioned once and thereafter referred to only as "he" won't be reliably retrieved by semantic search on "Gandhi." The keyword retriever fills this gap.

The vector retriever understands meaning: "automobile" and "car" produce similar vectors even with no shared keywords. The keyword retriever is precise but literal — it can't connect semantically related terms.

| | Vector Retriever | Keyword Retriever |
|---|---|---|
| Method | Cosine/KNN similarity | Keyword overlap |
| Strength | Paraphrase tolerance; semantic queries | Proper nouns, named entities, sparse data |
| Weakness | Sparse / pronoun-heavy text | Semantically related but lexically different queries |

AND merging gives a tighter, more precise result set. OR merging gives a broader, more inclusive one.

#### LlamaIndex — Key Objects

| Object | What It Does |
|--------|-------------|
| Embeddings model | Converts text to vectors (`embedding-001`, needs GCP API key) |
| VectorStoreIndex | Builds and manages the vector database |
| KeywordTableIndex | Extracts and stores chunk keywords |
| Custom Retriever | Wraps both retrievers, handles hybrid merge logic |
| Response Synthesizer | Holds LLM config; generates response from retrieved chunks + query |
| Query Engine | Top-level object tying retriever and synthesizer together |


#### Why a Separate Embeddings Model Is Needed

Decoder-only LLMs (GPT, Claude, LLaMA, Gemini) are designed to generate the next token — their architecture is built for generation, not for producing fixed-length vector representations of input.

Encoder models (BERT, sentence-transformers) are built specifically to take text in and produce a fixed-length, context-aware vector.

| | Encoder Model | Decoder LLM |
|---|---|---|
| Output | Fixed-length vector | Next generated token |
| Attention | Self-attention only | Self + cross-attention |
| Role in RAG | Vectorise documents and queries | Generate final response |
| Examples | BERT, embedding-001 | GPT-4, Claude, Gemini, LLaMA |

#### Chunk Size Matters

Chunk size (in tokens) is how large each stored piece of text is. Too small (64–128 tokens) and each chunk lacks enough context to be useful in isolation — retrieval returns fragments the LLM can't use effectively. Too large and retrieved chunks together may exceed the LLM's context window limit.The hard constraint: `top_k × chunk_size ≤ context_window_limit`.

#### A First Look at Agents

The RAG system built here is actually a minimal agent: it interacts with an external service (the vector database), retrieves information, and uses it to generate a grounded response. More advanced agents extend this pattern to live web search, email APIs, relational databases, calendar services — but the architecture is the same. RAG is the foundation of any agent system.

---

### Lecture 9 · Agentic AI — Multi-Agent Systems with CrewAI

#### What an Agent Actually Is

An agent is simply an automated workflow that takes actions toward a goal. In the LLM context, it's a workflow that uses an LLM as one of its components — for language understanding, reasoning, or generation. But the LLM doesn't need to be at every step; some steps might involve calling an API, querying a database, or running a Python function with no LLM involved at all. The defining characteristic is autonomy — it runs without a human triggering each individual step.

#### CrewAI — The Framework

CrewAI is a Python library for building multi-agent pipelines. It was chosen here because the code reads closely to plain English, making it accessible even to people new to agent frameworks. 

#### The Three Building Blocks

**Agent** — defined by a natural language role, goal, and backstory. Assigned an LLM for any reasoning or generation tasks, and a list of tools (external APIs or functions it can call).

**Task** — defined separately from the agent. Includes a plain-language description of what to do, a description of the expected output, optional tools, and which agent executes it.

**Crew** — assembles agents and tasks together and specifies the execution mode:
- `Process.sequential` — tasks run one after another; each task's output feeds the next.
- `Process.parallel` — tasks run simultaneously, useful for independent operations.

#### A Two-Agent Demo: Research + Write

The session built a two-agent pipeline to retrieve and summarise content from the Bhagavad Gita.

*Agent 1 — Researcher:* Uses a Google Search tool to retrieve content from a given URL or topic. The search itself is a direct API call — the LLM isn't involved in the retrieval step. The LLM is specified in the agent definition for framework compatibility, but plays no role here.

*Agent 2 — Writer:* Uses the Gemini LLM and no external tools. Takes the Researcher's output and reformats it as a structured bullet-point summary. This step is entirely LLM-driven.

```python
crew = Crew(
    agents=[researcher_agent, writer_agent],
    tasks=[research_task, write_task],
    process=Process.sequential
)
result = crew.kickoff(inputs={"topic": "Bhagavad Gita Chapter 4"})
```

With sequential processing, the Research task's output is automatically available to the Write task. The `kickoff()` call starts the pipeline.

#### Scaling the Pipeline

The two-agent example is just a starting point. In practice, pipelines can have as many agents and tasks as needed:

| Step | Agent | Task | Tool |
|------|-------|------|------|
| 1 | Researcher | Retrieve information | Google Search |
| 2 | Writer | Draft from research | LLM (Gemini) |
| 3 | Reviewer | Critique the draft | LLM |
| 4 | Rewriter | Incorporate feedback | LLM |
| 5 | Sender | Deliver final output | Email API |

Tasks must be in the correct sequence for sequential execution. Independent tasks can be run in parallel — for example, two Researcher agents scraping different sites simultaneously, with their outputs fed to a single Writer.

#### Tools in CrewAI

Tools are how agents interact with the external world. Any Python function or external API can be registered as a tool. Examples covered: Google Search, Brave Search, email APIs (SendGrid, Gmail), SQL database connectors, and custom Python functions. Each tool requires its own authentication credentials. The agent decides when and how to invoke its tools based on the task description — the developer doesn't specify this manually.

#### RAG vs Agentic AI

| | RAG | Agentic AI |
|---|---|---|
| Knowledge source | Pre-indexed documents | Live web, external APIs, databases |
| Complexity | Lower — retrieval + generation | Higher — coordinated agents and tools |
| Use case | Q&A over a fixed corpus | Multi-step automation with external services |
| LLM role | Response generation | One component; not in every step |

RAG is the right fit when the knowledge domain is fixed and pre-indexable. Agentic AI is the right fit when the task involves live data, external services, or multi-step workflows that benefit from specialisation.


---
