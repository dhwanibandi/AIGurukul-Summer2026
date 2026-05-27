# Lecture 1: Introduction to Generative AI & Large Language Models

## Origins of LLMs

- Development of LLMs can be traced back to the field of Natural Language Processing
- Development of compact, offline LLMs to cater to remote areas


## Generative AI and the Rise of LLMs

- Generative AI refers to AI systems capable of generating content. E.g. text, images, music, code, voice and even biological sequences (e.g. gene sequencing).
- The term "large language model" originally referred to text-based models but now encompasses multimodal models (image generation, audio etc.).
- ChatGPT reached 100 million active users in 2 months. It was the fastest adoption in tech history, creating a viral effect that accelerated both usage and teaching of LLMs.
- Key milestones:
    - 2017 - "Attention Is All You Need" paper (Google) → introduced Transformer architecture
    - 2018 - BERT (Google) → first major encoder-based language model, open-sourced
    - GPT series (OpenAI) → GPT-3 onwards, now a family of models
    - Anthropic / Claude → founded by ex-OpenAI researchers; part of the same lineage


## Open Source LLM Ecosystem and Research

- LLMs are no longer just commercial (OpenAI, Anthropic, Google). The open-source ecosystem, especially on HuggingFace, has more models than commercial ones.
- Open-source LLMs allow organizations with privacy and security concerns to self-host models with full weights.
- IIT Delhi study- used LLMs to crack JEE papers with 95 percentile (used prompting, CoT etc.)
- Now, many commercial models are offering reasoning.


## Probabilistic Nature of LLMs

- LLMs are **not deterministic,** they are **probabilistic models**.
- Given a sequence of words, the model calculates a probability distribution over the entire vocabulary and outputs the word with the highest probability.
- This is conceptually similar to **regression,** not classification.


## Pre-training and Fine-tuning

- Pre-training: Pre-trained model is fed generic data(e.g. all text data available on internet) and learns from it.
- Fine-tuning: To make the model specialized for a particular task, it is fed specific data.
    - Example: **PaLM-Med** (Google)- 52B parameter model fine-tuned on medical data.
    - Risk: fine-tuning can cause the model to unlearn prior knowledge if not done carefully.


## Neural Networks

- neural networks- inspired from human brain
- you can fix number of parameters, exposing it to more data makes it better, but the size won’t grow.


## Transformer Architecture

- **Transformer Architecture**
    - **Paper:** "Attention Is All You Need"- Google, 2017
    - Differences in transformer vs previous models(RNN, LSTM, GRU, N-gram etc.)

        | Limitation of older models | Transformer solution |
        | --- | --- |
        | Processed words **sequentially** (slow) | **Parallel processing** → massive speed gains |
        | Could only capture **short-range dependencies** | Captures **long-range dependencies** across entire context |
        | Fixed, small context window | Large, flexible context window |

    - Dependency parsing- finding out dependencies between different words and creating dependency maps using algorithms that are rule based
    - Neural network- black-box polynomial regression model
    - Attention mechanism- The core innovation of transformers. Attention allows the model to learn which words are related to each other in a sentence and weight them accordingly when making predictions.
    - Words are represented as vectors in a high-dimensional space (embeddings)
        - Dimensions can represent features like: `[animal, domesticated, pet, fluffy]`
        - These dimensions are not human-defined, the model learns them from data.
        - Similar words cluster together, related words appear in similar directions.
    - Architecture components:
        - Encoder - processes input
        - Decoder - generates output
        - Attention mechanism - learns relationships between words
        - Feed-forward neural networks - transforms representations
    - Transformers = Neural Network + Attention


## How Models Learn

- How does the model learn?
    - Next token prediction
        - Hide the last word in a sentence; force the model to predict it.
        - Penalize wrong predictions, reward correct ones → model maximizes probability of right word.
    - Masked Language Modeling (MLM)
        - Randomly mask words throughout a sentence (both left and right context visible).
        - Force the model to predict the masked words
        - This is the approach used in BERT

