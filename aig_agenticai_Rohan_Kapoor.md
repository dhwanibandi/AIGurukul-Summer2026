# AI Gurukul — Lecture Notes

Course on Generative AI / LLMs.

---

## Lecture 1 - What is Generative AI & LLMs

**Big picture**

- LLMs started from NLP research. Inflection point: 2017 paper *"Attention Is All You Need"* (transformers, Google). The lineage that followed:

```txt
Attention Is All You Need (2017, transformers)
      ↓
    BERT
      ↓
  GPT series
```

- Generative AI isn't just text: also images (Stable Diffusion, Studio Ghibli-style gen), music, voice, code, even gene sequencing.
- Many open-source LLMs exist (Hugging Face hosts thousands, with weights). Useful when you have privacy/cost concerns and want to self-host instead of using paid APIs.

**How LLMs actually work**

- They are **probabilistic models**, not deterministic. Given input words, they compute the probability of every possible next word and output the highest-probability one.
- Example: "color of the sky is ___" → "blue" gets ~91% probability → picked.

**Two training phases**

1. **Pre-training** — feed huge generic text (Wikipedia, books, web). Model learns general patterns. *Analogy: a baby observing everything before school.*
2. **Fine-tuning** — train further on domain-specific data (e.g. medical, history Q&A) to specialize. *Analogy: school curriculum for a specific subject.*
   - Fine-tuning is delicate: bad data can make the model **unlearn** previous knowledge. Data must be carefully curated.

**Where is the "memory"?**

- Stored as **weights/coefficients** in the neural network (inspired by brain neurons).
- Number of parameters is **fixed** in advance. More data → better learning, but size doesn't grow (like an adult brain learning more without growing bigger).

**Attention mechanism (the key idea)**

- Words in a sentence connect to each other with different strengths (e.g. "she" → "girl"/"nurse", not "Bob").
- **Context-dependent meaning**: "bank" = financial institution OR riverbank — decided by neighboring words. Attention lets the model weigh surrounding words to disambiguate.
- Old rule-based systems (dependency parsing) couldn't handle this well, transformers learn it from data.

**Words as vectors (embeddings)**

- Words are represented as vectors (lists of numbers) in N-dimensional space.
- Similar/related words cluster together, you can even do arithmetic: `man - woman + queen ≈ king`.
- Came from Word2Vec (2012-13), fully exploited by LLMs.

**Key caution: don't trust LLMs for knowledge/facts**

- They just reproduce whatever's in their training data — can be biased or wrong (e.g. politically skewed answers on certain countries).
- Use them for **general capabilities** (reasoning, summarizing) — and ground them with the right context, not as a fact oracle.

**Pre-training technique introduced: Masked Language Modeling (MLM)**

- Hide some words in sentences, force the model to predict them, penalize wrong predictions via a loss function. This updates the weights.

---

## Lecture 2 - Transformers, Pre-training & Prompts

**Fine-tuning is its own training run**

- It does NOT concatenate to old data — it's a *fresh* training on new data only, old learning stays captured in existing weights.
- "New data" for recent events = real public articles, but you must **reformat into Q&A** ("data generation"). Needs serious compute (e.g. 160 GB RAM / H100 GPUs).

**Embedding dimensions**

- Vector size (e.g. 512, 4096, 8192, usually a multiple of 512) = number of learned characteristics, NOT number of words.
- Bigger dimension → better separation of meanings, but more storage. In practice people stay under ~16,000.

**Transformer architecture**

- Transformer = Neural Network + Attention (+ other math transforms).
- **Encoder** = converts words → vectors. **Decoder** = calculates next-word probabilities.
- Original paper: 6 stacked encoders + 6 stacked decoders.

**Architecture types:**

| Type | Example | Best for |
|---|---|---|
| Encoder-only | BERT | Understanding tasks (sentiment, classification, summarization) |
| Decoder-only | GPT, LLaMA | Generating text — **most chatbots** |
| Encoder-decoder | BART (Bidirectional and Auto-Regressive Transformers) | Translation |
 
**Why transformers beat older models (RNN/LSTM/GRU)**

