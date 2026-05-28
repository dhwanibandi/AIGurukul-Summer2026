## LEC 1 AND 2

What is Generative AI?
GenAI = AI that creates new content (text, images, code, audio) rather than just classifying things
- LLMs are the most prominent class — they deal with language/text,images
- Examples: GPT-4 (OpenAI), Claude (Anthropic), Llama (Meta), Falcon
- Both commercial and open-source models exist

LLM = Large Language Model(probabilistic models)
- Made of billions of parameters (weights in a neural network)
- Trained on hundreds of billions to trillions of tokens (text)
- Because of scale, they learn to generalize — one model can do many tasks

Pre-training and Fine-tuning
- pretraining is generic data whereas fine tuning is more specific(can unlearn things in fine tuning)

Transformers:
- Scalability — scales to billions of params efficiently
- Parallelism — processes all words at the same time, not one-by-one
- Long-range context — can link a pronoun to a noun 50 words away
- Produces more coherent, accurate text than older architectures
 
Attention Mechanism:
when processing word X, which other words in the sentence should I pay attention to?
- Every word is compared against every other word — a relevance score is computed
- Higher score = more influence on the current word's meaning
- Multi-head attention = multiple attention computations run in parallel, each learning different relationships (grammar, meaning, who-refers-to-who, etc)
 Example - 'She went to the store and she bought milk' — attention helps the model link both 'she' references to the same person
         - we use the word 'bank' has two meaning depending on othwer words we interpret right meaning
 
Encoder and Decoder:
- Encoder(bert only encoder) — reads input text, converts to numerical vectors (embeddings) that capture meaning + context(good at understanding)
- Decoder(gpt only decoder) — generates output text one token at a time, using encoder embeddings + its own previous outputs (good at generating)
- both encoder + decoder(good at seq-to-seq tasks like translation, summarization)

How LLMs are Trained 
a) Next Token Prediction 
Model sees a sequence and predicts the next word
   example:  "Hannah is a ___"  →  sister / friend / marketer / comedian
-	Trained on billions of examples like this
-	Learns grammar, facts, reasoning, common sense — all just from predicting next words
-	Uni-directional (reads left to right only)

b) Next Sentence Prediction / NSP 
Model sees two sentences — does sentence B logically follow sentence A?
-	50% of pairs = real consecutive sentences from training data
-	50% = random, non-consecutive pairs
-	Teaches the model cross-sentence coherence

c) Masked Language Modeling / MLM 
A word in the MIDDLE of a sentence is hidden — model must predict it
   example:  "Jacob (MASK) reading"  →  loves / fears / enjoys / hates
-	Bi-directional — model sees words before AND after the mask
-	Makes BERT great at understanding tasks (not ideal for open-ended generation)

Model Size & Training Quality
Effect of size:
-	Bigger models = dramatically better, especially zero-shot and few-shot
-   Around 100B+ params, 'emergent abilities' appear — multi-step reasoning, analogies, in-context learning — things smaller models just can't do
-   Accuracy depends on size as well as how we train the model

Effect of training method (RLHF):
-	standard pre-training worse accuracy then fine tuning
-	RLHF teaches the model to follow instructions the way humans actually mean them
-	Algorithm: PPO (Proximal Policy Optimization) it keeps updates stable, doesn't deviate too much from old policy each step

NLP vs LLMs
-	Old NLP: rule-based, manual feature engineering, task-specific (one model per task)
-	LLMs: data-driven, automated feature learning, multi-task (one model for everything)
-	Old NLP: can't do zero-shot or few-shot learning. LLMs: can
-	Old NLP: low resource needs. LLMs: high (need GPU/TPU)
-	Dev time: Old NLP = medium effort per task. LLMs = high upfront, then LOW per new task
-   LLMs have a high initial cost but once you have the base model, adding a new task is cheap/fast

In-context learning: model adapts to the task from examples in the prompt — no weight updates needed

Types of prompting:
-	Zero-shot — just the instruction, no examples. Works for clear/simple tasks
-	One-shot — one example provided before the real query. Helps model understand expected output format
-	Few-shot — 3-10 examples provided. Best accuracy on complex/ambiguous tasks
-   Always try zero-shot first. If output isn't good enough, add few examples before trying fine-tuning

How ChatGPT Actually Works
 myth: ChatGPT searches the internet for answers
