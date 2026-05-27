# Lecture 1

### 1. History of LLM

A sudden change in development of LLM came in 2017, after the release of the paper titled 'Attention is All You need', where the transformer architecture was introduced. 

Before the introduction of transformer architecture, text generation was done by Recurrent Neural Networks, where the next word was predicted by looking at few of the previous words, the higher the number of previous words looked at, higher the computation for the RNN. Since we are looking at only a few of previous words, this caused the context window to be very small and hence the prediction quality was low and it would not produce coherent and contextually relevant text. 

The transformer fixed this issue by processing the data in parallel and process the whole sequence of words together. 

### 2. How does a Language Model Think?

All these LM are probabilistic. For a specific input, the model will find the probability of all possible future words and give out the highest probability word. This can be thought of like regression, where we predict something, here we are predicting the probability.

The 'memory' of these models is the weights and coefficients of the Neural network, that the model learned during training (i.e. when we feed it the data). 

These Weights and coefficients are called parameters of the model, which are fixed before the training begins, hence the model has a fixed 'memory' to work with before we start training it. 

### 3. Data representation: Words as vectors 

As we put data into the model, it will make a vector representation of the words, where words with same meaning are together and words with opposite meaning are farther apart. 

This representation of words as vectors is done by an algorithm called 'words2vec' 

We define some dimensions or feature that represents some quality, like is it an animal? is it a pet? is it domesticated? etc. So, for each word we will assign some number (this is what the word2vec does, finding what number to assign), hence each word is a n-dim vector. For ex- the pet dimension for a cat and dog will have similar numbers but for lion that number will be very different from that of cat and dog. 

Looking at words as vectors allow us to do math on them, for ex-

MAN - WOMEN + QUEEN = KING 
### 4. What is Attention?

Transformer use Neural Networks, which can be thought of as black box polynomial regression models, so when you feed a lot of data into the NN, it would learn those coefficients, hence it would know the dependencies between the words, like girl being used together with She, so it would form a connection between these 2 words. Hence it would learn to pay 'attention' to certain words. 

This further allows us to know the context of a word, i.e. for example the world Bank can mean either a financial Bank or a river Bank, the model will know what Bank it is by looking (or paying 'attention' to other words) at other words in the sentence and hence see what the 'context' of Bank is. We can't make a simple rule-based system, it would be too complex, we need something that will understand the context. This is what Attention mechanism does. 

### 5. Training Pipeline

1. Pre-Training: We train the model on some generic data so it learns the underlying rules of that specific data.

2. Fine-Tuning: We now train the model on some specific non-generic data to make it specialised in that domain. For ex- Google has a model fine-tuned on medical data called Med-PaLM. 
Fine-Tuning has to be designed properly since we already have a pre-trained model, so we need to design data we feed during fine-tuning such that the model doesn’t unlearn/forget/re-learn things it already knows. 

As a side note - LLMs should not be used for knowledge retrieval, rather they should be used for reasoning on some given facts. We can't rely on model to know certain facts or have certain knowledge since it may not have been trained on that data or might have been given the incorrect fact. 

# Lecture-2 

### 1. Transformer Architecture on a high level

On a high level we can think of Transformers having an encoder and a decoder, which are a combination of attention mechanism and a feed forward neural network. 

The encoder creates the encoded vectors and the decoder takes these in and outputs the probabilities. The chatbots we interact with are only decoder based. 

### 2. How pre-training actually happens

One of the techniques used to train a model on a large amount of data is **Next Token Prediction**, where we purposefully hide some of the words and force the model to predict them, penalising it when it gets them wrong via a loss function (which we can use since we know the correct word). 

Another technique is **Masked Language-modelling** where we hide a word in sentence and force the model to predict it, this is bi-directional since the model knows the previous and next words. Again, we penalise the model via a loss function since we know the correct word. 

These techniques were proposed for pre-training of a model BERT which was one of the first language models. 

### 3. Traditional Transformer Architecture

The traditional transformer has 2 main components: The encode and the decoder.

