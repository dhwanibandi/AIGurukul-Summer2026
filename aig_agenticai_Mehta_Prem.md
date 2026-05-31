# Lecture  1 : 

## Large Language Models (LLM)

 - Language models composed of billions of parameters trained on extensive text data, up to hundreds of billions to trillion tokens.
 - They have capacity to learn and generalize from extensive and diverse training data.

## Language Models (LMs)

All Language models are Probabilistic models , not deterministic. It gives responses of highest probability of being correct based on the input provided.

 **How are these probabilities calculated?**

 - We train the model to a given data (Pre-training) .   
 - After Training, we Fine-tune it, i.e., train it using Domain or Task-specific data or providing data that the model hasn't seen yet. (E.g. Palm)
 - The model takes the user's input sequence and calculates the probability for every possible next word in its vocabulary, ultimately predicting the word with the highest probability.
 - This is called Transfer Learning. 
 
## Transformer Architecture
- Combination of Neural Network and Attention Mechanism.
- Attention Mechanism functions by teaching a model how to "pay attention"  to the connections between different words. It teaches the ability to understand context-dependent meaning.
- It uses Word to Vector Algorithm, which allows words to be converted into mathematical representations, mapping them across multiple dimensions.(E.g. -man minus woman plus queen equals king)

# Lecture 2: 

## Additional points of Lec 1
- Language Models calculate probabilities of different words.
- Fine-tuned model does a better job at answering queries than Non-fine tuned models. It improves the conciseness and quality of answers.
- 1 H-100 = 80GB of memory.
- Dimensions (Also called Embeddings) size will mostly be a multiple of 512 (generally not above 16000). 
- In Transformer Architecture, Encoder converts Word to Vectors called embeddings, that represent the meaning and context of each word and Decoder calculates the probability of each word and generates the output text. (Most LLMs use just a decoder, instead of both)

## How to Pre-Train

**Next Token Prediction**
In this approach, the training system takes a sentence, and hides the final word for the model to predict. As we know the correct word, we can create a loss function and train the model with it.

**Masked Language Modelling**
In this approach, words are randomly hidden from anywhere in the sentence, while giving the model surrounding context(left and right words), and then forced to predict.

## Accuracy

 1. **Effect of Model size** - It is observed that as size increases, accuracy increases.
 2. **Effect of Training** -   It is equivalently important on how the model is trained. If the approach of training is really good, then the size of the model does not matter much.

## Prompt
Natural Language instruction in which we interact with an LLM is called a Prompt. Prompt construction is called Prompt Engineering.

 - **Zero Shot Learning** - Giving the model a natural language instruction without providing any examples of the desired output.
 - **Few Shot Learning** - Providing the model with one or more examples to help it recognize the pattern or format you want (related or unrelated).

# Lecture 3: 

## Environment Set-Up
-   Fork the Git repo into your own GitHub account.
-   Open Google Colab, which serves as a Python IDE (Integrated Development Environment).
-   Clone your forked repository into the Colab environment using the command !git clone (repo-name).
-   The files with the .ipynb extension are Jupyter Notebooks.
The Code and What It Does
## Additional points on the Code
-   The notebook interacts with a hosted Large Language Model (LLM) via an API. The model used is sonar-reasoning from Perplexity AI.
-   As a reasoning model, it applies chain-of-thought reasoning, meaning you do not need to explicitly prompt the model to think step-by-step.
-   Perplexity AI does not build its own models from scratch; instead, they fine-tune existing open-source models (likely LLaMA).
-   You must generate a Perplexity API key to use the model.
-   Because LLMs sample words from a probability distribution, running the exact same question multiple times will yield slightly different answers.
-   When defining a long prompt across multiple lines in Python, you must add a backslash  at the end of each line to tell the code the string continues, which prevents errors.

## Layers of Information when making a Request

1.  **System Prompt:** These are set by the company that built the model (e.g., Perplexity or OpenAI). This defines core behaviors, such as refusing to answer sensitive questions. Users cannot see or change the system prompt, and it always holds a higher priority than the user prompt.
2.  **User Prompt:** The specific rules you define for the interaction with the LLM. 
3.  **Question:** The actual query you want the model to answer.
4.  **Context:** Additional external information provided alongside the question.