-	It generates text word-by-word, picking the most probable next token each time
-	All knowledge is from training data — there's a cutoff date
-	Hallucination — when the model confidently says something wrong, because it's optimizing for plausibility not truth
-   ChatGPT makes a series of guesses. That's why it can argue wrong answers as if they're completely true

    Model and Context Window Size
-	Context window size matters a LOT for real-world tasks
-	A bigger model with small context window can fail where a smaller model with large context window succeeds
-	Always check context window limits when choosing a model for your use case

## LEC 3 and 4
LLM Model — Sonar Reasoning (Perplexity AI)

Why Sonar Reasoning?
- Affordability:Identified as one of the most affordable reasoning models — a year of work costing $30 on Perplexity might cost $800–$900 on GPT
- Automatic Chain-of-Thought: Unlike standard models, Sonar Reasoning applies chain-of-thought reasoning automatically, without needing a specific prompt
- Model Behaviour: LLMs generate text based on probability distributions. They do not know facts — they predict the next word, which is why the same question can be rephrased differently every time

Why Responses Differ Every Time:
- LLMs are probabilistic systems — not databases. At every step, the model calculates a probability distribution over possible next tokens and samples from it. The same input can produce different phrasing, structure, and wording every run. This is normal, expected behaviour
- The model doesn't know what it's saying — it's sampling the most likely continuation of text based on training. Temperature and top-p control how adventurous that sampling gets

API Architecture:
Most developers never run large models locally. Instead, applications send requests to hosted APIs — the actual model runs on centralised GPU servers elsewhere. The local application communicates with the model over the network only

Prompt Engineering:
- A prompt is the set of instructions given to the model to control its output behaviour

The Four-Layer Prompt Pipeline:
- Modern AI systems combine multiple text layers before sending a request. The model processes all layers together as one combined prompt

Layers:
a) System Prompt - Fixed, hidden developer instructions — safety rules, behaviour limits, platform restrictions; priority is Highest 
b) User Prompt - What you type — tone, format, roleplay, output language; priority is User-set 
c) Context - Supporting info injected at runtime — PDFs, chat history, retrieved documents; priority is Injected 
d) Final Question - The actual task to solve; priority is User-set
- Real-world example: DeepSeek refuses topics like Taiwan because a system prompt enforces it — set by the company, hidden from users, and cannot be changed via the commercial UI. When you write your own code hitting the API directly, you can set both layers yourself

Persona Design:
- You can define User 1 (the AI) and User 2 (the student) to set the tone of the conversation
- Example: Setting a persona for Lord Krishna with instructions to reflect the serenity of a cosmic being and use Hindi/Sanskrit words
- Experiment:Change the prompt to different personas (e.g., Lord Ram) and observe how the AI rephrases its responses

Strict Prompt Constraints:
- Length: Limit responses to 5–6 sentences
- Grounding: Instruct the model to quote specific texts like the Bhagavad Gita or Ramayana

Temperature & Top-p:
Setting and its effects:
a) Temperature = 0 - Selects only highest-probability tokens. Conservative, consistent, deterministic
b) High temperature - Samples lower-probability tokens. More creative but higher hallucination risk
c) Top-p (e.g. 0.9) - Probability threshold limiting which tokens can even be sampled. Works alongside temperature

Context Window:
- Every model has a hard token limit. The entire conversation — system prompt + user prompt + context + response — must all fit within it
- Sonar Reasoning limit: 128,000 tokens = 96,000 words
- Token conversion: 4 tokens = 1 word
- Included in the window (context lenght):User Prompt + Question + Context + Model Response
- If exceeded: The model starts dropping or truncating information — causing lost memory, broken reasoning, and incomplete responses. Context management is critical in production systems

Adding Context & RAG:
- Beyond the system/user prompt and question, you can pass context(a paragraph, PDF text, book excerpt) so the model answers from that instead of guessing. This is RAG — Retrieval-Augmented Generation
- RAG:Provide the model with specific context (like a PDF or textbook) so it answers based on that data rather than general training knowledge. Prevents vague or gibberish responses to specialised questions
- Demonstrated by uploading a compiler-design book PDF and asking what is this course about? — the model answers from the document
- Fine-tuning: Training an existing model on custom data (e.g., Mahakum data) to create a specialised model. The developer has full control over the system prompt. Essentially transfer learning — expensive but powerful

