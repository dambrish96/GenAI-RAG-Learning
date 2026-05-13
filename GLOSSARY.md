# Complete RAG Glossary (60+ Terms)

Quick reference for all RAG and AI terminology used throughout this learning path.

---

## A

**ANN (Approximate Nearest Neighbor)**
- Algorithm for finding similar vectors without comparing all pairs
- Enables fast retrieval in large vector spaces
- Examples: HNSW, IVF, LSH
- Used in: Vector database search

**Agentic RAG**
- RAG system with autonomous decision-making capabilities
- Agent decides which tools/databases to query
- Can perform multi-step reasoning
- More complex than standard RAG

**Augmentation**
- In RAG: Adding retrieved context to the prompt
- Enriching the input data for better generation
- Critical step between retrieval and generation

**API (Application Programming Interface)**
- Interface for software to communicate
- Examples: OpenAI API, Anthropic API
- Used to access: LLMs, embeddings, vector databases

---

## B

**BLEU (Bilingual Evaluation Understudy)**
- Metric for evaluating machine translation quality
- Also used for general generation quality
- Compares with reference translations
- Range: 0-1 (higher is better)

**BM25 (Best Match 25)**
- Ranking function for full-text search
- Keyword-based retrieval algorithm
- Sparse retrieval (not semantic)
- Fast and interpretable

**Batch Processing**
- Processing multiple queries/documents together
- Reduces per-item overhead
- More efficient than single processing
- Common in production systems

**Bi-Encoder**
- Model that encodes query and document independently
- Fast but less accurate for ranking
- Used for initial retrieval
- Example: Sentence Transformers

---

## C

**Chunking**
- Breaking documents into smaller pieces
- Critical preprocessing step in RAG
- Affects retrieval quality and cost
- Typical chunk size: 300-500 tokens

**Context Window**
- Maximum tokens an LLM can process
- Limits how much retrieved context fits
- Examples: 4K (GPT-3.5), 128K (GPT-4 Turbo), 200K (Claude 3)
- Important constraint in RAG

**Cross-Encoder**
- Model that scores relevance of query-document pairs together
- Used for reranking retrieved results
- More expensive than bi-encoders
- More accurate for ranking

**Cosine Similarity**
- Measures angle between two vectors
- Common metric in vector search
- Values: 0 to 1 (0=orthogonal, 1=identical)
- Fast to compute

**CRUD Operations**
- Create, Read, Update, Delete
- Basic database operations
- Important for maintaining vector index
- Standard in data management

---

## D

**Dense Retrieval**
- Semantic-based document retrieval
- Uses neural embeddings for similarity
- More accurate but slower than sparse
- Better for meaning-based queries

**DPR (Dense Passage Retrieval)**
- Technique for learning dense embeddings
- Important foundational paper (2020)
- Foundation for modern vector search
- Influential in RAG field

**Domain-Specific Knowledge**
- Knowledge specific to a particular field
- Examples: Medical, legal, financial, technical
- RAG enables LLMs to access current domain data
- Key advantage of RAG over fine-tuning

**Dimensionality**
- Number of components in a vector
- Typical embedding dimensions: 384-3072
- Higher dimensions = more information but more compute
- Trade-off between quality and speed

---

## E

**Embedding**
- Vector representation of text
- Fixed size (e.g., 768 dimensions)
- Enables semantic similarity measurement
- Core to RAG retrieval

**Embedding Model**
- Neural network that converts text to embeddings
- Examples: text-embedding-3-small, all-MiniLM-L6-v2
- Choice affects retrieval quality and cost
- Can be open-source or commercial API

**Ensemble Methods**
- Combining multiple models for better results
- Example: Hybrid search (BM25 + semantic)
- Improves robustness
- Common in production RAG

**Evaluation Metric**
- Quantitative measure of system performance
- Examples: precision, recall, NDCG, BLEU, ROUGE
- Critical for monitoring RAG quality
- Different metrics for different tasks

---

## F

**Faithfulness**
- Whether generated text is consistent with retrieved sources
- Evaluated by metrics and human review
- Key RAG quality measure
- Prevents hallucinations

