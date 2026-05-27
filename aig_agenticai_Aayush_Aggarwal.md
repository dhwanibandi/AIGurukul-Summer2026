# Lecture 1
- LLM are models composed of biilions of parameters trained on extensive text data
- Transformers Paper ---> Attention is all you need (by Google)
- Open Source models on Hugging Face
- Language Models are probabilistic models trained to learn statistical patterns on natural language (a model basically predicts the next word based on previous words using probability)
- Pretrained Model (generic data) ---> Fine tuned model (cleaned data)
- How does a model learn?(basically how does a model store data which acts as a memory) ---> We use a neural network for this which is heavily inspired from human brain. The capability of this neural net is somewhat proportional to the number of parameters it has.
- Importance of Transformers ---> Contributes in maintaining long term context, parallelism and scalability
- Attention ---> Here it means the relation between words, basically finding dependecy between words which can help in predicting the next word and figure out context dependent meaning
- Word2Vec (NLP)
- Does Dimension(WordVector) == Number of Parameters?

--------------------

# Lecture 2
- Transformer Architecture - Encoder and Decoder
- Most LLMs only have decoders
- Transformer solve memory botlleneck and parallelism
- Decoder based architecture works well for generating text, Encoder based architecture is useful for taks like sentiment classification where understanding text is important and encoder-decoder architecture is useful for tasks like language translation
- Accuracy of model is a fuction of how you train a model and also the size of the model
- Zero Shot Learning ---> Ability of a LLM to respond to a prompt without the help of any example
- N Shoy Learning ---> Needs N examples to respond to a prompt


-----------------------------


# Lecture 3 
- System Prompts has higher priiority than user prompts
- RAG (Context) ---> Retrieval Augmented Generation is used to pass context to the model to improve its response
- You cannot give prompt+context+question+response of greater than conext length (context window)


----------------


# Lecture 4
- Demo of accessing a model using a python backend
- Novi AI demo
- If you dont have the budget of fine tuning a model then you can use a RAG
- Temperature ---> sort of a measure for Creativity/Experimentalness in model responses


---------


# Lecture 5

### RAG Framework
> - Language Model 
> - Embeddings Model
> - Vector Db
> - Search Type 
> - Chain Type

### Vector Db
- Load the source data in a vector db
- For every query, the model understands the context and query (which basically means, it creates a vector of this query) and then it queries the vector db to retrieve 'most similar' results which are ranked on the basis of 'similarity score'

### Token
- Tokens are word sized sequences 
- Each natural language word can be broken down in token using a tokenizer
- Words ---> Token ---> Vector , this is done by an embedding model


-------------


# Lecture 6
- Keyword Augmented Retreival ---> Instead of finding similarity to queries in Vector Db, here we use keywords to find similar data
- Fine Tuning ---> for this we need a dataset, massive compute power. Question-Answer pairs are used to update the parameters of mdoels
- LoRA ---> Low-Rank Adaptation is very efficient compared to full fine tuning. Its called 'Parameter Efficient' as it does not update all the parameters. LoRa rank is directly proportional to the trainable parameters (trainable parameters can be reduced to less than 1%)
- Synthetic Datasets can be generated using LLMs by providing them context of what kind of data we want to generate
- 1 parameter takes 4 bytes
- Quantisation ---> Reducing number of bits required to represent a value, in this context it means reducing precision of LLM's parameters


------


# Lecture 8
- Fine tuning is extremely expensive but a 1 time operation
- A good analogy to understand fine tuning and RAG in comparison to each other is ---> Suppose a person is trained as a civil engineer but you want them to have knowledge of a computer scientist.
> - First approach is to train them as a computer scientist which would take a lot of time and resources the but at the end they would have genuine expertise of a computer scientist hence a great comprehensive output. This approach is analogous to Fine Tuning
> - Second approach is to hand them a book and now for every question they can use the info in that book to give an answer. This is extremely resource efficient but the knowledge of the person remains only theoretical and their skills are not that comprehensive. This approach is analogous to RAG.
> - There is no better or worse choice among these approaches, the better choice completely depends on your specific task

- Before making text into vectors, we have to break it into an optimal chunk size as per the text


---------


# Lecture 9
- A code and website demo about Agents, not much to node down here
- There should be a lot of emphasis on security when dealing with Ai Agents and deciding what access and data is exposed to LLMs.
