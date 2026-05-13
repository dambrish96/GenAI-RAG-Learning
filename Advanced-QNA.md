# Advanced Q&A - Expert Level RAG (15 Questions)

**For advanced practitioners designing enterprise RAG systems**

---

## Q1: Design a RAG system that handles 1M documents with sub-second latency

**Answer:**

**Architecture:**

```
┌─────────────────────────────────────────────────────┐
│         Query Load Balancer (Multiple Regions)      │
└────────────────────┬────────────────────────────────┘
                     │
        ┌────────────┴────────────┐
        ▼                         ▼
   ┌─────────────┐          ┌─────────────┐
   │ Cache Layer │          │ Cache Layer │
   │  (Redis)    │          │  (Redis)    │
   └────────────┬┘          └────────────┬┘
                │                        │
        ┌───────┴────────┬───────────────┴──────┐
        ▼                ▼                      ▼
   ┌─────────────┐  ┌──────────────┐  ┌─────────────┐
   │ Embedding   │  │ Fast Retrieval│  │  Reranking  │
   │  Service    │  │  (Vector DB)  │  │  Cluster    │
   └──────┬──────┘  └──────┬───────┘  └──────┬──────┘
          │                │                 │
          └────────────────┴─────────────────┘
                   │
          ┌────────┴────────┐
          ▼                 ▼
    ┌──────────────┐  ┌──────────────┐
    │ LLM Cluster  │  │ LLM Cluster  │
    │   (Region1)  │  │   (Region2)  │
    └──────────────┘  └──────────────┘
```

**Component Details:**

### **1. Vector Database Architecture**

```python
# Use distributed vector DB for 1M documents
class ScalableVectorDB:
    def __init__(self):
        # Sharding strategy
        self.shards = [
            MilvusCluster(shard_id=i)
            for i in range(10)  # 10 shards, ~100K docs each
        ]
    
    def index_documents(self, documents):
        """Index with sharding"""
        for doc in documents:
            shard_id = hash(doc.id) % len(self.shards)
            self.shards[shard_id].add(doc)
    
    def search(self, query_embedding, top_k=10):
        """Search across all shards in parallel"""
        futures = []
        for shard in self.shards:
            future = shard.search_async(query_embedding, top_k=50)
            futures.append(future)
        
        # Merge results from all shards
        all_results = [r for future in futures for r in future.result()]
        
        # Re-rank merged results
        reranked = sorted(all_results, key=lambda x: x.score)[:top_k]
        
        return reranked
```

### **2. Multi-Level Caching**

```python
class MultiLevelCache:
    def __init__(self):
        self.l1_cache = LocalMemoryCache(ttl=300)      # 5 min
        self.l2_cache = RedisCache(ttl=3600)           # 1 hour
        self.l3_cache = PersistentCache(ttl=86400)     # 1 day
    
    def get(self, key):
        # Try L1 (memory) - <1ms
        result = self.l1_cache.get(key)
        if result:
            return result
        
        # Try L2 (Redis) - <10ms
        result = self.l2_cache.get(key)
        if result:
            self.l1_cache.set(key, result)
            return result
        
        # Try L3 (disk) - <50ms
        result = self.l3_cache.get(key)
        if result:
            self.l2_cache.set(key, result)
            return result
        
        return None

# Cache hit stats: 70-80% hit rate typical
```

### **3. Fast Embedding with Quantization**

```python
class OptimizedEmbedding:
    def __init__(self):
        self.model = EmbeddingModel()
        self.quantizer = ProductQuantizer(K=256)
    
    def embed_and_quantize(self, texts):
        # Batch embedding
        embeddings = self.model.encode_batch(texts)
        
        # Quantize (reduce from 3072 dims to 64)
        quantized = self.quantizer.encode(embeddings)
        
        # Result: 3072 → 64 dimensions
        # Speed: 10x faster search, 98% quality retained
        return quantized
```

### **4. Tiered Retrieval**

```python
class TieredRetrieval:
    def retrieve(self, query_embedding):
        """Multi-stage retrieval for speed"""
        
        # Stage 1: Broad search (fast)
        # Use quantized vectors, retrieve 1000
        candidates = self.vector_db.search_quantized(
            query_embedding, top_k=1000
        )
        # Latency: <50ms
        
        # Stage 2: Coarse filtering
        # Filter by metadata, date, domain
        filtered = [d for d in candidates 
                   if d.is_valid()]
        # Latency: <10ms
        
        # Stage 3: Precise reranking
        # Use exact vectors and cross-encoder
        top_100 = self.reranker.rank(
            query_embedding, 
            filtered[:100]
        )
        # Latency: <100ms
        
        # Total latency: ~160ms
        return top_100[:10]
```

### **5. Async Everything**