---

# Lecture 2: LLM Architecture, Model Size and Prompting

## What is a Language Model

- A language model calculates the probability of the next word given a sequence of words. It picks the word with the highest probability as the output.
- Training data sources- Wikipedia, books and open websites. This is used to build a pre-trained model.

## Fine-Tuning

- Fine-tuning is a separate training exercise on new domain-specific data. You do not add new data to the original training set. The model starts from its existing weights and learns from only the new data.
- E.g. a small language model was fine-tuned on data about Mahakumbh 2025 and ISRO's space docking experiment. Neither event was in the model's pre-training data. After fine-tuning, the model answered questions about these events correctly 80 out of 100 times, compared to the base model.

## Attention Mechanism

- Different words in a sentence have different strengths of connection with other words. The word "she" connects to "girl". The word "he" connects to "boy." The attention mechanism helps the model learn and use these connections.
- Context-dependent meaning- the word "bank" can mean a financial institution or a river bank. The model figures out the correct meaning by paying attention to surrounding words. A rule-based system would struggle with this because word meanings change with context.

## Word Vectors (Embeddings)

- Words are represented as vectors, which are lists of numbers. Similar words cluster together in this vector space.
- Opposite but related words like man and woman fall along the same directional axis.
- Words derived from the same root appear close to each other.
- The dimension of a vector represents characteristics learned by the model. Typical sizes- 512, 4096, 8192 or other multiples of 512. The maximum seen so far is around 16,000 dimensions.
- Increasing dimensions improves the model's ability to cluster similar words but also increases storage requirements.
- The model learns what the dimensions represent on its own. Humans only specify the size.

## Transformer Architecture

- Transformers- neural network plus attention mechanism.
- The encoder converts words into vectors.
- The decoder takes those vectors and calculates probabilities of the next set of words.
- Encoder only- BERT (used for sentiment classification, summarization tasks)
- Decoder only- GPT
- Encoder-decoder- BART (for language translation tasks)
- The original transformer paper proposed six stacks of encoders and six stacks of decoders. Inside each encoder- an attention layer and a feed-forward neural network. Inside each decoder- self-attention, cross-attention (encoder-decoder attention) and a feed-forward neural network.
- "A feedforward neural network (FNN) is the simplest type of artificial neural network where data flows in only one direction- forward, from the input layer, through hidden layers, to the output layer."

## Why Transformers Replaced RNNs and LSTMs

- RNNs and LSTMs processed words one at a time sequentially. This was slow and limited their context window.
- Gmail's autocomplete feature used RNNs and could only reliably predict around six to seven words ahead.
- LSTMs and GRUs improved on this but still had limited memory. They also lacked the attention mechanism needed for context-dependent meaning. Transformers solved both problems by processing all words in parallel and using attention to capture long-range dependencies.

## Model Size and Accuracy

- As model size (number of parameters) increases, accuracy generally goes up.
- BERT large- 340 million parameters.
- LLaMA 3.3- 70 billion parameters. Google PaLM- 540 billion parameters.
- GPT-4 and newer models likely have trillion-scale parameters but OpenAI has not disclosed the exact size.
- Size is not the only factor. How you train the model matters just as much.
    - A 1.5 billion parameter model trained with a standard approach scored around 2 to 2.5 on a Likert scale.
    - The same size model after supervised fine-tuning scored above 3.5.
    - Using the InstructGPT training approach, the same size model reached 4.5.
- **Key takeaway**- a small model trained well can outperform a large model trained poorly. This makes small language models practical and useful when fine-tuned properly.
- Fine-tuning improves the model not just on knowledge but also on the conversational quality of responses, how concise they are and how on-point the answers are. Human evaluators giving Likert scores will factor in these aspects along with factual accuracy.

## Prompting Techniques

