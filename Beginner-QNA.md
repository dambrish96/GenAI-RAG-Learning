# Beginner Q&A - RAG Fundamentals (15 Questions)

**For those just starting their RAG journey**

---

## Q1: What is Retrieval-Augmented Generation (RAG)?

**Answer:**
Retrieval-Augmented Generation (RAG) is a technique that combines **information retrieval** with **text generation** to create more accurate, up-to-date, and reliable AI responses.

**Simple flow:**
1. **Retrieve**: Find relevant documents/information based on user query
2. **Augment**: Add retrieved information to the prompt
3. **Generate**: Use LLM to generate answer based on retrieved context

**Example:**
- **Without RAG**: "What were the latest AI breakthroughs in 2025?"
  - LLM responds: "I was trained until April 2024, I don't know about 2025 events"
  
- **With RAG**: "What were the latest AI breakthroughs in 2025?"
  - System retrieves 2025 news articles
  - LLM generates: "Based on the latest news, in 2025 the breakthrough was..."

**Why RAG matters:**
✅ Prevents hallucinations  
✅ Works with current information  
✅ More trustworthy answers  
✅ Cost-effective vs fine-tuning  

---

## Q2: How does RAG differ from fine-tuning?

**Answer:**

| Aspect | RAG | Fine-Tuning |
|--------|-----|-------------|
| **Approach** | Retrieve context, then generate | Train model on data |
| **Time** | Minutes | Hours/Days |
| **Cost** | Cheap (inference) | Expensive (training) |
| **Knowledge Update** | Real-time | Need to retrain |
| **Data Size** | Works with any size | Needs large dataset |
| **Customization** | Limited | High |
| **Best For** | Current info, Q&A | Style, specific tasks |

**When to use RAG:**
- Real-time knowledge needed
- Budget constraints
- Frequently changing information
- General knowledge Q&A

**When to use Fine-tuning:**
- Need specific writing style
- Limited inference budget
- Stable information
- Domain-specific behavior

**Can combine both**: RAG + fine-tuned model = best of both worlds

---

## Q3: What is a vector embedding?

**Answer:**
A vector embedding is a **numerical representation of text** - converting words/documents into a list of numbers (typically 300-3000 numbers).

**Example:**
```
Text: "I love pizza"
Embedding: [0.2, -0.1, 0.8, 0.3, ..., 0.5]  (e.g., 768 numbers)
```

**Key concepts:**
- **Fixed size**: Every text becomes same-length vector
- **Semantic meaning**: Similar meanings = similar vectors
- **Dimensions**: Number of elements (768 is common)

**How embeddings work:**
1. Convert word to vector
2. Run through neural network
3. Get semantic representation
4. Use for similarity comparison

**Similarity = Meaning:**
- "Dog" embedding ≈ "Puppy" embedding (similar meaning)
- "Dog" embedding ≠ "Car" embedding (different meaning)

**Used in RAG:**
- Convert document chunks to embeddings
- Store in vector database
- Find similar chunks to query

---

## Q4: What is a vector database?

**Answer:**
A vector database is a **specialized database** that stores and retrieves **vector embeddings** efficiently.

**How it works:**
1. Store millions of vectors + metadata
2. Query: "Find top-10 vectors similar to this query"
3. Return: Most similar vectors in milliseconds

**Traditional Database:**
```sql
SELECT * FROM documents WHERE title LIKE '%RAG%'
```

**Vector Database:**
```python
results = vector_db.search(
    query_embedding=[0.2, -0.1, 0.8, ...],
    top_k=10  # Return top 10 similar
)
```

**Popular Options:**
- **Pinecone** - Managed (easiest)
- **Weaviate** - Open-source, graph-based
- **Milvus** - High-performance
- **Qdrant** - Rust-based, fast
- **Chroma** - Lightweight, local
- **PgVector** - PostgreSQL extension

**Why needed:**
- Regular DBs can't handle similarity search
- Need fast nearest neighbor search
- Handle high-dimensional data
- Scale to millions of vectors

---

## Q5: What is semantic similarity?

