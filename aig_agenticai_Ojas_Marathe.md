# AI Gurukul Lecture Notes

# Lecture 1

Generative AI became a Turning Point \- ChatGPT reached 100 million in \~2 months. This caused an explosion in AI products, AI research, AI education, startups, and open-source LLMs.

- Most modern LLM research came from corporate research labs, not universities like OpenAI, Anthropic

## How LLMs Evolved

- Pre-2017 \- Rule Based and statistical NLP methods  
- 2017 \- Transformer Revolution \- Google published *“Attention is All You Need”*   
- 2018 \- BERT \- encoder based  
- 2018 \- present \- OpenAI GPT series

## GenAI

LLMs are no longer text-only; their applications include text generation, image generation, music generation, solving JEE papers, etc.

## Open Source vs Commercial Models

Hugging Face \- biggest repository.  
Advantages of open-source models : self-hosting, no API dependency, customization, fine-tuning  
Commercial Models : guardrails, limited fine-tuning, cannot freely redistribute

**LLMs Are Probabilistic Next-Token Predictors** LLMs do not “know" facts deterministically. Instead, they compute a probability distribution over the next possible word/token at each step. The highest-probability token is selected. Entire paragraphs are generated one token at a time through this repeated process.

## How LLMs Learn

**Phase 1: Pretraining**  
Model trains on massive internet-scale text; learns general language patterns, grammar, and world knowledge.  
Analogy : Baby observing the world

**Phase 2 : Fine-tuning**  
The model is further trained on domain-specific data (e.g., medical Q\&A, legal documents) to develop expertise. Fine-tuning can actually degrade a model if done carelessly — a phenomenon called *catastrophic forgetting*, where the model loses previously learned knowledge while absorbing new information. Well-curated datasets are essential.  
Analogy : Specialised schooling

## Attention Mechanism

The single most important innovation in modern NLP. Before transformers, models like RNNs struggled to associate words that were far apart in a sentence. The attention mechanism allows every word to "look at" every other word simultaneously, enabling context-aware understanding. Example: "The girl said she is a nurse." The model learns that "she" relates strongly to "girl" and "nurse," not to irrelevant words. This disambiguation is what attention captures. 

Why it matters for meaning: The word "bank" can mean a financial institution or a riverbank. Attention allows the model to resolve this using surrounding context — something older models could not do reliably.

## Word Embeddings / Vector Representations 

Words are converted into high-dimensional numerical vectors. Semantically similar words cluster geometrically close in this vector space. 

Example: king − man \+ woman ≈ queen 

This arithmetic works because meaning is encoded as geometry. The model learns these relationships from co-occurrence patterns in training data, not from explicit rules. The technique was pioneered by Word2Vec.

## Hallucinations and Bias 

Because LLMs learn statistical patterns, they are susceptible to: 

* Hallucinations: Confidently generating incorrect information   
* Bias: Reflecting imbalances and distortions present in training data   
* Cultural/political skew: Western media dominance in training corpora influences model worldview 

Practical recommendation: Use LLMs primarily for reasoning, summarisation, transformation, and analysis tasks, not as authoritative factual databases.

# Lecture 2

## Transformation Architecture

Transformers combine neural networks with the attention mechanism. The original design (from "Attention Is All You Need", 2017\) has two components: 

* Encoder: Converts input text into rich vector representations (understanding side)  
* Decoder: Uses those representations to generate output text (generation side) 

Simplified formula: Transformer \= Neural Network \+ Attention

Encoder-only \- BERT \- Classification, sentiment analysis, understanding tasks  
Decoder-only \- GPT, Llama \- Text generation, chatbots  
Encoder-Decoder \- BART \- Translation, sequence-tosequence tasks

## Older NLP Models Before Transformers

Before transformers: RNNs, LSTMs, GRUs

Problems:

* sequential processing  
* slow  
* limited memory  
* poor long-range understanding

RNNs could predict only \~6–7 words effectively.

Transformers solved this because:

* all words are processed in parallel  
* attention captures long-range dependencies

This massively improved: quality, speed

## Pretraining Objectives 

Two main methods are used:   
**A. Next-Token Prediction (GPT-style):** The model sees a partial sentence and must predict the next word. Wrong predictions are penalised via a loss function, which updates the model's weights. Repeated billions of times across the entire training corpus.  
**B. Masked Language Modelling (BERT-style):** Random words in a sentence are hidden (masked) and the model must predict them from the surrounding context. This trains bidirectional understanding.

