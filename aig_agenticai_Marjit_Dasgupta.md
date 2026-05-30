# Agentic AI

---

# Lecture 1

## Core Concepts: What is an LLM anyway?
The Generative AI boom has happened at an insane pace. The very foundation of this shift are on LLMs. Structurally speaking, LLMs refer to neural networks which use billions or even hundreds of billions of parameters. 

As opposed to the conventional NLP methods that focused heavily on rule-based engineering, generative AI uses purely data-driven techniques. Once fed with large amounts of unstructured texts, the network will automatically generalize and learn from them, thus allowing the same core model to perform a variety of tasks, including but not limited to:
* Sentiment analysis by analyzing the mood from user inputted text.
* Summarizing long and bulky pages of case study.
* Translating text effortlessly between various languages without using any dictionary.
* Composing functional software codes.

## The Logic: LLMs are just guessing games
LLMs are nothing but guessing games
One point cannot be overstated enough: these models are not knowledge bases. Rather, they know nothing for sure. They are purely probabilistic in nature.

What happens when you give an input? The model is not pondering what the right answer is. It merely calculates the probability that the next token or word is the one with the maximum probability given your previous input.

For example, in case of the probability of "blue" being the next word is 91%, then in that case, "blue" is going to come out as the prediction.

From a technical perspective, the algorithms and neural networks used by these models function like high-order polynomial regression equations. Mathematically speaking, the approach makes perfect sense, especially from an economics point of view. But here is the thing, it does not understand the ground truth of whatever comes out. If the model has been fed inaccurate data, or if some probabilities skew the results a certain way, then it is going to lie and say something that is entirely false.

If you look under the hood, the neural networks running these scripts are essentially functioning as high-order polynomial regression models. From an economics standpoint, this makes complete sense. It is predicting an outcome using mathematical weights and coefficients. The only difference is it's scaled up to billions of parameters instead of a handful of variables. Because it operates on pure probability, it doesn't understand ground truth. If its training data contains bad information, or if the math swings a certain way, it will hallucinate and confidently state things that are completely false.

## Historical Milestones: The Shift to Transformers
Before 2017, the NLP space relied on sequential processing models like Recurrent Neural Networks (RNNs). These architectures had a massive bottleneck: they could only parse text one single word at a time. This mechanical setup made training cycles crawl, and the models completely lost track of context over long documents.

Everything changed when Google engineers dropped the "Attention is All You Need" paper, launching the Transformer blueprint. This architecture completely eliminated sequential blockages through a few core structural changes:
* **True Parallel Processing:** Instead of reading word-by-word, transformers ingest entire data blocks all at once, which radically accelerated training speeds and scalability.
* **Long-range Dependencies:** They capture context across long gaps. A concept mentioned in the opening paragraph can still actively influence how the model processes a sentence down in the middle of the document.
* **Attention Mechanism:** This is how the system handles context tracking. It maps how words relate to one another in a sentence and assigns mathematical weights. That is how the model distinguishes whether "bank" means a river bank or a financial institution—it analyzes the surrounding words to settle the meaning.

## Turning Words into Math (Vectors)
Obviously, computers cannot understand human language. As such, in order to create language models that understand our language, the computer has to convert human language into numbers or alignments called vector/embeddings.
* Human language is converted to coordinates in a multi-dimensional mathematics space.
* It recognizes hidden relations based on counting how many times certain words occur together in the input text.
* Because language is converted into pure geometry, similar meanings wind up clustering close together in space—think *cat* and *kitten*—while totally unrelated concepts get pushed far away.
* This unlocks actual vector math on concepts. The classic example from AI research looks like this: 
$$\text{King} - \text{Man} + \text{Woman} = \text{Queen}$$

## But how exactly do they do it?
Learning is divided into two clearly distinguished phases:

**1. The Pre-training Phase:**
In other words, it is the childhood phase for the model in which basic knowledge is learned. You start with an untrained model and train it on a massive slice of the internet. It learns about grammar, syntax, and word associations using masked language modeling, i.e., filling in the blank space within a sentence, for example, "Jacob `[MASK]` reading." The training loop masks out particular words to make the model predict what should go there. The whole neural network iterates through the internal parameter changes until its prediction matches the input data.