**Answer:**
Semantic similarity measures **how similar two pieces of text are in meaning**, not just words.

**Example:**
- "I love dogs" vs "Dogs are great" → **High similarity** (same meaning)
- "I love dogs" vs "I hate cats" → **Low similarity** (different meaning)
- But both contain "dogs"!

**How it's measured:**
- Convert both texts to embeddings
- Calculate distance between vectors
- Closer vectors = more similar meanings

**Common metrics:**

**1. Cosine Similarity** (most popular)
- Measures angle between vectors
- Range: 0 to 1 (0=different, 1=identical)
- Example: 0.92 = very similar

**2. Euclidean Distance**
- Straight-line distance between points
- Smaller = more similar
- Less common than cosine

**3. Dot Product**
- Simple multiplication
- Fast but needs normalized vectors

**In RAG context:**
- Query embedding compared to document embeddings
- Find documents most semantically similar to query
- Cosine similarity typically used

---

## Q6: Explain document chunking and why it matters

**Answer:**
Document chunking is **breaking large documents into smaller pieces** before storing them in a vector database.

**Why chunking?**

Problems without chunking:
- Document too large → Embedding quality poor
- Can't retrieve exact relevant section
- LLM context window too small
- Wasted storage & computation

**Example:**

**Bad (No chunking):**
```
Document: "The solar system... [10,000 words] ...Jupiter has 79 moons"
Query: "How many moons does Jupiter have?"
Result: Returns entire 10,000 word document ❌
```

**Good (With chunking):**
```
Chunk 1: "The solar system contains 8 planets..."
Chunk 2: "Mercury is the smallest planet..."
...
Chunk 42: "Jupiter has 79 moons and is the largest planet"
Query: "How many moons does Jupiter have?"
Result: Returns Chunk 42 ✅
```

**Common chunking strategies:**

1. **Fixed-size chunking** (simplest)
   - Split every 500 tokens
   - Add 50 token overlap
   - Fast but may cut mid-sentence

2. **Semantic chunking** (smarter)
   - Split at sentence/paragraph boundaries
   - Preserve meaning
   - Slower but better results

3. **Language-specific** (best)
   - Respect language structure
   - Python code chunks ≠ English text chunks

**Chunk size tradeoff:**
- **Small chunks (100 tokens)**: Precise but more chunks
- **Large chunks (1000 tokens)**: Less precise, fewer chunks
- **Typical**: 300-500 tokens with 50 token overlap

**In RAG:**
- Chunking affects retrieval quality
- Affects cost (more chunks = more API calls)
- Critical preprocessing step

---

## Q7: What's the difference between BM25 and semantic search?

**Answer:**

| Feature | BM25 | Semantic Search |
|---------|------|-----------------|
| **Type** | Keyword/Sparse | Dense/Semantic |
| **How it works** | Keyword matching | Embedding similarity |
| **Speed** | Very fast | Slightly slower |
| **Quality** | Exact matches | Meaning-based |
| **Example** | "dog" finds "dog" | "dog" finds "puppy" |

**BM25 (Traditional):**
```
Query: "machine learning basics"
Finds: Documents containing those exact words
Speed: Very fast (milliseconds)
But: Misses "deep learning intro" (same topic, different words)
```

**Semantic Search:**
```
Query: "machine learning basics"
Finds: Documents about ML fundamentals
Quality: Understands meaning
Speed: Slightly slower
Also finds: "AI learning principles" (similar meaning)
```

**Hybrid approach (Best):**
- Use both BM25 and semantic search
- Combine results
- Get speed + quality
- Typical setup in production

**Example hybrid:**
```
Query: "How to start learning AI?"
BM25 results: "AI for beginners", "Learning AI"
Semantic results: "Machine learning intro", "Getting started with deep learning"
Combined: Best of both
```

---

## Q8: What is a prompt in the context of RAG?

**Answer:**
A prompt is the **complete instruction given to an LLM** in a RAG system, including:
1. Original user query
2. Retrieved context
3. Instructions for generation

**Example RAG prompt:**