**Encoder** processes the input text and converts it to a vector embedding, that represents the meaning and context of each word in that input paragraph or sentence.

**Decoder** generates the probabilities of each word and hence predicts the next best word using these vector embeddings and the previous words.

**Attention** is used to focus on the most relevant parts of input and output text and to capture the long-range dependencies and relationships between the words.

Then **training** happens on a large set of text data to minimise the difference in predicted and actual words, i.e. to reduce the loss as much as possible. 

### 4. What is GPT?

The GPT model has only one component: the decoder

Decoder only models don’t have an explicit encoder to summarise the input text, they predict the next word by using the vector embeddings and previous words. 

### 5. Types of Transformers 

Transformers are generally categorised into 3 categories:

1. **Encoder only**: These are useful for tasks where entire sequence of words needs to be understood, such as in sentiment classification. e.g.- BERT.
2. **Decoder only**: These are useful for tasks where the text needs to be completed, i.e. for generating next word. e.g.- GPT.
3. **Encoder-Decoder**: For language translation tasks these are more useful. e.g.- BART by Facebook.

### 6. Accuracy

1. **Effect of model size**: Generally, as size (number of parameters) increase, the accuracy also increases. But this is not the only factor.
2. **Effect of training**: A larger model might perform bad than a smaller model if we train the smaller model better than the larger one. Ex- for **GPT**, if we *prompt* it properly, then we observe that the **Likert score**(a way of measuring accuracy, higher the score better the model) increases drastically as number of parameter increase, but for *fine-tuned* model we don’t observe such drastic increase in the score, which implies that size is not the only factor, but how you fine-tune or train the model also determines the accuracy. Hence, we can use small-scale models and train and fine-tune them properly to get better results rather than simply using a larger model.

### 7. Different ways to prompt a model
There are many ways to prompt or ask a model certain questions, let’s say we ask (or prompt) the question, *How do u like this movie?* And ask the model to do a sentiment classification on customer reviews.
**Zero Shot Learning**: Ability of LLM to answer a question **without any example. **
In the above example, the LLM will be asked to classify the sentiment of *I loved this movie* review, without being given any example, or any other reviews. 

**One Shot Learning**: When an **single example** is provided to the model and then its asked to Ans the question. 

In above example, the LLM is given the sentiment for the question *I loved this movie* as *positive* and then asked to predict the sentiment for *I don’t like this chair*.

**Few Shot Learning**: When **multiple example** are given to the model and then it’s asked to answer a question. 

It’s preferred to use few shots learning since the models learns to associate certain words with certain sentiments, hence it can predict better. The examples we give don’t have to be related to our query/question, the model simply learns how to answer question. 

# Lecture 3

We use Perplexity API to use sonar-reasoning model, which natively has chain of thought. 

### 1. What is a prompt?

A prompt is an instruction to an LLM. 

**There are two types of prompts**:
1. **System Prompt**: Hardcoded prompts in the model, we can't change these prompts and these prompts have a higher preference than user prompts and are generally not revealed by the company who made these models. (e.g. DeepSeek refuses to answer sensitive questions, which is result of system prompts.)
2. **User Prompt**: The prompts given by us, we can pass a prompt variable to the model, it will reply accordingly. This was passed to the model via *prompt* variable in the code. 

**Context**: We can also pass context to the model along with user prompts. The model won't Ans from its memory but rather the context we provide it with. This ensures that the model sticks to the material provided. For ex- if we want the model to Ans about a course, we can upload the book for that and model will answer from that. This framework is called *RAG (Retrieval-Augmented Generation)*.

**User Query**: This is the question we ask the model; this is sent along with the above discussed parameters. This is represented by the *rephrased_memoery* variable in the code.

### 2. How much can we ask the model?

Since compute resources are finite, we can't send any arbitrary length of content/prompt to a model. Depending on what model we use we can have varying context-windows or context-lengths
* The number of tokens or words we can pass to a model is called the *context-length* of the model. 
* Context length includes the `user prompt + the user question + the context + the response`. Hence this value should be less than the model's max context-window (for e.g. - 128k tokens). If we exceed this limit the model might give truncated response. 

