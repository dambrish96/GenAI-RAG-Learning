# Curated Resources - GenAI RAG Learning Path

Complete collection of tools, courses, papers, and communities for RAG mastery.

---

## 📚 Books & Comprehensive Guides

### Essential Reading

1. **"Building Generative AI Applications with Gradio"** - Hugging Face
   - Free online course
   - Hands-on with LLMs
   - URL: huggingface.co/learn

2. **"LLM.int8: 8-bit Matrix Multiplication"** - Research Paper
   - Quantization techniques
   - Making models efficient
   - URL: arxiv.org/abs/2110.02861

3. **"Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"**
   - Original RAG paper
   - Lewis et al., 2020
   - Foundation reading
   - URL: arxiv.org/abs/2005.11401

4. **"Dense Passage Retrieval for Open-Domain Question Answering"**
   - DPR paper
   - Karpukhin et al., 2020
   - Key retrieval techniques
   - URL: arxiv.org/abs/2004.04906

---

## 🎓 Free & Paid Courses

### Free Courses

1. **Hugging Face Course** (Free)
   - URL: huggingface.co/course
   - Topics: Transformers, NLP, embeddings
   - Time: 3-5 weeks
   - Quality: ⭐⭐⭐⭐⭐

2. **Stanford CS224N: NLP with Deep Learning** (Free)
   - URL: web.stanford.edu/class/cs224n
   - Lectures, assignments
   - Time: 10 weeks
   - Quality: ⭐⭐⭐⭐⭐

3. **Fast.ai - Practical Deep Learning** (Free)
   - URL: fast.ai
   - Video lectures
   - Time: 7 weeks
   - Quality: ⭐⭐⭐⭐⭐

4. **DeepLearning.AI Courses** (Free)
   - URL: deeplearning.ai
   - Topics: LLMs, RAG, Agents
   - Time: 2-3 hours each
   - Quality: ⭐⭐⭐⭐⭐

5. **Anthropic Claude 3 Prompt Engineering** (Free)
   - URL: github.com/anthropics/prompt-engineering
   - Best practices
   - Time: 1-2 hours
   - Quality: ⭐⭐⭐⭐⭐

### Paid Courses

1. **Andrew Ng's AI for Everyone** ($39)
   - URL: coursera.org
   - Beginner friendly
   - Time: 3-4 hours
   - Quality: ⭐⭐⭐⭐

2. **Udemy: Complete LLM & Generative AI Course** ($15-50)
   - URL: udemy.com (search LLM)
   - Hands-on projects
   - Time: 20-30 hours
   - Quality: ⭐⭐⭐⭐

3. **LinkedIn Learning: Generative AI** ($15/month)
   - URL: linkedin.com/learning
   - Professional content
   - Time: 2-3 weeks
   - Quality: ⭐⭐⭐⭐

---

## 🛠️ Vector Databases

### Managed Services (Easiest)

| Tool | Pricing | Learning Curve | Scalability | Use Case |
|------|---------|---|---|---|
| **Pinecone** | Free tier + paid | Very easy | High | Production RAG |
| **Weaviate Cloud** | Free tier + paid | Easy | High | Enterprise |
| **LanceDB** | Free | Very easy | Medium | Quick POC |
| **Supabase pgvector** | $5+/month | Medium | Medium | Developer friendly |

### Self-Hosted (Control & Cost)

1. **Weaviate** (Open-source)
   - Docker: `docker run -d -p 8080:8080 semitechnologies/weaviate`
   - URL: weaviate.io
   - Graphical DB
   - Good for experimentation

2. **Milvus** (Open-source)
   - URL: milvus.io
   - High performance
   - Horizontal scaling
   - For large datasets

3. **Qdrant** (Open-source)
   - URL: qdrant.tech
   - Rust-based
   - Very fast
   - Good documentation

4. **Chroma** (Embedded)
   - pip: `pip install chroma-db`
   - In-app vector DB
   - Perfect for development
   - No setup needed

5. **pgVector** (PostgreSQL extension)
   - SQL-based
   - Familiar for SQL users
   - Smaller datasets
   - Great for prototyping

### Comparison

```python
# Pinecone (Managed)
from pinecone import Pinecone
pc = Pinecone(api_key="...")
index = pc.Index("my-index")
results = index.query(embedding, top_k=5)

# Weaviate (Self-hosted)
import weaviate
client = weaviate.Client("http://localhost:8080")
results = client.query.get("Document", ["text"]).with_near_vector({
    "vector": embedding
}).with_limit(5).do()

# Chroma (Embedded)
import chromadb
client = chromadb.Client()
collection = client.get_or_create_collection("docs")
results = collection.query(query_embeddings=[embedding], n_results=5)
```

---

## 🤖 LLM APIs & Services

### Popular Providers