```
System: You are a helpful AI assistant.

Context (from retrieval):
Jupiter is the fifth planet from the Sun and 
the largest planet in the Solar System. It has 
79 known moons. Jupiter's Great Red Spot is a 
storm larger than Earth.

User Query: How many moons does Jupiter have?

Please answer based on the provided context.
```

**LLM sees:** Context + Query  
**LLM generates:** "Jupiter has 79 known moons."

**Key elements:**
1. **System prompt** - Instructions to LLM
2. **Retrieved context** - Information from documents
3. **User query** - Original question
4. **Instructions** - How to answer

**Why important:**
- Formatting affects quality
- Context order matters
- Instructions guide generation
- Can prevent hallucinations

**Simple prompt template:**
```
Answer the user question based on this context:

CONTEXT:
[Retrieved documents]

QUESTION:
[User question]

ANSWER:
```

---

## Q9: What are embeddings models? Name some popular ones.

**Answer:**
Embedding models are **neural networks that convert text to vectors**.

**How they work:**
```
Input text: "Hello world"
↓ (through neural network)
↓
Output embedding: [0.1, 0.2, -0.3, ..., 0.5]
```

**Popular Open-Source Models:**

1. **Sentence Transformers** (all-MiniLM-L6-v2)
   - Small, fast (22MB)
   - 384 dimensions
   - Great for beginners
   - Good quality/speed balance

2. **all-mpnet-base-v2**
   - Better quality
   - Slower
   - 768 dimensions
   - Production-ready

3. **BGE Large** (BAAI)
   - State-of-the-art quality
   - 1024 dimensions
   - Popular in 2024-2025

**Commercial APIs:**

1. **OpenAI text-embedding-3-small**
   - 512 dimensions
   - $0.02 per 1M tokens
   - Reliable, popular

2. **OpenAI text-embedding-3-large**
   - 3072 dimensions
   - $0.13 per 1M tokens
   - Best quality

3. **Cohere Embed v3**
   - Good quality
   - Competitive pricing
   - Good for multilingual

**How to choose:**
- **Quality**: OpenAI 3-large
- **Speed**: all-MiniLM
- **Balance**: all-mpnet or BGE
- **Cost**: Open-source models
- **Multilingual**: Cohere or multilingual BERT

---

## Q10: Explain how RAG retrieval works step-by-step

**Answer:**

**Step 1: Prepare (Offline - Done Once)**
```
Document: "Paris is the capital of France"
↓
Break into chunks
↓
Chunk: "Paris is the capital of France"
↓
Convert to embedding: [0.2, -0.1, 0.8, ...]
↓
Store in vector database
```

**Step 2: Query (Online - Per Question)**
```
User: "What's France's capital?"
↓
Convert query to embedding: [0.21, -0.09, 0.79, ...]
↓
Search vector database: "Find similar embeddings"
↓
Vector DB returns: Similar chunks
↓
Result: 
  - Chunk: "Paris is the capital of France" (similarity: 0.98)
  - Other: "France is in Europe" (similarity: 0.45)
```

**Step 3: Augment**
```
Retrieved context: "Paris is the capital of France"
User query: "What's France's capital?"
↓
Combine into prompt:
"Based on this context: Paris is the capital of France
Answer: What's France's capital?"
```

**Step 4: Generate**
```
LLM receives: Context + Query
↓
LLM generates: "Paris is the capital of France"
↓
Return to user
```

**Full pipeline:**
```
Query → Embed → Search → Retrieve → Augment → Generate → Answer
```

**Typical metrics:**
- Embedding: 50ms
- Search: 10ms
- Generation: 500-2000ms
- **Total: 0.6-2.1 seconds**

---

## Q11: What's the difference between retrieval and ranking?

**Answer:**

**Retrieval:** Finding *candidate* documents
- Search vector DB for similar items
- Get top-1000 candidates quickly
- Speed is critical
- Quality can be approximate

**Ranking:** Ordering them by *relevance*
- Take candidates from retrieval
- Precisely score each one
- Sort by score
- Return top-10 to user

**Example:**