- Zero-shot learning- you give the model a question with no examples.
- One-shot learning- you give one labelled example before asking the question.
- Few-shot learning- you give two or more labelled examples before asking the question. Examples do not need to be related to each other. The model learns the pattern of the task from the examples, not from the topic.

---

# Lecture 3: Prompt Engineering and RAG

## Setting Up the Environment

- Fork the Git repo into your own GitHub account.
- Open Google Colab. Colab is a Python IDE (integrated development environment).
- Clone the forked repo into Colab using the command `!git clone <repo-name>`. This copies the files into the Colab environment.
- Files with the extension `.ipynb` are Jupyter notebooks. You can open and edit them in Colab or download them to run locally if you have Jupyter installed.

## The Code and What It Does

- The notebook used in this session calls a hosted LLM via an API. The model used is sonar-reasoning from Perplexity AI. This is one of the more affordable reasoning models available. Other reasoning models like o1, o3 and o3-mini exist but are more expensive.
- Sonar-reasoning applies chain-of-thought reasoning by default without needing to be explicitly told to do so in every prompt.
- Perplexity AI does not build its own models from scratch. They fine-tune existing open-source models.
- To use the model, you need a Perplexity API key. Even $5 worth of credits is enough for several months of moderate use.
- The code makes a POST request to a URL where the model is running.
- Every time you run the same question you may get a slightly different answer. This is because the model samples from a probability distribution and does not produce a fixed deterministic output.
- The backslash `\` at the end of each line in a multi-line Python string tells the code that the string continues on the next line. It does not affect the prompt content itself.

## System Prompt vs User Prompt

There are two types of prompts:

- System prompt- set by the company that built the model. Users cannot see or change it. This controls how the model behaves at a fundamental level. The system prompt has higher priority than the user prompt.
- User prompt- what you define.
- Question- the actual query you send.
- Context- additional information you pass alongside your question.

All four of these get combined and sent to the model together. 
System prompt + user prompt + question + context all count toward the context window limit.


## Context Window

- The context window is the total number of tokens a model can process in one request.
- The model used in this session has a 128k token context window. One word is roughly 4 tokens so 128k tokens is approximately 24,000 to 32,000 words.
- If your combined input and output exceeds the context window the model will not generate a response or will give a very short one.

## RAG (Retrieval Augmented Generation)

- It is a technique where you pass additional context to the model alongside your question. This allows the model to answer questions about documents, books or PDFs that were not in its original training data.
- RAG is useful when the question is too specific or domain-specific for the model to answer from its training data alone.

---

# Lecture 4: API Code, RAG vs Fine-Tuning and Temperature

## Recap of Lecture 3

- The sonar-reasoning model from Perplexity was used to make API calls.
- Two versions of code were shown. In one the prompt was hardcoded. In the other the prompt could be edited.
- Sequence: Colab notebook → instructor's modal.run server → Perplexity API → back to colab as output
- System prompt + user prompt + question + context all count toward the context window limit.
- If context window exceeds the limit the response may be truncated or reduced to a single word.

## Code Walkthrough: What Is Inside the API

```python

import pprint
import requests                    # requests = for making API calls
import json
import re                          # re = for find and replace in strings
import pandas as pd
from pydantic import BaseModel
import logging as logger
from typing import Optional

class Input(BaseModel):
    rephrased_memories: Optional[str] = None   # the user's question
    api_key: Optional[str] = None              # perplexity API key
    modelname: Optional[str] = None            # e.g. "sonar-reasoning"
    user1: Optional[str] = None                # the AI persona e.g. "Lord Krishna"
    user2: Optional[str] = None                # the person asking e.g. "Anupama"
    gender: Optional[str] = None               # used to pick the right salutation

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

        # headers = the cover note sent with the request to prove we are a valid user
        # Authorization: Bearer is the standard way of sending an API key
        # Content-Type: application/json tells the server the data is in JSON format
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

---

## How to call this function from Colab