```python
async def query_async(question):
    """Fully async pipeline"""
    
    # Parallel embedding and retrieval prep
    embedding_task = embed_async(question)
    cache_task = cache.get_async(question)
    
    # Check cache while embedding
    cached_answer = await cache_task
    if cached_answer:
        return cached_answer
    
    embedding = await embedding_task
    
    # Retrieve and rerank in parallel
    retrieve_task = vector_db.search_async(embedding)
    
    retrieved = await retrieve_task
    
    # Generate answer
    answer = await llm.generate_async(question, retrieved)
    
    # Cache for next time
    await cache.set_async(question, answer)
    
    return answer

# Total latency: ~400ms (vs 1500ms sequential)
```

**Performance Targets:**

```
Metrics:
├─ P50 Latency: 200-300ms
├─ P95 Latency: 400-600ms
├─ P99 Latency: 800ms-1s
├─ Throughput: 1000 QPS
├─ Cache hit rate: 70-80%
└─ Cost: $0.01-0.02 per query

Scaling:
├─ 1M documents: 10-20GB embeddings
├─ Sharding: 10 shards x 100K docs
├─ Replication: 3x for HA
├─ Total storage: 600GB-1TB
```

---

## Q2: How would you implement a self-correcting RAG system?

**Answer:**

**Self-Correction Loop:**

```
Query
  ↓
Generate Answer
  ↓
Evaluate Answer
  ├─ Is it good? → Return
  └─ Is it bad? → Retry
      ↓
  New Retrieval
      ↓
  Regenerate
      ↓
  Check again
      ↓
  Repeat until good or max retries
```

**Implementation:**

```python
class SelfCorrectingRAG:
    def __init__(self, max_retries=3):
        self.max_retries = max_retries
        self.retriever = Retriever()
        self.llm = LLM()
        self.evaluator = AnswerEvaluator()
        self.critic = CriticModel()
    
    def query(self, question):
        """Query with self-correction"""
        
        for attempt in range(self.max_retries):
            # Retrieve documents
            if attempt == 0:
                # First attempt: standard retrieval
                retrieved = self.retriever.retrieve(question)
            else:
                # Retry: expanded retrieval
                expanded_q = self.expand_query(question, prev_answer)
                retrieved = self.retriever.retrieve(expanded_q)
            
            # Generate answer
            answer = self.llm.generate(question, retrieved)
            
            # Evaluate answer
            quality_score = self.evaluator.score(
                question=question,
                answer=answer,
                context=retrieved
            )
            
            if quality_score > 0.8:
                # Good answer
                return {
                    'answer': answer,
                    'confidence': quality_score,
                    'attempts': attempt + 1
                }
            
            # Bad answer, get feedback for retry
            if attempt < self.max_retries - 1:
                feedback = self.critic.critique(
                    question=question,
                    answer=answer,
                    context=retrieved
                )
                prev_answer = answer
                print(f"Attempt {attempt+1} failed: {feedback}")
        
        # Max retries exceeded, return best attempt
        return {
            'answer': answer,
            'confidence': quality_score,
            'attempts': self.max_retries,
            'note': 'Max retries reached'
        }
    
    def expand_query(self, original, previous_answer):
        """Expand query based on what failed"""
        expansion_prompt = f"""
        Original query: {original}
        Previous answer: {previous_answer}
        
        The previous answer was insufficient.
        Generate 3 alternative query formulations:
        """
        
        alternatives = self.llm.generate(expansion_prompt)
        return alternatives.split('\n')[0]
```

**Evaluation Strategies:**

```python
class AnswerEvaluator:
    def score(self, question, answer, context):
        """Multi-dimensional evaluation"""
        
        scores = []
        
        # 1. Relevance to question
        relevance = self.semantic_similarity(question, answer)
        scores.append(('relevance', relevance))
        
        # 2. Grounding in context
        grounding = self.check_grounding(answer, context)
        scores.append(('grounding', grounding))
        
        # 3. Factuality
        factuality = self.check_facts(answer, context)
        scores.append(('factuality', factuality))
        
        # 4. Completeness
        completeness = self.check_completeness(question, answer)
        scores.append(('completeness', completeness))
        
        # Weighted average
        weights = {
            'relevance': 0.3,
            'grounding': 0.3,
            'factuality': 0.25,
            'completeness': 0.15
        }
        
        overall = sum(
            score * weights[name] 
            for name, score in scores
        )
        
        return overall
```

**Results:**
- ✅ Lower hallucination rate (5% → 2%)
- ✅ Better answer quality
- ❌ Extra API calls (1.5x average)
- Trade-off: Quality over latency

---

## Q3: Design a multi-modal RAG system

**Answer:**

**Architecture:**