```
Query: "Best pizza places in NYC"

RETRIEVAL (Fast):
Vector DB returns 1000 documents containing "pizza"
- Restaurant 1
- Restaurant 2
- ... (more pizza places)
- Time: 50ms

RANKING (Precise):
Re-score top 1000:
- Restaurant 1: Score 0.95 ← Best match
- Restaurant 3: Score 0.92
- Restaurant 15: Score 0.88
- ... (sorted by score)
- Time: 200ms

FINAL RESULT (Top-10):
Return only top 10 ranked restaurants
```

**Methods:**

**Retrieval Methods:**
- BM25 (keyword)
- Semantic search (embeddings)
- Hybrid (both)

**Ranking Methods:**
- Cross-encoders (most accurate)
- LLM-based ranking
- Learning-to-rank
- Simple heuristics

**Tradeoff:**
- More ranking = better quality but slower
- Less ranking = faster but lower quality
- Typical: Retrieve 1000, rank top 100, return 10

---

## Q12: What are the main challenges in RAG?

**Answer:**

**1. Retrieval Quality**
- Challenge: Wrong documents retrieved
- Impact: LLM can't answer even with context
- Solution: Better embedding models, hybrid search, reranking

**2. Hallucination**
- Challenge: LLM makes up info not in context
- Impact: False answers despite having right docs
- Solution: Better prompts, grounding checks, fact verification

**3. Context Window Limits**
- Challenge: Too much context doesn't fit in prompt
- Impact: Can't use all relevant information
- Challenge: LLM has max token limit (e.g., 128K)
- Solution: Summarization, selective context, newer models with larger windows

**4. Latency**
- Challenge: Retrieval + generation takes time
- Impact: Slow user experience
- Typical: 1-3 seconds per query
- Solution: Caching, parallel processing, optimization

**5. Cost**
- Challenge: Multiple API calls (embedding, LLM, retrieval)
- Impact: Expensive at scale
- Solution: Cheaper models, caching, batching

**6. Knowledge Freshness**
- Challenge: Documents become outdated
- Impact: Wrong information
- Solution: Regular reindexing, document versioning

**7. Domain Mismatch**
- Challenge: General models vs specific domain
- Impact: Poor quality for specialized topics
- Solution: Domain-specific embeddings, fine-tuning

**8. Multi-lingual**
- Challenge: Model might not understand all languages
- Impact: Poor retrieval for non-English
- Solution: Multilingual models, language detection

---

## Q13: What does "grounding" mean in RAG?

**Answer:**
Grounding means **basing generated answers on retrieved documents**, not on the LLM's internal knowledge.

**Without Grounding:**
```
User: "What's the CEO of MyCorp as of May 2026?"
LLM: "I don't have current information, my knowledge ends at April 2024"
Problem: LLM relies on training data, not fresh info ❌
```

**With Grounding:**
```
User: "What's the CEO of MyCorp as of May 2026?"
Retrieve: "MyCorp announces new CEO John Smith on May 10, 2026"
LLM: "According to recent news, John Smith is the CEO of MyCorp"
Success: LLM grounded in retrieved document ✅
```

**Why grounding matters:**
- Prevents hallucinations
- Increases trustworthiness
- Can cite sources
- Allows fact verification

**How to enforce grounding:**

1. **Explicit instruction in prompt:**
```
"Answer only based on provided context.
If context doesn't contain answer, say so."
```

2. **Remove LLM's prior knowledge:**
- Smaller, less trained models
- Specific instruction: "Only use context"

3. **Fact verification:**
- Check generated answer against context
- Highlight which sentence supports answer

4. **Citation:**
- Return which document/chunk was used
- User can verify

---

## Q14: How do you measure RAG quality?

**Answer:**
RAG quality has multiple dimensions:

**1. Retrieval Quality**
- **Precision**: % of retrieved docs that are relevant
- **Recall**: % of relevant docs that are retrieved
- **NDCG** (Normalized Discounted Cumulative Gain): Quality of ranking
- **MRR** (Mean Reciprocal Rank): Position of first relevant result

