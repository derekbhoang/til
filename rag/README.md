# Retrieval-Augmented Generation (RAG)

It's a technique that combines a large language model (LLM) with an external knowledge source, allowing the model to retrieve relevant information before generating a response.

## How it works:

1. **User asks a question**

   - Example: "What were our company's Q1 sales figures?"

2. **Retrieve relevant information**

   - The system searches a knowledge source such as:

     - Documents
     - PDFs
     - Databases
     - Websites
     - Internal company knowledge bases

3. **Augment the prompt**

   - The retrieved information is added to the model's context.

4. **Generate an answer**

   - The AI uses both its general knowledge and the retrieved information to produce a response.

## Why RAG:

**Advantages**

- Reduces hallucinations.
- Uses up-to-date information.
- Allows AI to answer questions about private data.
- No need to retrain the model whenever data changes.
- Can provide citations to source documents.

**Without RAG**

```
LLM → Answer from training data only
```

**With RAG**

```
LLM + Your Documents → Grounded Answer
```

**Common RAG architecture**

```
Documents
    ↓
Embedding Model
    ↓
Vector Database
    ↓
Similarity Search
    ↓
Relevant Chunks
    ↓
LLM
    ↓
Response
```