* If the context-length (before counting the response in it) exceeds, we won't get a response. 
  
# Lecture 4

We will be learning about how the API works under the hood.

### 1. What gets sent to the API from the user side?

The function call to the API contains the Header and the Payload.

* **Header**: This contains the actual API key; it's sent separately to ensure the authentication isn’t mixed with the actual data.
* **Payload**: This is the main data being sent to the model, this has the System Prompt, User Prompt, User query, Model-name.

These 2 things are sent to the Perplexity AI, and we get the *response object* back which has:

* **response** - The actual response of the model as text.
* **status** - The status of the response, if it’s an error then we won’t bother reading the response. If we don't get a 200 status that means there is some error, so we catch these non-200 errors in a try-if block. 
  
To make this API interactive we wrap this entire API calling and API response in a User Interface, this is how we get Chat-Bots. But in the above setup, the model won't remember what you asked it previously while answering your current question, until and unless you send your previous data as context to the API call.

### 2. Memory of LLM 

LLMs don't remember our previous question or whatever payload we sent, they are inherently stateless. Hence for a working chat-bot we need to continuously append the previous query-response into a backend database and then add those to the prompt every time as context whenever a new query is sent to the API. As seen in the chatbot demo, we can also store user preferences in the backend and append those to payload every time the user sends a prompt. We can also get another model to summarise daily conversation and then pass that into context for each new user-prompt. 

### 3. RAG v/s Fine-Tuning 

A research comparison of RAG and Fine-Tuning, what is better at what 

1. **RAG**: It performed better on factual retrieval, outperforming Fine-Tuning by a significant margin. This is so because the model retrieves the answers from specific chunks of text from the given PDF or document, puts those text chunks into the prompt to prevent the model from giving random information about the question.
2. **Fine-Tuning**: Perfumed better at conversation aspect than RAG, this is because here we are changing the model weights and hence the model memory itself. But it consumes a lot of computes in doing so. 

Hence if you want to retrieve a bunch of info form fixed sourced, you should use RAG, but if you want your model to change fundamentally about certain aspects like giving it certain personality, Fine-Tuning is better there.

### 3. Some Hyperparameters 

1. **Top-p**: This sets the probability threshold for the model to predict the next word. If its 0.9 then model will only consider the words with 90% or more probability as the possible outputs. More accurately since a single word is less likely to have a 90% probability, what we do is, we consider cumulative probability to be 90% or more, i.e. we keep taking words until their total probability is 90%, hence we get a word 'pool'. 
2. **Temperature**: This dictates how the model choses the next word from the word pool we have after the top-p filter. 
    * A temperature of 0 makes the model output the highest probability word.
    * A higher temperature makes the model creative in the sense that it can sample from the lower probability word from the given pool, but this also increases the chances of model hallucinating.

# Lecture 5: RAG

What happens when the model lacks fundamental knowledge regarding certain domain, or a book (ex- the Bhagavad Gita) that the user wants to query the model on? As discussed, we can use 2 approaches- Fine-Tuning or RAG. 

### 1. What is Retrieval Augmented Generation (RAG)

The basic components of a RAG are - 
* **Language Model** - Commercial ones like GPT 3.5 or Open-source models like Google Flan T5
* **Embedding models** - Commercial ones like text-search-davinci or open source models like Flan UL2 
* **Vector Database** - like Faiss or ChromaDB
* **Search Type** - Some of the common search types are Similarity and Maximal Marginal Relevance 
* **Chain Type** - 'stuff','map_reduce’, etc. 

### 2. Architecture of RAG

1. **Vector Database**
We have some source data with us, we use an embedding model to convert all the text data to a vector embedding and store it in a Vector DB. A little clarification on how vectors are made from the words - first we convert our words to token using a tokenizer then it’s converted to token numbers which is then converted to an n-dim vector. All of this is handled by the embedding model. 