RAG vs Fine-tuning Comparison:
a) What it does : RAG - Keep a general model, supply external text as context at query time; FT - Retrain/update the model's parameters on specific data 
b) Cost : RAG - Cheap; FT - Expensive — needs GPUs (like internalising a subject over years) 
c) Wins on : RAG - Knowledge & conciseness; FT - Conversational style 
d) Data needed : RAG - Low; FT - Lots of data to shine 
e) Best for : RAG - Quick development; FT - Specialised deployments 
- For most projects, prefer RAG. Fine-tuning costs significantly more. Building a model from scratch is infeasible — always use existing models + RAG or fine-tuning instead

##LEC 5

Why RAG Exists:
- LLMs rely entirely on their internal training data, which creates three key problems:
- Hallucination: Models generate confident but incorrect answers when they lack reliable knowledge
- Static memory: Training data is frozen — it cannot reflect private, proprietary, or constantly changing information
- Incomplete knowledge: The model may know a source partially or not at all (example: detailed content from the Bhagavad Gita)
- RAG solves all three problems — without retraining the model. Instead of generating from memory, the system retrieves relevant information from an external source and uses it to augment the generation process

Components of a RAG System:
- To implement a RAG framework, you need four things:
a) Language Model (LLM) - Commercial (e.g. GPT) or open-source — generates the final answer 
b) Embeddings Model - A separate model whose only job is converting text → vectors 
c) Vector Database - Stores vectors and enables fast similarity search 
d) Search + Chaining Logic - Orchestrates retrieval, ranking, and prompt construction 
- embeddings model is separate from the LLM — it handles vectorisation only

 Tokens and Vectors:
- These are often confused. The embeddings model handles a 3-step pipeline: Text -> Tokens -> Vectors  
a) Text to Tokens: The tokenizer splits text using a fixed vocabulary (~60,000 tokens). Words not in the vocabulary get broken into sub-pieces
   Example: punctuation ->punct + ation
b) Tokens to Token Numbers: Each token is replaced by its index (serial number) in the vocabulary
c) Token Numbers to Vectors: Token numbers are mapped into an N-dimensional space where similar or related words land close together mathematically
   Example dimensions for the word cat: domesticated, pet, fluffy, mammal — each dimension gets a numerical value
   Words with related meanings cluster near each other in this space
- Key distinction: Tokens = numerical indexes. Vectors = high-dimensional coordinates representing meaning

The Full RAG Pipeline:
A) Indexing Pipeline (Offline Preparation)
- Run once when setting up the system:
  Source data (CSV, HTML, Word, PDF…) -> Extract Text -> Split into chunks (paragraphs, sections, sliding windows) -> Embeddings model -> Vectors -> Vector Database
- Why chunking? - Large files cannot fit directly into prompts. Splitting documents into smaller chunks keeps token usage manageable and makes retrieval more precise
B) Query Pipeline (At Question Time)
- Runs every time a user asks a question:
User query -> Embeddings model (same model used during indexing) -> Query vector -> Similarity search in vector DB -> Rank results by similarity score -> Select top-K chunks -> Inject into prompt as context -> LLM generates grounded answer

Similarity Search & Ranking:
a) How Similarity Scores Work:
- Score = 1 → vectors are identical (perfect match)
- Score = 0→ vectors are completely unrelated
- Scores closer to 1 indicate stronger semantic similarity
b) Ranking & Top-K Selection
- Discard score-0 (irrelevant) chunks
- Among remaining chunks, rank by similarity score
- Select top-K (e.g. K = 5 or K = 10) highest-ranked chunks
- These chunks become the evidence fed to the LLM
- Why limit to top-K? Too many chunks would exceed the model's context window
c) Optional Filters
- Metadata filtering:Drop chapters or sections known to be irrelevant before search
- Score cutoff:Set a minimum similarity threshold to exclude weak matches
d) Search Algorithms
1) Similarity search - Finds chunks closest to the query vector (like Google's PageRank, but on vectors) 
2) Maximum Marginal Relevance (MMR) Balances relevance with diversity — avoids returning redundant chunks 

Feeding Chunks to the LLM:
- Once top-K chunks are retrieved, there are several strategies for consuming them:
a) Stuff - Dump all retrieved chunks straight into the prompt. Simplest approach 
b) Map-reduce - Process each chunk independently, then combine results. Good for large sets
c) Refine - Iteratively refine the answer chunk by chunk
d) Map-rerank - Score each chunk's answer, select the best
- Map-reduce, Refine, and Map-rerank are borrowed from big-data concepts (e.g. Hadoop map-reduce)