**2. The Fine-Tuning Phase:**
When your base model already knows how to use language properly, you give it a job. You train it further on carefully selected datasets, such as finance-related reports or medical health data, and make the model an expert at handling a particular task. In this case, the most crucial point is that fine-tuning will not increase the baseline file size or expand your model's capacity by adding new parameters. You optimize the current architecture to achieve better performance on a certain target task.

## Open Source vs Commercial Ecosystems
The pace of development within the open source world has been absolutely insane recently. If one were to compare the number of models in places such as Hugging Face, they would find that the amount of open models vastly surpasses the number of closed ones made by commercial corporations. If a person were creating an app where data privacy was a concern, then open-source was usually the only way to go.

---

# Lecture 2

## Core Logic: Its All About Probability
In line with our discussion from last lecture, keep in mind that LLMs do not reason anything at all. Instead, they use probability to determine their next-word distribution.
* If you enter to the model “the color of the sky is”, it calculates probabilities for every word in its dictionary.
* It chooses the highest possible scoring word (such as "blue") and prints it out.
* Even when it produces a long essay of several pages, it keeps doing the same thing again and again.

## Word Vectors Deep-Dive
To map text into numbers, models project words across thousands of high-dimensional coordinates. Standard sizes hover around 512, 4096, 8192, or even 16,000 distinct dimensions.
* The model uncovers unique patterns on its own during training, creating vector dimensions that represent specific traits (like sorting whether an object is a domestic pet or a wild animal).
* Humans do not define these categories. The model uncovers them on its own through data ingestion; humans only define the raw vector size constraint.
* If you ramp up the number of vector dimensions, the model gets a much finer tool for grouping related terms. The catch is that it hits your system with higher storage and compute requirements.

## Inside the Transformer: Encoders vs Decoders
The classical design of the transformer network has two types of operating components, although most web-based models only require one half of them:
* **Encoders:** This model reads and processes text by examining the surrounding context on both sides of the word. They are used for advanced processing tasks, such as sentiment analysis or entity extraction. (Previous models such as BERT were **encoder-only**).
* **Decoders:** These components perform pure text generation. They process text in a unidirectional way, analyzing only the left context to predict the next word.
* As a result of this incredible prediction power in conversational text, almost all mainstream chatbots available nowadays are made solely of decoders without any need for an encoder.
* Then you have **Encoder-Decoders:** These models analyze the input sequence and map it to a new output sequence. Thus, they are very efficient for tasks such as language translation.

## Scale vs Precision
Generally, increasing the number of parameters in the model, scaling it from a couple hundred million parameters to tens or even trillions of parameters, guarantees you better accuracy levels. However, scale is not everything; the way in which training is done is as important as the model's scale itself.

For example, a smaller model, but with very fine-tuned training processes (such as InstructGPT or Supervised Fine-Tuning), is always able to perform better than a much bigger model that was poorly trained.

## Prompt Engineering: Zero-Shot vs Few-Shot
The way you write your prompt has a direct bearing on the results from your model.
* **Zero-Shot Learning**: It implies asking the model a question directly without providing it with examples as to what your desired output format should look like.
* **One-Shot Learning:** Asking the question after providing the model with just one single example of how the output should look like when the question is asked.
* **Few-Shots Learning:** Asking questions after providing a few structured examples.
* Using few-shot prompts dramatically increases the chances of getting accurate responses. It is rather bizarre but it is not mandatory for the prompts to have anything at all to do with your present subject of interest because the model learns from the *structure* of the prompts.
* 
---

# Lecture 3

## Preparing for Coding Environment
This session was dedicated to a very hands-on lab where we learned how to interact with a live reasoning model through its API using Google Colab.
* **Forking vs. Cloning:** As usual, when you work with Git repositories, first *fork* a project, which means creating a new repository based on a public one hosted on someone else's account, then *clone* it to your GitHub account to be able to modify it. Afterward, you clone the project to your local environment for execution.
* **Google Colab:** An online platform for running Python code via Jupyter notebooks directly in your web browser without installing anything locally.
* **Perplexity AI:** This lab was set up to interface with Perplexity's API using their reasoning model **Sonar Reasoning**. It should be noted that Perplexity does not develop such models, but only fine-tunes pre-existing open-source models. By default, Sonar models perform chain-of-thought processing, i.e., reasoning within itself before producing output.

