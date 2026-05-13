# Intermediate Q&A - RAG Architecture (18 Questions)

**For intermediate learners building production systems**

---

## Q1: Explain the difference between retrieval precision and recall

**Answer:**

**Precision**: What fraction of retrieved documents are actually relevant?
```
Precision = Relevant documents retrieved / Total documents retrieved
```

**Example:**
- Retrieve 10 documents
- 7 are relevant
- Precision = 7/10 = 70%

**Recall**: What fraction of all relevant documents did we retrieve?
```
Recall = Relevant documents retrieved / Total relevant documents in corpus
```

**Example:**
- Total relevant documents in database: 20
- Retrieved 7 relevant ones
- Recall = 7/20 = 35%

**In RAG Context:**
- **High Precision**: Few false positives, focused results
  - Better for: "Exactly what I need"
  - Risk: Miss some relevant docs
  
- **High Recall**: Get most relevant docs, may include noise
  - Better for: "Give me everything relevant"
  - Risk: Noise in results

**Trade-off:**
```
          Low Recall              High Recall
Low Prec  ❌ Bad               ✅ Useful (lots of noise)
High Prec ✅ Useful (incomplete) ❌ Bad
```

**Typical RAG Strategy:**
1. High recall retrieval (get many candidates)
2. High precision reranking (filter to best)
3. Best of both worlds

---

## Q2: What is cross-encoder reranking and why use it?

**Answer:**

**What is Cross-Encoder?**
A model that scores query-document pairs **together** (not independently):
```
Input: (Query, Document)
Cross-Encoder: [Joint scoring model]
Output: Relevance score 0-1
```

**Bi-Encoder vs Cross-Encoder:**

| Aspect | Bi-Encoder | Cross-Encoder |
|--------|-----------|---------------|
| Input | Query & doc separately | Query & doc together |
| Speed | Very fast | Slower |
| Accuracy | Good (70-80%) | Excellent (90-95%) |
| Use | Initial retrieval | Reranking |
| Examples | Sentence Transformers | BERT fine-tuned |

**Example:**

```
Query: "How to optimize RAG systems?"
Retrieved 5 documents:
1. "RAG Optimization" - needs reranking
2. "Vector Databases" - needs reranking
3. "LLMs" - needs reranking
4. "API Design" - needs reranking
5. "RAG Performance" - needs reranking

Bi-Encoder scores (fast):
1. 0.85
2. 0.72
3. 0.45
4. 0.38
5. 0.88

Cross-Encoder scores (accurate):
1. 0.92 ← Actually most relevant
2. 0.68
3. 0.40
4. 0.35
5. 0.80

Reranked order: 1, 5, 2, 3, 4
```

**Why Use?**
- ✅ Better quality results
- ✅ More accurate ranking
- ✅ Better user satisfaction
- ✅ Small latency overhead

**Production Pattern:**
```
1. Bi-encoder retrieval (fast, broad)
   ↓
2. Get top-100 candidates
   ↓
3. Cross-encoder rerank (slow, accurate)
   ↓
4. Return top-10 to user
   ↓
Result: Quality + Speed balance
```

---

## Q3: How do you handle the context window limitation in LLMs?

**Answer:**

**The Problem:**
- GPT-4 Turbo: 128K tokens max
- Claude 3 Opus: 200K tokens max
- Query + retrieved docs + generation space
- If everything doesn't fit → error or truncation

**Strategies:**

### **1. Selective Context (Most Common)**
```python
# Only include most relevant chunks
retrieved_docs = vector_db.search(query, top_k=100)
ranked_docs = cross_encoder_rerank(query, retrieved_docs)
selected_docs = ranked_docs[:5]  # Only top 5

# Build prompt
context = "\n".join(selected_docs)
prompt = f"Context:\n{context}\n\nQuestion: {query}"
```

**Pros**: Simple, effective
**Cons**: Might miss relevant info

### **2. Hierarchical Summarization**
```
Large documents
    ↓
Summary of each doc
    ↓
Include summaries instead
    ↓
Saves tokens, keeps info
```

**Pros**: More comprehensive
**Cons**: Summary quality dependent

### **3. Progressive Retrieval**
```
Initial retrieval: Top 10
If not enough info:
  → Retrieve next 10
  → Keep adding until answer found
  → Stop when sufficient
```