## RAG (Retrieval-Augmented Generation)

-   RAG is a technique where you supply external documents (like a PDF or book) directly to the model as Context.
-   Instead of relying on its pre-training memory to answer a query, the model acts on the provided document. RAG prevents the model from hallucinating a generic response by forcing it to summarize the exact document provided.

## Context Length/Window

-   The context window is the total text capacity a model can process in a single interaction.
-   The absolute limit is consumed by: **User Prompt + Question + Context + Response**.
-   If the combined total of these four elements exceeds the model's limit (which is 128,000 tokens for the sonar-reasoning model), the model will fail to process it and will output a blank or cut-off response. (1 token ~= 4 chars in English or 3/4 words)

# Lecture 4: 

## Exploring API 

1. We created a class called Input with the following variables
 - Rephrased memories: For user's questions
 - API_key: Perplexity API Key used
 - Model_Name: sonar-reasoning used
 - User1 & 2: The persona between whom the conversations are happening
 - gender: Male/Female
 
2. We wrote a system prompt to train the AI on how it needs to give responses.
3. **Headers** - It is the cover sent to show that it is a valid request.
- Authorization : for the API Key (Bearer is the standard way of sending an API key)
- Content type: application/json tells the server the data is in JSON format.
4. **Payload** - includes actual content of the request.
- Model : Model name
- Messages: System prompt + User prompt
5. If status code == 200, we have a proper response.

## RAG vs Fine-Tuning

- Fine-Tuning requires much more computation power than RAG.
- Fine-Tuning produces models with much more conversational flow and conciseness than RAG.
- RAG performs better in terms of Factual knowledge.
- Hence, RAG is recommended approach if the answers are required to be strictly grounded in a specific data, else, Fine Tuning is the better approach.

## Temperature and Top P

 - Top P restricts the model to only selecting next words that meet or exceed that probability.
 - Temperature controls the randomness of the model. A temperature of zero forces the model to be highly conservative and predictable, picking only the words with the absolute highest probability.
 - Higher temperatures allow the model to bypass the Top P threshold and sample words with lower probabilities, making the output more creative but significantly increasing the risk of hallucination.

We can also have a previous_conversation variable for the context of previous information.

# Lecture 5:

## RAG (Retrieval Augmented Generation)

A framework used to ensure Large Language Models (LLMs) provide accurate answers grounded in specific external data.

It is composed of 

 1. **Language Model** : The Model that generates the final response. It can be commercial or open source.
 2. **Embeddings Model** : Converts text into vector representation.
 3. **Vector Database** : Storage part of all vectors created by Embeddings Model.
 4. **Search and Retrieval** : The process of finding which stored vector is most similar to the query vector and retrieving those and passing to the Language Model as context.
- We can use Top K similarity search or Maximum Marginal Relevance for searching.
- Searching is done through similarity score. ( 0 for not similar, 1 for most similar)

## Tokenization vs Vectorization

For any query, words are first broken down into tokens, which are pre-defined. These tokens are then mapped to an number index. Finally, these numbers are projected as vectors by the Embedding System.

A major advantage of a custom RAG system is Data Security. With a custom RAG architecture, you retain control of your vector database and only temporarily pass specific chunks of data to the LLM.

# Lecture 6: 

## KAR (Keyword Augmented Retrieval)
- RAG matches query vectors with document vectors by similarity.
- This fails when query vectors appear rarely or as pronouns.
- KAR solves this by extracting keywords from the query and matching them directly with keywords extracted from document to find context.
- Hence, It is very useful for sparse data sets.

It is good to use RAG for factual knowledge and Fine-Tuning for teaching the model how to converse and for Personalization.

## Synthetic Data Generation
This involves writing prompts for a massive LLM (Like GPT) to automatically generate thousands of accurate question-answer pairs. These pairs are then used to train and fine-tune a smaller open-source model.

DeepSeek uses Reinforcement learning which uses human feedback as it's data.

## Full Fine-Tuning vs LoRA (Low Rank Adaptation)

