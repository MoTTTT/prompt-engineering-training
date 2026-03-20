# 2.03 — RAG Architecture

## Key Insight

Retrieval-Augmented Generation (RAG) solves the problem of grounding an LLM in private or up-to-date data without retraining it. The three-stage pipeline — ingestion, retrieval, generation — is straightforward to describe but full of tuning decisions that determine whether your RAG system actually answers questions correctly or just appears to. This section covers the architecture decisions that matter, including the frequently-asked question of when to use RAG versus fine-tuning.

---

## The Three-Stage Pipeline

### Stage 1: Ingestion

Run once (or on a schedule when documents change), not on every user request.

```
Raw documents (Markdown, PDF, HTML, database records)
    |
    v
[Loader] — read documents into memory
    |
    v
[Chunker] — split into manageable segments (see 2.02 for strategies)
    |
    v
[Embedding model] — convert each chunk to a vector
    |
    v
[Vector store] — persist chunks + vectors
```

**Java (Spring AI + PgVector):**

```java
@Service
public class DocumentIngestionService {

    private final VectorStore vectorStore;

    public DocumentIngestionService(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void ingestDocuments(List<Path> documentPaths) {
        List<Document> rawDocs = documentPaths.stream()
            .map(this::loadDocument)
            .toList();

        // Chunk: 500 tokens per chunk, 100-token overlap
        List<Document> chunks = new TokenTextSplitter(500, 100).split(rawDocs);

        // Embed and store (Spring AI handles embedding transparently)
        vectorStore.add(chunks);

        log.info("Ingested {} chunks from {} documents", chunks.size(), documentPaths.size());
    }

    private Document loadDocument(Path path) {
        try {
            String content = Files.readString(path);
            Map<String, Object> metadata = Map.of(
                "source", path.getFileName().toString(),
                "ingested_at", Instant.now().toString()
            );
            return new Document(content, metadata);
        } catch (IOException e) {
            throw new IngestionException("Failed to load document: " + path, e);
        }
    }
}
```

**Python (Anthropic + Qdrant):**

```python
from anthropic import Anthropic
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
from openai import OpenAI  # using OpenAI embeddings with Anthropic for generation
import uuid
from pathlib import Path

anthropic_client = Anthropic()
openai_client = OpenAI()  # for embeddings only
qdrant_client = QdrantClient(host="localhost", port=6333)

COLLECTION_NAME = "documentation"
EMBEDDING_MODEL = "text-embedding-3-small"
EMBEDDING_DIM = 1536

def setup_collection():
    qdrant_client.recreate_collection(
        collection_name=COLLECTION_NAME,
        vectors_config=VectorParams(size=EMBEDDING_DIM, distance=Distance.COSINE)
    )

def embed_text(text: str) -> list[float]:
    response = openai_client.embeddings.create(model=EMBEDDING_MODEL, input=text)
    return response.data[0].embedding

def chunk_text(text: str, chunk_size: int = 500, overlap: int = 100) -> list[str]:
    words = text.split()
    chunks = []
    i = 0
    while i < len(words):
        chunk_words = words[i:i + chunk_size]
        chunks.append(" ".join(chunk_words))
        i += chunk_size - overlap
    return chunks

def ingest_document(file_path: str):
    text = Path(file_path).read_text()
    chunks = chunk_text(text)

    points = []
    for chunk in chunks:
        vector = embed_text(chunk)
        points.append(PointStruct(
            id=str(uuid.uuid4()),
            vector=vector,
            payload={"content": chunk, "source": file_path}
        ))

    qdrant_client.upsert(collection_name=COLLECTION_NAME, points=points)
    print(f"Ingested {len(points)} chunks from {file_path}")
```

### Stage 2: Retrieval

On every user request:

```
User question
    |
    v
[Embedding model] — same model used for ingestion
    |
    v
[Similarity search] — find top-K most similar chunks
    |
    v
[Optional: relevance threshold filter] — discard low-score chunks
    |
    v
Retrieved chunks → used in Stage 3
```

**Java (Spring AI + PgVector):**

```java
public List<Document> retrieve(String userQuestion, int topK, double minSimilarity) {
    SearchRequest request = SearchRequest.query(userQuestion)
        .withTopK(topK)
        .withSimilarityThreshold(minSimilarity);

    return vectorStore.similaritySearch(request);
}
```

**Python (Qdrant):**

```python
def retrieve(query: str, top_k: int = 3, min_score: float = 0.6) -> list[dict]:
    query_vector = embed_text(query)

    results = qdrant_client.search(
        collection_name=COLLECTION_NAME,
        query_vector=query_vector,
        limit=top_k,
        score_threshold=min_score
    )

    return [
        {"content": hit.payload["content"], "score": hit.score, "source": hit.payload["source"]}
        for hit in results
    ]
```

### Stage 3: Generation

Inject retrieved context into the grounded prompt and call the LLM:

**Java (Spring AI):**

```java
public String generateAnswer(String question, List<Document> retrievedChunks) {
    if (retrievedChunks.isEmpty()) {
        return "Information not found in internal documentation.";
    }

    String context = retrievedChunks.stream()
        .map(Document::getContent)
        .collect(Collectors.joining("\n\n---\n\n"));

    return chatClient.prompt()
        .system("""
            You are an internal Support Assistant.
            Use ONLY the provided context to answer the question.
            If the answer is not in the context, say: "Information not found in internal docs."
            """)
        .user(u -> u.text("""
            Context:
            {context}

            Question:
            {question}
            """)
            .param("context", context)
            .param("question", question))
        .call()
        .content();
}
```