## The Prompt Hierarchy
When you interact with an LLM through an API, the final text payload hitting the server is a combined stack of several distinct layers:

| Prompt Layer | Priority | Operational Role |
| :--- | :--- | :--- |
| **System Prompt** | **Highest** | Coded by the developers; completely invisible to the end user. This sets the fundamental safety boundaries, operational rules, and core persona limits. It will always override user inputs if there is a conflict. |
| **User Prompt** | **Medium** | The programmatic framework you build to control the AI's persona or formatting constraints (e.g., forcing the AI to speak as Lord Krishna and hard-capping responses at 5 sentences). |
| **Question** | **Standard** | The active variable input typed by the end user at that exact moment (like "What is motion?"). |
| **Context** | **Grounding** | Background documentation, data tables, or raw files injected alongside the prompt so the model extracts facts from a specific source instead of guessing from memory. |

## The Context Window Limit
You can't just pass infinite data into an LLM. Every model has a hard memory limit known as the **Context Window**, which is measured in tokens (roughly 4 tokens for every 3 words). 
* **Crucial detail:** The context window evaluates the sum total of *everything* passed in that single back-and-forth interaction. That means: 
$$\text{System Prompt} + \text{User Prompt} + \text{Context Docs} + \text{The User Question} + \text{The Generated Answer}$$
* If you dump too much data at the model and cross its processing ceiling, the system will break down. It will either drop the conversation completely, clip the text mid-sentence, or just print a single-word error. You have to actively budget your token intake.

---

# Lecture 4: API Code, RAG vs Fine-Tuning and Temperature

## Code Deep-Dive: What is inside an API?
To understand what is happening inside the URL endpoint (`model.run`), we opened up the Python backend script that speaks directly with Perplexity. We saw how the code structures data classes using object-oriented principles and relies heavily on Pydantic to handle input validation.

### 1. Data Structure Validation (Pydantic)
We construct an `Input` data frame using Pydantic's `BaseModel` structure. You can visualize this class as a clean, structured data container that bundles all incoming user variables safely before passing them to the processing logic. It explicitly validates parameters like the `rephrased_memories` text, the user's Perplexity `api_key`, the target `modelname`, and incoming chat profiles.

### 2. Traditional Logic Layer
Before hitting the AI server, traditional, deterministic code handles the data. For example, a basic `if-else` block scans the user's gender input:
* If `gender == "male"`, it assigns a tracking variable to `"prabhu"`.
* If `gender == "female"`, it assigns it to `"mataji"`.
This salutation is then dynamically slotted into the system prompt string. This highlights that real-world AI builds rely on plenty of standard, classic coding alongside the probabilistic LLM layer.

### 3. Headers and Payload Structure
The script bundles everything into two standard Python dictionary objects to fire a `requests.post()` call to the endpoint:
* **Headers:** Handles system validation via the standard key format: `"Authorization": f"Bearer {data.api_key}"`.
* **Payload:** Contains the model configuration metrics. It maps the designated model name and a messages array that packages both the hidden `system` prompt boundaries and the active `user` message.

### 4. Output Text Parsing
Because reasoning models process ideas internally, they output a lot of background thinking wrapped in custom tags. The script uses Python's `re.sub` string parsing utility to scan for those `<think>` segments and erase them entirely. Stripping these tags out keeps the user dashboard looking professional, ensuring only the polished final answer gets displayed.

## Architectural Case Study: Companion App
we did a walkthrough on an actual production app that was built with a **React** frontend and this specific architectural style for the backend. The app had several personas, where user data is analyzed via two different pipelines happening in the background:
* **Memories:** Logs of previous conversations, which are categorized and generate interface tabs such as *Favorites* and *Hopes & Goals*.
* **Diaries:** Another pipeline analyzing user behavior on a daily basis.
* **Important Feature in Architecture:** The logs generated through the Memories pipeline cannot be accessed by the conversational AI while generating the conversation. Instead, another LLM is employed behind the scenes to analyze logs and save them in the database only to track the interface of the user.

## Controlling Output: Temperature and Top P
LLMs extract words from a probability distribution, and we adjust their focus using two main configuration metrics:
* **Temperature:** Adjusts output randomness. 
  * `Temperature = 0` locks the model down into a completely deterministic pattern, forcing it to choose *only* the word with the highest mathematical score. This results in highly predictable, factual, and repetitive text blocks.
  * Higher settings force the model to sample words with lower probability rankings, making the output highly creative but heavily raising the risk of hallucination.