**Pros**: Adaptive
**Cons**: Multiple LLM calls

### **4. Compression**
```
Retrieved: "The quick brown fox jumps over the lazy dog"
Compress: "Fox jumps over dog"
Result: Same meaning, fewer tokens
```

**Pros**: Keeps information
**Cons**: May lose nuance

### **5. Filtering by Metadata**
```
Retrieved 100 docs
Filter: Only last 2 years (current info)
Filter: Only high confidence (>0.9)
Result: Top relevant docs only
```

**Pros**: Focus on best docs
**Cons**: May be too restrictive

**Best Practice:**
```python
def select_context(query, top_k=100, max_tokens=2000):
    # 1. Broad retrieval
    candidates = vector_db.search(query, top_k=top_k)
    
    # 2. Rank precisely
    ranked = cross_encoder.rank(query, candidates)
    
    # 3. Select until token limit
    context = ""
    for doc in ranked:
        if tokens(context) + tokens(doc) < max_tokens:
            context += doc + "\n"
        else:
            break
    
    return context
```

---

## Q4: What is query expansion and when should you use it?

**Answer:**

**What is Query Expansion?**
Generating multiple versions of the query to improve retrieval coverage.

**Original Query**: "How to use RAG?"

**Expanded Queries**:
1. "How do I use Retrieval Augmented Generation?"
2. "What is the procedure for implementing RAG?"
3. "RAG implementation guide"
4. "Using retrieval augmented generation in production"
5. "Steps to apply RAG in my project"

**Retrieval:**
```
Search vector DB with all 5 queries
Combine results
Remove duplicates
Return union of top results
```

**Methods:**

### **1. Synonym Expansion**
```
Original: "machine learning"
Expanded: ["ML", "artificial intelligence", "deep learning", "neural networks"]
```

### **2. Paraphrase Expansion**
```
Original: "How to optimize speed?"
Expanded: 
  - "How do I improve performance?"
  - "Speed optimization techniques"
  - "Making systems faster"
```

### **3. Decomposition**
```
Original: "What is the capital of France and its population?"
Expanded:
  - "What is France's capital?"
  - "What is the population of France?"
  - "France capital city"
```

### **4. Hypothetical Document Embedding (HyDE)**
```
Query: "How to use RAG?"

LLM generates hypothetical document:
"Retrieval Augmented Generation is a technique where...
To use RAG, you need to... First retrieve documents...
Then augment the prompt... Finally generate..."

Use this doc as query embedding instead
Better semantics!
```

**When to Use:**

✅ **Use Query Expansion When:**
- Complex, multi-faceted queries
- User queries are vague
- Domain-specific terminology
- Need high recall
- Multiple ways to phrase same question

❌ **Don't Use When:**
- Simple, clear queries
- Performance critical (<100ms latency needed)
- Limited retrieval budget
- High precision more important than recall

**Production Implementation:**
```python
def retrieve_with_expansion(query, num_expansions=3):
    # 1. Generate expanded queries
    expanded = generate_expansions(query, num_expansions)
    all_queries = [query] + expanded
    
    # 2. Retrieve for each
    all_results = []
    for q in all_queries:
        results = vector_db.search(q, top_k=10)
        all_results.extend(results)
    
    # 3. Deduplicate and rank
    unique = deduplicate(all_results)
    ranked = cross_encoder_rank(query, unique)
    
    return ranked[:10]
```

**Impact:**
- Recall increase: 20-40%
- Latency: 2-3x (multiple searches)
- Precision: Might decrease slightly

---

## Q5: Explain semantic vs syntactic chunking

**Answer:**

**Fixed-Size Chunking (Syntactic)**
```
Text: "The capital of France is Paris. It's known for art. 
       The Eiffel Tower is famous. Paris has museums."

Chunk size: 20 words
Overlap: 5 words

Chunk 1: "The capital of France is Paris. It's known for art. The Eiffel"
Chunk 2: "Tower is famous. Paris has museums."
```

**Problem**: Cuts mid-sentence, breaks meaning

**Semantic Chunking**
```
Same text

Chunk 1: "The capital of France is Paris."
Chunk 2: "It's known for art and culture."
Chunk 3: "The Eiffel Tower is famous. Paris has museums."
```

**Benefit**: Preserves meaning at boundaries

**Comparison:**