2. **Query Embedding** 
For the LLM to understand the query it need to  convert the query to vector embedding with same embedding model, then we compare that against the vector DB we just made, it will extract the data from vector DB based on search type we use, let’s say we use *Similarity search*, then the model will find the vectors that are similar to the user prompt vector. 

3. **Top-K Filter**
Say we get 1000s of vector which are Similar to our query vector, how do we know which vectors to choose? We can't choose all since it might exceed the context-length of the model we are using. Hence, we rank the vectors based on the similarity score we get. After this we will simply define a cutoff, that below a certain similarity score we drop those vectors. Hence, we have some Top-K vectors. We can do this similarity in various ways. Also, we can use certain facts about the query to eliminate even more similar vectors before applying the top-k filter. (e.g. - we can pass off in our prompt that the query refers to chapter 1 of Bhagwat Gita and hence can ignore all the vectors from other chapter.)
4. **Passing the retrieved content to the LLM**
Now that we have say, 10 chunks of text after the above step, we can use Chain Type to put all these text chunks into the prompt, we can simply put our 10 chunks into the prompt or use the various Chaining Types. 

So finally, we send - `The User Query + The prompt + The Retrieved document or text` to the language model. Basically, we pass our retrieved text chunks from the vector DB that we found using the user query to the model as context. 

**Q**:*Why do we make RAG instead of simply uploading PDFs into chat gpt web interface?*
If we upload certain private or corporate data into chat gpt, it will probably be used to train the model and our data is stored on Open-AI servers and if someone else asks about us the model might reveal our personal data to them. Hence an offline self-hosted RAG is better to ensure data privacy.

# Lecture 6: RAG Limitations and Fine-Tuning

### 1. Limitation of RAG

When our data is very sparse, in the sense that say in a 50-page document, the name of the person only appears once, for rest of the paper the person is referred by 'they' or 'he' or similar pronouns. The way the normal RAG works is - it converts the document to a vector embedding then runs the similarity search on the vector DB with the query vector, the vector similarity search will more often than not fail to retrieve relevant info about the person due to how sparse their name appears in the data and hence in the vector DB.

The Fix is to use something called *Keyword Augmented Retrieval(KAR)*. This method works by extracting key-words from the user query and then comparing them with the keywords extracted from the document. This method outperforms standard RAG on sparse datasets. 

### 2. The Hybrid Architecture of RAG + Fine-Tuning

Every time we want to summarize certain document or material, we can't fine-tune the model on that since as discussed previously, fine-tuning is expensive. Hence real-world applications involve a hybrid of RAG - for factual content retrieval and of Fine-Tuning - to make the model follow certain persona or behaviour while answering the user query, this is permanent so we only fine-tune the model once this way and we are done. 

### 3. How to get the data for fine-tuning 

Fine-Tuning involves personalisation of the LLM to fit certain traits or a personality; this requires a lot of data in the form of Question and Answer which we train the model on. One way is to manually make the question answer list, but since we need huge amount of data this will be very time consuming and expensive.

**The Synthetic Data trick**

Instead of manually writing 1000s of Q&A pairs, we use sophisticated frontier models like GPT-4 to generate this Q&A pairs for us by prompting the model with our persona defined in the prompt we give to the model. Hence, we get a massive synthetic dataset generated by a frontier model.

Now we train or fine-tune a much smaller model (like Llama 3.1 70B) model on the dataset we made, which results in the smaller model mimicking the much larger model with respect to the personality we wanted our model to have, without incurring massive API costs.

### 4. Hardware constraints in fully fine-tuning a model 

In fine-tuning we are updating the weights of a model. To do so we need large amount of GPU Video RAM. The breakdown of how much we need is- 
* To store a 1B parameter model into V-RAM we need 4GB of storage. (One parameter = 4 bytes in std 32-bit float, hence 1B parameter is ~4GB)
* During a full fine-tune we must store the full model, the gradients, the optimizer states (like of Adam), and the activation in the GPU simultaneously. 
* So, for a small 70B parameter model, we would need at least 2 A100 NVidia GPUs, which is very costly (~160GB VRAM)
* Also, since we are updating all the weights, what might happen is that the model might forget the pre-existing things it learned during the full fine-tune. 