* **Top P (Nucleus Sampling):** Sets a cumulative threshold limit (like 0.9). The model list out the top words whose probabilities add up to 90%, and completely throws out the remaining bottom 10% options. Raising your temperature lets the model dip lower into this pre-filtered Top P pool.

## Maintaining Context through Chat Interfaces
Since LLMs have no notion of states, every request made to the API views you as a new visitor without any trace of memories of previous conversations. For you to pretend like there is a continuous dialogue, then you’ll have to make use of manual data transmission under the hood.

When you ask a question for the umpteenth time, your backend script will extract your **entire chat history from the database**, add your **newly composed query** to the bottom of this long string, and send the whole thing to the API request.

---

# Lecture 5: Retrieval-Augmented Generation (RAG)

## The Core Problem with Base Models
Relying entirely on a standard pre-trained model means you are at the mercy of whatever frozen data it picked up during its initial training lifecycle. If you try to quiz it on narrow company documents, specific class textbooks, or recent world news, it won't have the answers. It will either fail outright or fabricate an answer that sounds completely plausible but is factually detached from reality. To bypass this issue without the massive expense of training new model layers, we deploy a framework called RAG.

## What actually is RAG?
Think of a standard LLM as a student sitting for a closed-book exam relying strictly on memory. **RAG turns it into an open-book exam**. Before your prompt even hits the model, a retrieval layer scans a targeted database we control, pulls the exact relevant text blocks, and hands them to the model to read right then and there. The architecture breaks down into:
1. **The Language Model (LLM):** The generative engine handling text synthesis.
2. **The Embeddings Model:** A separate model that processes text strings into long lists of numerical coordinates called **vectors**.
3. **The Vector Database:** The storage unit where document vectors are indexed for fast mathematical lookups.

## The RAG Data Lifecycle
1. **Parsing and Text Splitting:** Raw documentation (like PDFs, spreadsheets, or raw source text) is loaded and broken down into minor, digestible snippets called chunks.
2. **Database Loading:** These chunks are run through an embedding model to calculate their mathematical coordinates and are saved as index points inside our vector database.
3. **Query Conversion:** When an end-user inputs a question, the system takes that string and runs it through the exact same embedding model to create a matching query vector.
4. **Vector Coordinate Matching:** The database runs similarity search equations to calculate where the user's query coordinates align with the stored document coordinates, outputting relevance scores between 0 and 1.
5. **Data Caps and Metadata Tuning:** To ensure we don't overflow the model's context window, we apply a Top K setting to only fetch a strict number of top-scoring text blocks. We can also append metadata tags to filter our searches to specific chapters or dates.
6. **Context Insertion and Synthesis:** The system collects the chosen document chunks, inserts them as raw context text right above the user's question, and routes the full prompt package to the LLM. The model scans the text and writes a response grounded strictly in the facts provided on the page.

## Advanced Retrieval Strategies
There are a number of advanced methods for chunk routing to the LLM:
* **Stuffing:** The most popular and straightforward option. It consists of sending all Top K chunks directly into the prompt block altogether.
* **Map/Reduce:** The process is completed independently for each chunk. First, chunks are sent to the LLM and an answer is generated for each of them ("Map"), and afterward, all separate answers are united and sent back to the model in one batch ("Reduce").
* **Refinement:** The chunks are processed sequentially. The first chunk is asked to receive an answer and then the initial answer together with the next chunk is asked.
* **Map/Reranking:** In this process, chunks are independently sent to the LLM along with a request to produce an answer accompanied by a confidence score.

## Enterprise Security Benefits
The act of copying and pasting sensitive internal company documents or confidential research documents within the consumer AI portal presents huge security concerns since the foreign server collects all the inputs in order to build its model and hence will be able to get your confidential information.

When you employ a RAG pipeline, your document stays protected within your network on a secure vector database. For each search query made by the user, the system only transmits isolated snippets of text necessary to generate the answer without exposing your entire enterprise documentation.

---

# Lecture 6: Fine-Tuning, LoRA and Evaluation Metrics