The Core Equation:
Final input to LLM = Prompt (instruction) + Query (question) + Context (top-K retrieved chunks)
Without RAG: LLM receives Prompt + Query only; Result- Answers from training data — may stray, hallucinate, or miss specifics 
With RAG: LLM receives Prompt + Query + retrieved Context; Result- Answer stays grounded in the exact supplied text — no room to wander 
- Example: Feeding only Chapter 18 of the Bhagavad Gita as context → the model answers only from that chapter, with no guesswork

Context vs RAG:
- Context = plain text inserted into the prompt manually
- RAG = the entire pipeline that retrieves, ranks, and injects that context automatically
- Passing context is part of RAG. RAG is the whole retrieve → augment → generate process

Enterprise Security Benefits:
RAG significantly improves data privacy for organisations:
- The vector database stays within your own infrastructure — you control it entirely
- Only the most relevant chunks are sent to the external LLM API — not your full dataset
- This minimises data exposure and reduces the risk of confidential information leaking
- Uploading full documents to services like ChatGPT is risky — you don't know if it's performing RAG or folding your data into its training (which would be irreversible and could expose it to others)

Garbage In, Garbage Out:
- Response quality depends entirely on supplying the right context
- The LLM can only be as accurate as the context it receives

RAG enables "chat with your document" experiences:
- Upload a document (PDF, resume, textbook, etc.)
- The system automatically chunks and vectorises it
- You ask questions — the system retrieves relevant parts of your specific file to answer
- Responses are grounded in your document, not generic AI training data
This is significantly cheaper to build using open-source models than relying on paid services like ChatGPT

##Lec 6
- RAG converts a document corpus into vectors stored in a vector database. When a user submits a query, it is also converted into a vector and matched against stored document vectors. The retrieved documents are then passed alongside the original prompt as context for the model to generate an answer
- The key point: RAG does not update or change the model itself. It simply feeds additional contextual information to the model at query time

Keyword-Augmented Retrieval (KAR) — An Alternative to RAG:
- Instead of matching vectors, KAR extracts keywords from both the user query and the document corpus. If the keywords in a document paragraph match the query's keywords, that paragraph is passed as context

Where RAG Falls Short:
- Vector-based similarity search has a weakness with sparse data — information that appears only once or twice in an entire document
- Examples:
a) A person's name like "Narendra Modi" mentioned once, then referred to as "Modi" or "PM" via pronouns
b) Employee IDs, serial numbers, unique location codes
c) Rare identifiers that get lost inside semantic averaging
- In these cases, vector similarity fails to pick the right context because the rare keyword doesn't dominate the embedding. KAR finds it by directly matching the rare keyword

When to Use Each:
- KAR is preferred for sparse datasets where critical identifiers appear infrequently
- RAG is preferred for general semantic retrieval across dense, rich corpora
- Production systems often combine both — semantic vector search for meaning, keyword matching for exact identifiers. This hybrid approach dramatically improves retrieval reliability

RAG vs Fine-Tuning:
- These are not competitors. They solve different problems and most production systems use both together
a) Model changes?: RAG - No — model is untouched; FT - Yes — parameters are updated 
b) Best for: RAG - Changing knowledge (facts, documents, data); FT - Stable behaviour (style, tone, persona) 
c) Cost to update: RAG - Cheap — just update the vector DB; FT - Expensive — requires retraining 
d) Speed to update: RAG - Instant; FT - Slow 

Vidyang Platform:
- RAG for knowledge: Study material keeps changing — new books, notes, YouTube transcripts, PDFs, XLS, CSV, DOC files. Re-fine-tuning every time new content arrives would be too slow and expensive. RAG handles this by feeding new information via the vector database at query time
- Fine-tuning for personalization: The persona and tone (speak like a professor, school teacher, or doctor) rarely changes. Since it is stable, it makes sense to bake it permanently into the model via fine-tuning

Instructor Personalities and the Big Five Framework:
- Fine-tuning for persona requires a structured way to define personality. The Big Five framework provides this:
A) Extraversion
B) Conscientiousness
C) Neuroticism
D) Openness
E) Agreeableness
- Four common instructor personas were identified, all mapped onto the Big Five dimensions. All personas score low on neuroticism and vary mainly in openness and agreeableness