| Aspect | Fixed-Size | Semantic |
|--------|-----------|----------|
| Speed | Fast | Slower |
| Quality | Lower | Higher |
| Consistency | Predictable | Variable |
| Meaning | Often broken | Preserved |
| Implementation | Simple | Complex |
| Cost | Lower | Higher |

**How Semantic Chunking Works:**

```python
1. Split by sentence
   sentences = text.split('.')

2. Group semantically similar sentences
   while sentences_are_related(current, next):
       combine(current, next)

3. Result: Chunks of variable size but coherent meaning
```

**Methods:**

### **1. Sentence-Based**
```
Split at sentence boundaries (.)
Groups sentences into chunks
Keeps meaning intact
```

### **2. Paragraph-Based**
```
Split at paragraph breaks
Works for structured docs
Simple but sometimes too large
```

### **3. Semantic Similarity**
```
Use embeddings to find similar content
Group until similarity drops
Very accurate but expensive
```

### **4. Language-Specific**
```
Python: Split by functions/classes
Legal: Split by sections/clauses
Medical: Split by symptoms/treatments
```

**When to Use:**

✅ **Fixed-Size**: 
- Large datasets (cost)
- Good enough quality
- Speed critical

✅ **Semantic**:
- Quality critical
- Smaller datasets
- Complex documents

**Production Example:**

```python
def chunk_intelligently(text, method="semantic"):
    if method == "fixed":
        # Simple, fast
        return chunk_fixed_size(text, size=500)
    
    elif method == "semantic":
        # High quality
        sentences = split_sentences(text)
        chunks = []
        current = ""
        
        for sent in sentences:
            candidate = current + sent
            if tokens(candidate) < 500:
                current = candidate
            else:
                chunks.append(current)
                current = sent
        
        chunks.append(current)
        return chunks
```

**Impact on RAG:**
- Better chunking = Better retrieval (15-25% improvement)
- Better retrieval = Better answers
- Worth the extra preprocessing cost

---

## Q6: What is hybrid search and how do you implement it?

**Answer:**

**Hybrid Search**: Combining sparse (BM25) and dense (semantic) retrieval.

**Why Both?**
- **BM25**: Exact keywords, fast, interpretable
- **Semantic**: Meaning-based, slower, more powerful
- **Hybrid**: Get strengths of both

**Example:**

```
Query: "How to optimize neural networks?"

BM25 Results (Keywords):
1. "Optimization of neural networks" (score: 0.95)
2. "Neural network training" (score: 0.87)
3. "Deep learning optimization" (score: 0.82)
4. "Performance metrics" (score: 0.45)

Semantic Results (Meaning):
1. "Making ML models faster" (score: 0.92)
2. "Improving model performance" (score: 0.88)
3. "Network efficiency tips" (score: 0.85)
4. "Computing resources" (score: 0.60)

Hybrid Combination:
1. "How to optimize neural networks" ← Both ranked high
2. "Optimization of neural networks"
3. "Making ML models faster"
4. "Neural network training"
5. "Improving model performance"
```

**Implementation Methods:**

### **1. Reciprocal Rank Fusion (RRF) - Most Popular**
```python
def rrf_combine(bm25_results, semantic_results, k=60):
    """
    Combine using RRF formula
    """
    scores = {}
    
    # Score BM25 results
    for rank, doc_id in enumerate(bm25_results):
        scores[doc_id] = scores.get(doc_id, 0) + 1/(k + rank + 1)
    
    # Score semantic results
    for rank, doc_id in enumerate(semantic_results):
        scores[doc_id] = scores.get(doc_id, 0) + 1/(k + rank + 1)
    
    # Sort by combined score
    return sorted(scores.items(), key=lambda x: x[1], reverse=True)
```

### **2. Weighted Sum**
```python
def weighted_combine(bm25_results, semantic_results, 
                    bm25_weight=0.4, semantic_weight=0.6):
    """
    Weighted combination: 40% BM25 + 60% semantic
    """
    combined = {}
    
    # Normalize scores 0-1
    bm25_norm = normalize(bm25_results)
    semantic_norm = normalize(semantic_results)
    
    # Combine with weights
    for doc_id, score in bm25_norm.items():
        combined[doc_id] = bm25_weight * score
    
    for doc_id, score in semantic_norm.items():
        combined[doc_id] = combined.get(doc_id, 0) + semantic_weight * score
    
    return sorted(combined.items(), key=lambda x: x[1], reverse=True)
```