## Model Scale

Key insight: Larger models generally perform better, but training quality and strategy can make smaller models surprisingly competitive. Model size alone does not determine performance.

## Prompt Engineering 

A prompt is the natural language instruction given to an LLM. Prompt quality dramatically influences output quality.   
Types of prompting:   
Zero-shot \- No examples provided \- "Classify the sentiment of this review."   
One-shot \- One example provided before the task \- One labelled example then the query   
Few-shot \- Multiple examples provided \- Several labelled examples then the query.

Few-shot prompting consistently produces better results. The examples demonstrate the pattern the model should follow, even if they are not semantically related to the query.

## Hardware Realities 

Fine-tuning a 70B parameter model (e.g., Llama 3.1 70B) requires approximately: 2× H100 GPUs × 80 GB VRAM each \= 160 GB GPU memory total This is because training requires storing model weights, gradients, optimizer states, and activations simultaneously. LLM work is not purely a software challenge — it requires serious compute infrastructure.

# Lecture 3

The LLM Application Stack Modern AI applications are built on: 

* APIs – hosted interfaces that let code communicate with a remote LLM  
* Prompts – instructions that control model behaviour   
* Context – external information passed alongside a query   
* Retrieval systems – mechanisms to fetch relevant documents   
* Orchestration – logic to combine all these components 

The majority of production AI work involves combining these elements, not training large models from scratch.

## System Prompt vs User Prompt 

System Prompt \- The application/company sets it \- Hidden from end-user \- Safety rules, persona constraints, censorship   
User Prompt \- The user/developer sets it \- Visible \- Task instructions, persona, output format 

System prompts have higher priority. This is how companies like OpenAI or DeepSeek enforce behavioural guardrails even when users ask differently. Building on open-source models gives developers full control over system prompts.

## Context and Context Windows 

Context \= any external information passed to the model alongside the query. This can include uploaded PDFs, previous conversation history, or retrieved documents.   
Context window \= the maximum number of tokens the model can process in a single request. Everything counts toward this limit:   
System Prompt \+ User Prompt \+ Context \+ Query \+ Response ≤ Context Window   
Typical context windows range from 32K to 128K tokens (\~24K to \~96K words). If the total exceeds the limit, the model truncates, degrades in quality, or fails entirely.

## LLM Outputs Are Non-Deterministic 

The same query submitted twice will produce slightly different responses. This is because models sample from a probability distribution rather than always selecting the single highest-probability token. This is controlled by a parameter called temperature — higher temperature \= more creative/variable output; lower \= more consistent/predictable.

## Introduction to RAG (Retrieval-Augmented Generation) 

**Problem**: Pretrained models do not know your private documents, recent company data, or information after their training cutoff.   
**Solution**: Instead of relying purely on memorised training knowledge, retrieve relevant documents at query time and inject them as context.   
With Prompt (instructions), Retrieved Context (relevant docs) and Query the LLM generates a grounded, document-based answer.  
**Benefit**: Dramatically reduces hallucinations and keeps responses factually anchored to real source material.

# Lecture 4

## Reasoning Models — Sonar Reasoning Specifically

The session used Perplexity's sonar-reasoning model. The key insight: most reasoning models (OpenAI's O1, O3, GPT-4.1) require you to explicitly tell them to apply reasoning. Sonar reasoning applies chain-of-thought reasoning automatically even without being prompted to do so. That makes it more practical for everyday use. Perplexity offers three tiers — sonar, sonar-reasoning, and sonar-reasoning-pro — with similar quality between the latter two based on their own evaluation work.

## Cost Comparison — Perplexity vs GPT APIs

Made explicit here with actual numbers: the same volume of work done over 8 months cost roughly $30 on Perplexity's API. The equivalent on GPT APIs would have cost close to $800–900. For students and researchers, Perplexity is the practical starting point.

## System Prompts Were Leaked

Someone published all major model system prompts to a public GitHub repo. This means you can now read exactly how companies like OpenAI and Anthropic instruct their models internally, even though you still cannot change those prompts from the outside. The only way to control your own system prompt is to build and host your own fine-tuned model.

## The Backslash in Multi-line Prompts