| Provider | Model | Cost | Quality | Speed |
|----------|-------|------|---------|-------|
| **OpenAI** | GPT-4 Turbo | $0.03/$0.06 per 1K | Excellent | Fast |
| **Anthropic** | Claude 3 Opus | $0.015/$0.075 per 1K | Excellent | Medium |
| **Google** | Gemini 1.5 | $0.075/$0.3 per 1M | Very Good | Very Fast |
| **Meta** | Llama 2/3 | Open-source | Good | N/A |
| **Mistral** | Mistral Large | $0.008/$0.024 per 1K | Good | Very Fast |

### Open-Source Models

```bash
# Ollama (Run locally)
ollama run llama2
ollama run mistral

# Hugging Face
from transformers import AutoModelForCausalLM
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b")
```

---

## 🧠 Embedding Models

### Popular Options

| Model | Dimensions | Speed | Quality | Cost | Type |
|-------|-----------|-------|---------|------|------|
| **text-embedding-3-small** | 512 | Fast | Very Good | $0.02/1M | API |
| **text-embedding-3-large** | 3072 | Fast | Excellent | $0.13/1M | API |
| **all-MiniLM-L6-v2** | 384 | Very Fast | Good | Free | Open-source |
| **all-mpnet-base-v2** | 768 | Fast | Excellent | Free | Open-source |
| **BGE-large-en-v1.5** | 1024 | Fast | Excellent | Free | Open-source |

### Code Examples

```python
# OpenAI Embeddings
from openai import OpenAI
client = OpenAI()
embedding = client.embeddings.create(
    model="text-embedding-3-small",
    input="Hello world"
).data[0].embedding

# Sentence Transformers (Open-source)
from sentence_transformers import SentenceTransformer
model = SentenceTransformer('all-MiniLM-L6-v2')
embeddings = model.encode(["Hello world"])

# Hugging Face
from transformers import AutoTokenizer, AutoModel
tokenizer = AutoTokenizer.from_pretrained("BAAI/bge-large-en-v1.5")
model = AutoModel.from_pretrained("BAAI/bge-large-en-v1.5")
```

---

## 🔗 RAG Frameworks & Libraries

### Top Frameworks

1. **LangChain** (Most Popular)
   - URL: python.langchain.com
   - Install: `pip install langchain`
   - Features: Chains, agents, memory, RAG
   - Docs: Excellent
   - Use: Production & prototyping

2. **LlamaIndex** (Document-focused)
   - URL: docs.llamaindex.ai
   - Install: `pip install llama-index`
   - Features: Document indexing, querying
   - Docs: Very good
   - Use: Document Q&A, RAG

3. **Haystack** (Pipeline-based)
   - URL: haystack.deepset.ai
   - Install: `pip install haystack-ai`
   - Features: Pipelines, retrievers
   - Docs: Good
   - Use: Complex RAG pipelines

4. **Semantic Kernel** (Microsoft)
   - URL: microsoft.github.io/semantic-kernel
   - Languages: Python, C#, Java
   - Features: Skills, connectors
   - Use: Enterprise applications

### Quick Start Example

```python
# LangChain RAG
from langchain.vectorstores import Chroma
from langchain.embeddings.openai import OpenAIEmbeddings
from langchain.text_splitter import CharacterTextSplitter
from langchain.chains import RetrievalQA
from langchain.llms import OpenAI

# 1. Load documents
documents = [{"page_content": "Your text"}]

# 2. Split text
splitter = CharacterTextSplitter(chunk_size=1000)
docs = splitter.split_documents(documents)

# 3. Create embeddings
embeddings = OpenAIEmbeddings()

# 4. Store in vector DB
vectorstore = Chroma.from_documents(docs, embeddings)

# 5. Create RAG chain
qa = RetrievalQA.from_chain_type(
    llm=OpenAI(),
    chain_type="stuff",
    retriever=vectorstore.as_retriever()
)

# 6. Query
answer = qa.run("Your question here")
```

---

## 📊 Evaluation & Benchmarking Tools

### Evaluation Frameworks

1. **RAGAS** (RAG Assessment)
   - GitHub: explodinggradients/ragas
   - Metrics: Faithfulness, answer relevance
   - Use: Automatic RAG evaluation
   - Install: `pip install ragas`

2. **Trulens** (Feedback Functions)
   - URL: trulens.org
   - Metrics: Hallucination, groundedness
   - Use: Monitor RAG quality
   - Install: `pip install trulens-eval`

3. **LangSmith** (Debugging & Monitoring)
   - URL: langsmith.io
   - Features: Tracing, testing, monitoring
   - Cost: Free tier available
   - Use: Production monitoring

4. **DeepEval** (Evaluation Framework)
   - GitHub: confident-ai/deepeval
   - Metrics: Hallucination, answer relevance
   - Use: CI/CD for LLM apps
   - Install: `pip install deepeval`

### Benchmarks to Test Against

1. **SQuAD** - Question Answering
2. **Natural Questions** - Real user questions
3. **TriviaQA** - Trivia knowledge
4. **HotpotQA** - Multi-hop reasoning
5. **MS MARCO** - Information Ranking

---

## 💬 Communities & Forums

### Active Communities

