# Chunking Strategy Demo

This folder demonstrates several ways to split text into chunks for retrieval-augmented generation (RAG). The examples start with simple character-based splitting and build up to LLM-assisted "agentic" chunking.

The code is intentionally small and uses tiny chunk sizes so you can see the boundaries clearly in the console output.

## Setup

From this directory:

```powershell
uv venv
uv pip install -r requirements.txt
```

The demo uses both Ollama and OpenAI:

- Ollama is used for local chat and local embeddings in the final RAG step.
- OpenAI is used for semantic chunking and proposition extraction.

Expected local Ollama models:

```powershell
ollama pull llama3.2
ollama pull nomic-embed-text
```

Required environment variable:

```powershell
$env:OPENAI_API_KEY="your-api-key"
```

Optional model overrides:

```powershell
$env:OLLAMA_CHAT_MODEL="llama3.2"
$env:OLLAMA_EMBEDDING_MODEL="nomic-embed-text"
$env:OPENAI_MODEL="gpt-4.1-mini"
$env:OPENAI_EMBEDDING_MODEL="text-embedding-3-small"
```

Run:

```powershell
.\.venv\Scripts\python.exe main.py
```

## Why Chunking Matters

LLMs and embedding models work best when the context they receive is focused. A chunk should usually be large enough to preserve meaning, but small enough that retrieval can return the exact material needed for a question.

Bad chunking can cause two common problems:

- Chunks are too small, so important context is split apart.
- Chunks are too large, so retrieval brings back irrelevant material.

The best strategy depends on the source material. Plain prose, Markdown, source code, and dense factual writing often benefit from different splitting rules.

## Strategies Demonstrated

### 1. Manual Character Splitting

The first example slices a string every fixed number of characters.

This is useful for showing the basic idea of chunking, but it is usually a poor production strategy because it can cut through words, sentences, or concepts.

Use it when:

- You are teaching or debugging chunk boundaries.
- You need a quick baseline.

Avoid it when:

- The text has meaningful sentence, paragraph, or document structure.

### 2. CharacterTextSplitter

`CharacterTextSplitter` automates fixed-size splitting. In this demo it uses an empty separator, so it behaves similarly to manual character slicing.

Compared with manual slicing, it is easier to configure and plugs directly into LangChain document creation.

Use it when:

- Your input has simple separators.
- You want a basic LangChain splitter with predictable behavior.

### 3. RecursiveCharacterTextSplitter

`RecursiveCharacterTextSplitter` tries a list of separators in order, usually starting with larger boundaries like paragraphs, then lines, then spaces, then individual characters.

This is often the default practical choice for general text because it tries to keep natural language structure intact before falling back to smaller splits.

Use it when:

- You have prose or mixed plain text.
- You want a robust default splitter.
- You care about preserving paragraphs and sentences where possible.

### 4. Document-Specific Splitting

Document-specific splitters understand the shape of a particular format.

This demo includes:

- `MarkdownTextSplitter` for Markdown headings and sections.
- `PythonCodeTextSplitter` for Python code.
- `RecursiveCharacterTextSplitter.from_language(..., Language.JS)` for JavaScript.

These splitters are better than generic character splitting because they respect structure such as headings, classes, functions, and language syntax.

Use them when:

- Your documents have a known format.
- You want chunks that align with headings, functions, classes, or code blocks.

### 5. Semantic Chunking

`SemanticChunker` uses embeddings to compare neighboring sentences. When the semantic difference between sentences is large enough, it creates a split.

Instead of splitting by length alone, it attempts to split where the topic changes.

Use it when:

- Topic boundaries matter more than exact chunk size.
- The source is prose with multiple ideas mixed together.
- You want chunks that feel conceptually coherent.

Tradeoffs:

- It is slower than character splitting.
- It requires an embedding model.
- In this project it comes from `langchain_experimental`, so the API may change.

### 6. Proposition-Based Chunking

The proposition step asks an LLM to convert paragraphs into small standalone factual claims.

For example, a paragraph might become several atomic statements such as:

- Text splitting divides large text into smaller segments.
- Smaller text segments can improve processing efficiency.
- Text splitting helps with extracting specific context.

This separates the act of understanding the text from the act of grouping it.

Use it when:

- The source text contains dense factual information.
- You want retrieval units closer to individual claims.
- You want later grouping to be based on meaning instead of original paragraph layout.

Tradeoffs:

- It requires an LLM call.
- The quality depends on the model's extraction accuracy.
- Extracted propositions may lose style, tone, or surrounding nuance.

### 7. Agentic Chunking

`AgenticChunker` groups propositions into topic-based chunks. For each new proposition, it asks an LLM whether the proposition belongs in an existing chunk or should start a new one.

Each chunk stores:

- A short id.
- A title.
- A summary.
- A list of propositions.

When a proposition is added to a chunk, the chunk title and summary can be updated so the chunk metadata stays useful.

Use it when:

- You want chunks organized by topic rather than source order.
- You have many small propositions that need semantic grouping.
- You want human-readable chunk titles and summaries for inspection.

Tradeoffs:

- It makes many LLM calls.
- It is slower and more expensive than deterministic splitting.
- Chunk assignment can vary by model and prompt.

## Final RAG Step

After agentic chunking, the demo converts each grouped chunk into a LangChain `Document`, stores those documents in Chroma, retrieves relevant chunks, and asks Ollama to answer:

```text
What is the use of Text Splitting?
```

This final step shows the end-to-end purpose of chunking: make retrieval return focused context so the model can answer from the retrieved material.

## Choosing a Strategy

Use this rough guide:

| Source type | Good starting strategy |
| --- | --- |
| Plain prose | `RecursiveCharacterTextSplitter` |
| Markdown docs | `MarkdownTextSplitter` |
| Python code | `PythonCodeTextSplitter` |
| JavaScript code | Language-aware recursive splitting |
| Topic-heavy essays | Semantic chunking |
| Dense factual text | Proposition-based chunking |
| Knowledge-base synthesis | Agentic chunking |

For most RAG systems, start simple with recursive character splitting. Move to semantic, proposition-based, or agentic chunking only when retrieval quality justifies the extra complexity and model calls.