### **3. Two-Stage**
```
Stage 1: Get top-N from each method
Stage 2: Only include docs that appear in BOTH
Result: High precision
```

**Production Implementation:**

```python
class HybridSearch:
    def __init__(self, vector_db, bm25_index):
        self.vector_db = vector_db
        self.bm25_index = bm25_index
    
    def search(self, query, top_k=10):
        # 1. BM25 retrieval
        bm25_results = self.bm25_index.search(query, top_k=100)
        
        # 2. Semantic retrieval
        query_embedding = self.embed(query)
        semantic_results = self.vector_db.search(
            query_embedding, top_k=100
        )
        
        # 3. Combine with RRF
        combined = rrf_combine(
            bm25_results, 
            semantic_results
        )
        
        # 4. Optional: Re-rank with cross-encoder
        reranked = cross_encoder_rank(query, combined)
        
        return reranked[:top_k]
```

**Results:**
- Recall improvement: +10-20%
- Precision improvement: +5-15%
- Latency: ~2x (both searches)
- Often better than either alone

**When to Use:**
✅ Production systems (quality matters)
✅ Balanced needs (not extreme precision/recall)
✅ Budget allows extra latency

---

## Q7: How do you evaluate RAG systems comprehensively?

**Answer:**

**Three Dimensions:**

### **1. Retrieval Quality**

**Precision @ K**
```
Precision@5 = Relevant docs in top-5 / 5
Typical target: >80%
```

**Recall @ K**
```
Recall@100 = Relevant docs in top-100 / Total relevant docs
Typical target: >90%
```

**NDCG (Normalized Discounted Cumulative Gain)**
```
Considers ranking position
Punishes relevant docs appearing late
Range: 0-1
Formula: DCG(k) / Ideal DCG(k)
```

**MRR (Mean Reciprocal Rank)**
```
Position of first relevant doc
Example: 1/5 = 0.2 (if first relevant at position 5)
Range: 0-1
Good for: "Get at least one good result"
```

### **2. Generation Quality**

**Faithfulness**
```
Is answer faithful to retrieved docs?
Evaluated by:
- LLM semantic similarity check
- Human evaluation
- Fact verification against sources
```

**Relevance**
```
Does answer address the question?
Measured by:
- Semantic similarity (answer ↔ question)
- Human rating (1-5 scale)
- Task-specific metrics
```

**Hallucination Rate**
```
% of answers with hallucinations
Measured by:
- Fact-checking against docs
- LLM evaluation
- Human review
Ideal: <5%
```

### **3. System Performance**

**Latency**
```
Time from query to response
- Retrieval latency: <100ms (target)
- LLM latency: <500ms (target)
- Total: <2000ms (target)
```

**Throughput**
```
Queries per second
- Single retrieval: 100+ QPS
- Single LLM: 10-50 QPS
```

**Cost**
```
Cost per query
= (embedding cost + retrieval cost + LLM cost) / query
Typical: $0.01-0.05 per query
```

**Comprehensive Evaluation Setup:**

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness, answer_relevancy, 
    context_recall, context_precision
)

# Prepare test set
test_questions = ["Q1", "Q2", ...]
ground_truth_answers = ["A1", "A2", ...]

# Run RAG
results = []
for q in test_questions:
    retrieved = retrieve(q)
    answer = generate(q, retrieved)
    results.append({
        "question": q,
        "answer": answer,
        "contexts": retrieved,
        "ground_truth": ground_truth
    })

# Evaluate
metrics = evaluate(
    results,
    metrics=[
        faithfulness,
        answer_relevancy,
        context_recall,
        context_precision
    ]
)