Synthetic Data Generation for Fine-Tuning:
- Fine-tuning requires thousands of question-answer pairs per persona. Manual creation at that scale is infeasible
- The solution is synthetic data generation:
a) Use a large, sophisticated LLM that understands the Big Five framework to generate Q&A pairs automatically
b) Group the generated data by persona type, labeled accordingly
c) Use this synthetic data to fine-tune a smaller open-source model (e.g. LLaMA 3.1 70B)
- The open-source Big Five Chat Dialogues database can also be used as training data for this purpose. This approach reduces data collection costs heavily while producing realistic personality-consistent training examples

What Fine-Tuning Actually Does:
- Fine-tuning adjusts how the model responds more than it teaches new facts. It permanently modifies model behaviour — writing style, response structure, tone, formatting conventions, and personas
- Pushing too much new factual data through fine-tuning risks overwriting existing knowledge, a problem called catastrophic forgetting

Full Fine-Tuning vs LoRA:
A) Parameters updated: FFT - All; LoRA - Tiny fraction, often less than 1% 
B) RAM and compute: FFT - Very heavy — also stores a copy of old weights; LoRA - Around 80% less 
C) Forgetting risk: FFT - Higher; LoRA - Lower 
D) When to use: FFT - Only if you can afford it and manage forgetting; LoRA - Preferred for efficiency with large corpora 

How LoRA Works:
- LoRA stands for Low-Rank Adaptation and is a form of Parameter-Efficient Fine-Tuning (PEFT)
- Instead of updating every parameter, LoRA freezes the original model weights and inserts tiny trainable matrices into the network. Only a very small percentage of parameters actually get trained
- The LoRA rank hyperparameter controls what percentage of parameters are trainable:
- Low rank = fewer trainable parameters = faster and cheaper
- As rank approaches infinity, LoRA becomes functionally identical to full fine-tuning
- Full fine-tuning is therefore a special case of LoRA at maximum rank

Compute and Storage Requirements
- Fine-tuning large models is hardware-intensive. Here is why:
- 1 parameter = 4 bytes in 32-bit float format
- 1 billion parameters ≈ 4 GB of storage
- Training also stores optimizer state (Adam), gradients, and activations — roughly a 4×8×4×8 overhead multiplier
- Fine-tuning LLaMA 3.1 70B requires approximately 2× A100 GPUs totalling around 160 GB RAM
- Regular consumer PCs cannot handle this workload

Quantization:
- Quantization compresses model weights into lower precision formats to reduce memory and storage requirements
- Examples:
a) 32-bit float → 16-bit
b) 16-bit → 4-bit
- This reduces VRAM usage, storage requirements, and inference costs. The trade-off is a small accuracy drop, which is generally acceptable for most use cases

Evaluation Metrics:
- To determine whether RAG, full fine-tuning, or LoRA is performing best for a specific use case, proper evaluation metrics are needed. The right metric depends on what the system is optimised for
- HELM (Holistic Evaluation of Language Models) is a Stanford framework that profiles models across multiple dimensions:
a) Accuracy
b) Calibration
c) Robustness
d) Fairness
e) Bias
f) Toxicity

## lec 8
The Knowledge Gap in LLMs:
- Generic LLMs are trained on publicly available data. They have no knowledge of private company records, internal databases, or domain-specific information they were never exposed to
- Analogy: an off-the-shelf LLM is like a civil engineer. If you ask them computer science questions, they cannot answer — they simply were never exposed to that information
- There are two ways to solve this knowledge gap. - RAG and Fine Tuning

RAG vs Fine-Tuning:
RAG — External Knowledge Retrieval:
- RAG augments the prompt with relevant information retrieved from an external source at query time. The model does not memorise the data internally — it reads the retrieved context during inference
- Analogy: handing the civil engineer a computer science textbook and asking them to find the answer by reading it
Fine-Tuning — Internal Knowledge :
- Fine-tuning retrains the model so it internalises the information by updating its weights directly. The model permanently learns patterns from the custom dataset
- Analogy: sending the civil engineer back to university to take data structures and operating systems courses