### 5. The fix: LoRA

To bypass the hardware constraints of full fine-tuning, we use LoRA.

* Instead of updating all the parameters of the model, LoRA only updates a tiny number of relevant weights. (<1% of the total parameters). 
* Since we don't deal with full model now, and only ~1% of the total parameters, we don't need the full base model in the VRAM but a tiny percent of it, hence we require much less VRAM for LoRA. 
* We can manually adjust LoRA rank(r) and as we increase the rank the more parameters are changed during fine-tuning, as the rank approaches infinity LoRA is same as Full Fine-tuning. A lower rank is lower memory usage but it might have lower leaning capability.
* To further reduce memory, we can *quantize* the models by compressing the 32-bit float numbers to 16-bit or 4-bit numbers to shrink the model size and hence reduce compute costs. 

### 6. LLM Evaluation metrics (HELM)

To objectively prove that a fine-tuning or RAG implementation improved the model, we use evaluation frameworks. One of them is Holistic Evaluation of Language Models (HELM) by Stanford. Model is asked certain questions and then the answers are evaluated on certain attributes like Accuracy, Fairness, Bias, Toxicity, Robustness, etc and a score is generated. 

# Lecture 7: Agentic AI 

### 1. What is an Agent?

An Agent is a **automated workflow process**, it may or may not use an LLM. For e.g.- An agent assigned to scrape the web is executing simple scripts to do so, whereas an agent tasked to summarise the text will use an LLM.

### 2. CrewAI: Agent Orchestration 

To make a bunch of these agents work together, each doing a specific task, we use the crewAI framework. Some of the other frameworks we can use are LangGraph and Langroid. 

### 3. An example of a workflow 

We need to define specific roles and the exact work those roles have to do, just like in a real company where different job roles have different responsibilities. For ex- In a Law firm, for a case, the paralegal will find the relevant case files (The Researcher) and the attorney uses those to draft litigation (The Writer).

We look at a crewAI script, where we define 2 agents - a "Researcher" and a "Writer". We also define the tasks each of these agents will do.

### 4. Agents can be assigned tools to interact with real world data.

Each agent gets assigned specific tasks in Natural Language. The researcher executes his task via a google search API and similarly the writer agent uses an LLM to do its tasks, which is explicitly defined. Each agent here was equipped with a tool, which are used by agent to interact with real world data, here the research interacts with internet via custom google search and the Writer was equipped with an LLM, here we used gemini-flash-2.0. 

### 5. Chaining the workflow 

We use crewAI to organise these into a sequential workflow, meaning the data collected by research is passed onto the writer. We can also use crewAI to prompt the agents and under the hood these agents are used in the workflow we defined. We can also organise a few agents to scrape websites in parallel, then continue with our sequential workflow. 

### 6. More Agents 

We can extend our 2 Agent setup in a larger automated pipeline. An example 5-step workflow might look like - 

1. **Researcher**: Gathers data using google search.
2. **Writer**: Drafts a response using the data it received from Researcher using an LLM.
3. **Reviewer**: Uses another LLM to check the draft made by Writer 
4. **Formatter**: Uses one more LLM to implement the Reviewer's changes into a strict user defined template. 
5. **Sender**: Uses a 3rd party Email API tool to auto send the final draft to someone, or even the user so that user can verify it before sending it to someone else.

This will be done sequentially, as in the Writer coming after researcher and so on. We could also scale the researcher by running multiple of them in parallel to scrape multiple websites together. 

### 7. Too much automation might not be good

As we make workflows more and more automated, we should not give too much permission to the Agent, tasks like making purchases without oversight can end with agent executing harmful purchases or other harmful action using the user's personal data. 