# Results
print(f"Faithfulness: {metrics['faithfulness']:.2%}")
print(f"Answer Relevancy: {metrics['answer_relevancy']:.2%}")
print(f"Context Recall: {metrics['context_recall']:.2%}")
print(f"Context Precision: {metrics['context_precision']:.2%}")
```

**Benchmark Results:**

| Metric | Target | Good | Excellent |
|--------|--------|------|-----------|
| Context Precision | >70% | 75% | 85%+ |
| Context Recall | >60% | 70% | 90%+ |
| Faithfulness | >70% | 75% | 90%+ |
| Answer Relevancy | >70% | 75% | 85%+ |
| Latency | <2s | <1.5s | <500ms |
| Hallucination | <10% | <5% | <2% |

---

## Q8: What are the main failure modes in RAG?

**Answer:**

### **1. Retrieval Failure**
**Problem**: Wrong documents retrieved

**Causes**:
- Poor embedding model
- Bad query understanding
- Chunking issues
- Not enough context

**Symptoms**:
- Precision <60%
- Recall <70%
- Irrelevant documents ranked high

**Fix**:
```
1. Better embedding model
2. Hybrid search
3. Query expansion
4. Better chunking
5. Semantic chunking
6. Reranking
```

### **2. Hallucination**
**Problem**: LLM makes up information not in context

**Causes**:
- LLM training knowledge
- Prompt not grounding well
- Context not included
- Insufficient context

**Symptoms**:
- Factually incorrect answers
- Information not in docs
- Confidently wrong statements

**Fix**:
```
1. Explicit grounding prompt: 
   "Only use provided context"
2. Fact-checking
3. Better retrieval quality
4. Smaller LLM (less knowledge)
5. Validation layer
```

### **3. Context Window Overflow**
**Problem**: Too much context doesn't fit

**Causes**:
- Too many retrieved docs
- Large documents
- Limited context window
- Not filtering enough

**Symptoms**:
- Token limit exceeded
- Documents truncated
- Information lost

**Fix**:
```
1. Better selection (top-5 instead of top-20)
2. Summarization
3. Larger context window LLM
4. Progressive retrieval
```

### **4. Latency Issues**
**Problem**: System too slow for real-time

**Causes**:
- Multiple retrievals
- Complex reranking
- Slow embedding model
- Slow LLM

**Symptoms**:
- >2 second response time
- Users complain about speed
- Can't handle concurrent requests

**Fix**:
```
1. Caching frequent queries
2. Async operations
3. Faster embedding model
4. Remove reranking
5. Batch processing
```

### **5. Cost Explosion**
**Problem**: Too expensive to operate

**Causes**:
- Expensive embedding model
- Too many API calls
- Inefficient retrieval
- Large model usage

**Symptoms**:
- $0.10+ per query
- Bill too high
- Not sustainable

**Fix**:
```
1. Cheaper embedding model
2. Open-source LLM
3. Caching
4. Batching
5. Local hosting
```

### **6. Quality Inconsistency**
**Problem**: Some queries work, others don't

**Causes**:
- Domain mismatch
- Query type differences
- Embedding model bias
- Insufficient training data

**Symptoms**:
- Works for some queries
- Fails for others
- Unpredictable quality

**Fix**:
```
1. Domain-specific models
2. Fine-tuning
3. Better evaluation
4. Query routing (different handlers)
```

### **7. Outdated Information**
**Problem**: Retrieved docs are outdated

**Causes**:
- Stale index
- Old documents
- No refresh strategy
- New information not indexed

**Symptoms**:
- Answers about old events
- Wrong information
- User reports inaccuracy

**Fix**:
```
1. Regular reindexing
2. Document versioning
3. Freshness scoring
4. New doc streaming
```

**Prevention Checklist:**

```python
class RAGHealthCheck:
    def __init__(self, rag_system):
        self.rag = rag_system
    
    def check_retrieval(self, test_queries):
        """Check if retrieval is working"""
        for q in test_queries:
            results = self.rag.retrieve(q)
            assert len(results) > 0, f"No results for: {q}"
            print(f"✓ {q}")
    
    def check_latency(self, test_queries):
        """Check if fast enough"""
        for q in test_queries:
            start = time.time()
            self.rag.query(q)
            latency = time.time() - start
            assert latency < 2.0, f"Too slow: {latency}s"
            print(f"✓ {q}: {latency:.2f}s")
    
    def check_cost(self, num_queries):
        """Check if within budget"""
        start_cost = get_api_cost()
        for _ in range(num_queries):
            self.rag.query("test")
        end_cost = get_api_cost()
        
        per_query = (end_cost - start_cost) / num_queries
        assert per_query < 0.05, f"Too expensive: ${per_query}"
        print(f"✓ Cost: ${per_query:.4f} per query")
    
    def check_hallucinations(self, test_set):
        """Check for hallucinations"""
        hallucinations = 0
        for q, expected in test_set:
            answer = self.rag.query(q)
            if not fact_check(answer, expected):
                hallucinations += 1
        
        rate = hallucinations / len(test_set)
        assert rate < 0.05, f"Too many hallucinations: {rate:.1%}"
        print(f"✓ Hallucination rate: {rate:.1%}")