## When Vector Search Fails: The Sparse Data Problem
Standard RAG relies heavily on dense vector semantic meaning, which breaks down when handling **sparse data**.
* If a specific name, serial code, or tracking ID shows up only once or twice in a 500-page book, or if a person is mostly referenced using pronouns ("he", "she"), a vector search will often fail to pull that paragraph because the overall semantic context of the block doesn't match perfectly.
* **The Solution: Keyword Augmented Retrieval (KAR):** Instead of calculating vector space coordinates, KAR runs direct keyword matching (essentially an automated CTRL+F search). It tracks exact word matches to guarantee the system grabs the specific paragraph containing that rare key term.
* **Best Practice:** The industry standard is moving toward a Hybrid Retrieval model. This setup layers a Vector Retriever together with a Keyword Retriever, combining their outputs so you capture both high-level conceptual ideas and rare pinpoint terms.

## Balancing between RAG and fine-tuning (The VidyaRang Study)
We have covered a case study that demonstrated the perfect equilibrium between both:
* **RAG manages fluctuating information:** The factual information, schedules, and even reference guides frequently fluctuate. It is simply not economically viable to fine-tune the model each time a new document appears due to the computational costs. This task is managed flawlessly through RAG, fetching the latest documents.
* **Fine-tuning manages behavioral consistency:** While the facts change all the time, the underlying behaviors and personality traits of a particular individual remain constant. Fine-tuning the model in perpetuity enables it to adopt the communication style of a specific persona (the particular teaching style, for example, based on the **Big Five Model** such as Extraversion and Openness).
 
## Synthetic Data Generation & Distillation
To run a successful fine-tuning script, you need thousands of highly curated Question-Answer training pairs demonstrating exactly how that specific persona should speak. Drafting these manually would take forever. 

Instead, developers leverage a massive, advanced closed model (like GPT-4) to synthetically generate huge batches of specialized training pairs. These synthetic pairs are then used to train a smaller, cheaper open-source architecture (like a LLaMA 70B model). This training process is called **Distillation**, allowing a lightweight local model to successfully mimic the capabilities and tone of a massive model at a fraction of the operating cost.

## Fine Tuning Hardware and Mathematics Demands
Fine tuning using full precision is an extremely costly procedure that demands enterprise-level hardware infrastructure:
* **Basic Storage Requirement:** 1 weight consumes 4 bytes of information (a 32-bit float). Thus, the storage requirement for a 70 Billion model becomes 280 GB only to be stored and left idle inside VRAM.
* **Active Training Multiplication Factors:** When a model undergoes active backpropagation training, the hardware storage space needed more than doubles due to the necessity to store additional math states like 2 Adam Optimizers (+8 bytes), Gradients (+4 bytes), and Activations (+8 bytes). This brings the active storage capacity required for a 70B model over 1.5 TB. Active fine-tuning requires many enterprise-level GPUs that give access to large amounts of fast VRAM.
* **Optimization via Quantization:** To reduce computational costs considerably, the developers use mathematical compression known as **quantization**. The technique converts heavyweight 32-bit parameters to lightweights such as 16-bit, 8-bit, and even 4-bit. This reduces the required hardware storage by 80%, allowing for running massive models on less expensive GPUs without any noticeable drop in performance precision.

## Parameter Efficiency: Full Fine-Tuning vs. LoRA
When running a script where you fully adjust every parameter weight of a multi-billion parameter model, there is an extremely dangerous phenomenon known as **Catastrophic Forgetting**. While updating the mathematical weights of the model through training for it to remember your new dataset, it risks overwriting all the essential core capabilities the model previously knew.

This problem is solved by employing the concept of **LoRA (Low-Rank Adaptation)**. It is a technique of **Parameter-Efficient Fine-Tuning (PEFT)**:
* **The Basic Concept:** LoRA involves **freezing** all the initial weights of the original model and never making changes to those mathematical parameters.
* **Adapter Matrices:** However, it also injects light-weight adaptable matrix adapters to the layers of the model.
* **Speed and Effectiveness:** With this, you drastically decrease the amount of adjustable parameters to less than 1% of the original architecture. You save GPU computational power significantly and train the model blazingly fast, without risking destroying its core intelligence permanently. (In fact, if you set the parameter of the LoRA adapter, $r$, to infinity, LoRA becomes a full fine-tuning technique).
 
