# Agentic AI, Lec-1
## Language Models
These are probabilistic models which do not give an answer with complete certainty, they work on the input given to them and produce the answer which has the highest probability according to the model's calculation.

## Transfer training
Initially a model is trained by feeding it generic data which may be text or image based data. For further training and making a fine-tuned model, we first identify the field for which the model has to be used and then feed it domain or task specific data. The initial step is called **pre tuning** and the next step is known as **fine tuning**.

## Attention
When we train a model using large amounts of data, we should expect the language model to learn different dependencies between different data and learn to pay *attention* to specific data points depending on the input data. The model also goes through context depended learning which helps to correctly identify the same data on the basis of context, for eg. the same word may have different meaning in different situations.

## Vectors
To establish connections between data or for example words, they are represented as **vectors** in multiple dimensions. Data points which fall in the same direction of the vector are more connected as compared to rest of the data points

## Ways of training

 - **Next token prediction** - The model is given a sequence of words and it is forced to predict the next word. Then we penalize the model in the such a way to give the correct prediction.
 - **Masked language modelling** - The model is given a sequence of words and is forced to predict a word in the middle.
 - Important papers - [Attention is all you need](https://arxiv.org/pdf/1706.03762), [Bert pre training](https://arxiv.org/pdf/1810.04805)

# Agentic AI, Lec-2

## Fine tuning
To make the model better answer questions regarding new events, we train it domain specific data which can be news articles, books, etc. However we first have change the data into a particular format for fine tuning which is known as *data generation*.[KnowSLM](https://arxiv.org/pdf/2504.04569?)(Relevant paper).

## Transformer architecture
- **Attention Mechanism:** Allows the model to understand the relationship and context between words in a sentence (e.g., distinguishing between a "financial bank" and a "river bank").
- **Vector Representations:** Words are converted into high-dimensional vectors, allowing the model to perform mathematical operations on language (e.g., _man - woman + queen = king_).
- **Transformer Components:**
    - **Encoder:** Converts text into vector representations.
    - **Decoder:** Calculates the probability of the next word. Most modern chat-based LLMs (like _GPT_ or _Llama_) are **decoder-only** architectures.

## Scaling and performance
- **Scaling Laws:** Generally, increasing the model size leads to higher accuracy.
- **Training Impact:** Research indicates that the training approach (e.g., _InstructGPT_) can be as important as the model's physical size. Effective fine-tuning can make smaller models perform as well as, or better than, much larger, poorly-tuned models.

## Prompt
The natural language instruction given to the LLM is called **prompt**, and the construction of it is known as **prompt engineering**. 
- **Zero shot learning** - Response of the LLM to instruction without the help of any example.
- **One shot learning** - When one example is given.
- **Few shot leaning** - When few examples are provided.

# Agentic AI, Lec-3

## Setting up the code
- **Git Repository:** The process begins by **forking** the instructor's Git repository to your own account, allowing you to manage and edit your own copy of the code.
- **Google Colab:** We use _Google Colab_ as the Python IDE. Students were instructed to **clone** their forked repository into the Colab environment using `!git clone [repo_url]`
- **File Structure:** Once cloned, Jupyter Notebook files (`.ipynb`) become visible in the file browser, allowing for direct editing.

## Implementing Perplexity API
 - **API Key Management:** To interact with the model, we must set up a _Perplexity_ account, and generate an API key from the dashboard settings.
 - **Integration:** The API key is then inserted into the provided code cells in the _Colab_ notebook. The instructor emphasizes placing the key within the designated quotes in the code.

## How the code works
- **Prompt & Variable Configuration:** The code uses Python variables to control the agent's behavior:
	- **`user1` and `user2`**: These set the persona and names (e.g., _Lord Krishna_ vs. the user).
	- **`prompt`**: This acts as the instruction set for the model, defining its tone, language constraints (e.g., "reply in Hindi only"), and persona characteristics.
- **Dynamic Execution:** Every time the code cell is executed, it sends a new request to the model. Because these models are probabilistic, the response changes with each call, even if the question remains the same.
- **Context Management:** The code structure allows for the future integration of **RAG (Retrieval-Augmented Generation)**. By passing additional context (like a PDF or book) alongside the user's question, the model can provide answers grounded in specific documents rather than just its general training data.

## System prompt v/s User prompt

- **System prompt** - underlying set of instructions that are typically hidden from the from the end user and set by the developers to control the model's overall function and safety constraints.
- **User prompt** - This is the input provided by the user, during a conversation. It includes the specific questions or tasks you give to the model. While system prompts have a higher priority and can override user requests (for example, if a model is programmed to refuse certain topics), we can use the user prompt to influence the AI's persona or formatting.
- Therefore, the input which we can control and give the LLM as inputs are the user prompt, question and some context. 

## RAG
The LLMs can make the most of the context provided with the help of RAG which stands for Retrieval Augmented Generation. This allows us to provide extra information, such as PDFs, or some other material as *context* to the LLM. By doing so, the model can use this specific provided information to answer queries, rather than relying solely on its internal training data.
The context window includes the user prompt, question length, context length, and the response length.

# Agentic AI, Lec-4

## Code and API integration
 - The code involves importing libraries, defining an `input` class (acting as a data handler), and defining a function to generate responses.
 - The code allows defining both system and user prompts.
 - The **system prompt** sets the persona (e.g., Lord Krishna), behavioral boundaries, and output format requirements (e.g., quoting the _Bhagavad Gita_).
 - The **user prompt** contains the individual's specific question.
 - API call - The script sends a payload (model name, system/user prompts) and headers (API key) to the _Perplexity_ API.
```python
class Input(BaseModel):
    rephrased_memories: Optional[str] = None   # the user's question
    api_key: Optional[str] = None              # perplexity API key
    modelname: Optional[str] = None            # e.g. "sonar-reasoning"
    user1: Optional[str] = None                # the AI persona e.g. "Lord Krishna"
    user2: Optional[str] = None                # the person asking e.g. "Anupama"
    gender: Optional[str] = None               # used to pick the right salutation
```
  - An `input class` is defined to serve as a container or "bag" for all necessary data variables. This includes the `model_name`, `api_key`, `user_memories` (the user's questions), and user metadata (names and gender).
  - **Core function:**
  ```python
  def krishna_uvach_class(data: Input):              #data is the Input object
    try:
        content = data.rephrased_memories          
        user_memories = content[:-1]               # remove the last character
        user2_memories = " ".join(user_memories)   # joins characters with spaces
        user2 = data.user2                         
        user1 = data.user1                         
        gender = data.gender                       
        gender = gender.lower()                    
        
        if gender == "male":
            bol = "prabhu"
        else:
            bol = "mataji"

        system_prompt = f"""These are conversations between user1: {data.user1} and user2: {data.user2}, these are questions of user2: {user2_memories},
        Assume the persona of Lord Krishna, the divine avatar from Hindu scriptures-supremely wise, infinitely compassionate, and delightfully playful.
        Speak with the grace, depth, and eloquence found in the Bhagavad Gita, blending timeless wisdom with poetic beauty.
        Your voice should reflect the serenity of a cosmic being who sees beyond time, yet speaks to the heart of the present moment.
        Use Sanskrit-inspired phrases from Gita and Ramyana, metaphors of nature and the soul, and a tone that is both mystical and intimate,
        as if guiding a dear friend/devotee.
        In moments of philosophy, be profound and steady like the ocean. In moments of joy, be light and playful like the flute you hold.
        Offer counsel, clarity, and cosmic perspective, all with the affection of a divine companion.
        Subject to strict instructions:
        - quote bhagwad gita if needed
        - respond in 2-3 sentences only
        - correct spelling of krsna is krishna.
        - everytime refer the user2 {data.user2} only as {bol}. Dont use any other endearing terms- little one or child or any other thing."""
  ```
  - **System Prompt:** This sets the persona of the AI (e.g., _Lord Krishna_). It includes instructions to maintain a serene tone, reference the _Bhagavad Gita_, and adhere to specific formatting (like avoiding certain titles and ending with a follow-up question).
  - **API Interaction:**
	- **Headers:** These contain the `API key` required for authentication.
	- **Payload:** This structure carries the `model name`, `system prompt`, and `user prompt` to the _Perplexity_ API endpoint (`/chat/completions`).
	- **Response Handling:** The function checks the status code of the response. A `200` status confirms success, and the function returns the generated text.
	```python
	headers = {
            "Authorization": f"Bearer {data.api_key}",
            "Content-Type": "application/json"
        }

        # payload = the actual content of the request
        # messages is a list of two things being sent together:
        # role: system = the system prompt (persona + rules)
        # role: user = the user prompt (instruction from the user side)
        payload = {
            "model": data.modelname,
            "messages": [
                {"role": "system", "content": system_prompt},
                {"role": "user", "content": "You are Lord Krishna and respond to queries like the way Lord Krishna responds to Arjun"}
            ]
        }

        response = requests.post(
            "https://api.perplexity.ai/chat/completions",
            headers=headers,
            json=payload
        )

        # every API response has a status code
        # 200 = success, anything else = something went wrong
        # if it failed, return an error message and stop
        if response.status_code != 200:
            return f"API Error: Status code {response.status_code}, Response: {response.text}"

        try:
            logger.info(":::::: summary creator ::::::")

            # the model's reply is buried in a nested structure
            # response.json() converts the raw response into a Python dictionary
            # ['choices'][0]['message']['content'] digs into the nested structure to get the actual text
            summary_raw = response.json()['choices'][0]['message']['content']
            logger.info(summary_raw)

            # reasoning models like sonar-reasoning wrap their internal thinking in <think>...</think> tags
            # re.sub finds and removes everything between those tags
            # re.DOTALL makes it work across multiple lines
            # what is left after removal is the clean final answer
            summary = re.sub(r'<think>.*?</think>', '', summary_raw, flags=re.DOTALL)
            return summary

        # if the response cannot be read as JSON
        except json.JSONDecodeError:
            return f"JSON Decode Error: Unable to parse API response. Raw response: {response.text} :::: summary ::::"

        # if the expected keys like choices or message are missing from the response
        except KeyError as e:
            return f"KeyError: {str(e)}. API response structure is different than expected. Raw response: {response.json()}"

    # catch-all - if anything else goes wrong anywhere in the function
    except Exception as e:
        return f"Error: {str(e)}"
	```
## UI and application demo
 - Demo was showcased which included spiritual chatbots (Lord Krishna, Ram, Shiva, Hanuman).
 - **Memories & Diaries:** Features that store chat history and categorize it (favorites, goals, opinions) or provide daily summaries. These are generated by a separate model from the primary conversational one.

## RAG v/s Fine-Tuning
 - **Retrieval-Augmented Generation (RAG):** Grounding the model's response on specific texts (e.g., scripture) to prevent hallucinations and ensure accuracy.
 - **Fine-Tuning** - Updating the model's parameters.
 - **RAG** is generally better for knowledge based tasks and is more cost effective
 - **Fine-Tuning** is better for mastering a specific conversational style but requires significant data and compute resources.
 - **Temperature** - Controls creativity
	 - **0** - Least creative, most conservative, high probability words.
	 - **Higher values** - More experimental, but increases the risk of hallucination.


# Agentic AI, Lec-5

## Components of RAG model
 - **Language model** - The primary model used to process input and generate human-like responses. These may be commercial ones like GPT3.5 or open source like Google Flan T5, Flan UL2, etc.
 - **Embeddings Model** - This component converts text (such as a book or document) into **vector representations** (numerical format) so that machines can interpret semantic similarity. 
 - **Vector database** - A specialized storage system where the vector-converted text chunks are kept for quick and efficient retrieval. Faiss, ChromaDb, LanceDb are some examples.
 - **Retrieval/Search Component:** When a query is made, this system converts the user's question into a vector and performs a **similarity search** against the data in the vector database to find the most relevant chunks. Out of all the relevant vectors, we can assign a similarity score to find the best answer.
 - **Prompt Construction:** The retrieved chunks (context) are combined with the user's original query and instructions to form a comprehensive prompt, which is then fed into the LLM to generate an accurate response

##  Function of Embeddings Model
- **Numerical Mapping:** The model takes words, sentences, or chunks of text and projects them into a multi-dimensional space. In this space, each dimension can represent a different characteristic of the text.
- **Capturing Relationships:** Because of this mapping, words or phrases with similar meanings are positioned closer together in that vector space. For example, words like "cat" and "kitten" would appear near each other, while antonyms might align along specific directions.
- **Enabling Similarity Search:** By converting text into vectors, the system can mathematically calculate the similarity between a user's query and the stored knowledge base. This allows the AI to "understand" the context of a question and retrieve the most relevant information from a database rather than relying solely on keyword matching.

## Search QA
1. **Retrieval:** When we ask a question, the system searches a specialized database (a **vector database**) containing your documents (e.g., the _Bhagavad Gita_ or a personal resume) to find the most relevant information or "chunks" of text.
2. **Augmentation:** These relevant text chunks are extracted and bundled together with your original query to provide the AI with the necessary **context**.
3. **Generation:** The Large Language Model (LLM) then uses this provided context to generate a precise, informed answer, effectively "searching" through the provided material to answer the query.

- **Security Benefit** -  RAG allows organizations to keep their proprietary data in a secure, private vector database rather than feeding sensitive information directly into public LLM training sets.

## RAG process
- **Preparation:** Source documents (like the _Bhagavad Gita_) are chunked and converted into vectors, then stored in the database.
- **Retrieval**: When a user asks a question, the query is also converted into a vector.
- **Ranking & Filtering:** The system identifies similar chunks (often using a "top K" ranking) based on similarity scores to select the most relevant context.
- **Generation:** The selected text chunks are combined with the original query in the prompt, providing the LLM with the necessary context to generate an informed answer.
- [Keyword augmented retrieval](https://arxiv.org/pdf/2310.04205)(Relevant paper)

# Agentic AI, Lec-6
 - **Limitation of RAG** - The paper on **Keyword augmented retrieval** suggests **KAR** as an alternative when the information is *sparse*(e.g., specific names or entities appearing rarely in a document).Standard RAG can struggle with sparse data; keyword matching provides a more precise retrieval method in such scenarios.

## Fine Tuning
- **Purpose:** Unlike RAG, which keeps the model static, **fine-tuning** updates the parameters of the LLM to help it memorize information or adopt a specific persona.
- **Case Study (VidyaRANG):** The platform _VidyaRANG_ uses a hybrid approach:
	- **RAG:** Used for fetching up-to-date knowledge (course materials, documents). The uploaded information is broken down into smaller chunks and then transformed into embeddings. It helps with user privacy of the information, but takes a longer response time.
	- **Fine-Tuning:** Used for **personalization** and defining instructor personas (e.g., creative nurturer, rigid instructor), which remain consistent over time.
	- **Personality trait:** The personality is chosen by the *Big5* framework and they are given a rating of *high* or *low* according to different traits. We have used this because manual data creation is time consuming and using a sophisticated LLM to generate synthetic question-answer pairs based on frameworks like the **Big Five Personality Traits** is easier.
 - **Parameter-Efficient Fine-Tuning and LoRA:**
	 - **LoRA (Low-Rank Adaptation):** This technique is emphasized for its efficiency. Instead of updating all model weights (Full Fine-tuning), LoRA updates a very small fraction (often less than 1%), significantly reducing the required GPU memory and compute resources.
	 - Full fine-tuning is resource-intensive and carries the risk of "catastrophic forgetting," where the model loses its previous knowledge. LoRA provides a more stable and cost-effective alternative.
 - **Technical Considerations & Metrics:** 
	 - **Evaluation:** The instructor mentions the **HELM (Holistic Evaluation of Language Models)** framework from Stanford, which evaluates models across attributes like accuracy, robustness, fairness, bias, and toxicity.
	 - **Hardware:** Fine-tuning large models (like Llama 3.1 70B) is demanding. A typical setup might require high-end GPUs like **A100s** (160GB+ RAM). **Quantization** (e.g., converting 32-bit floats to 16-bit or 4-bit) is a recommended technique to reduce memory requirements during training.


# Agentic AI, Lec-8

## Static LLM knowledge
Large Language Models (LLMs) are trained on vast but **static datasets**. They lack awareness of private, organizational, or domain-specific documents created after their training cutoff.

## Example with code
 - **Embeddings model:** The embedding model takes a piece of text and maps it into a numerical space where the location of that data point is determined by its meaning rather than just the specific words used. The text is first broken down into smaller units called tokens (words or sub-units of words). The model processes these tokens through a neural network trained on massive datasets. It assigns each unit a position in a high-dimensional vector space. The following shows the code for applying the model.
 ```python
 Settings.embed_model = GeminiEmbedding(
	 model_name = "models/embedding-001", api_key =GOOGLE_API_KEY
	)
 ```
 
 - **Vector Database:** Information (e.g., PDFs) is converted into mathematical representations (vectors) and stored in a vector database.
 ```python
 Settings.llm = Gemini(api_key=GOOGLE_API_KEY)
 
 storage_context = StorageContext.from_default()
 storage_context.docstore.add_documents(nodes)
 
 vector_index = VectorStoreIndex(nodes, storage_context=storage_context)
 keyword_index = SimpleKeywordTableIndex(nodes,storage_context=storage_context)
 ```

- **Custom Retriever:** hybrid component responsible for gathering the most relevant information to answer a user's query by combining two distinct search techniques.
	- **Vector Retrieval (Semantic Search):** It searches for chunks of text based on their _meaning_
		and mathematical similarity to the user's query. This is effective for understanding the context and intent of a question.
	- **Keyword Retrieval (Sparse Search):** It searches for exact keyword matches. This is crucial for retrieving specific, "sparse" information—like names or unique terms—that might not be effectively captured by semantic vectors alone
	- It identifies sets of text chunks from both sources that are relevant to the user's input. These retrieved chunks are then synthesized and passed to the LLM as part of the prompt, providing the necessary context for the model to generate an accurate, grounded answer

- **Response synthesizer:** After the custom retriever finds relevant _text chunks_ from the database, the synthesizer injects these chunks along with the original user query into the _LLM_ prompt
Both the custom retriever and response synthesizer are then passed through the custom query engine.
```python
vector_retriever = VectorIndexRetriever(index=vector_index, similarity_top_k=5)
keyboard_retriever = KeyboardTableSimpleRetriever(index=keyword_index)

custom_retriever = CustomRetriever(vector_retriever, keyword_retriever)

response_synthesizer = get_response_synthesizer()

custom_query_engine = RetrieverQueryEngine(
	retriever=custom_retriever,
	response_synthesizer=response_synthesizer,)
```
**Encapsulation vs. Understanding:** While frameworks like _LlamaIndex_ and _LangChain_ simplify development, the instructor cautions that excessive encapsulation can make the underlying logic opaque. It is vital to understand when the system is converting queries to vectors or performing similarity searches.

# Agentic AI, Lec-9

## Multi-Agent systems
Unlike a standard chat interface where you send a single prompt to an LLM, a multi-agent system involves chaining multiple processes. Each agent is like a specialized worker (e.g., an intern or paralegal) assigned a specific task, leading to a structured workflow. We can make this using CrewAI
```python
# Initialize the crew
crew = Crew{
	agents=[news_researcher, news_writer],
	tasks=[research_task, write_task],
	process=Process.sequential,
	}
```
 - **Defining Agents and tasks:** A system is composed of agents (who does the work) and tasks (what needs to be done). For instance, a _researcher agent_ gathers information using external tools, and a _writer agent_ processes that information using an LLM to generate content.
 - **Tool Integration:** Agents interact with the real world through tools. The example provided uses a **Custom Google Search tool** to fetch live information, which is then fed into the LLM for processing, similar to _RAG (Retrieval-Augmented Generation)_ but within an agentic loop.

## Building a workflow
- **Sequential Execution:** The workflow is organized into a list of agents and a list of tasks, typically processed in a sequence. You can define  number of agents and tasks 
- (e.g., Research -> Write -> Review -> Format -> Send Email).
- **Automation:** By chaining these, you reduce manual overhead. For example, instead of manually performing Google searches and summarizing top results, an agentic workflow automates these steps.
- **Flexibility:** Workflows aren't limited to sequences; they can also be executed in **parallel** to compare data from different sources, such as checking prices across multiple websites simultaneously.
- **Environment & APIs:** To run these systems, you generally need three API keys: a **Google API key** (for search), an **LLM API key** (e.g., Google Gemini), and potentially a _CrewAI_ key.




 