- Old models processed words **one at a time** (slow) and had **short memory** (~6-7 words). No real attention.
- Transformers process all words **in parallel** (fast, scalable) and capture **long-range dependencies**.

**Pre-training techniques**

- **Next-token prediction**: hide the last word, force prediction.
- **Masked Language Modeling (MLM)**: randomly hide words, predict them using left+right context.

**Size isn't everything — training method matters**

- For the same model size, better training (supervised fine-tuning) raised answer quality (Likert score) far more than just adding size.
- **Takeaway: small models trained/fine-tuned well can beat large untrained-prompted ones.** Fine-tuning also improves conversational style and conciseness, not just knowledge.

**Prompts & shot learning**

- A **prompt** = natural-language instruction to the model.
- **Zero-shot**: ask with no examples.
- **One-shot / few-shot**: give 1 or more examples in the prompt to guide it.
- Giving examples (even unrelated ones) helps the model infer the pattern — models are surprisingly good at this.

---

## Lectures 3 & 4 - API & Prompt Engineering

**Why responses differ each time**

- The model samples words from a probability distribution — same question can give different phrasing every run. It doesn't "know" what it's saying.

**System prompt vs User prompt**

- **System prompt**: set by the company, hidden, has higher priority. (This is why DeepSeek refuses certain topics, e.g. Taiwan — enforced via system prompt.) You can't change it on commercial UIs.
- **User prompt**: what you type. You can change this.
- When you write your **own code** hitting the API directly, you can set **both** system and user prompt.

**How the API call is structured**

- `headers` carry the **API key**.
- `payload` carries the **model name + system prompt + user prompt** (roles: `system` and `user`).
- POST to Perplexity's `chat/completions` URL → get a `response` object with `response.text` and `response.status` (200 = OK, 400/401/404 = error).
- Inputs bundled in an **input class**, `data` is the object holding all variables (model name, API key, question (`refresh_memory`), user names, gender). *Analogy: a bag holding your notebook/pen/laptop, `data.xxx` reaches inside.*

**Context window**

- Total budget = (system prompt + user prompt + question + **context** + response) in tokens. Must stay ≤ model's context window (e.g. 128k).
- Exceed it → response truncated, quality drops, or you get one-word output.
- Rough conversion: ~4 tokens ≈ 1 word, so 128k tokens ≈ ~96,000 words.

**Temperature & Top-p**

- **Temperature = 0** → least creative, only highest-probability words → conservative, consistent.
- **Higher temperature** → samples lower-probability words → more creative but **more hallucination risk**.
- **Top-p**: probability threshold (e.g. 0.9) limiting which words can be sampled.

**Adding context**

- Beyond system/user prompt and question, you can pass **context** (a paragraph, PDF text, book) so the model answers from *that* instead of guessing. Demonstrated by uploading a PDF (e.g. compiler-design book) and asking "what is this course about?" — this is **RAG** (next lecture).

**RAG vs Fine-tuning (from a comparison paper)**

| Dimension | RAG | Fine-tuning |
|---|---|---|
| What it does | Keep a general model, supply external text as context at query time | Retrain/update the model's parameters on specific data |
| Cost | Cheap | Expensive, needs GPUs (like internalizing a subject over years) |
| Wins on | Knowledge & conciseness | Conversational style |
| Data needed | Low | Lots of data to shine |

- **For quick development, prefer RAG.** Fine-tuning is essentially transfer learning.
- Building a model from scratch is infeasible — instead use existing models + RAG or fine-tuning.

---

## Lecture 5 - RAG (Retrieval-Augmented Generation)

**Why RAG?** Relying on the model's own training data is risky — it may not know your specific text (e.g. Bhagavad Gita / Bhagavatam) or only know it partially. RAG (or fine-tuning) fixes this. RAG = retrieve relevant info, then generate an answer using it.

**RAG components**

1. **Language model** (commercial or open-source).
2. **Embeddings model** — converts text → vectors.
3. **Vector database** — stores those vectors.
4. **Search + chaining** logic.

**Tokens vs Vectors (clarified)** — a 3-step pipeline, all done by the embeddings model:

```txt
Text
  ↓
Tokens
  ↓
Token numbers
  ↓
Vectors
```