Compute and Infrastructure Trade-Offs:
a) When compute happens: RAG - Inference time only; FT - Training time 
b) Compute cost: RAG - Around 0.1% extra overhead; FT - Extremely expensive — one-time 
c) Hardware needed: RAG - Minimal; FT - 2× A100 GPUs (~160 GB) just to load a 70B model, plus 3–4 more GPUs to store weights during training 
d) Knowledge storage: RAG - External vector database; FT - Inside the model itself 
e) Infrastructure needed: RAG - Vector DB, embedding pipeline, retrieval system; FT - None after training 
f) Update frequency: RAG - Instant — just update the vector DB; FT - Requires full retraining 
- For most projects, start with RAG. Fine-tuning is a compute-heavy, one-time exercise reserved for stable behavioural patterns that rarely change

Data Preparation for RAG:
- Large files exceed token limits and cannot be passed directly into prompts. The document must go through a preparation pipeline first
- Source documents (PDF, Word, CSV, TXT, HTML...) -> Extract text -> Chunk the text (commonly 1024 or 2048 tokens per chunk) -> Pass through embeddings model -> vectors -> Store text chunks + vectors in vector database -> Chunk size matters — there are dedicated research papers (e.g. "Chunk Size Optimization") on finding the best chunk size for specific LLM context windows. Too small and you lose context; too large and you waste token budget

Hybrid Retrieval — Vector + Keyword Search;
- Pure vector similarity search is often not enough, especially for sparse information — rare names or identifiers that appear only a few times in a document and get replaced by pronouns elsewhere. The vector for "Modi" and "the Prime Minister" may not score high enough to surface the relevant chunk
- A robust retrieval system combines two approaches:
Vector Similarity (Semantic Search):
- Converts the user query into a vector using the same embeddings model used during indexing
- Compares the query vector against stored document vectors
- Rank-orders by similarity score and retrieves the top-K closest chunks (e.g. top 4 or 5)
- Matches on meaning, not just exact words
Keyword Similarity:
- Extracts important words (mainly nouns) from both the query and each chunk
- Drops stop-words, pronouns, and prepositions
- Matches based on keyword overlap — no understanding of meaning, purely exact-match
- Catches rare identifiers that semantic search misses
Combining Them:
-A custom retriever class combines both sets of results using an AND or OR condition. This hybrid approach dramatically improves retrieval reliability across both dense and sparse information

Encoders vs Decoders — Why Embeddings Need an Encoder:
- This distinction matters because you cannot use just any model to generate embeddings
- Most LLMs (GPT, LLaMA) are decoder-only models. Their job is to predict the next token. They never directly output a vector, even though they encode internally
- Decoders use cross-attention, which mixes input and output information — useful for tasks like translation but not for generating standalone embeddings
- Encoders use self-attention only. Their job is to understand language and produce a numeric vector representation of the input text
- For RAG, you need an encoder to convert both documents and queries into vectors. The query and the documents must use the same embeddings model, otherwise the vector spaces will not match

Agents — RAG as Agent:
- RAG is considered the simplest form of an AI agent. An agent is just an LLM integrated with one or more external services or tools
- In a RAG system, the LLM interacts with a vector database to fetch information
- More advanced agents interact with external APIs to take real-world actions — booking a ticket, sending an email, querying a live database
- The same principle applies: the LLM is the reasoning core, and tools extend what it can access or do

Limitations and Next Steps:
- The hybrid vector + keyword retrieval approach will eventually hit limits. When it does, the next steps to explore are:
a) Dense retrieval methods beyond standard similarity search
b) Alternative vector databases
c) Larger top-K values
d) Chunk size tuning based on your specific LLM context window
e) Research into chunk size optimisation papers for your use case

##lec 9
What an Agent Actually Is:
- An AI agent is essentially workflow process automation. It is not magical autonomous intelligence or something out of the ordinary. An agent is a worker that takes an action by itself — automating tasks that humans would otherwise do manually
- Key distinctions:
a) A standard chat interface can only respond to one instruction at a time
b) An agent can chain multiple tasks together into a pipeline
c) A multi-agent system assigns different responsibilities to different specialised workers
d) Not every agent in a workflow needs to use an LLM — some agents rely entirely on standard scripts or external APIs
- The LLM acts as the core reasoning brain for certain parts of the workflow, but it is just one component among many