A small but practical Python detail: when writing a long prompt string across multiple lines in code, you need a backslash \\ at the end of each line to tell Python the string continues on the next line. Without it, the interpreter treats it as a new statement and the prompt breaks silently.

## DeepSeek's Failure Modes Under Load

More detail than before: DeepSeek's problems are two specific things — it hits rate limit errors under heavy use (not scalable), and once it starts hallucinating it compounds badly with no recovery. The instructor attributed this partly to their heavy system-prompt restrictions around certain topics, which interferes with the model's general reasoning consistency.

# Lecture 5

## Retrieval-Augmented Generation

Meaning:

1. Retrieve relevant information  
2. Use that information while generating answer

Instead of:

“model answering from memory”

the model now answers from:

“retrieved documents/context”

## High-Level RAG Pipeline

The instructor described the pipeline:

Documents→ Convert to vectors→ Store in vector DB→ User asks query→ Query converted to vector→ Similarity search→ Retrieve relevant chunks→ Pass chunks as context to LLM→ Generate answer

This is THE core RAG architecture.

## Tokenisation 

Words are not fed directly into models. 

The pipeline is: Word → Token → Token Index → Vector Embedding 

A tokeniser maintains a fixed vocabulary (e.g., 60,000 tokens). Unknown or rare words are broken into subword pieces (e.g., "punctuation" → "punct" \+ "uation"). Crucially, a token is not the same as a vector — tokenisation is a preprocessing step before embedding. 

## Embeddings 

An embedding model converts text (words, sentences, chunks) into dense numerical vectors. Semantically similar text produces geometrically proximate vectors. Practical consequence: A query about "moksha" will retrieve passages about "liberation," "salvation," and "spiritual freedom" because they cluster nearby in vector space — even if the exact word "moksha" does not appear in those passages. Embedding dimensions typically range from 512 to 16,000. Higher dimensions improve expressiveness but require more storage and compute. 

## Vector Databases

Vector databases are purpose-built for similarity search at scale. After embedding, all document chunks are stored in a vector DB. When a query arrives, it is embedded and compared against all stored chunk vectors. 

Similarity score: 

Score → 1.0: highly semantically similar 

Score → 0.0: unrelated

## Chunking and Top-K Retrieval 

Large documents must be broken into chunks before embedding — entire books exceed practical context limits. After similarity search, hundreds of relevant chunks may be returned. Not all can be passed to the LLM (context window limits). 

The solution: 

Top-K Retrieval — rank all chunks by similarity score and select only the top K (e.g., top 5 or top 10). This balances relevance, context size, and inference cost. 

Metadata filtering can further improve precision — e.g., filtering to only chunks from a specific chapter or document when the query context is known.

## Retrieval Strategies

**Stuff** \- Concatenate all retrieved chunks directly into the prompt 

**MapReduce** \- Process each chunk separately, then combine results 

**Refine** \- Iteratively refine the answer by processing chunks sequentially 

**MapRerank** \- Score each chunk-based answer and select the best 

For most applications, "Stuff" is the simplest starting point.

## Fine-Tuning vs RAG

Instructor made distinction:

### **Fine-tuning \-** Updates model permanently.

### **RAG \-** Leaves model unchanged; only injects temporary context.

Security Considerations   
When using closed commercial APIs (e.g., ChatGPT) with proprietary documents: 

* You do not know whether documents become part of future training data   
* You do not know if RAG, fine-tuning, or other retention mechanisms are applied 

Building private RAG systems (self-hosted vector DB \+ open-source LLM) keeps sensitive data internal.

# Lecture 6

## Why Fine-Tuning Is Still Necessary 

RAG is excellent for knowledge retrieval, but it cannot change how a model behaves. Fine-tuning is required when you need to modify the model's: 

* Tone and communication style (formal, friendly, strict, motivational)   
* Persona (e.g., a specific instructor archetype)   
* Consistent output formatting   
* Domain-specific reasoning patterns 

Real-world example: An educational chatbot system might use: 

* RAG for course notes, PDFs, and evolving content (changes frequently)   
* Fine-tuning for teaching persona (changes rarely)

## The Data Challenge in Fine-Tuning 

Fine-tuning requires large quantities of labelled question-answer pairs formatted to demonstrate the desired behaviour. Creating these manually is impractical at scale. Solution: Synthetic Data Generation 