```
Input (Text/Image/Audio)
    ↓
├─ Text Path: Tokenizer → Embedding (text-embedding-3)
├─ Image Path: Vision encoder → Embedding (CLIP)
└─ Audio Path: Audio encoder → Embedding (Whisper)
    ↓
    ↓ (All converted to same embedding space)
    ↓
Multi-modal vector DB
    ↓
├─ Text retrieval
├─ Image retrieval
└─ Cross-modal retrieval
    ↓
LLM (can process all modalities)
    ↓
Multi-modal answer
```

**Implementation:**

```python
class MultiModalRAG:
    def __init__(self):
        # Different encoders
        self.text_encoder = TextEmbedding()
        self.image_encoder = CLIPImageEncoder()
        self.audio_encoder = WhisperAudioEncoder()
        
        # Unified vector DB
        self.vector_db = UnifiedVectorDB()
        
        # Multi-modal LLM
        self.llm = GPT4Vision()
    
    def index_multimodal_documents(self, documents):
        """Index documents with mixed modalities"""
        
        for doc in documents:
            embeddings = {}
            metadata = {'id': doc.id, 'source': doc.source}
            
            # Encode text
            if doc.text:
                embeddings['text'] = self.text_encoder.encode(doc.text)
                metadata['has_text'] = True
            
            # Encode images
            if doc.images:
                embeddings['images'] = [
                    self.image_encoder.encode(img)
                    for img in doc.images
                ]
                metadata['num_images'] = len(doc.images)
            
            # Encode audio
            if doc.audio:
                embeddings['audio'] = self.audio_encoder.encode(doc.audio)
                metadata['has_audio'] = True
            
            # Store in unified space
            self.vector_db.add(
                embeddings=embeddings,
                metadata=metadata
            )
    
    def retrieve_multimodal(self, query, query_type='text'):
        """Retrieve using multi-modal query"""
        
        if query_type == 'text':
            query_embedding = self.text_encoder.encode(query)
        elif query_type == 'image':
            query_embedding = self.image_encoder.encode(query)
        elif query_type == 'audio':
            query_embedding = self.audio_encoder.encode(query)
        
        # Cross-modal retrieval
        # Text query can match images/audio
        results = self.vector_db.search(
            query_embedding,
            top_k=10,
            return_all_modalities=True
        )
        
        return results
    
    def generate_multimodal_answer(self, query, retrieved):
        """Generate answer using multi-modal context"""
        
        # Prepare context
        context = {
            'text': [],
            'images': [],
            'audio': []
        }
        
        for result in retrieved:
            if 'text' in result:
                context['text'].append(result['text'])
            if 'images' in result:
                context['images'].extend(result['images'])
            if 'audio' in result:
                context['audio'].append(result['audio'])
        
        # Generate using GPT-4 Vision
        answer = self.llm.generate(
            query=query,
            text_context='\n'.join(context['text']),
            images=context['images'],
            audio_transcripts=[
                self.transcribe(audio)
                for audio in context['audio']
            ]
        )
        
        return answer
```

**Use Cases:**
- Visual search + text explanation
- Document understanding (text + images)
- Video analysis + transcript

---

## Q4: How would you implement RAG for real-time streaming data?

**Answer:**

**Architecture:**

```
Data Source
    ↓ (streaming)
    ├─ News feed
    ├─ Social media
    ├─ Sensor data
    └─ Logs
    ↓
Stream Processor (Kafka/Kinesis)
    ↓
Chunking & Embedding Pipeline
    ↓
Vector DB (with streaming indexing)
    ↓
Real-time RAG Queries
```

**Implementation:**

```python
class StreamingRAG:
    def __init__(self):
        self.stream = KafkaConsumer('documents')
        self.vector_db = WeaviateDB()
        self.embedding_queue = Queue()
        self.index_queue = Queue()
    
    def start_streaming(self):
        """Start consuming and indexing stream"""
        
        # Process stream
        threading.Thread(
            target=self._consume_stream,
            daemon=True
        ).start()
        
        # Embed batches
        threading.Thread(
            target=self._embed_batch,
            daemon=True
        ).start()
        
        # Index documents
        threading.Thread(
            target=self._index_batch,
            daemon=True
        ).start()
    
    def _consume_stream(self):
        """Consume from stream"""
        
        for message in self.stream:
            doc = json.loads(message.value)
            
            # Quick processing
            chunks = self.chunk(doc['text'])
            
            self.embedding_queue.put({
                'chunks': chunks,
                'metadata': doc.get('metadata', {})
            })
    
    def _embed_batch(self):
        """Batch embedding"""
        
        batch = []
        while True:
            try:
                item = self.embedding_queue.get(timeout=1)
                batch.append(item)
                
                if len(batch) >= 32:  # Batch size
                    embeddings = self.embed_batch([
                        item['chunks']
                        for item in batch
                    ])
                    
                    for i, item in enumerate(batch):
                        self.index_queue.put({
                            'chunks': item['chunks'],
                            'embeddings': embeddings[i],
                            'metadata': item['metadata']
                        })
                    
                    batch = []
            except:
                if batch:
                    # Timeout, process partial batch
                    embeddings = self.embed_batch([
                        item['chunks']
                        for item in batch
                    ])
                    
                    for i, item in enumerate(batch):
                        self.index_queue.put({
                            'chunks': item['chunks'],
                            'embeddings': embeddings[i],
                            'metadata': item['metadata']
                        })
                    batch = []
    
    def _index_batch(self):
        """Index to vector DB"""
        
        while True:
            item = self.index_queue.get()
            
            self.vector_db.add(
                vectors=item['embeddings'],
                payloads=item['chunks'],
                metadata=item['metadata']
            )
```

