# Retrieval Augmented Generation (RAG)

**19 September 2026**

> Study notes — Claude Certified Architect (Foundations), Anthropic Academy

**Contents**
- [The Problem with Large Documents](#the-problem-with-large-documents)
- [Option 1 — Include Everything in the Prompt](#option-1--include-everything-in-the-prompt)
- [Option 2 — Break Documents into Chunks](#option-2--break-documents-into-chunks)
- [This is RAG](#this-is-rag)
- [Chunking Strategies](#chunking-strategies)
- [Text Embeddings](#text-embeddings)
- [The Full RAG Flow](#the-full-rag-flow)
- [BM25 Lexical Search](#bm25-lexical-search)
- [Multi-Index RAG Pipeline](#multi-index-rag-pipeline)
- [Re-ranking Results](#re-ranking-results)
- [Contextual Retrieval](#contextual-retrieval)

---

**Retrieval Augmented Generation (RAG)** is a technique that helps you work with large documents when using Claude. Instead of cramming an entire 800-page financial report into a single prompt, RAG lets you intelligently find and include **only the most relevant sections** for each question.

---

## The Problem with Large Documents

Imagine you have a massive financial document and want to ask Claude specific questions about it, like *"What risk factors does this company have?"*

You face a fundamental challenge:

> How do you get the **right information** from the document into Claude so it can answer your question effectively?

---

## Option 1 — Include Everything in the Prompt

The first approach seems straightforward: extract all the text from the document and stuff it directly into your prompt along with the user's question.

**Problems with this approach**

- There's a hard limit on how much text Claude can process — your document might be too long
- Claude becomes less effective with very long prompts
- Larger prompts cost more money and take longer to process

> **Worth noting (2026):** current Claude models have a **1M-token context window**, so the "document is simply too long" constraint binds far less often than it used to — an 800-page report may well fit. The other arguments for RAG still hold: **cost**, **latency**, and **attention** (Claude stays sharper on a focused prompt than a vast one). Treat Option 1 as genuinely viable for a single mid-size document, and reach for RAG when you have collections, tight cost targets, or repeated queries over the same corpus.

---

## Option 2 — Break Documents into Chunks

The second approach is more sophisticated. You break the document into smaller chunks during a **preprocessing step**, then find and include only the chunks relevant to each user question.

**How it works**

When a user asks *"What risks does this company face?"*, you search through your chunks to find the one about **"Risk Factors"** and include only that section in your prompt to Claude.

### Benefits vs Challenges

| ✅ Benefits | ⚠️ Challenges |
|---|---|
| Claude can focus on only the most relevant content | Requires a preprocessing step to split documents |
| Scales up to very large documents | Need a searching mechanism to find "relevant" chunks |
| Works with multiple documents | Included chunks might not contain all the context Claude needs |
| Smaller prompts cost less and run faster | Many ways to chunk text — which approach is best? |

> **The missing-context problem:** if you only include the "Risk Factors" section, you might miss important context from the "Strategy Outlook" section that addresses *how the company plans to handle those risks*.

---

## This is RAG

Option 2 **is** Retrieval Augmented Generation. Despite its complexity, RAG offers significant advantages for working with large documents — but it comes with technical challenges that require careful consideration.

### Key Components

1. **Document preprocessing and chunking**
2. **A search mechanism** to find relevant chunks
3. **Intelligent selection** of which chunks to include in prompts

```
Document  →  Chunk  →  Search  →  Select  →  Prompt  →  Claude
             (preprocess)  (per question)
```

### When to Use It

When considering RAG for your application, evaluate whether the benefits outweigh the additional complexity **for your specific use case**.

| | |
|---|---|
| **Shines when** | Working with large document collections where you need precise, contextual answers |
| **Costs you** | More upfront engineering work than simply including entire documents in prompts |

---

## Chunking Strategies

### 1. Size-Based

Size-based chunking is the most straightforward approach. You simply divide your document into chunks of approximately equal character or word count. It's easy to implement and works reliably across different document types.

However, this approach has clear downsides. Words get cut off mid-sentence, and chunks lose important context. For example, a chunk might not include the section header that would explain what the content is actually about.

The solution is to add **overlap** between chunks. Each chunk includes some characters from neighbouring chunks, ensuring better context preservation and avoiding abrupt cutoffs.

```python
def chunk_by_char(text, chunk_size=150, chunk_overlap=20):
    chunks = []
    start_idx = 0

    while start_idx < len(text):
        end_idx = min(start_idx + chunk_size, len(text))
        chunk_text = text[start_idx:end_idx]
        chunks.append(chunk_text)

        start_idx = (
            end_idx - chunk_overlap if end_idx < len(text) else len(text)
        )

    return chunks
```

### 2. Structure-Based

Structure-based chunking leverages the natural organisation of your documents. If you're working with markdown files, you can split on headers. For other formats, you might split on paragraphs or other structural elements.

This approach works beautifully when you have guarantees about document structure. For markdown documents, you can split on section headers:

```python
import re

def chunk_by_header(text):
    # Split by # headers in markdown
    sections = re.split(r'\n#+\s+', text)
    return sections
```

### 3. Semantic-Based

Semantic-based chunking is the most sophisticated approach. It analyses the meaning and relationships between sentences to group related content together. This typically involves:

- Breaking text into sentences
- Using NLP techniques to measure semantic similarity
- Grouping related sentences into coherent chunks

While this can produce the highest quality chunks, it's computationally expensive and more complex to implement. For most applications, the simpler approaches work well enough.

### Choosing the Right Strategy

Your choice of chunking strategy depends entirely on your specific use case:

| Your situation | Use |
|---|---|
| Consistent document structure | **Structure-based** — cleanest results |
| Mixed document types | **Sentence-based** often works well |
| Code or technical content | **Character-based** — most reliable |
| Unknown document formats | **Character-based** — safest bet |

> Chunking is often an **iterative process**. Start with a simple approach, test it with your specific documents and use cases, then refine based on the results. The "best" chunking strategy is the one that works reliably for your particular data and requirements.

---

## Text Embeddings

After extracting text chunks from a document, the next step in a RAG pipeline is **finding which chunks are most relevant** to a user's question. This is essentially a **search problem** — you need to look through all your chunks and identify the ones that relate to what the user is asking about.

### Semantic Search

The **most common approach** for finding relevant chunks is **semantic search**. Unlike traditional **keyword-based search**, semantic search uses **text embeddings** to understand the actual meaning of both the user's question and each text chunk. This allows the system to find **conceptually related content** even when the exact words don't match.

### What Are Text Embeddings?

A text embedding is a **numerical representation** of the meaning contained in some text. Think of it as converting words and sentences into a format that computers can work with mathematically.

Here's how the process works:

- You feed text into an embedding model
- The model outputs a long list of numbers (the embedding)
- Each number ranges from `-1` to `+1`
- These numbers represent different qualities or features of the input text

### Embeddings on Vertex AI

Claude **can't generate embeddings directly**. Instead, you need to use a specialised embedding model. On Vertex AI, the model we'll use is called `text-embedding-005`.

---

## The Full RAG Flow

### Step 1 — Chunk Your Source Text

First, we take our source document and break it into manageable chunks. For this example, we'll use two simple text sections:

| Section | Text |
|---|---|
| **1 — Medical Research** | "This year saw significant strides in our understanding of XDR-47, a 'bug' we have not seen before." |
| **2 — Software Engineering** | "This division dedicated significant effort to studying various infection vectors in our distributed systems" |

### Step 2 — Generate Embeddings

Next, we convert each text chunk into numerical embeddings. To make this easier to understand, let's imagine we have a perfect embedding model that always returns exactly two numbers, and we know what each number represents.

In our imaginary model:

- **First number:** how much the text talks about medicine
- **Second number:** how much the text talks about software engineering

| Section | Embedding | Why |
|---|---|---|
| Medical Research | `[0.97, 0.34]` | Very medical, somewhat software-related due to the word "bug" |
| Software Engineering | `[0.30, 0.97]` | Very software-focused, but "infection vectors" has medical connotations |

### Normalization

Before storing these embeddings, they go through a **normalization** process that scales each vector to have a magnitude of `1.0`. This is typically handled automatically by your embedding API, but it's important to understand it happens.

After normalization, our embeddings become `[0.944, 0.331]` and `[0.295, 0.955]`. We can visualise these on a **unit circle**, where both points lie exactly on the circle's edge.

### Step 3 — Store in a Vector Database

The normalized embeddings get stored in a **vector database** — a specialised database optimised for storing, comparing, and searching through long lists of numbers like our embeddings.

> ⚠️ **Check this — likely note-taking slip.** The notes say *"ChromeDB … a managed vector database service that runs on Google Cloud."* The database is almost certainly **ChromaDB** (Chroma), and Chroma is an **open-source** vector database you run yourself or embed in your app — not a Google-managed service. Google Cloud's own managed option is **Vertex AI Vector Search**.

### Step 4 — Process the User Query

At this point, we pause. All the work so far has been **preprocessing** that happens ahead of time. Now we wait for a user to submit a query.

The query gets embedded as `[0.1, 0.89]` — low medical score, high software engineering score. After normalization, it becomes `[0.112, 0.993]`.

### Step 5 — Find Similar Embeddings

Now we ask the vector database: *"Find the stored embedding that's closest to this user query embedding."* The database returns the **software engineering** section because it's the most similar.

But how does the database determine "closest"? It uses **cosine similarity**.

**Cosine similarity** — the vector database calculates the cosine of the angle between vectors to measure similarity. This gives us a number between `-1` and `1`:

| Value | Meaning |
|---|---|
| `1.0` | Vectors point in exactly the same direction — **very similar** |
| `0.0` | Vectors are perpendicular — **unrelated** |
| `−1.0` | Vectors point in opposite directions — **very different** |

### Step 6 — Build the Final Prompt

Finally, we take the user's question and the most relevant text chunk (the software engineering section) and combine them into a prompt for Claude.

> And that's the complete RAG pipeline. The system successfully found the most relevant context for the user's software engineering question and provided it to Claude for generating an informed response.

This process happens automatically every time a user submits a query, allowing Claude to answer questions based on **your specific documents** rather than just its general training knowledge.

---

## BM25 Lexical Search

When building a RAG pipeline, you'll quickly discover that semantic search alone doesn't always return the best results. Sometimes you need **exact term matches** that semantic search might miss. The solution is to combine semantic search with **lexical search** using a technique called **BM25**.

### The Problem with Semantic Search Alone

Say you're searching for a specific incident ID like `INC-2023-Q4-011` in a document. While this exact term appears multiple times in relevant sections, semantic search might return unrelated sections that are semantically similar but don't actually contain the specific term you're looking for.

This happens because semantic search focuses on **meaning** rather than exact matches. When you need precise term matching, you need a different approach.

### Hybrid Search Strategy

The solution is to run two searches **in parallel** and merge the results:

| Search | Role |
|---|---|
| **Semantic search** | Uses embeddings and vector databases for meaning-based matching |
| **Lexical search** | Uses classic text search for exact term matching |
| **Merge results** | Combines both result sets for better coverage |

### How BM25 Works

**BM25** (Best Match 25) is a popular algorithm for lexical search in RAG pipelines. It processes a search query in four key steps:

1. **Tokenize the query** — break the user's question into individual terms
2. **Count term frequency** — see how often each term appears across all documents
3. **Weight terms by rarity** — terms used less frequently get higher importance scores
4. **Find best matches** — return chunks that contain more instances of the higher-weighted terms

> The key insight is that **rare terms** like `INC-2023-Q4-011` are much more important for search than common words like "a" or "the".

---

## Multi-Index RAG Pipeline

When semantic search (using vector embeddings) and lexical search (using BM25) are combined, we get a hybrid search strategy that leverages the strengths of both approaches while mitigating their weaknesses. This is called a **multi-index RAG pipeline**.

### Creating a Unified Interface

Both search implementations share nearly identical APIs — they both have `add_document()` and `search()` methods that work the same way. This consistency makes it straightforward to wrap them in a single **Retriever** class.

The Retriever acts as a **coordinator** that:
1. Forwards user queries to both indexes
2. Collects their results
3. Merges them into a single ranked list

```
              ┌─→  Vector index (embeddings)  ─┐
Query  →  Retriever                             ├─→  RRF merge  →  Ranked results
              └─→  BM25 index (lexical)       ─┘
```

### Reciprocal Rank Fusion

The challenge is merging results from different search methods that use **different scoring systems**. Vector search returns cosine similarity scores, while BM25 returns relevance scores — you can't simply combine these numbers directly.

Instead, we use a technique called **Reciprocal Rank Fusion (RRF)**. This method focuses on the **rank position** of results rather than their raw scores.

### Testing the Hybrid Approach

When testing with the query *"what happened with INC-2023-Q4-011?"*, the hybrid approach delivers much better results than vector search alone.

### Benefits of the Hybrid Architecture

| Benefit | Detail |
|---|---|
| **Modular design** | Each search index is implemented independently with the same API |
| **Easy extensibility** | Add new search methods by implementing the same `search()` and `add_document()` interface |
| **Better accuracy** | Combines semantic understanding with exact keyword matching |
| **Flexible fusion** | The RRF algorithm works regardless of how many search indexes you combine |

---

## Re-ranking Results

The hybrid retrieval approach we've built works well, but it still has some rough edges. When we search for *"what did the eng team do with INC-2023-Q4-011?"*, we'd expect the Software Engineering section to rank higher, since it specifically mentions the engineering team and the incident. However, the Cybersecurity section still comes first.

This is where **re-ranking** comes in — a post-processing technique that can significantly improve retrieval accuracy.

### How Re-ranking Works

Re-ranking adds an **extra step after** your hybrid search process. Instead of just returning the merged results from your vector and BM25 indexes, you pass those results through an LLM for intelligent reordering.

The process is straightforward:

1. Run your existing hybrid search (vector + BM25)
2. Merge the results as before
3. Send the merged results to Claude with a **re-ranking prompt**
4. Get back a reordered list of the most relevant documents

```
Hybrid search  →  Merge  →  Claude (re-rank prompt)  →  Reordered results
```

### Trade-offs

Re-ranking comes with trade-offs to consider:

| | Trade-off |
|---|---|
| ✅ | **Improved accuracy** — the LLM can understand context and intent better than pure similarity scores |
| ⚠️ | **Increased latency** — you now need to wait for an additional LLM call to complete |
| ⚠️ | **Cost considerations** — each search now requires an LLM API call |

---

## Contextual Retrieval

Contextual retrieval is a technique that improves RAG pipeline accuracy by solving a fundamental problem: **when you split a document into chunks, each chunk loses its connection to the broader document context.**

### The Problem with Standard Chunking

When you take a source document and break it into chunks for your vector database, each individual piece no longer knows where it came from or how it relates to the rest of the document. This can hurt retrieval accuracy because the chunks lack important contextual information.

### How Contextual Retrieval Works

Contextual retrieval adds a **preprocessing step before** inserting chunks into your retriever database. The process:

1. Take each individual chunk **and** the original source document
2. Send both to Claude with a specific prompt asking it to add context
3. Claude generates a short snippet that **"situates"** the chunk within the larger document
4. Combine this context with the original chunk to create a **"contextualized chunk"**
5. Use the contextualized chunk in your vector and BM25 indexes

```
Chunk + Source document  →  Claude  →  Context snippet  →  Contextualized chunk  →  Indexes
```

### Handling Large Documents

A common problem is when your source document is **too large to fit into Claude's context window**. You can still use contextual retrieval by providing a reduced set of context.

Instead of including the entire document, provide:

- A few chunks from the **start** of the document (often containing summaries or abstracts)
- Chunks **immediately before** the chunk you're contextualizing

This approach gives Claude enough information to understand the document structure and immediate context without overwhelming the prompt.

### When to Use Contextual Retrieval

This technique is most valuable when:

- Your documents have **complex internal relationships** between sections
- Chunks **reference concepts defined elsewhere** in the document
- Understanding the **document structure** is important for accurate retrieval
- You're working with **technical documents, reports, or academic papers**

> While contextual retrieval adds processing time and cost (since you're making additional API calls), it can significantly improve retrieval accuracy for complex documents where context matters.