```python
# create the Input object with all your values
data = Input(
    rephrased_memories="how to get moksha",   
    api_key="your_api_key_here",               
    modelname="sonar-reasoning",               
    user1="Lord Krishna",                      
    user2="Anupama",                           
    gender="female"                            
)

result = krishna_uvach_class(data)
pprint.pprint(result)
```

---

## RAG vs Fine-Tuning

A paper compared RAG and fine-tuning on the same datasets.

Results from the paper (out of 30 questions):

- Knowledge accuracy: RAG won 26 out of 30. Fine-tuning won only 4 out of 30.
- Conversational style: fine-tuning won 17 out of 30. RAG won 13 out of 30.
- Conciseness: RAG did better.

Why fine-tuning had better conversational style: fine-tuning makes the model internalize the domain knowledge. It is like a student who has studied a subject deeply and can explain it naturally. RAG is like giving the same student a book and asking them to look up the answer. The answer is accurate but the delivery is more mechanical.

Why RAG had better knowledge accuracy: fine-tuning in this study used only 7,000 records. Fine-tuning generally starts working well when you have around 50,000 records. With more data and better hyperparameters fine-tuning results improve significantly.

Practical recommendation: for quick development use RAG. Fine-tuning requires more data and more GPU compute and is therefore more expensive.

Fine-tuning is transfer learning. You start from an existing pre-trained model and update its parameters on new domain-specific data. You do not retrain from scratch.

## Temperature

Temperature is a parameter that controls how creative or conservative the model's word selection is.

- Temperature = 0: the model picks only words with the highest probability. Responses are conservative and predictable. Least creative.
- Higher temperature: the model starts sampling words with lower probabilities. Responses become more varied and creative. Higher chance of hallucination because the model may pick words that are less relevant to the question.

There is a related parameter called Top P. Top P sets a probability threshold. The model only picks words with a probability above that threshold (e.g. 0.9). Increasing temperature effectively lowers this threshold.

---

# Lecture 5: Retrieval Augmented Generation (RAG)

## What RAG Stands For

Retrieval Augmented Generation.

- Retrieval- finding the relevant information from your own documents
- Augmented- adding that information to what you send the model
- Generation- the model uses that information to generate a response

## Components of a RAG System

**1. Language Model**
The LLM that generates the final response. Can be commercial or open-source.

**2. Embeddings Model**
Converts text into vectors. The three step process it handles is:

- Word to token (tokenization)
- Token to token number
- Token number to vector

**3. Vector Database**
Where all the vectors are stored. When a query comes in it also gets converted into a vector and then compared against everything in the vector database.

**4. Search and Retrieval**
The process of finding which stored vectors are most similar to the query vector. Two approaches are:

- Similarity search- compare query vector to stored vectors and rank by similarity score (0 = not similar, 1 = identical)
    - Define a cutoff- top k
- Maximum marginal relevance

## How to Pass Retrieved Chunks to the LLM

1. **Stuff**
    - Take all your top K retrieved chunks and just dump them all directly into the prompt at once.
2. **Map Reduce**
    - Instead of stuffing everything in at once you process each chunk separately.
    - Map- Each chunk gets sent to the LLM individually and gets its own response.
    - Reduce- Then all those individual responses get combined and sent to the LLM one more time to produce a final summarized answer
3. **Refine**
    - Also processes chunks one at a time but differently. It starts with the first chunk and gets an initial answer.
    - Then it takes that answer plus the second chunk and asks the LLM to refine the answer.
    - Then takes the refined answer plus the third chunk and refines again.
    - It keeps going chunk by chunk each time building on the previous answer.
4. **Map Rerank**
    - Sends each chunk to the LLM separately and asks it to generate an answer along with a confidence score for how relevant that chunk was. Then it picks the answer with the highest confidence score as the final response.

## Tokens vs Vectors

- Token- a unit of text.
    - Most words are one token. Longer or unusual words get broken into multiple tokens.
    - Punctuation is often its own token.
    - The tokenizer has a fixed vocabulary (e.g. 60,000 tokens). If a word is not in the vocabulary it gets broken into smaller pieces that are.