1. **Text to tokens**: tokenizer splits text using a fixed vocabulary (~60k tokens). Words not in vocab get broken into sub-pieces (e.g. "punctuation" → "punct" + "uation").
2. **Tokens to token numbers**: each token replaced by its index/serial number in the vocabulary.
3. **Token numbers to vectors**: mapped into N-dimensional space where similar/related words land close together.
- The **embeddings model is separate from the LLM**.

**Indexing pipeline (offline prep)**

```txt
Source data (CSV, HTML, Word, PDF…)
      ↓
Extract text
      ↓
Embeddings model
      ↓
Vectors
      ↓
Vector DB
```

**Query pipeline (at question time)**

```txt
User query
      ↓
Embeddings model
      ↓
Query vector
      ↓
Similarity search in vector DB
      ↓
Rank, select top-K
```

1. Convert the user query into a **query vector** using the embeddings model.
2. **Similarity search** in the vector DB (like Google's old PageRank, but on vectors). Similarity score: 1 = identical, 0 = unrelated.
3. Discard score-0 chunks, among the rest you may still get hundreds/thousands of relevant chunks.
4. **Rank** by similarity score and select **top-K** (e.g. K=5 or 10). Can also filter using **metadata** (e.g. drop chapters known to be irrelevant) or a similarity-score cutoff.
   - Reason for limiting: too many chunks would exceed the **context window**.
5. Search algorithms: **similarity search** and **maximum marginal relevance (MMR)**.

**Feeding chunks to the LLM** — how to consume the retrieved chunks:

- **Stuff**: dump all retrieved chunks straight into the prompt (simplest).
- **Map-reduce**, **Refine**, **Map-rerank**: more advanced strategies borrowed from big-data (Hadoop map-reduce) concepts.

**The core equation**

```
Final input to LLM = Prompt (instruction) + Query (question) + Context (top-K retrieved chunks)

```
- Without RAG: just prompt + query → model answers from its own training data (may stray across sources).
- With RAG: you supply the exact relevant context (e.g. only Chapter 18 of the Gita) → answer stays grounded, no room to wander.

**Context vs RAG**: not interchangeable. Passing context is **part of** RAG. RAG is the whole retrieve→augment→generate process, the retrieved context is what gets passed alongside the prompt.

**Why RAG is also safer**

- The **vector DB stays in your own organization**, you only send the most relevant chunks to the LLM, not your entire confidential dataset.
- Uploading full docs to ChatGPT is risky — you don't know if it's doing RAG or folding your data into its training (which would be irreversible and could leak to others).

**Garbage in, garbage out**: response quality depends entirely on supplying the *right* context (feeding Gita text but asking Ramayana questions → poor answers).

---

## Lecture 6 - Fine-tuning (and LoRA), plus Keyword-Augmented Retrieval

**Keyword-Augmented Retrieval (KAR) — an alternative to RAG**

- Instead of matching *vectors*, extract **keywords** from both the query and the document, then match keywords to pick the relevant paragraph as context.
- **Where RAG fails**: on **sparse data** — info appearing only once or twice (e.g. a person's name "Narendra Modi" mentioned once, then "Modi"/"PM" via pronouns). Vector similarity misses it, KAR finds it.
- So KAR > RAG for sparse datasets, otherwise RAG is fine.

**RAG vs Fine-tuning — the core difference**

- **RAG**: model is untouched. Relevant info pulled from a vector DB and added to the prompt at query time.
- **Fine-tuning**: the **model itself is updated** — it actually learns/memorizes the new information by adjusting parameters.

**Case study:**

- **RAG for knowledge** — study material keeps changing (new books, notes, YouTube transcripts). Re-fine-tuning every time would be too slow/expensive, so feed new info via RAG. Supports PDF/XLS/CSV/TXT/DOC + YouTube links.
- **Fine-tuning for personalization** — the *persona/style* (talk like a professor / school teacher / doctor) rarely changes, so bake it in via fine-tuning.

**Instructor personalities ← Big Five framework**

- Big Five = 5 personality dimensions: Extraversion, Conscientiousness, Neuroticism, Openness, Agreeableness.
- They identified 4 common instructor personas, all mapped onto the Big Five (all low on neuroticism, vary mainly in openness/agreeableness).

**Where does fine-tuning data come from? (synthetic data generation)**

- You need **thousands** of Q&A pairs per persona — manual creation is infeasible.
- Solution: use a **large, sophisticated LLM** (that understands Big Five) to **generate Q&A pairs**, then use those to fine-tune a **smaller model** (e.g. LLaMA 3.1 70B). Feed data grouped per personality, labeled by persona type.

**What fine-tuning needs**

1. **Dataset** (Q&A pairs, e.g. the Big Five Chat dataset).
2. **Compute / GPUs** — a 70B model needs ~2× the storage+RAM during training. Fine-tuning LLaMA 3.1 70B needs ~**2× A100** (≈160 GB RAM). Regular PCs won't do.
3. **Care against catastrophic forgetting** — full fine-tuning can make the model **forget** older knowledge while learning new data.

**Full fine-tuning vs LoRA (Low-Rank Adaptation = PEFT)**

| Aspect | Full fine-tuning | LoRA (PEFT) |
|---|---|---|
| Params updated | All | Tiny fraction (often **<1%**) |
| RAM / compute | Very heavy (also stores a copy of old weights) | ~80% less |
| Forgetting risk | Higher | Lower |
| When to use | Only if you can afford it & manage forgetting | Efficiency with large corpora |

- **LoRA rank** controls trainable %: higher rank → more trainable params. As rank → ∞, LoRA ≈ full fine-tuning (i.e. full fine-tuning is a special case of LoRA).

**Fine-tuning = teaching style/format more than new facts** — it adjusts how the model responds, pushing too much new data risks overwriting (forgetting) existing knowledge.

**Evaluation matters** — to compare RAG vs full FT vs LoRA, use **evaluation metrics** (choice depends on use case).

- **HELM** (Holistic Evaluation of Language Models, Stanford): profiles models on accuracy, calibration, robustness, fairness, bias, toxicity.

**Storage math (why it's so heavy)**

- 1 parameter = 4 bytes (32-bit float) → 1B params ≈ 4 GB.
- Training also stores **optimizer state (Adam), gradients, activations** → roughly ×(4+8+4+8) overhead. That's how 70B → ~160 GB.

**Quantization** — shrink storage by converting 32-bit floats → **16-bit or 4-bit**. Small accuracy drop, big memory savings.

---

## Lecture 8 - RAG End-to-End: Hands-on Code (Hybrid Vector + Keyword Search)

**Recap: why RAG or fine-tuning at all?**

- An off-the-shelf LLM is trained on limited/generic data — it has no knowledge of your *organization-specific* data. *Analogy: a civil engineer asked computer-science questions can't answer.*
- Two fixes:
  - **RAG** — augment the prompt with relevant info at query time. *Analogy: give the engineer a book to refer to while answering.*
  - **Fine-tuning** — retrain the model so it internalizes the info. *Analogy: send the engineer to take CS courses.*
- **Compute**: fine-tuning is expensive (happens at **training** time) but one-time. RAG adds only ~0.1% extra compute (at **inference** time). E.g. fine-tuning a 70B model needs ~2× A100 (160 GB) just to load, more to update → 3-4 GPUs. So **start with RAG**.

**Storage difference**

- RAG **requires a vector DB** (you "carry the book"). Fine-tuning needs none (knowledge is "in the brain").

**Hybrid retrieval = vector similarity + keyword similarity**

- Pure vector similarity often isn't enough for **sparse info** (e.g. a rare name appearing a few times, mostly replaced by pronouns).
- **Keyword similarity** catches those. So the demo uses **both** (hybrid search).
- **Keyword retriever**: extracts important words (nouns), drops stop-words/pronouns/prepositions, matches query keywords against each chunk's keywords (overlap-based — no understanding of meaning).
- **Vector retriever**: matches on **semantic** meaning. Needed because not every chunk containing a keyword is actually relevant.
- Combine the two sets with an **AND / OR** condition (defined in the custom retriever class).

**Encoder vs decoder (why embeddings need an encoder)**

- LLMs (GPT, LLaMA) are **decoder-only** → they only predict the next *word*, never output a vector. (Internally they encode, but you'd have to pull it out.)
- For embeddings you use an **encoder** (self-attention only). Decoders add **cross-attention** (for mixing input/output, e.g. translation) — not needed for encoding.
- The query and the documents **must use the same embeddings model**.

**Code walkthrough (Google Colab)**

1. **Install dependencies**
2. **Import** libraries.
3. Set the **Google API key**
4. **Embeddings model** — converts text → vectors.
5. **Build the vector DB**:
   - Put PDFs in a `data/` folder.
   - Extract text → **chunk** it → embed each chunk → store. First column = text chunk, second = its vector.
   - **Chunk size** matters (typical 1024 / 2048)
   - **Persistence**: in-memory (lost on restart) vs persistent (`persist=True`). The vector DB lives **on your own machine**, not client-side cloud.
6. **Custom retriever** = vector retriever (`similarity_top_k=5`) + keyword retriever, combined.
7. **Response synthesizer** holds the LLM config, the **query engine** wraps retriever + synthesizer, converts the query to a vector, retrieves chunks, injects them into the prompt, calls the LLM.
8. Ask a question → only answers from the uploaded book. "Who is Prashant?" (not in the Gita) → "unable to determine" (no matching chunks).

**Limitations & next steps** — vector+keyword retrieval will hit limits, then explore **dense retrieval**, other vector DBs, larger top-K, chunk-size tuning.

---

## Lecture 9 - Multi-Agent Systems (CrewAI)

**What's an agent?**

- An agent is **just a workflow / process automation**. A worker that takes an action by itself.
- It may or may not use an LLM. In this demo, the LLM is only *part* of the workflow.
- A **chat interface** can only respond to one instruction. An **agent** can **chain multiple tasks** together.

**Library used: CrewAI** — builds multi-agents, very readable code. (Alternatives: LangGraph, LangDroid)

**The demo: 2 agents, 2 tasks**

- **Researcher agent** → does a **Google search** (tool = custom Google search, no LLM). *Analogy: a paralegal sent to the library to find similar cases.*
- **Writer agent** → uses the **LLM** (`llm=llm`) to write/summarize the retrieved info. *Analogy: the paralegal drafting it up for the attorney.*
- Each agent has a **task** defined in **plain natural-language** (a `description` + `expected_output`).
- Example run: "read from [chapter/URL], summarize as bullet points" → researcher scrapes, writer summarizes.

**Build steps**:

```txt
Install libs
      ↓
    Import
      ↓
Set API keys (Google, Gemini/LLM, CrewAI)
      ↓
Define agents
      ↓
Define tasks
      ↓
process = sequential (or parallel)
```

**Chaining = the whole point**

```txt
Research
   ↓
Write
   ↓
Review
   ↓
Rewrite based on review
   ↓
Send email
```

- You can have **N agents / N tasks** chained in sequence (the diagram above is one such chain).
- Each new action may need its own API + auth (e.g. an email API key to send mail).
- **Parallel** processes are possible too — e.g. scrape two websites at once, then feed both into a comparison/filter step.
- You can even insert a **filtering layer** (LLM picks only relevant text, RAG-style) between research and writing.

**RAG vs this multi-agent setup**

| Aspect | RAG | Multi-agent |
|---|---|---|
| Structure | Single agent, single task | Multiple agents/tasks chained |
| Info source | Your **vector DB** (e.g. Gita PDF) | Live **Google search** |
| How info used | Injected into the prompt (not preloaded into model) | Passed to writer task, LLM summarizes — whole workflow re-runs each query |

**Other tricks mentioned**

- Agents can hit a **database**: agent 1 writes a SQL query from a schema, agent 2 runs it, agent 3 uses the result — "no data engineer needed" (but flaky on complex schemas).
- **No memory** between runs — each run is independent, to remember, you'd pass the whole history back in.
- **Caching** (homework) — store prior results so you skip re-doing the research step every time.
- **Why two identical runs differ**: the Google result is the same, but the LLM is **probabilistic** — output wording varies run to run.

**Caution on agents (important)**

- Don't over-automate.
- **Never auto-automate payments / credit cards.**
- Also legal risk: LLMs trained on others' data → regulators / copyright suits emerging. Be careful publishing LLM-generated books.

---