Powerful LLM (e.g., GPT-4 / Claude) — Generate thousands of Q\&A pairs in desired style — Use those pairs to fine-tune a smaller, cheaper model 

This is a mainstream technique in modern AI. The smaller model learns to imitate the larger one's behaviour on a targeted task.

## Full Fine-Tuning and Its Costs

In full fine-tuning, all model parameters are updated. Memory explodes because training requires holding the model weights, gradient calculations, optimizer states (e.g., Adam momentum terms), and intermediate activations simultaneously.

## Catastrophic Forgetting 

When a model is fully fine-tuned on new data, it may overwrite previously learned knowledge. The model "forgets" older capabilities while acquiring new ones. This is a fundamental challenge in continual learning. Mitigation strategies include careful dataset curation, training on a mix of old and new data, and using parameter-efficient methods like LoRA. 

## LoRA (Low-Rank Adaptation) 

LoRA is the most widely adopted parameter-efficient fine-tuning technique.   
**Core idea:** Instead of updating all billions of parameters, LoRA adds small trainable matrices to selected layers and updates only those — often less than 1% of total parameters — while freezing the rest.   
**Analogy:** Rather than rewriting an entire textbook, you insert a thin addendum with only the new information.

**LoRA Rank:** A hyperparameter controlling how many parameters are trained. Higher rank \= more expressive but more expensive. As rank → infinity, LoRA approaches full fine-tuning.

## Quantisation

Reducing numerical precision lowers memory requirements significantly with only a small accuracy tradeoff. Quantisation is commonly combined with LoRA for affordable fine-tuning.

## Evaluation Metrics 

After fine-tuning, rigorous evaluation is essential.   
Standard dimensions include: Accuracy – correctness of responses, Conciseness – appropriate response length, Robustness – consistency under varied phrasings, Fairness – absence of demographic bias, Toxicity – absence of harmful content, Calibration – appropriate confidence in outputs 

Stanford's HELM (Holistic Evaluation of Language Models) framework provides a structured approach to multi-dimensional LLM evaluation.   
Human evaluation using Likert scales remains the gold standard for conversational quality, as automated metrics often miss nuanced response characteristics.

## Keyword Augmented Retrieval (Extension of RAG) 