1. **r/MachineLearning** (Reddit)
   - URL: reddit.com/r/MachineLearning
   - Community: 1M+ members
   - Activity: High
   - Quality: Excellent

2. **Hugging Face Forums**
   - URL: discuss.huggingface.co
   - Community: Researchers & practitioners
   - Activity: High
   - Quality: Excellent

3. **LangChain Discord**
   - URL: discord.gg/langchain
   - Community: RAG & LLM builders
   - Activity: Very High
   - Quality: Excellent

4. **Weaviate Community**
   - URL: forum.weaviate.io
   - Community: Vector DB users
   - Activity: Medium
   - Quality: Good

5. **Semantic Web Forums**
   - URL: semanticweb.org
   - Community: Knowledge graphs
   - Activity: Medium
   - Quality: Expert-level

### Twitter/X Accounts to Follow

- @OpenAI - LLM updates
- @AnthropicAI - Claude updates
- @huggingface - NLP/ML news
- @langchainai - RAG updates
- @weaviate_io - Vector DB news

---

## 📈 Cost Estimation for Different Setups

### Scenario 1: Hobby Project ($0-50/month)

```
Components:
- Embedding Model: Sentence Transformers (free, self-hosted)
- Vector DB: Chroma (free, embedded)
- LLM: Ollama/Llama2 (free, self-hosted)
- Hosting: Home server/laptop
Total: $0/month (excluding electricity)
```

### Scenario 2: Startup MVP ($100-500/month)

```
Components:
- Embedding Model: OpenAI text-embedding-3-small (~$20/month)
- Vector DB: Pinecone (free tier or $50/month)
- LLM: GPT-3.5-turbo ($50-200/month depending on usage)
- Infrastructure: Serverless (~$50/month)
Total: $120-320/month
```

### Scenario 3: Production Scale ($500-5000+/month)

```
Components:
- Embedding Model: OpenAI text-embedding-3-large (~$100/month)
- Vector DB: Pinecone (Pro $100+) or Self-hosted Milvus ($200)
- LLM: GPT-4 or Claude 3 ($500-2000/month)
- Infrastructure: Dedicated servers ($200-1000/month)
- Monitoring: LangSmith Pro ($100-500/month)
Total: $1000-5000+/month depending on scale
```

### Cost Optimization Tips

1. **Use cheaper models for less critical tasks**
   - Retrieval: Fast embedding models
   - Ranking: Smaller cross-encoders
   - Generation: GPT-3.5 instead of GPT-4

2. **Batch processing**
   - Embed multiple documents at once
   - Reduce API calls

3. **Caching**
   - Cache frequent queries
   - Reduce repeated computations

4. **Open-source where possible**
   - Self-host embeddings
   - Use local LLMs for dev/testing

5. **Sampling during development**
   - Don't evaluate on full dataset
   - Use representative samples

---

## 📋 Recommended Learning Path with Resources

### Week 1-2: Foundations
- Read: "Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks"
- Course: DeepLearning.AI - Intro to LLMs (2 hours)
- Practice: Explore vector databases (Pinecone, Weaviate)

### Week 3-4: Core Concepts
- Read: "Dense Passage Retrieval" paper
- Course: Hugging Face Course - Embeddings section
- Practice: Build simple embedding pipeline

### Week 5-7: Advanced Architecture
- Course: Fast.ai Deep Learning part 2
- Code: Build RAG with LangChain
- Research: Read latest RAG papers on arxiv

### Week 8-9: Production
- Course: LangSmith documentation
- Implement: Monitoring and evaluation
- Benchmark: RAGAS evaluation framework

### Week 10-12: Advanced Patterns
- Research: Multi-hop, graph-based RAG
- Build: Capstone project
- Present: Share learnings

---

## 🔍 Research Papers (Must Read)

1. **Retrieval-Augmented Generation** (2020)
   - arxiv.org/abs/2005.11401

2. **Dense Passage Retrieval** (2020)
   - arxiv.org/abs/2004.04906

3. **REALM: Retrieval-Augmented Language Model** (2020)
   - arxiv.org/abs/2002.08909

4. **Fusion-in-Decoder** (2021)
   - arxiv.org/abs/2007.01282

5. **ColBERT: Efficient and Effective Passage Search** (2020)
   - arxiv.org/abs/2004.12832

6. **HyDE: Hypothetical Document Embeddings** (2022)
   - arxiv.org/abs/2212.10496

7. **Self-RAG: Learning to Retrieve, Generate, and Critique** (2023)
   - arxiv.org/abs/2310.11511

---

## 🎯 Project Ideas (Building Experience)

### Beginner Projects
1. Q&A chatbot for your own documents
2. Resume analyzer
3. PDF question answerer

### Intermediate Projects
1. Multi-source knowledge base
2. Customer support chatbot
3. Research paper summarizer

### Advanced Projects
1. Multi-modal RAG (text + images)
2. Real-time news analysis system
3. Domain-specific search engine

---

**Total Resource Estimated Time**: 100+ hours to completion

**Last Updated**: 2026-05-13