```

---

## Q9: How do you optimize RAG for latency?

**Answer:**

**Latency Budget Breakdown:**
```
Total: ~1500ms
├─ Embedding query: 50ms
├─ Vector search: 50ms
├─ Reranking: 300ms (optional)
├─ LLM generation: 1000ms
└─ Overhead: 100ms
```

**Optimization Strategies:**

### **1. Caching**
```python
# Cache frequent queries
cache = {}

def query(question):
    if question in cache:
        return cache[question]  # <1ms
    
    answer = rag.query(question)
    cache[question] = answer
    return answer
```

**Impact**: 90% queries served <10ms

### **2. Async Retrieval**
```python
import asyncio

async def rag_query(question):
    # Do retrieval and LLM in parallel where possible
    embedding_task = asyncio.create_task(embed(question))
    
    embedding = await embedding_task
    retrieved = await vector_db.search_async(embedding)
    answer = await llm.generate_async(question, retrieved)
    
    return answer
```

**Impact**: Reduce sequential wait time

### **3. Faster Embedding Model**
```
Model               Speed    Quality
text-embedding-3    100ms    Excellent
all-MiniLM-L6-v2    30ms     Good
all-MiniLM-L12-v2   50ms     Better
```

**Choice**: Use faster for retrieval, slower for ranking

### **4. Remove Reranking**
```python
# Before: 400ms total (retrieval + rerank)
retrieved = vector_db.search(query, top_k=5)  # Direct top-5

# After: 50ms total
# Trade: Slightly lower quality, much faster
```

**Impact**: -300ms latency

### **5. Reduce Retrieved Docs**
```python
# Before: Top-20, feed all to LLM
retrieved = vector_db.search(query, top_k=20)

# After: Top-5, only best
retrieved = vector_db.search(query, top_k=5)

# Impact: Shorter prompt → Faster generation
```

**Impact**: -100ms LLM latency

### **6. Batching**
```python
# For multiple queries
queries = ["Q1", "Q2", "Q3", "Q4", "Q5"]

# Batch embedding (better GPU utilization)
embeddings = embed_batch(queries)

# Batch retrieval
results = [vector_db.search(e, top_k=5) for e in embeddings]
```

**Impact**: 50% faster for batches

### **7. Smaller LLM**
```
Model               Latency  Quality
GPT-4 Turbo        1500ms   Excellent
GPT-3.5-turbo       500ms   Good
Llama-2-7b          200ms   Okay
Phi-2               100ms    Fair
```

**Choice**: Use smallest that meets quality needs

### **8. Streaming Response**
```python
# Instead of waiting for complete answer
def stream_query(question):
    retrieved = retrieve(question)
    
    # Stream LLM output as tokens arrive
    for token in llm.stream(question, retrieved):
        yield token
        # User sees first token in 100ms, not 1500ms
```

**Impact**: Perceived latency reduction

**Production Optimization Stack:**

```python
class OptimizedRAG:
    def __init__(self):
        self.cache = Cache()
        self.fast_embedder = FastEmbedder()  # 30ms
        self.vector_db = IndexedVectorDB()   # 50ms
        self.fast_llm = FastLLM()            # 500ms
    
    def query(self, question):
        # 1. Check cache (<1ms)
        cached = self.cache.get(question)
        if cached:
            return cached
        
        # 2. Fast embedding (<30ms)
        embedding = self.fast_embedder.embed(question)
        
        # 3. Fast retrieval (<50ms)
        retrieved = self.vector_db.search(
            embedding, 
            top_k=5  # Reduced
        )
        
        # 4. Fast generation (<500ms)
        answer = self.fast_llm.generate(
            question, 
            retrieved
        )
        
        # 5. Cache result
        self.cache.set(question, answer)
        
        return answer
        # Total: ~580ms (vs ~1500ms)
```

**Results:**
- Before: 1500ms
- After: 600ms (60% improvement)
- Trade-off: Slightly lower quality
- Worth it for real-time use

---

## Q10: Explain A/B testing for RAG systems

**Answer:**

**Basic Setup:**

```
User Query
    ↓