A limitation of standard vector similarity search: important named entities (e.g., a person's name) may appear sparsely in documents, causing retrieval failures.   
**Solution**: Augment vector similarity with direct keyword matching. The hybrid pipeline compares both query vectors and query keywords against the document, improving recall for rare but critical

# Lecture 8

## Why Generic LLMs Fall Short for Specific Use Cases

An off-the-shelf LLM is trained on general internet data. If your organisation has proprietary data — internal documents, records, domain-specific knowledge — the model has never seen it and cannot answer questions about it. The analogy used: a civil engineer asked about computer science has no knowledge of it because he was never trained on it. The same logic applies to LLMs.

Two solutions exist: RAG (inject information at query time) or Fine-Tuning (retrain the model to absorb the information). RAG is always the starting point because fine-tuning is far more compute-intensive.

## Compute Comparison: RAG vs Fine-Tuning

Fine-tuning a 70B parameter model requires roughly 3–4 × A100 GPUs (each with 80 GB VRAM) because you need memory not just to store the model but also to hold the weights being updated, gradients, and optimizer states simultaneously. It is expensive but a one-time exercise.

RAG adds only about 0–1% extra compute on top of normal inference. You are not touching the model at all — you are just enriching the prompt.

## The RAG Mental Model

Think of it this way: fine-tuning is like teaching a civil engineer computer science so the knowledge lives in his brain permanently. RAG is like handing him a reference book at the time he needs to answer — he reads from it and responds.

In RAG, the "book" lives in a **vector database**. You never retrain the model. You retrieve relevant passages and pass them inside the prompt alongside the query.

## Vector Database

A vector database stores text in two parallel forms: text chunk and vector.

When a query comes in, it is also converted into a vector, then compared against all stored vectors to find the closest matches. The corresponding text chunks (not the vectors themselves) are what get sent to the LLM.

## The Full Pipeline

**Ingestion (done once):**

1. Load source documents (PDFs, CSVs, etc.)  
2. Extract all text  
3. Break text into smaller **chunks** (typical sizes: 1024 or 2048 tokens — there is actual research on optimal chunk sizing)  
4. Run each chunk through an **embeddings model** to get its vector  
5. Store (chunk text \+ vector) in the vector database

**Query time (done on every user question):**

1. Convert the user's query into a vector using the same embeddings model  
2. Run similarity search — compare query vector against all stored chunk vectors  
3. Retrieve the top-K most similar chunks (e.g., top 5\)  
4. Construct the final prompt: system instruction \+ retrieved chunks \+ user query  
5. Send to LLM → get grounded response

## Why the Same Embeddings Model Must Be Used for Both

The query vector and the document chunk vectors must live in the same vector space to be meaningfully comparable. If you embed documents with one model and queries with another, the similarity scores become meaningless. Consistency across ingestion and retrieval is mandatory.

## Hybrid Search: Vector Similarity \+ Keyword Matching

Pure vector similarity has a weakness with **sparse information** — names, locations, unique identifiers that appear only a few times in the document. The model may use pronouns or indirect references most of the time, so vector similarity alone might miss those chunks.

**Keyword retrieval** complements this by doing exact or near-exact keyword matching between the query and the chunks — similar to a SQL WHERE clause but on words instead of structured fields.

The session used a **custom hybrid retriever** combining both:

* Vector retriever: finds chunks semantically similar to the query  
* Keyword retriever: finds chunks sharing important keywords with the query  
* The two result sets are merged (AND or OR logic, configurable)

## Encoder vs Decoder — Why a Separate Embeddings Model Is Needed

GPT-style LLMs are decoder-only architectures. They are designed to predict the next token, not to produce a fixed vector representation of input. They use cross-attention (for mixing input and output sequences) rather than self-attention alone.

An encoder model (like BERT variants) is what produces the embedding — it processes the full input bidirectionally and outputs a dense vector representation of meaning. For RAG, you need an encoder-style model dedicated to embedding, separate from the generative LLM.

## Chunk Size and Top-K Are Tunable Parameters

* **Chunk size:** Larger chunks carry more context per piece but reduce retrieval precision. Smaller chunks are more precise but may lose surrounding context. A paper on arXiv ("Chunk Size Optimization" / "Rethinking Chunk Size for Long Documents") covers optimal sizing for different LLM context windows.  
* **Top-K:** How many chunks to retrieve. More chunks \= more context but risks hitting the context window limit and adding noise. Typical starting values are 5–10.

## What the Code Actually Does

building a Bhagavad Gita Q\&A app using:

* **Google Gemini** as the LLM (with a GCP API key, $300 free credit available)  
* **embeddings-001** as the embeddings model (Gemini-compatible)  
* **LlamaIndex** as the orchestration framework  
* **Streamlit** for the UI

Key code components:

* Dependencies installed via pip in Colab  
* PDF loaded, text extracted, chunked, embedded, stored in an in-memory vector index  
* Custom retriever combining vector index and keyword table  
* Query engine wiring together the retriever and LLM (response synthesizer)

When asked "Who is Prashant?" (a name that does not appear in the Gita), the system correctly returned that it could not determine the answer — because no similar vectors were found, no chunks were returned, and the LLM had no context to work with.

# Lecture 9

## What Is an Agent?

An agent is just an automated workflow. It is a process that takes action by itself based on instructions you have defined. Whether it uses an LLM or not, it is still an agent. The term has been over-hyped, at its core, multi-agent AI is just multiple automated processes chained together, some of which happen to use an LLM as one of their steps.

## CrewAI — The Framework Used

CrewAI is a Python library for building multi-agent pipelines. Other frameworks exist (LangGraph, LangDroid) but CrewAI was chosen because the code is readable and the abstractions are clean. It is a startup product, not an established industry standard — the space is still evolving rapidly.

Three API keys are needed to run the session's code:

* **Google Cloud Platform (GCP) key** — for Gemini LLM access  
* **Google Custom Search key \+ CSE ID** — for the Google Search tool  
* **CrewAI key** — from the CrewAI website

## The Two-Agent Pipeline

The session built a research-and-writing pipeline with two agents and two tasks:

**Agent 1 — Researcher**

* Task: Search Google for information on a given topic  
* Tool used: Custom Google Search tool (calls Google Search API)  
* The LLM is specified for compatibility but is not doing the actual searching

**Agent 2 — Writer**

* Task: Take the research output and summarise it in bullet points  
* Tool used: LLM (Gemini)  
* No external tool — purely LLM-based generation

The analogy used: in a law firm, a junior associate goes to the library and pulls relevant case files (researcher), then a paralegal drafts the written brief (writer), and the senior attorney reviews it. Each role is a separate agent with a separate task.

## How Agents and Tasks Are Defined

Everything is written in plain natural language — no special syntax for the task instructions themselves. Each task has:

* **Description:** Natural language explanation of what the agent should do  
* **Expected output:** What format or content the result should take  
* **Tools (optional):** External services the agent can call (e.g., Google Search, email API)  
* **LLM (optional):** Which language model to use for generation steps

Tasks are collected into a list and the crew is told to run them sequentially (or in parallel if designed that way).

Chaining More Agents — The Extensible Pattern

The two-agent setup is just a starting point. You can keep adding agents and tasks in sequence:

| Step | Agent | Task | Tool |
| ----- | ----- | ----- | ----- |
| 1 | Researcher | Search and gather information | Google Search API |
| 2 | Writer | Summarise into structured content | LLM |
| 3 | Reviewer | Check quality and flag issues | LLM |
| 4 | Rewriter | Apply reviewer's corrections | LLM |
| 5 | Sender | Deliver final output | Email API |

Each additional step just requires a new agent definition, a new task definition, and appending both to the list. The pipeline handles the rest.

## Parallel vs Sequential Processing

By default, tasks run sequentially — output of one feeds into the next. CrewAI also supports parallel execution where two tasks run simultaneously and their outputs are merged before passing to the next step.

A good use case for parallel: scraping two competitor websites at the same time for price comparison, then passing both results to a single writing/analysis agent.

## Connecting to Any Data Source

The Google Search tool used here is just one example of an external connector. You can swap in or add:

* Your own relational database (pass the schema to the agent, have it write and execute SQL queries)  
* A vector database (same as the RAG session)  
* An email API (to read or send emails)  
* Any service with an API and authentication credentials

The pattern is always the same: define the tool, pass credentials, reference it in the relevant task. The instructor demonstrated writing SQL queries via an agent — it works well for simple schemas but becomes unreliable when schemas are complex.

## How This Relates to RAG

The distinction is worth being clear on:

| RAG (previous sessions) | Multi-Agent (this session) |
| ----- | ----- |
| Information pre-loaded into a vector DB | Information fetched live via Google Search |
| Retrieval happens at query time from stored vectors | Retrieval happens by calling an external API |
| Single agent, single task | Multiple agents, multiple tasks chained |
| LLM answers from retrieved chunks | LLM writes/summarises from live search results |

Both are forms of grounding an LLM with external information. RAG is better for controlled, private, or static knowledge bases. Multi-agent search is better for live, public, or open-ended research tasks.

## Caching — An Important Optimisation

One participant raised a good question: if the same research is being fetched every time the workflow runs, why not store it? This is called **caching**. By storing the research output after the first run, subsequent calls with the same topic skip the search step entirely and go straight to writing (or whatever downstream task). This cuts latency and API costs significantly. Left as a suggested exercise.

## What Agents Cannot Do Well

* **Agents do not remember previous runs.** Each workflow execution is independent. There is no memory across sessions unless you explicitly pass history as context.  
* **LLM outputs are probabilistic.** Even with identical inputs, two runs may produce slightly different outputs. The Google Search results will be the same, but the written summary may vary.  
* **Do not automate financial actions.** The instructor shared a personal anecdote about accidentally being charged $800 by clicking register on a college event. Giving agents access to payment credentials is genuinely dangerous.  
* **Agents sometimes stop following instructions.** There have been documented cases of agents deviating from defined instructions. The simple safeguard: do not automate irreversible actions, and always keep a human checkpoint before anything consequential happens.  
* **Copyright risk.** LLMs are trained on others' data. Content generated by LLMs and published without attribution or rights-checking is legally risky. Regulators are already pursuing this.

## The AGI Caveat

The instructor mentioned attending Snowflake Summit 2025 where Sam Altman discussed AGI. The broader point: do not become too dependent on these tools. Use agents to make your life easier, not to replace your own thinking entirely. The analogy — if agents start communicating with each other in Morse code or ultrasonic frequencies and stop following human instructions, the only solution is to pull the plug. Keep humans in the loop for anything consequential.