## Benchmarking Performance: The HELM Framework
To remove guesswork when running fine-tuning loops or adjusting RAG systems, developers rely on objective engineering metrics rather than subjective analysis. The gold standard for this is Stanford University's HELM framework. HELM scales its evaluation across a deep multi-dimensional tracking profile, grading models on factual accuracy, performance calibration, structural robustness, response fairness, hidden data bias, language toxicity, and execution efficiency.

---

# Lecture 8: Hybrid Retrieval (Vector + Keyword)

## The Technical Execution Order
When executing a production hybrid search system using frameworks like LlamaIndex or LangChain, the architectural pipeline runs through a rigid, sequential structure:
1. **Load Documents:** Ingesting single or multiple PDFs/text repositories into the memory environment.
2. **Chunking:** Slicing the long raw texts into standardized sizes (typically 1024 or 2048 tokens). This happens *before* any embeddings are generated.
3. **Dual Indexing:** The chunks are cloned into two separate data tables simultaneously:
   * **Vector Index:** Each chunk runs through the embedding model (like Google's `embeddings-001`) to generate a vector and is stored in the vector database.
   * **Keyword Index:** Key nouns and core identifiers are parsed out of each chunk while common "stop words" (pronouns, prepositions) are completely discarded. These are mapped into a local keyword look-up table alongside the chunk ID. Setting a flag like `persist=True` saves these indexes directly to local disk storage so they survive environment reboots.
4. **Hybrid Custom Retriever:** A custom programming class combines both retrieval architectures, using specific parameter flags (like `similarity_top_k = 5`) to fetch the top text blocks from both semantic and structural layers.
5. **Response Synthesizer & Query Engine:** The response synthesizer merges the custom retriever logic with your active LLM configuration. This is encapsulated inside a high-level `query_engine` object. While it abstracts away the complex multi-dimensional search math under a clean, high-level code structure, it securely handles taking an input text query, vectorizing it, executing the hybrid search match, and packaging the grounded text directly to the LLM.

---

# Lecture 9: Multi-Agent AI with CrewAI

## Shifting Gears to Multi-Agent Workflows
Up to this point, engineering focus has centered on prompt structuring for a single chatbot interface. Modern application development is rapidly shifting into multi-agent design tracks. Instead of leaning on a singular monolithic prompt block to handle an entire project from start to finish, you build out an operational network of autonomous digital workers. These specialized components work as targeted agents, possessing individual toolkits, isolated goals, and the ability to route information among themselves on the fly.

To build and manage these workflows, we use a framework called CrewAI. It acts as an orchestra conductor, setting up the communication channels and processing queues between different language model nodes.

## Core Code Architecture Breakdown
Setting up a multi-agent workflow inside a Python notebook follows a highly standardized, sequential five-part structure:

### 1. Installation & Library Imports
Installing the `crewai` library along with any supplemental technical tool kits.

### 2. Setup and Authentication (API Keys)
To give your agents their core analytical "brains" and environmental tools, you must explicitly declare your API environment variables in the script:
* **Language Model Key:** Powers the core text reasoning and processing engine of each agent.
* **External Action Tools:** You must declare separate security tokens to grant your agents practical operational tools, such as the ability to hook into a Google Search API or read live tables from a cloud database instance.

### 3. Programmatic Agent Definitions
You programmatically construct your individual digital workers, explicitly establishing their job boundaries using specific parameters:
* **Researcher Agent:** Assigned a specific `role` and `goal` focused purely on crawling search APIs to scrape messy raw data from the web. Given access to the search tool.
* **Writer Agent:** Focused entirely on taking raw research logs and organizing them into a clean, well-structured text layout. Given access to the LLM core.
* **System Prompt Isolation:** Every agent receives its own distinct system prompt parameter behind the scenes. This strictly confines their persona, rules, and objectives, preventing them from stepping out of their designated job description.

### 4. Defining the Tasks (The Workflows)
You write out concrete **Tasks** using plain, natural language instructions and explicitly map them to a specific agent:
* The researcher is assigned the `research_task`.
* The writer is assigned the `write_task`.

### 5. Crew Assembly and Execution Loop
Finally, instantiated a `Crew` object, passing a list of your agents and tasks, and define the processing flow:
```python
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)