**Few-Shot Learning**
- Learning from very few examples
- Examples provided in prompt
- Alternative to fine-tuning or RAG
- Limited by LLM training knowledge

**Fine-Tuning**
- Training a pre-trained model on specific data
- Updates model weights permanently
- Expensive and time-consuming
- Different approach from RAG

**Full-Text Search**
- Searching text using keywords
- Traditional database search method
- Less semantic than vector search
- Fast and exact for keywords

**Function Calling**
- LLM ability to call external functions/APIs
- Used in agentic RAG
- Enables multi-step workflows
- Modern LLMs support this

---

## G

**Generative Model**
- Model that generates new text
- Examples: GPT-4, Claude, Llama
- Used for generation step in RAG
- Contrast: Discriminative models classify

**Grounding**
- Basing generated text on retrieved documents
- Prevents hallucinations
- Common RAG requirement
- Verifiable against sources

**Graph Database**
- Database storing relationships between entities
- Used in graph-based RAG
- Examples: Neo4j, ArangoDB
- Good for structured knowledge

**GPU (Graphics Processing Unit)**
- Hardware for fast computation
- Needed for embeddings and LLM inference
- Cost factor in RAG systems
- Options: NVIDIA, AMD, TPU

---

## H

**Hallucination**
- When LLM generates false/fabricated information
- Major problem RAG addresses
- Reduced by grounding in retrieved docs
- Difficult to fully eliminate

**Hierarchical RAG**
- RAG with multiple levels of document organization
- Better for large document collections
- Improves retrieval efficiency
- More complex architecture

**Hybrid Search**
- Combining sparse and dense retrieval
- Uses both BM25 and embeddings
- Often better results than either alone
- Standard in production systems

**HNSW (Hierarchical Navigable Small World)**
- Algorithm for nearest neighbor search
- Used in vector databases (e.g., Qdrant, Weaviate)
- Fast and memory-efficient
- Good for large-scale systems

---

## I

**Indexing**
- Process of preparing documents for fast retrieval
- Includes embedding and storing in vector DB
- Done once, used for all queries
- Critical for performance

**IVF (Inverted File)**
- Quantization technique for vector search
- Divides vectors into clusters
- Fast but less accurate than exact search
- Good balance for large datasets

**In-Context Learning**
- Learning from examples in the prompt
- Without training or fine-tuning
- LLMs are good at this
- Different from few-shot learning

**Inference**
- Running a trained model on new data
- Opposite of training
- What happens in RAG generation step
- Usually refers to LLM generating response

---

## K

**Knowledge Cutoff**
- Date of LLM's training data
- After this date, model has no knowledge
- RAG solves this by providing current data
- Important limitation of LLMs

**Knowledge Graph**
- Structured representation of relationships
- Nodes: entities, Edges: relationships
- Can be used in graph-based RAG
- Enables complex reasoning

**K (Top-K)**
- Number of most similar items to retrieve
- Example: Retrieve top-5 documents
- Hyperparameter in RAG systems
- Affects accuracy and latency

---

## L

**Latency**
- Time taken to generate response
- Critical for real-time applications
- RAG adds retrieval latency overhead
- Typically 0.5-2 seconds total

**LLM (Large Language Model)**
- Neural network trained on large text corpus
- Examples: GPT-4, Claude 3, Llama 2
- Core component of RAG generation
- Billions to trillions of parameters

**LangChain**
- Framework for building LLM applications
- Popular for RAG implementation
- Provides chains, agents, memory components
- Very active development

**LSH (Locality-Sensitive Hashing)**
- Technique for fast similarity search
- Hashes similar items to same bucket
- Used in vector search
- Probabilistic approach

---

## M

**Metadata**
- Data about data
- Examples: document title, date, category, author
- Used for filtering in retrieval
- Improves search accuracy

**Metrics**
- Quantitative measures of performance
- Examples: precision, recall, NDCG, BLEU, ROUGE
- Essential for evaluating RAG systems
- Different metrics for different aspects

**Milvus**
- Open-source vector database
- High-performance option
- Good for large-scale deployments
- Supports distributed architecture