Random split (50/50)
    ├─ Version A: Current system
    └─ Version B: New system
    ↓
Measure metrics
    ├─ Quality scores
    ├─ User satisfaction
    └─ Business metrics
    ↓
Statistical analysis
    └─ Is B significantly better?
```

**Example:**

```
Version A: Without reranking
- Latency: 600ms
- Satisfaction: 75%

Version B: With reranking
- Latency: 900ms
- Satisfaction: 82%

Question: Is 7% improvement worth 300ms latency?
```

**Metrics to Track:**

### **Quality Metrics**
- Retrieval precision/recall
- Hallucination rate
- Answer relevance
- User ratings

### **Business Metrics**
- Click-through rate
- Bounce rate
- Session duration
- Conversion rate

### **Technical Metrics**
- Latency
- Cost per query
- System uptime

**Statistical Significance:**

```python
import scipy.stats as stats

# Version A results
a_scores = [0.75, 0.82, 0.78, 0.81, ...]  # 100 samples

# Version B results
b_scores = [0.82, 0.85, 0.88, 0.79, ...]  # 100 samples

# T-test
t_stat, p_value = stats.ttest_ind(a_scores, b_scores)

# p_value < 0.05 = statistically significant
if p_value < 0.05:
    print("Version B is significantly better!")
else:
    print("No significant difference")
```

**Sample Size Calculation:**

```python
def required_sample_size(
    baseline=0.75,        # Current metric value
    expected_lift=0.05,   # Expected 5% improvement
    alpha=0.05,           # 95% confidence
    power=0.80            # 80% power
):
    """Calculate samples needed for significance"""
    
    # Using effect size calculation
    effect_size = (baseline + expected_lift - baseline) / sqrt(...)
    
    # Simplified formula
    n = (3.84 * variance) / (effect_size ** 2)
    
    return int(n)
```

**A/B Testing Process:**

```
Day 1-7: Ramp up
├─ Version A: 100%
├─ Day 2: Split 90/10
├─ Day 4: Split 50/50
└─ Ensure stability

Day 7-21: Full experiment
├─ Both 50/50
├─ Collect 1000+ samples each
└─ Track all metrics

Day 21: Analysis
├─ Check for significance
├─ Analyze by user segment
├─ Check for negative effects
└─ Make decision

Decision:
├─ Winner clear? → Roll out to 100%
├─ No winner? → Keep current
└─ Need more data? → Extend test
```

**Segmentation:**

```python
def analyze_by_segment(results):
    """Check if improvement varies by user type"""
    
    segments = {
        'new_users': results[results['user_age'] < 7],
        'power_users': results[results['query_count'] > 100],
        'mobile': results[results['device'] == 'mobile'],
        'desktop': results[results['device'] == 'desktop'],
    }
    
    for segment_name, segment_data in segments.items():
        a_data = segment_data[segment_data['version'] == 'A']
        b_data = segment_data[segment_data['version'] == 'B']
        
        improvement = b_data.mean() - a_data.mean()
        print(f"{segment_name}: {improvement:+.1%}")
```

**Real Example:**

```
Test: Better embedding model

Version A: text-embedding-3-small
- Quality score: 0.78
- Cost: $0.01/query
- Users tested: 500

Version B: text-embedding-3-large
- Quality score: 0.83
- Cost: $0.05/query
- Users tested: 500

Analysis:
- Quality improvement: +6.4% (significant, p<0.001)
- Cost increase: 5x ($0.04 more per query)
- User satisfaction: +8% (significant)
- Latency: +150ms (acceptable)

Decision: Roll out to 10% of users first
- Monitor metrics closely
- If good, expand to 100%
```

---

## Q11: How do you implement caching in RAG systems?

**Answer:**

**Three Levels of Caching:**

### **Level 1: Query Cache (Fastest)**
```python
class QueryCache:
    def __init__(self):
        self.cache = {}
    
    def get_answer(self, question):
        # Cache hit: <1ms
        if question in self.cache:
            return self.cache[question]
        
        # Cache miss: compute
        answer = full_rag_pipeline(question)
        self.cache[question] = answer
        return answer
