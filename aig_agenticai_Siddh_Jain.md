## What Are Language Models?

Language Models are probabilistic models.  
Language models predict the next token by calculating probability distributions over all possibilities.

**Traditional NLP** relied on rules and struggled with long-range dependencies.  
So we moved to **Transformers** (2017, Google - *Attention is All You Need*).

Data ---> Pre-trained model ---> Base Model ---> Fine-Tuned model (specialized - pick domain specific data) ---> Specified Task

LLMs are based on **Transformer Architecture** which contains neural networks.  
So in neural networks, weights or coefficients are updated to encode statistical patterns during training of LLMs.

When the model has to learn different things -> exposed to a lot of data -> need to learn dependencies -> learn how to pay **Attention** to different things.

## Attention: How strongly tokens relate to each other  
Cross Attention - Matching the info between input and output.
Self Attention - Relationship between tokens within the same sequence.
Example:
"The girl and the boy walked home. She"
The model links "She" to "The girl".

So during pre-training, the model creates linkage by observing the relationships and then when related info is asked, it gives the most probabilistic answer.

## Embeddings

Data is converted to vectors first.

- These vectors are multi-dimensional.
- For dogs, cats, rats, etc., vectors may have dimensions like:
  - animal
  - domestic
  - pet

Similar words cluster closely.

Example:
- king & queen
- man & woman

Transformers beat both RNN and LSTM because:
- They processed tokens sequentially.
- Slow training.
- Weak memory.

Transformers introduced:
- Attention
- Parallel processing


## Types of Modeling

1. **Next Token Prediction**
   - Give a sequence of words and make the model predict the last word.

2. **Masked Language Modeling**
   - Mask words in between so previous and next words are available.
   - The model predicts the missing word.

Encoders are best for understanding.  
Decoders are best for generating.

Most LLMs are decoder-based (masked self attention).


## Prompt
Natural language instruction given to an LLM.

- **Zero-shot** → no examples given
- **One-shot** → single example given
- **Few-shot** → multiple examples given

(Examples are classifications/answers to similar questions.)

## Structure

System Prompt + User Prompt + Question + Context + Rephrase Memory ---> LLM

- **System Prompt**
  - Company/host restrictions or predefined behavior.
  - Example: DeepSeek responses regarding Taiwan.

- **User Prompt**
  - Prompt given by the user.

- **Context**
  - External information such as retrieved documents.


## RAG (Retrieval Augmented Generation)

LLMs hallucinate when they lack reliable knowledge.  
Fixed memory isn't enough for constantly changing information.

RAG fixes this **without retraining**.

It retrieves external information and injects it into the prompt.

Its tokens must not exceed the model's context length.

## Context Window

Context Window = User Prompt + System Prompt + Question + Context + Response

Example: 128k context window.


## Code Within the API Called by the Model

1. Import libraries
2. Create class for input variables
   - rephrased memory
   - users
   - etc.

3. Function definition (generate response)
   - Header
     - API key
   - Payload
     - model name
     - user prompt
     - system prompt

4. Payload is sent to Perplexity API
   - returns:
     - response.txt
     - response status


## Temperature and TopP

These affect model output.

## TopP
TopP is usually fixed (example: 0.9).

The LLM only picks from the smallest set of tokens whose cumulative probability exceeds 0.9.

## Temperature
Controls randomness in token selection.

- Higher temperature:
  - more creativity
  - more randomness
  - higher hallucination chances


## RAG vs Fine-Tuning

**RAG**
Good for knowledge grounding.

Use when information changes frequently:
- facts
- reports
- research papers

Vector DB can be updated without retraining.

**Fine-Tuning**
Good for stable behavior:
- formatting
- behavioral personas
- conversational style

It permanently modifies the model.


## Token Processing

LLMs do not process words directly.

Text ---> Tokens ---> Token IDs ---> Embeddings ---> LLM


## Components of a RAG Framework

1. Language Model
2. Embedding Model
3. Vector Database
4. Search Type
5. Chain Type


## Vector Database

Source Data ---> Embedding Model ---> Stored as Vectors

## Retrieval Process

```text
Question
   ↓
Embed Query
   ↓
Query Vector
   ↓
Find Similar Vectors
   ↓
Extract Relevant Chunks
   ↓
Rank by Similarity Score
   ↓
Choose Top-K
   ↓
Pass Prompt + Relevant Docs to LLM
   ↓
Generate Response
```

## KAR (Keyword Augmented Retrieval)

When data is sparse, RAG may fail.

KAR solves this using keyword matching.

Input Document + Query ---> Keyword Matching ---> Prompt + Keywords as Context ---> LLM

## LoRA (Low-Rank Adaptation)

Parameter Efficient Fine-Tuning.

Full fine-tuning requires huge resources.  
LoRA updates only a small percentage of parameters.


## Quantization

Reducing the number of bits used to represent values.


## Multi-Agent Systems

Prompt ---> Agent1 ---> Agent2 ---> Final Response

Example:

Research Agent ---> Writer Agent ---> Final Output

Instead of one LLM doing everything, different agents handle different tasks.

**Advantages**

1. More accurate results
2. Easier debugging
3. Complex tasks handled easily


## CrewAI

Framework for creating and managing multiple AI agents.

## Components

1. Agents - Role assigned to the LLM

2. Tasks - Work done by agents

3. Crew - Agents + Tasks + Tools + Process

4. Tools - External utilities reducing hallucinations

5. Process Flow
   - Sequential
   - Parallel
   - Ordered workflows