**Multi-Modal RAG**
- RAG with multiple data types (text, image, audio)
- Handles diverse content
- More complex than text-only RAG
- Emerging capability

**Multi-Hop Retrieval**
- Retrieving across multiple documents
- Reasoning through chain of documents
- Handles complex questions requiring synthesis
- More advanced RAG technique

---

## N

**NDCG (Normalized Discounted Cumulative Gain)**
- Ranking quality metric
- Considers position of relevant items
- Higher is better (0-1 scale)
- Good for evaluating retrieval

**Neural Ranking**
- Using neural networks for relevance ranking
- More sophisticated than BM25
- Examples: CrossEncoders, neural re-rankers
- Improves RAG quality

**Normalization (Vector)**
- Converting vector to unit length
- Makes cosine similarity = dot product
- Standard preprocessing for embeddings
- Improves computation efficiency

---

## O

**Overlap (Chunking)**
- In chunking: duplicate tokens between chunks
- Prevents losing context at boundaries
- Typical overlap: 10-20% of chunk size
- Balance between coverage and redundancy

**Open-Domain QA**
- Question answering without restricted domain
- Requires general knowledge
- Traditional application of RAG
- Challenging task

---

## P

**Pinecone**
- Managed vector database service
- Cloud-based, easy to use
- Popular for RAG projects
- Free tier available

**Precision**
- Fraction of retrieved items that are relevant
- Precision = Relevant items retrieved / Total items retrieved
- Important metric for retrieval quality
- Focus: Avoid false positives

**Prompt Engineering**
- Crafting better prompts for LLMs
- Improves quality without training
- Complements RAG
- Active research area

**Prompt Injection**
- Security attack inserting malicious prompts
- Retrieved documents can contain injections
- Important production concern
- Prevention: Validation and filtering

---

## Q

**Qdrant**
- Vector database (Rust-based)
- Focus on performance and simplicity
- Good alternative to Pinecone
- Self-hosted and managed options

**Quantization**
- Reducing precision of embeddings
- Saves storage and computation
- Slight accuracy loss
- Common optimization

**Query Expansion**
- Generating multiple versions of query
- Better retrieval coverage
- Example: "What is RAG?" → ["What is RAG?", "Define RAG", "Explain RAG"]
- Improves recall

**Query Understanding**
- Processing and interpreting user queries
- Extracting intent and entities
- Improves retrieval effectiveness
- Preprocessing step

---

## R

**Reranking**
- Re-ordering retrieved documents by relevance
- Uses more sophisticated models
- Improves retrieval quality at cost of latency
- Standard in production RAG

**Recall**
- Fraction of relevant items that are retrieved
- Recall = Relevant items retrieved / Total relevant items
- Important metric for retrieval quality
- Focus: Avoid false negatives

**Retrieval**
- Fetching relevant documents/chunks
- First step of RAG
- Determines quality of final answer
- Critical component

**RAG (Retrieval-Augmented Generation)**
- Combining retrieval with language model generation
- Main focus of this learning path
- Solves hallucinations and knowledge gaps
- Increasingly popular approach

**ROUGE (Recall-Oriented Understudy for Gisting Evaluation)**
- Metric for evaluating summarization
- Also used for generation quality
- Based on overlapping n-grams
- Good for abstractive tasks

**RRF (Reciprocal Rank Fusion)**
- Combining multiple ranking lists
- Used in hybrid search
- Weights by reciprocal rank position
- Elegant combining approach

---

## S

**Semantic Search**
- Finding documents by meaning, not keywords
- Uses embeddings
- Better for understanding intent
- More powerful than keyword search

**Semantic Chunking**
- Chunking based on meaning boundaries
- Preserves semantic coherence
- More complex than fixed-size chunking
- Better quality but slower

**Sentence Transformers**
- Library for creating sentence embeddings
- Built on HuggingFace transformers
- Popular open-source option
- Easy to use

**Similarity Metric**
- Function measuring similarity between vectors
- Examples: cosine, euclidean, dot product
- Critical choice in vector search
- Different metrics have different properties