Core Components of an Agentic System:
Agents:
- Each agent is defined with a specific role, much like hiring an employee for a particular job. You give each agent:
a) a role (what kind of worker it is)
b) a goal (what it is trying to achieve)
c) tools (what external capabilities it can use)
d) task instructions (plain natural language describing what to do and what the expected output looks like)
Tasks:
- Tasks are written in plain natural language. Each task has a description of what to do and a definition of the expected output. Tasks are assigned to specific agents
Tools:
- Agents are equipped with external tools to execute their tasks
- Examples:
a) a researcher agent equipped with a custom Google Search tool to pull live web data
b) a writer agent using the LLM directly to process and summarise retrieved data
c) an action agent with access to email APIs or database connectors
Example Multi-Agent Workflow:
- A typical chained pipeline looks like this:
- Research -> Write -> Review -> Rewrite based on review -> Send email
- You can have N agents and N tasks chained in any sequence. Each new action that requires an external service (like sending email) needs its own API key and authentication

Researcher Agent:
- Gathers raw information using tools like search APIs, databases, or retrieval systems. The LLM itself does not browse the internet automatically — it triggers external tools to do so
Writer Agent:
- Converts raw research into structured content — summaries, bullet points, reports, or formatted documents. Uses the LLM to process and synthesise the retrieved information
Reviewer Agent:
- Critiques the output from the writer. Checks for clarity, formatting, accuracy, and rule compliance
Action Agent:
- The final agent in the chain interacts with external APIs — sending emails, updating databases, triggering downstream workflows

Sequential vs Parallel Execution:
Sequential:
- Tasks happen one after the other. Each agent waits for the previous one to finish before starting
- Example: Research -> Write -> Review -> Rewrite -> Send Email
Parallel:
- Tasks run simultaneously. Multiple agents work at the same time and their results are merged downstream
- Example: two research agents scrape different websites in parallel, then their results are fed into a single comparison or filter step. This improves speed significantly
- You can also insert a filtering layer between research and writing — an LLM picks only the relevant text from the scraped results before passing it to the writer, similar to how RAG injects context

RAG vs Multi-Agent Search:
- These serve different purposes and can be combined.
a) Structure: RAG - Single agent, single task; MAS - Multiple agents and tasks chained 
b) Information source: RAG - Pre-loaded static vector database; MAS - Live internet search via Google or other APIs 
c) How info is used: RAG - Retrieved chunks injected into the prompt; MAS - Passed to writer task, LLM summarises — whole workflow re-runs each query 
d) Best for: RAG - Private documents, offline corpora; MAS - Live, up-to-date information 
- Uploading the Bhagavad Gita to a vector DB means the system only answers from that stored text. The agentic search demo instead dynamically fetches fresh results from the internet and passes them to the writer agent to summarise

Advanced Patterns:
Agents Querying Databases:
- Agent 1 reads the database schema and writes a SQL query
- Agent 2 executes the query against the database
- Agent 3 uses the result to generate an answer
- This removes the need for a dedicated data engineer for simple queries, though it can be unreliable on complex schemas
Filtering Layer:
- An LLM-based filtering step can sit between research and writing, selecting only the relevant portions of scraped content before passing it downstream — essentially RAG-style context injection inside an agentic workflow
Caching:
- Caching stores prior results so the system skips re-running expensive research steps on repeated or similar queries
- This reduces API costs significantly by reusing stored outputs instead of repeating external calls
No Memory Between Runs:
- By default, each run is fully independent
- The agent has no memory of previous conversations or runs
- To maintain continuity, the full conversation history must be passed back in on each new run

Output Variability:
- Agent systems still rely on probabilistic LLMs
- Even when the Google search results are identical across two runs, the LLM's text generation will produce slightly different wording each time. This is expected behaviour, not a bug

Do Not Over-Automate:
- Autonomous systems require strict human oversight. Key dangers:
a) LLMs can hallucinate, misinterpret instructions, or execute unintended actions
b) A bad prompt loop can trigger repeated transactions or runaway automation very quickly
c) Never allow unrestricted automated access to financial systems, payment methods, sensitive credentials, or authentication tokens
d) Never hardcode credit card or payment automation into an agent workflow — agents can accidentally register for expensive services or make unintended purchases

The Ultimate Failsafe:
- If an agentic system becomes too complex or stops responding to human instructions, the simplest solution is to shut it down. Do not become so dependent on an automated system that you lose the ability to override it. Human oversight must always be preserved


