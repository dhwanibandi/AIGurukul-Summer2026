# GenAI Master Notes (Combined)

## 1. Introduction to GenAI
Generative AI is rapidly growing, showcasing massive potential in automating routine tasks and aiding in creative content creation. 
* **Large Language Models (LLMs):** They are responsible for latest Gen AI innovations.
* **Scale:** They are trained on vast amounts of text data and contain billions (modern things like Gemini(2026) have trillions) of parameters, allowing them to learn and generalize deeply from diverse data.
* **Capabilities:** LLMs can perform a wide range of tasks including sentiment analysis, text completion, summarization, Q&A, translation, and even writing code.



## 2.  Transformers & Attention
The true revolution in Generative AI began in 2017 with the introduction of the **Transformer Architecture** (from the famous paper *"Attention is All You Need"*).
* **Parallel Processing**  Transformers allow for parallel processing making them highly scalable and faster to train.
* **Attention Mechanism:** This allows the model to focus on the most relevant parts of the input text when generating an output. For instance, it correctly links pronouns to nouns across complex sentences based on surrounding context.
###  Architecture: Encoders and Decoders
Transformers generally fall into a few architectural types:
* **Encoders (e.g., BERT):** These are designed to deeply read and understand text. They look at context from both the left and the right simultaneously. They are good for classifying documents, analyzing sentiment, and extracting information.
* **Decoders (e.g., GPT, Llama):** These are predictive engines. They only look at the left context (what has been said so far) to guess the next word. Because they are so good at generating natural text, almost every popular chatbot today is built on a decoder-only architecture.
* **Encoder-Decoder Hybrids (e.g., BART):** These take an input sequence digest / process it and map it to an entirely new output sequence. Good for translation

# 3. Model Input/Ingress
### The Vector Space
Before a model can process a word, that word goes through a tokenizer, which chops it up into smaller pieces, and then an embedding model, which turns those pieces into a high-dimensional vector. 
Because language is mapped as geometry, you can actually perform math on concepts. The most famous example in AI research is the equation: `King - Man + Woman = Queen`. By mapping meaning to numbers, the model learns the underlying relationships between concepts without ever needing a dictionary.

# 4. Pre-Training and Fine-Tuning
### Phase 1: Pre-Training (The Formative Years)
During pre-training, an untrained model is fed a large amount of data scraped from various sources. Generally results in a foundational model needing further processing.

### Phase 2: Fine-Tuning (Specialized Schooling)
To further tune the model we train it on smaller curated examples however this process is quite CPU intensive requiring quite a bit of expensive infrastructure.
Fine tuning can potentially result in forgetting previously learning stuff.

## 5. Interaction through Prompting and APIs
* **System vs. User Prompts:** The *System Prompt* acts as the backend rules and always has priority over the *User Prompt*, which is what we actually type.
* **Context Windows:** Models have a hard memory limit for a single interaction (e.g., 128k tokens). This limit must fit the system prompt, user prompt, any background context, and the model's answer.
* **Controlling Output:**
    * *Temperature:* Controls randomness. Low temperature = predictable, factual. High temperature = creative, but risks hallucinations.
    * *Top-P:* Restricts the model's choices to only the most probable words, cutting off wild guesses.
* **N-Shot Learning:** Giving the model examples before asking your question. Zero-shot (no examples), One-shot (one example), Few-shot (multiple examples—highly effective for teaching formatting).

# 6.  RAG (Retrieval-Augmented Generation)
* **RAG:** RAG gives the model an list of important context to read from before answering, instead of relying purely on its internal memory this is useful because it is not possible for large models to remember each and every internal detail so we need to feed them the relevant context to maintain accuracy and prevent hallucination.
* **The Pipeline:**
    1. Break your documents into smaller text chunks.
    2. Convert them to vectors using an embedding model and store them in a Vector Database.
    3. When a user asks a question, convert the question into a vector.
    4. Do a similarity search to find the closest matching chunks.
    5. Pass those top chunks along with the user's question to the LLM to get a factual, grounded answer.

# 7. Advanced Search & Fine-Tuning Tech
* **Hybrid Search (KAR):** Pure vector search (RAG) struggles with "sparse data" like specific names or employee IDs. Keyword Augmented Retrieval (KAR) matches exact words. Using both together (Hybrid Search) gets the most reliable results.
* **LoRA (Low-Rank Adaptation):** Full fine-tuning is incredibly expensive because it requires massive GPUs. LoRA freezes the main model and only trains a tiny fraction (<1%) of the parameters. It's cheap, fast, and prevents catastrophic forgetting.
* **Quantization:** Shrinks the model size by reducing the precision of the weights (e.g., from 32-bit to 4-bit). Saves massive amounts of memory with only a minor drop in accuracy.
* **Synthetic Data:** If you need 10,000 Q&As to fine-tune a model, don't write them by hand. Use a massive model (like GPT-4) to generate the training data, then use that data to train a smaller, cheaper open-source model. (Can be tweaked to be used for distillation attack)
# 8 . Agentic AI & Multi-Agent Systems
* **Agent** An agent is simply an automated workflow. It doesn't always need an LLM for every step. It's given a role, a goal, and tools to interact with the real world.
* **Multi-Agent Systems:** Instead of writing one giant prompt to do everything -> split tasks.
* For example, a "Researcher Agent" searches the web and passes raw data to a "Writer Agent" to summarize, who then passes it to a "Reviewer Agent".
* **CrewAI Framework:** Python library to build these systems. Define Agents, assign them Tasks, give them Tools (like search APIs or SQL access), and chain them together sequentially or in parallel.
* **Potential Safety Considerations:** Never give an autonomous agent unrestricted access to money, credit cards, or irreversible actions ( like deleting important files).
* Can implement user confirmation for safety like in general agents like OpenCode


**NSVIF** (First Look)
	**Approach Overview**
		LLM output is often erroneous.
		Normal LLMs require output verification
		- Semantic Verification ( Assess things like tone of response and ...)
		- Logical Verification
		Reduce into CSP ( Constraint Satisfaction Problem) --> Decompose into appropriate set of logical constraints -> Solve using available logic solvers.