**Sparse Retrieval**
- Keyword-based retrieval (BM25)
- Represents sparse vectors
- Fast, exact matches
- Traditional approach

**Streaming**
- Processing data continuously
- Opposite of batch processing
- Enables real-time indexing
- Useful for live data

---

## T

**Tokenization**
- Breaking text into tokens
- Tokens are input to LLMs
- Different tokenizers give different tokens
- First step in processing

**Top-K Retrieval**
- Retrieving top K most similar items
- K is hyperparameter
- Typical values: 3, 5, 10
- Critical retrieval parameter

**Transformer**
- Neural architecture with attention mechanism
- Foundation of modern LLMs
- Introduced in 2017
- Revolutionary architecture

**Truncation**
- Cutting text to fit context window
- Necessary when text too long
- Can lose important information
- Handled automatically by APIs

---

## U

**User Feedback**
- Human evaluation of system outputs
- Used to improve system
- Gold standard for evaluation
- Expensive but valuable

---

## V

**Vector**
- Array of numbers representing text
- Used for semantic similarity
- High-dimensional data
- Core to RAG retrieval

**Vector Database**
- Database optimized for storing/searching vectors
- Enables fast similarity search
- Core component of RAG system
- Examples: Pinecone, Weaviate, Milvus

**Vector Search**
- Searching by similarity of vectors
- Enables semantic search
- More intuitive than keyword search
- Foundation of modern RAG

**Vectorization**
- Process of converting text to vectors
- Done via embedding model
- Critical preprocessing in RAG
- One-time operation

---

## W

**Weaviate**
- Open-source vector database
- Graph-based approach
- Good for scalable systems
- Excellent documentation

**Weighting**
- Assigning importance to different factors
- Example: weight BM25 and dense scores differently
- Common in hybrid search
- Fine-tuning technique

---

## X

**X-as-a-Service (XaaS)**
- Providing service via cloud API
- Examples: Pinecone (VectorDB-aaS)
- Reduces operational burden
- Pay-per-use model

---

## Z

**Zero-Shot Learning**
- Performing task without training examples
- LLMs good at this
- Different from few-shot and fine-tuning
- LLMs excel at zero-shot

---

## 📊 Acronym Quick Reference

| Acronym | Full Form |
|---------|-----------|
| ANN | Approximate Nearest Neighbor |
| API | Application Programming Interface |
| BLEU | Bilingual Evaluation Understudy |
| BM25 | Best Match 25 |
| CRUD | Create, Read, Update, Delete |
| DPR | Dense Passage Retrieval |
| GPU | Graphics Processing Unit |
| HyDE | Hypothetical Document Embeddings |
| HNSW | Hierarchical Navigable Small World |
| IVF | Inverted File |
| LLM | Large Language Model |
| LSH | Locality-Sensitive Hashing |
| NDCG | Normalized Discounted Cumulative Gain |
| QA | Question Answering |
| RAG | Retrieval-Augmented Generation |
| RRF | Reciprocal Rank Fusion |
| ROUGE | Recall-Oriented Understudy for Gisting Evaluation |
| XaaS | X-as-a-Service |

---

## 🔗 Related Terms by Category

### Retrieval Methods
- Sparse Retrieval, Dense Retrieval, Hybrid Search
- BM25, DPR, Semantic Search, Vector Search

### Vector Operations
- Embedding, Vectorization, Dimensionality
- Cosine Similarity, Euclidean Distance, Dot Product

### Databases
- Vector Database, Graph Database, Full-Text Search
- Indexing, CRUD Operations, Metadata

### Evaluation
- Precision, Recall, NDCG, BLEU, ROUGE
- Metrics, Evaluation Metric, Faithfulness

### LLM Concepts
- LLM, Context Window, Hallucination, Grounding
- In-Context Learning, Few-Shot Learning, Zero-Shot Learning

### Architectures
- Standard RAG, Hierarchical RAG, Agentic RAG, Multi-Modal RAG
- Multi-Hop Retrieval, Function Calling

---

**Last Updated**: 2026-05-13
**Total Terms**: 60+