**Python (Anthropic SDK):**

```python
def generate_answer(question: str, retrieved_chunks: list[dict]) -> str:
    if not retrieved_chunks:
        return "Information not found in internal documentation."

    context = "\n\n---\n\n".join(chunk["content"] for chunk in retrieved_chunks)

    response = anthropic_client.messages.create(
        model="claude-haiku-4-5",
        max_tokens=1024,
        system="""You are an internal Support Assistant.
Use ONLY the provided context to answer the question.
If the answer is not in the context, say: "Information not found in internal docs." """,
        messages=[{
            "role": "user",
            "content": f"Context:\n{context}\n\nQuestion:\n{question}"
        }]
    )
    return response.content[0].text

def ask(question: str) -> str:
    """Complete RAG pipeline: retrieve then generate."""
    chunks = retrieve(question)
    return generate_answer(question, chunks)
```

---

## RAG vs Fine-Tuning: The Decision Framework

This is one of the most frequently asked questions in enterprise AI adoption. The short answer: **start with RAG; fine-tune only when RAG has provably failed at something you cannot fix with better prompts or better retrieval.**

| Factor | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Data is private / not in training data** | Yes — inject at runtime | Possible, but requires retraining |
| **Data changes frequently** | Yes — update the vector store | Hard — requires retraining on new data |
| **Need citation/source traceability** | Yes — retrieved chunks are explicit sources | No — knowledge is baked into weights |
| **Data volume** | Handles any volume | Training data has practical limits |
| **Latency** | Adds retrieval step (~50–200ms) | No retrieval step |
| **Cost to update** | Low — re-ingest changed documents | High — retrain the model |
| **Behaviour/style/tone customisation** | Possible via prompt | Better — fine-tuning shapes model behaviour |
| **Consistent output format** | Via prompt + output validation | More reliable after fine-tuning |

**When RAG is clearly right:**
- Knowledge base assistant: "Answer questions about our internal documentation"
- Private data: "Summarise customer records from our CRM"
- Frequently changing data: "Answer questions about this week's incident reports"

**When fine-tuning may be worth considering:**
- You have 10,000+ high-quality labelled examples of the exact task
- The model needs to learn a specialised format or vocabulary that does not appear in its training data
- You have consistently failed to achieve the required quality with RAG + prompt engineering
- You have the budget and infrastructure for fine-tuning and evaluation

In practice, for most enterprise use cases, RAG + careful prompt engineering is sufficient. Fine-tuning adds significant complexity, cost, and maintenance overhead.

---

## Retrieval Quality Issues

### Chunk Size Trade-offs

| Chunk size | Benefit | Risk |
|-----------|---------|-----|
| Small (200–300 tokens) | Precise embeddings; specific retrieval | Lacks context; answers may be incomplete |
| Medium (400–600 tokens) | Balanced | Need to tune for your content type |
| Large (800–1200 tokens) | Rich context per chunk | Diluted embeddings; imprecise retrieval |

Start with 500 tokens. Test with a representative question set and measure whether users get complete answers.

### Top-K Selection

`topK=3` is a reasonable default. More chunks = more coverage = more cost + potential noise. Less chunks = cheaper + faster but may miss relevant content.

**Practical calibration approach:**

1. Build a test set of 20–30 question/answer pairs based on your documents
2. Measure recall at topK=1, 3, 5, 10 — what percentage of questions have the answer in the retrieved chunks?
3. Choose the lowest topK that achieves acceptable recall

### Relevance Threshold

Discarding low-similarity chunks prevents injecting noise into the context:

```java
// Discard chunks below 0.7 cosine similarity
.withSimilarityThreshold(0.7)
```

If your "Information not found" rate is too high, lower the threshold. If answers include too much irrelevant content, raise it. Monitor both metrics.

---

## Grounded Response Prompt Design

The grounded response prompt is critical to RAG quality. Key elements:

```
[ROLE]
You are an internal Support Assistant for Acme Corp engineering documentation.

[GROUNDING CONSTRAINT — this is the most important line]
Use ONLY the provided context to answer. Do not use your general knowledge.

[FALLBACK]
If the answer is not in the context, say exactly: "Information not found in internal docs."

[ANTI-HALLUCINATION]
Do not speculate or infer beyond what the context explicitly states.

[SOURCE CITATION — optional but valuable]
When you answer, mention which document or section the information comes from.
```

The fallback phrase should be:
- Literal and exact (so you can detect it programmatically)
- Unique enough that the model won't generate it for other reasons
- Useful to the user (tells them where to look next)

---

## Check Your Understanding

1. Walk through the three stages of a RAG pipeline in your own words. At which stage would you look first if users are getting "Information not found" responses to questions that should be answerable from your documents?

2. Your RAG system ingests a 200-page PDF. A user asks a question whose answer spans information from page 12 and page 87. Why might this produce a poor answer, and what retrieval or chunking strategy could help?

3. A colleague proposes fine-tuning a model on your company's documentation instead of building a RAG system. Using the decision framework above, what questions would you ask to determine whether fine-tuning is warranted?

4. Your RAG assistant is returning "Information not found" for 40% of queries, even though you believe the answers are in the documents. List three retrieval quality issues that could cause this and how you would diagnose each.

5. You are building a RAG system for a support assistant that handles 10,000 queries per day. With topK=5 and an average chunk size of 600 tokens, estimate the token cost of the retrieval context per day. What would reduce this cost by 50% while preserving quality?