**Example:**
```
Query: "How to make pasta?"
Retrieved 5 documents:
1. Pasta recipe ✅ (relevant)
2. Sauce recipe ✅ (somewhat relevant)
3. Pizza recipe ❌ (not relevant)
4. Cooking tips ✅ (relevant)
5. Dessert recipe ❌ (not relevant)

Precision: 3/5 = 60%
Recall: 3/4 = 75% (if 4 relevant docs total)
```

**2. Generation Quality**
- **BLEU**: Similarity to reference answer
- **ROUGE**: Overlapping n-grams
- **METEOR**: Considers synonyms
- **Human evaluation**: Is answer good?

**3. Hallucination**
- % of answers with hallucinations
- Fact-checking score
- Faithfulness to retrieved context

**4. Latency**
- Time from query to response
- Should be <2 seconds typically

**5. Cost**
- Cost per query
- Cost per 1000 queries

**6. User Satisfaction**
- Simple rating (1-5 stars)
- Would you use this?
- Thumbs up/down

**Best approach:**
Use multiple metrics:
```python
metrics = {
    "retrieval_precision": 0.85,
    "retrieval_recall": 0.78,
    "hallucination_rate": 0.05,
    "average_latency": 1.2,  # seconds
    "cost_per_query": 0.02,  # dollars
    "user_satisfaction": 4.2  # out of 5
}
```

---

## Q15: What's a simple RAG system architecture?

**Answer:**

**Minimal RAG (What you need):**

```
┌─────────────────┐
│  Documents      │  (Company docs, PDFs, web pages)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Chunk/Embed    │  (Break docs into chunks, convert to vectors)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Vector Database │  (Store embeddings for fast retrieval)
│ (Pinecone,      │
│  Weaviate, etc) │
└────────┬────────┘
         │
    ┌────┴────┐
    │ Retrieval Process (Online)
    │
    ▼
┌─────────────────┐
│  User Query     │  (Question from user)
│                 │
│ "How to get     │
│  started?"      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Embed Query    │  (Convert query to vector)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Find Similar   │  (Search vector DB)
│  Documents      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Retrieved Docs  │  (Top-K results)
│                 │
│ "Start with...  │
│  Basic concepts │
│  first..."      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Build Prompt   │  (Combine context + query)
│                 │
│ "Context: ...   │
│  Q: How to get  │
│  started?"      │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  LLM (GPT-4,    │  (Generate answer)
│   Claude, etc)  │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Generated      │  (Final answer)
│  Answer         │
│                 │
│ "To get started,│
│  you should..."|
└─────────────────┘
```

**Simple Python-like pseudocode:**

```python
# 1. Preparation (offline, once)
docs = load_documents()  # Your documents
chunks = split_into_chunks(docs, size=500)
embeddings = embed_all(chunks)  # Convert to vectors
vector_db.store(chunks, embeddings)  # Save

# 2. Query (online, per request)
query = "How to get started?"
query_embedding = embed(query)  # Convert query to vector

# 3. Retrieval
relevant_chunks = vector_db.search(
    query_embedding, 
    top_k=5  # Get top 5 similar
)

# 4. Augmentation
context = "\n".join(relevant_chunks)
prompt = f"Context: {context}\nQuestion: {query}"

# 5. Generation
answer = llm.generate(prompt)

# Return to user
return answer
```

**Components:**
- **Embedding Model**: Convert text to vectors
- **Vector Database**: Store and search
- **LLM**: Generate answers
- **Orchestration**: Combine everything (LangChain)

**Typical tools:**
- **Vector DB**: Pinecone (easy) or Weaviate (self-hosted)
- **Embedding**: OpenAI or Sentence Transformers
- **LLM**: GPT-4 or Claude
- **Orchestration**: LangChain
- **Time**: 2-4 hours to build POC

---

## 🎓 Next Steps

After mastering these 15 beginner questions:

1. **Review**: Go through each answer again
2. **Experiment**: Build simple RAG with code
3. **Practice**: Explain to someone else
4. **Move to**: Intermediate-QNA.md (18 questions)
5. **Build**: Simple RAG project

**Estimated completion**: 2-3 weeks for beginners

Good luck! 🚀

---

**Last Updated**: 2026-05-13