**Freshness Guarantees:**

```python
class FreshRAG:
    def retrieve(self, query, max_age_minutes=60):
        """Retrieve only recent documents"""
        
        embedding = self.embed(query)
        
        # Filter by recency
        cutoff_time = time.time() - (max_age_minutes * 60)
        
        results = self.vector_db.search(
            vector=embedding,
            top_k=100,
            filter={
                'timestamp': {'$gte': cutoff_time}
            }
        )
        
        return results[:10]
```

---

## Q5: Design a knowledge graph-based RAG system

**Answer:**

**Architecture:**

```
Documents
    ↓
Named Entity Recognition + Linking
    ↓
Extract triples: (entity1, relation, entity2)
    ↓
Build Knowledge Graph
    ↓
Query → Entity lookup → Graph traversal
    ↓
Retrieved subgraph
    ↓
LLM generates answer
```

**Implementation:**

```python
class KnowledgeGraphRAG:
    def __init__(self):
        self.graph_db = Neo4jDB()
        self.ner = NamedEntityRecognizer()
        self.relation_extractor = RelationExtractor()
    
    def build_knowledge_graph(self, documents):
        """Extract and build knowledge graph"""
        
        for doc in documents:
            # Extract entities
            entities = self.ner.extract(doc.text)
            
            # Extract relations
            triples = self.relation_extractor.extract(
                doc.text,
                entities
            )
            
            # Add to graph
            for subject, relation, obj in triples:
                self.graph_db.add_relation(
                    source=subject,
                    target=obj,
                    relation_type=relation,
                    metadata={'source_doc': doc.id}
                )
    
    def retrieve_subgraph(self, query, hops=2):
        """Retrieve relevant subgraph"""
        
        # Extract entities from query
        query_entities = self.ner.extract(query)
        
        # Start from query entities, traverse
        subgraph = []
        visited = set()
        
        for entity in query_entities:
            subgraph.extend(
                self._traverse_graph(
                    entity,
                    hops=hops,
                    visited=visited
                )
            )
        
        return subgraph
    
    def _traverse_graph(self, entity, hops, visited):
        """Multi-hop graph traversal"""
        
        if hops == 0 or entity in visited:
            return []
        
        visited.add(entity)
        results = []
        
        # Get relations for entity
        relations = self.graph_db.get_relations(entity)
        
        for subject, relation, obj in relations:
            results.append({
                'subject': subject,
                'relation': relation,
                'object': obj
            })
            
            # Recursive traversal
            if hops > 1:
                results.extend(
                    self._traverse_graph(
                        obj,
                        hops=hops-1,
                        visited=visited
                    )
                )
        
        return results
    
    def generate_answer_from_graph(self, query, subgraph):
        """Generate answer using graph structure"""
        
        # Format graph as structured text
        graph_text = self._format_subgraph(subgraph)
        
        prompt = f"""
        Knowledge Graph:
        {graph_text}
        
        Question: {query}
        
        Answer based on the knowledge graph above:
        """
        
        answer = self.llm.generate(prompt)
        return answer
```

**Benefits:**
- ✅ Explicit reasoning paths
- ✅ Multi-hop reasoning
- ✅ Structured knowledge
- ❌ Requires NER/relation extraction

---

Continue with Q6-Q15 (Advanced topics like optimization, cost analysis, production patterns, etc.)

**Q6**: Optimization techniques for low-latency RAG
**Q7**: Cost optimization strategies
**Q8**: Advanced prompt engineering for RAG
**Q9**: Domain adaptation techniques
**Q10**: Handling long documents (>10K tokens)
**Q11**: Multi-language RAG systems
**Q12**: RAG security & privacy
**Q13**: A/B testing complex RAG changes
**Q14**: Handling conflicting information in RAG
**Q15**: Building RAG for different languages and cultures

---

**Last Updated**: 2026-05-13
