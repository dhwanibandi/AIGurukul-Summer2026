# Agentic AI Notes

## Lecture 1

- **Probabilistic models:** Output the response with the highest probability; the model calculates probabilities internally.
- **Transfer learning:** Moving from a pre-trained model to a fine-tuned model, from generic to specialised knowledge.
- **PaLM:** Fine-tuned on medical data; around 540 billion parameters.
- **Neural networks:** Made up of neurons with weights that store learned information. The number of parameters is fixed after training.
- **Transformer architecture:** Introduced by Google in 2017. Captures long-range dependencies, supports parallelism and scalability, and models relationships using encoders and decoders.
- **Words as vectors:** Words are represented as vectors, where dimensions capture features or parameters.
- **Dimensionality reduction:** Reduces vector dimensions based on similarity.
- **Masked language modelling:** Predicts missing words using probabilistic analysis and penalises wrong predictions.

---

## Lecture 2

- **Attention mechanism:** Learns relationships between information that often appears together.
- **Fine-tuning:** Requires high processing power. Example: the Mahakumbh example used 160 GB RAM and 2 H100 GPUs.
- **Transformer architecture:** Encoder converts words to vectors; decoder calculates probabilities for the next words. Most LLMs use only decoders.
- **Next-token prediction:** Takes a sequence, hides the last word, and lets the LLM predict it.
- **Masked language modelling:** Hides or masks random words instead.
- **GPT vs BERT:** GPT is decoder-only, while BERT is encoder-only. Encoders are useful for understanding full text; decoders are useful for text completion.
- **Model size:** Measured by number of parameters. Accuracy often increases with size, but also depends on training quality.
- **Prompting:** Zero-shot, one-shot, few-shot, or N-shot learning, based on the number of examples given.

---

## Lecture 3

- **System prompt:** Backend prompt with higher priority; hidden from users.
- **User prompt:** Prompt given by the user. It can include the question, context, and RAG data.
- **RAG:** Retrieval-Augmented Generation. It uses external context to answer a question.
- **Context length:** Includes the user prompt, question, context, and response.
- **Context window:** Maximum number of tokens an LLM can process and generate in one interaction. The prompt plus generated response cannot exceed this limit.
- **Token estimate:** 1 token is around 4 characters or 3/4 of a word.

---

## Lecture 4

- **Code structure:** Import libraries, create a class for input variables, and define a function for response generation.
- **Input variables:** User1, User2, gender, rephrased memories, API key, and model name.
- **Function definition:** Header contains API key; payload contains model name, system prompt, and user prompt. Both are sent to the Perplexity API.
- **Output:** Response variable contains `response.txt` and response status.
- **Fine-tuning vs RAG:** RAG is better for knowledge and slightly better for conversational aspects. Fine-tuning is slightly better for conciseness, but expensive and more useful when records are high, usually above 50,000.
- **Temperature:** Controls randomness and creativity of the model's response.
- **Streamlit:** Used for Python-based UI development.

---

## Lecture 5

- **Vector store:** Stores word vectors; models search for information here.
- **Query vector store:** Stores queries in vector form. The model searches using the query vector and retrieves the most similar output.
- **Similarity ranking:** Similar vectors are ranked using similarity scores. The most relevant ones are selected to fit the context length.
- **Techniques to pass retrieved data:** Direct pass, stuff, map-reduce, refine, and map-rerank.
- **LLM choices:** Language model, embedding model, vector database, search type, and chain type.
- **Embedding model:** Converts text into vectors.
- **Prompt with context:** Context is external to the model and is passed with the query so the model can refer to it while answering.
- **RAG workflow:** Build vector store -> get query embedding -> perform index search, such as FAISS -> retrieve relevant documents -> pass query and documents to the LLM.
- **Tokenization:** Tokenizer breaks text into tokens, searches them in its vocabulary, and replaces them with index values.
- **Flow:** Words -> Tokens -> Vectors.

---

## Lecture 6

- **RAG limitation:** RAG can fail when information is sparse.
- **Keyword-Augmented Retrieval (KAR):** Uses keywords instead of vectors. Works well for sparse datasets and specific information, such as a person's name.
- Even with spelling mistakes, models can often predict what the user was probably looking for.
- **Vidyarang chatbot:** Uses a mix of RAG and fine-tuning. RAG is used for course creation based on uploaded documents.

### Fine-Tuning

- **Personality traits:** Big Five model examples include Creative Nurturer, Visionary Idealist, Traditional Caregiver, and Rigid Instructor.
- **Data collection:** Can be manual question-answer creation, which is time-consuming, or synthetic dataset generation, where an LLM generates questions based on the Big Five model.
- **Reinforcement learning:** DeepSeek used reinforcement learning with user input. This updates the model but is different from fine-tuning.
- **Personality fine-tuning:** The LLM is exposed to question-answer pairs linked with a personality, so it learns to answer like that personality.
- **Forgetting:** Models tend to forget older information during full fine-tuning.
- **LoRA (Low-Rank Adaptation):** Parameter-efficient fine-tuning. It reduces trainable parameters for downstream tasks; the dataset stays the same, but fewer relevant parameters are changed.
- **HELM (Holistic Evaluation of Language Models):** LLM evaluation metric.
- **Computation requirement:** Parameters use 4 bytes per parameter, 2 Adam optimisers add 8 bytes, gradients add 4 bytes, and activations add 8 bytes.
- **Quantisation:** Reduces the number of bits used to represent values, such as parameter precision. Benefits include smaller size, easier deployment/storage, lower power usage, and faster training.

---

## Lecture 8

- LLMs have limited training data, but organisations often have specific internal data that LLMs may not have seen.
- **Techniques to solve this:** RAG passes specific information in the prompt; fine-tuning trains the model on specific data.
- **RAG vs fine-tuning:** Fine-tuning is more expensive but usually one-time. RAG requires a vector database.
- **Similarity search in RAG:** Query and information are converted into vectors and matched based on similarity.
- **Vector similarity vs keyword similarity:** RAG vs KAR.
- Since LLMs are decoder-based, they output words and not vectors. An embedding model converts words into vectors.
- **RAG data flow:** Read all data -> divide into small chunks -> convert chunks into vectors -> store them in a vector database.
- **Persistent vs non-persistent databases:** Persistent databases store data permanently; non-persistent databases do not.
- **Custom retriever:** Combines vector retrieval and keyword retrieval.
- **Similarity:** Number of top similar vectors to extract.
- **Final prompt:** Query plus extracted chunks.
- **Optimal chunk size:** Depends on the context window and other factors of the LLM.

---

## Lecture 9

- Agents can do more than a simple chat interface.
- **Multi-agent workflows:** Install libraries, then define agents and their tasks, such as research and writing.
- Tasks need to be defined in **NLI (Natural Language Instruction)**.
- There can be N agents and N tasks, such as reviewing, modifying, drafting, and mailing.
- Research uses Google Search; writing, reviewing, and drafting use an LLM; mailing uses the mail API.
- **CrewAI:** Used to build multi-agent workflows.
