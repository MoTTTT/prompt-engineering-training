# 2.02 — Embeddings and Vector Search

## Key Insight

Embeddings convert text into vectors of meaning — lists of numbers where proximity in the vector space reflects semantic similarity. This is the mathematical foundation of semantic search: "find me documents similar in meaning to this query" rather than "find me documents containing these exact words." Understanding embeddings, similarity metrics, and vector databases gives you the tools to debug retrieval quality and make informed decisions about chunking strategy and database selection.

---

## What Are Embeddings?

An embedding is a fixed-length list of numbers (a vector) that represents the semantic meaning of a piece of text. The key properties:

- The vector has a fixed dimension regardless of input text length (common sizes: 768, 1536, 3072 dimensions)
- Texts with similar meaning produce similar (numerically close) vectors
- The relationship is geometric: meaning is encoded as position in a high-dimensional space

**Concrete example (simplified to 3 dimensions for illustration):**

```
"deploy to production"  →  [0.82, 0.15, 0.43]
"push to prod"          →  [0.79, 0.18, 0.41]   ← similar meaning, numerically close
"VPN configuration"     →  [0.12, 0.91, 0.67]   ← different meaning, numerically distant
```

In reality, vectors have hundreds or thousands of dimensions, but the principle is the same: meaning is encoded as position in a high-dimensional space.

**What embeddings are NOT:**

- Not the original text (you cannot reconstruct text from an embedding)
- Not a compression of the text (the vector has a fixed size regardless of text length)
- Not a simple bag-of-words frequency count

---

## How Similarity Search Works

To find documents semantically similar to a query:

1. Embed the query: convert it to a vector
2. Compare the query vector to all document vectors in the database
3. Return the documents with the highest similarity scores

The comparison step uses a similarity metric. Two common ones:

### Cosine Similarity

Measures the angle between two vectors. Range: -1 (opposite) to 1 (identical).

```
cosine_similarity(A, B) = (A · B) / (|A| × |B|)
```

Conceptually: if two vectors point in the same direction, they are similar regardless of magnitude. Cosine similarity is scale-invariant — a short document and a long document on the same topic can have high cosine similarity even though their raw vectors have very different magnitudes.

**Most vector databases use cosine similarity by default.** It is appropriate for text similarity in RAG.

### Dot Product

The raw dot product without normalisation. Magnitude affects the result. Used in some embedding models where the magnitude of the vector encodes additional information (e.g., overall "relevance" of the document).

**When to use dot product:** When the embedding model documentation specifically recommends it. Otherwise, use cosine similarity.

---

## Embedding Models

The embedding model determines the quality of your vector representations. Different embedding models have different:

- Dimension sizes (higher = more nuanced, more expensive to store)
- Max input length (documents longer than this are truncated)
- Specialisation (code, natural language, multilingual)

| Model | Provider | Dimensions | Max tokens | Best for |
|-------|----------|-----------|-----------|---------|
| `text-embedding-3-small` | OpenAI | 1536 | 8191 | General text; cost-effective |
| `text-embedding-3-large` | OpenAI | 3072 | 8191 | Higher accuracy; higher cost |
| `embed-english-v3.0` | Cohere | 1024 | 512 | English text optimised |
| `all-mpnet-base-v2` | Sentence Transformers | 768 | 514 | Self-hosted; strong general performance |
| Spring AI default | Via Spring AI config | Provider-dependent | Provider-dependent | Configured via `spring.ai.embedding.model` |

**Critical rule:** Use the same embedding model for ingestion and retrieval. If you embed documents with `text-embedding-3-large` and embed queries with `text-embedding-3-small`, the vector spaces do not match and similarity scores are meaningless.

---

## Vector Databases

A vector database stores embeddings and performs efficient similarity search at scale. Without a vector database, finding the most similar vector among one million documents would require comparing the query to all one million vectors — O(n) per query. Vector databases use approximate nearest-neighbour (ANN) algorithms to do this in O(log n) or better.

### Comparison of Common Options

| Database | Type | When to choose | Managed option |
|----------|------|---------------|---------------|
| **PgVector** | Postgres extension | You already run Postgres; want SQL + vectors in one place | Supabase, Neon, AWS RDS |
| **Pinecone** | Managed cloud-native | Simplest managed option; no infrastructure | Pinecone.io |
| **Qdrant** | Open-source, self-host or managed | Rich filtering, high performance, free open-source | Qdrant Cloud |
| **Weaviate** | Open-source, self-host or managed | Schema-based; good for structured + vector hybrid search | Weaviate Cloud |
| **Chroma** | Embedded, open-source | Development and local testing; not production at scale | None |

**For Spring Boot teams:** PgVector is the natural starting choice if you are already running Postgres. Spring AI has first-class PgVector support. You avoid introducing a new database technology.

**For Python teams starting fresh:** Qdrant has an excellent Python client, is fully open-source, and handles high throughput. Chroma is the default for local development.

### Setting Up PgVector with Spring AI