- Vector- the mathematical representation of a word after it has been through the embedding model.
    - A list of numbers across n dimensions (e.g. 4096 dimensions). Similar words cluster together in vector space.

## How the RAG Process Works Step by Step

- Take your source documents (PDF, CSV, HTML, Word, anything)
- Extract the text
- Convert all text into vectors using the embeddings model
- Store all vectors in a vector database
- User submits a query
- Query gets converted into a vector using the same embeddings model
- Similarity search runs against the vector database
- Top K most similar chunks are retrieved
- Those chunks plus your prompt plus the query all get sent together to the LLM
- The LLM generates a response grounded in the retrieved chunks

## RAG and Data Privacy

- Building your own RAG system means the vector database stays within your own infrastructure.
- You only pass the most relevant chunks to the LLM at query time so your full document never leaves your environment. This is a significant advantage for enterprise use cases with sensitive data.

---

# Lecture 6: Fine-Tuning, LoRA and Evaluation Metrics

- In fine-tuning we actually update the model's parameters. The model learns and memorizes the new information directly into its weights.

## Paper on Keyword Augmented Retrieval

- Standard RAG converts text into vectors and matches query vectors to document vectors by similarity.
- This fails when the relevant information appears very rarely in the document (sparse data).
- Keyword Augmented Retrieval solves this by extracting keywords from both the query and the document and matching them directly instead of comparing vectors. If the keywords in a paragraph match the keywords in the query that paragraph becomes the context.

## When to Use RAG vs Fine-Tuning

Use RAG when:

- Information keeps changing or evolving
- Fine-tuning every time new content is added is too expensive and time consuming
- We need the model to refer to specific up to date external documents

Use fine-tuning when:

- The behavior or style of the model needs to change
- Personalization is required
- The pattern of response is something that will stay consistent and not change frequently

## How the Fine-Tuning Data Was Generated

- Fine-tuning needs a lot of data in question-answer pair format.
- Use a large sophisticated LLM to generate synthetic question-answer pairs.
- These synthetic pairs are then used as training data to fine-tune a smaller model (LLaMA 3.1 70B).
- Deepseek- uses reinforcement learning(the agent takes actions to achieve a specific goal and receives feedback in the form of rewards or penalties, learning over time to maximize its cumulative score)
- When we fine-tune a model you update its parameters (weights) using the new question-answer pairs. The process is the same as the original training loop. We show it a question, it predicts an answer, the loss function measures how wrong it was and the parameters update to reduce that error. This repeats across all the training pairs.

## LoRA (Low Rank Adaptation)

- Full fine-tuning: updates all parameters of the model.
- LoRA: only updates a very small subset of parameters (<1% of total parameters). The rest of the model's weights stay frozen.
- Also called parameter-efficient fine-tuning (PEFT).
- LoRA rank- As you increase the rank more parameters become trainable.
- Reduces RAM and GPU requirements by roughly 80%.

## Evaluation Metrics

- After fine-tuning or building a RAG system we need to measure whether it is actually performing better.
- **HELM Framework** (Holistic Evaluation of Language Models) uses the following dimensions:
    - Accuracy
    - Calibration
    - Robustness
    - Fairness
    - Bias
    - Toxicity
    - Efficiency
- The right metric to optimize depends entirely on our use case. There is no single universal metric.

## Compute Requirements for Fine-Tuning

1 Parameter = 4 bytes (32 bit float)

1 billion parameters = 4 GB of storage

2 Adam Optimisers = +8 bytes per parameter 

Gradients = +4 bytes per parameter

Activations = +8 bytes per parameter

Total = 4 bytes per parameter + 20 extra bytes per parameter

Total memory requirement for full fine-tuning of a 70B model: approximately 160 GB RAM. This requires two A100 GPUs (each with 80 GB RAM).

## Quantization