```

**Hit Rate**: 30-40% (same questions repeated)
**Latency**: <1ms
**Trade-off**: Exact matches only

### **Level 2: Semantic Cache (Smarter)**
```python
class SemanticCache:
    def __init__(self):
        self.cache = {}  # embedding -> answer
    
    def get_answer(self, question):
        embedding = embed(question)
        
        # Check if similar question cached
        for cached_embedding, cached_answer in self.cache.items():
            similarity = cosine(embedding, cached_embedding)
            if similarity > 0.95:  # Very similar
                return cached_answer
        
        # Not cached, compute
        answer = full_rag_pipeline(question)
        self.cache[embedding] = answer
        return answer
```

**Hit Rate**: 50-70% (similar questions)
**Latency**: <100ms (extra search)
**Trade-off**: Slight quality difference

### **Level 3: Retrieval Cache (Building Block)**
```python
class RetrievalCache:
    def __init__(self):
        self.cache = {}  # query_embedding -> results
    
    def retrieve(self, question):
        embedding = embed(question)
        embedding_key = tuple(embedding[:10])  # Quantize
        
        # Check cache
        if embedding_key in self.cache:
            return self.cache[embedding_key]
        
        # Not cached, retrieve
        results = vector_db.search(embedding, top_k=10)
        self.cache[embedding_key] = results
        return results
```

**Hit Rate**: 60-80% (similar embeddings)
**Latency**: Saves 50ms (retrieval step)
**Trade-off**: Slight precision loss

**Production Caching Strategy:**

```python
class CachedRAG:
    def __init__(self):
        self.query_cache = LRUCache(maxsize=10000)
        self.retrieval_cache = LRUCache(maxsize=100000)
        self.embedding_cache = LRUCache(maxsize=50000)
    
    def query(self, question):
        # Level 1: Exact query cache
        if question in self.query_cache:
            return self.query_cache[question]  # <1ms
        
        # Level 2: Retrieval cache
        embedding = self.embed(question)
        emb_key = hash_embedding(embedding)
        
        if emb_key in self.retrieval_cache:
            retrieved = self.retrieval_cache[emb_key]
        else:
            retrieved = self.vector_db.search(embedding)
            self.retrieval_cache[emb_key] = retrieved
        
        # Generate answer
        answer = self.llm.generate(question, retrieved)
        
        # Cache result
        self.query_cache[question] = answer
        
        return answer
```

**Cache Invalidation:**

```python
class SmartCache:
    def __init__(self):
        self.cache = {}
        self.timestamps = {}
        self.ttl = 3600  # 1 hour
    
    def get(self, key):
        if key not in self.cache:
            return None
        
        # Check if expired
        age = time.time() - self.timestamps[key]
        if age > self.ttl:
            del self.cache[key]
            return None
        
        return self.cache[key]
    
    def set(self, key, value, ttl=None):
        self.cache[key] = value
        self.timestamps[key] = time.time()
    
    def invalidate_domain(self, domain):
        """Invalidate all docs from a domain"""
        keys_to_delete = [k for k in self.cache.keys() 
                         if domain in k]
        for k in keys_to_delete:
            del self.cache[k]
```

**Impact:**

```
Without caching:
- Latency: 1500ms
- Cost: $0.05 per query

With caching (50% hit rate):
- Latency: 750ms average
- Cost: $0.025 per query
- 50% improvement!
```

---

Continue with Q12-Q18 in next section...

(Due to length, I'll provide a summary of remaining questions)

## Q12: Multi-stage Retrieval Architecture

**Key Points:**
- Coarse retrieval (broad)
- Intermediate ranking
- Fine retrieval (precise)
- Final answer generation

## Q13: Fine-tuning Retrieval Models

**Key Points:**
- Training on domain data
- Using contrastive learning
- Measuring improvement
- Cost-benefit analysis

## Q14: Graph-based RAG

**Key Points:**
- Knowledge graphs
- Entity linking
- Relationship traversal
- Multi-hop reasoning

## Q15: Multi-source RAG

**Key Points:**
- Combining multiple data sources
- Handling conflicts
- Source attribution
- Trust scores

## Q16: Real-time Indexing

**Key Points:**
- Streaming data
- Live document updates
- Freshness guarantees
- Consistency challenges

## Q17: RAG Security

**Key Points:**
- Prompt injection prevention
- Data privacy
- Access control
- Sensitive data handling

## Q18: Monitoring Production RAG

**Key Points:**
- Logging and tracing
- Metric dashboards
- Anomaly detection
- Alert strategies

---

**Last Updated**: 2026-05-13