```sql
-- Enable the pgvector extension
CREATE EXTENSION IF NOT EXISTS vector;

-- Spring AI creates this table automatically if spring.ai.vectorstore.pgvector.initialize-schema=true
-- Or create manually:
CREATE TABLE IF NOT EXISTS vector_store (
    id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
    content TEXT NOT NULL,
    metadata JSON,
    embedding VECTOR(1536)  -- dimension must match your embedding model
);

CREATE INDEX ON vector_store USING ivfflat (embedding vector_cosine_ops) WITH (lists = 100);
```

```properties
# application.properties
spring.ai.vectorstore.pgvector.index-type=IVFFLAT
spring.ai.vectorstore.pgvector.distance-type=COSINE_DISTANCE
spring.ai.vectorstore.pgvector.dimensions=1536
spring.datasource.url=jdbc:postgresql://localhost:5432/ragdb
```

---

## Chunking Strategies

Before embedding documents, you must split them into chunks. The chunking strategy directly affects retrieval quality.

### Why Chunking Matters

The embedding of a 10-page document represents the "average" meaning of all 10 pages. A query about one specific paragraph on page 7 may not match this average well, even though the relevant content is present.

Smaller chunks give more precise embeddings. Larger chunks provide more context. The tension between these is the core chunking trade-off.

### Strategy 1: Fixed-Size Chunking

Split documents into chunks of N tokens (or characters), with M tokens of overlap between adjacent chunks.

```java
// Spring AI: TokenTextSplitter with chunk size and overlap
List<Document> chunks = new TokenTextSplitter(500, 100).split(rawDocuments);
// 500 tokens per chunk, 100-token overlap between chunks
```

```python
# Python with LangChain (common in Python RAG pipelines)
from langchain.text_splitter import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=500,         # characters, not tokens
    chunk_overlap=100,      # overlap to preserve context at boundaries
    length_function=len,
)
chunks = splitter.split_text(document_text)
```

**When to use:** Quick to implement; works reasonably well for uniform text like API documentation or FAQs. The overlap ensures that content at chunk boundaries is not lost.

**When it fails:** Documents with natural section boundaries (headings, numbered lists, paragraphs) get split mid-section, producing chunks that lack context for their content.

### Strategy 2: Semantic Chunking

Split at natural boundaries: paragraph breaks, section headings, or sentence boundaries.

```python
def chunk_by_paragraphs(text: str, max_tokens: int = 600) -> list[str]:
    """Split at double newlines (paragraph breaks), merge short paragraphs."""
    paragraphs = [p.strip() for p in text.split('\n\n') if p.strip()]
    chunks = []
    current_chunk = ""

    for para in paragraphs:
        if len(current_chunk) + len(para) > max_tokens * 4:  # rough character estimate
            if current_chunk:
                chunks.append(current_chunk.strip())
            current_chunk = para
        else:
            current_chunk = current_chunk + "\n\n" + para if current_chunk else para

    if current_chunk:
        chunks.append(current_chunk.strip())
    return chunks
```

**When to use:** Documentation with clear structure (Markdown, HTML, well-formatted PDFs). Semantic chunks preserve context better, leading to higher retrieval quality.

**When it fails:** Poorly structured documents (OCR output, email threads, informal notes) lack reliable boundaries.

### Strategy 3: Hierarchical Chunking

Maintain both a document-level embedding (for broad topic matching) and section-level embeddings (for precise retrieval). Retrieve at the document level first, then narrow to sections.

```
Document: "Spring Boot Security Guide"
├── Document embedding: represents the whole guide
├── Section: "Authentication" → section embedding
│   └── Subsection: "JWT Configuration" → subsection embedding
└── Section: "Authorisation" → section embedding
    └── Subsection: "Role-Based Access" → subsection embedding
```

**When to use:** Large documents where the chapter or section structure is important. Good for technical manuals and books.

**Complexity:** More complex to implement and query. Consider whether the quality improvement justifies the effort.

### Chunking Decision Guide

| Document type | Recommended strategy | Chunk size |
|---------------|---------------------|-----------|
| FAQ, short articles | Fixed-size | 300–500 tokens |
| Technical documentation | Semantic (by section) | 400–800 tokens |
| Long-form books/manuals | Hierarchical | Section: 600–1200 tokens |
| Code files | By function/class boundary | Per function |
| Emails/transcripts | Fixed-size with overlap | 200–400 tokens |

---

## Check Your Understanding

1. You have two documents: "How to configure HTTPS in Nginx" and "How to enable TLS termination in a load balancer." A user queries "how do I set up SSL?" Explain why semantic search (embeddings) would likely retrieve both documents while keyword search might retrieve neither.

2. You build a RAG system for a 500-page technical manual. Users report that answers to questions about Chapter 3 are consistently wrong, even though the manual is fully ingested. What chunking problem might cause this, and how would you investigate?

3. A team has been using `text-embedding-3-small` for ingestion and has just switched to `text-embedding-3-large` for new documents. What problem does this create, and how do you fix it?

4. Compare PgVector and Pinecone as vector database choices for a Spring Boot team that already runs Postgres. In what circumstances would you choose each?

5. Your retrieval system returns chunks with similarity scores of 0.62, 0.61, 0.58, 0.52, and 0.31. The last chunk is clearly off-topic. How would you implement a similarity threshold, and what are the trade-offs of setting it too high versus too low?