- Technique to reduce memory requirements.
- Converts 32-bit float numbers to 16-bit or 4-bit. This reduces storage size significantly with only a small drop in model performance.
- Often used alongside LoRA.

---

# Lecture 8: Hybrid Retrieval (Vector + Keyword)

## Libraries and Setup

- Google API key is needed for the embeddings model used (embeddings-001 which is compatible with Gemini LLMs).

## Step 1: Load Documents

- One PDF or multiple PDFs can be loaded at the same time.

## Step 2: Chunking

- Chunk size- common sizes are 1024 or 2048 characters.
- The optimal chunk size depends on the context window of the model.

Chunking happens before embedding. The order is:

- Load text from documents
- Break text into chunks
- Convert each chunk into a vector using the embeddings model
- Store chunks and their corresponding vectors in the vector database (Pinecone, FAISS)

## Step 3: Vector Index and Keyword Index

Two separate indexes are created from the chunks:

**Vector Index**

- Each chunk is converted into a vector using the embeddings model. These vectors are stored in the vector database.

**Keyword Index (Keyword Table)**

- Keywords are extracted from each chunk. Stop words (prepositions, pronouns, common words) are discarded. Important words like nouns are kept. These keywords are stored in a keyword table alongside the chunk they came from.
- Storage location: by default the vector database is stored in memory on the local machine.
- If you set `persist=True` it gets saved to disk so it survives notebook restarts.

## Step 4: Hybrid Custom Retriever

A custom retriever is built that combines both the vector retriever and the keyword retriever. This is the hybrid search approach.

**Vector Retriever**
Finds chunks that are semantically similar to the query. The parameter `similarity_top_k = 5` means it picks the top 5 most similar chunks.

**Keyword Retriever**
Finds chunks that share keywords with the query. Looks for exact or near-exact keyword overlap between the query and stored chunks. 

**Why hybrid?**

- Vector similarity alone fails for sparse terms (a name appearing once)
- Keyword matching alone ignores meaning and context
- Together they cover both cases

The custom retriever can use AND or OR logic to combine results from both retrievers. The final result is a set of relevant chunks to pass as context.

## Step 5: Response Synthesizer

The response_synthesizer object takes two inputs:

- The custom retriever
- The LLM configuration

## Step 6: Query Engine

The query engine combines the custom retriever and the response synthesizer. When you pass a query to it:

- The query gets converted to a vector
- The retriever fetches relevant chunks (via vector and keyword search)
- The chunks + the query + the prompt instruction go into the LLM
- The LLM generates a response grounded only in those chunks

---

# Lecture 9: Multi-Agent AI with CrewAI

- CrewAI is a Python library for building multi-agent systems.
- Define multiple agents, assign each one a task and chain them together sequentially or in parallel.
- Other frameworks: LangGraph, LangChain.

## The Two-Agent Example

**Researcher agent**

- Task: search the web for information on a given topic
- Tool used: custom Google Search API

**Writer agent**

- Task: take the research output and summarize it into bullet points
- Tool used: LLM

## Code Structure

The code has five parts in order:

1. Install libraries
2. Import libraries
3. Set API keys (Google API key, Gemini/GCP key, CrewAI key)
4. Define agents and their prompts
5. Define tasks- defined in plain natural language

## How Chaining Works

Tasks are put into a list in the order you want them to run:

```python
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, write_task],
    process=Process.sequential
)
```

`Process.sequential` means tasks run one after another.

## Tools and APIs

Each agent can be given a tool. The tool is what it uses to take action in that step.

- Google Search- used by the researcher agent. Requires a Google API key and a Custom Search Engine (CSE) ID from GCP.
- LLM- used by the writer, reviewer and rewriter agents.
- Email API- used by a sender agent. Requires authentication credentials for that email service.
- Database- can connect a relational database too.

## Parallel Processing

- You can run two agents in parallel when their tasks are independent of each other.
- Example: scrape two different e-commerce websites simultaneously to compare prices.