- Full Fine-Tuning updates all the parameters of the model. Hence, it is extremely resource intensive and requires high computation power.
- It is also possible for the model to forget the training data when Full Fine-tuning.
- Instead of updating everything, LoRA updates only a tiny fraction of parameters (less that 1%). This reduces the necessary computation power and maintains performance.
- LoRA rank is a hyperparameter that determines the percentage of a model's parameters that will be actively updated and trained. As rank is increased, LoRA becomes identical to Full Fine-Tuning.

## Evaluation Metrics and Quantization
- To measure the performance of a model, we use Evaluation Metrics.
- HELM (Holistic Evaluation of Language Model) Metric by Stanford uses 
	- Accuracy
	- Calibration
	- Robustness
	- Fairness
	- Bias
	- Toxicity
	- Efficiency
- Total Computation Requirement includes Parameters + Optimizers + Gradients + Activations (1 Parameter requires 4 Bytes + 20 Bytes extra) 
- Hence, to improve this memory requirement, we use Quantization. This involves converting the model's standard 32-bit floating numbers into smaller 16-bit or 4-bit numbers, which significantly shrinks the model's storage with minimal impact on its performance.

# Lecture 8:

## Hybrid Retrieval
Steps :
1. Documents are uploaded.
2. **Chunking** - The text of the documents is broken into chunks. Commonly chunks are of size 1024 or 2048 characters.  Order of Chunking : 
	- Load text from documents.
	- Break text into smaller chunks.
	- Convert each text chunk into a vector using the embeddings model.
	- Store the chunks and their corresponding vectors in a vector database.
3. Two separate indexes are created:
	-   **Vector Index:** Each chunk is converted into a vector using the embeddings model and stored in the vector database.
	-   **Keyword Index (Keyword Table):** Keywords are extracted from each chunk, discarding stop words (like prepositions and pronouns) and keeping important words like nouns. These are stored in a keyword table alongside the chunk they originated from.
	-   **Storage Location:** By default, the vector database stores this data in memory on the local machine. However, you can configure it to persist (e.g., `persist=True`) so that the database is saved to your disk or cloud storage.
4. **Custom Retriever** includes:
	-   **Vector Retriever:** Finds chunks that are similar to the query. Using a parameter like similarity_top_k = 5 tells the system to pick the top 5 most similar chunks.
	-   **Keyword Retriever:** Finds chunks that share exact or near-exact keyword overlaps with the user's query.
	- The custom retriever can use "AND" or "OR" logic to combine the results from both methods.
5. **Response Synthesizer:**  The response_synthesizer class prepares the final information for the model. It takes two primary objects as inputs: the retriever object and the LLM configuration.
6. **Query engine:** When you pass a query into the engine
	- The engine handles converting the query into a vector.
	-  The custom retriever fetches the most relevant text chunks using both vector and keyword search.
	-  The retrieved chunks, the original query, and the prompt instructions are all combined and passed into the LLM.
	- The LLM generates a response that is strictly grounded in the retrieved chunks.

# Lecture 9:

## Agentic AI
- Crewai is a library used to build multi-agent workflows.
- An AI Agent is essentially a worker in a workflow automation process.
- We created two agents to decrease the workload: a Researcher agent(acts like an intern gathering information) and a Writer agent (acts like a paralegal drafting the final response).
- Tasks are natural language instructions assigned to the agents.
-    **Researcher Agent:** This agent is equipped with a specific tool—a Custom Google Search tool. It actively queries Google to fetch the most relevant text from the web.
-   **Writer Agent:** This agent does not use the search tool. Instead, it is explicitly configured to use the LLM. It takes the raw data retrieved by the Researcher and uses the LLM to write the final summary.
-    Once the agents and tasks are defined, they are chained together into a workflow.
-   **Sequential Processing:** The instructor's code runs sequentially. The Researcher completes the Google search first, and its output is automatically passed to the Writer agent to generate the final text.
-   **Parallel Processing:** The framework also supports running tasks in parallel, such as deploying two agents to fetch two different websites simultaneously before sending their combined data to a reviewing agent.
- Storing retrieved information (E.g. by the Researcher Agent) is known as Caching. This technique can make AI Agents more efficient if we want to reuse the retrieved information.
