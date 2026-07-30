# 🤖 RAG (Retrieval-Augmented Generation) Protocol in AI - Complete Interview Guide

## 📚 **What is RAG?**

**RAG (Retrieval-Augmented Generation)** is an AI framework that combines:
- **Retrieval**: Finding relevant information from external knowledge sources
- **Generation**: Using LLMs to generate responses based on retrieved information

Think of it as giving an AI assistant access to a library (retrieval) and then asking it to write an answer (generation) using books from that library.

---

## 🎯 **Why RAG is Important?**

### **Problems RAG Solves:**
1. **Knowledge Cutoff**: LLMs have training data cutoffs
2. **Hallucination**: LLMs sometimes generate false information
3. **Domain-Specific Knowledge**: Need for specialized, up-to-date information
4. **Cost**: Fine-tuning large models is expensive

### **RAG Benefits:**
- ✅ Access to real-time information
- ✅ Reduced hallucinations
- ✅ Domain-specific accuracy
- ✅ Cost-effective compared to fine-tuning
- ✅ Transparent source attribution

---

## 🏗️ **RAG Architecture**

```
User Query → Retrieval System → Knowledge Base
                ↓
Retrieved Documents → LLM → Generated Response
```

### **Core Components:**

1. **Knowledge Base** (Vector Database)
2. **Retrieval System** (Embedding + Search)
3. **Generation Model** (LLM)
4. **Orchestration Layer**

---

## 💻 **Complete RAG Implementation Example**

Let's build a **Customer Support RAG System** for a tech company:

### **1. Project Structure**

```
rag-system/
├── data/
│   ├── documents/
│   └── processed/
├── src/
│   ├── __init__.py
│   ├── document_processor.py
│   ├── vector_store.py
│   ├── retriever.py
│   ├── generator.py
│   ├── rag_pipeline.py
│   └── api.py
├── requirements.txt
├── config.py
└── main.py
```

### **2. Requirements (`requirements.txt`)**

```txt
# Core RAG components
langchain==0.1.0
openai==1.3.0
chromadb==0.4.18
sentence-transformers==2.2.2

# Document processing
pypdf2==3.0.1
python-docx==0.8.11
beautifulsoup4==4.12.2

# Vector operations
faiss-cpu==1.7.4
numpy==1.24.3

# API and utilities
fastapi==0.104.1
uvicorn==0.24.0
pydantic==2.5.0
python-dotenv==1.0.0
streamlit==1.28.0

# Monitoring and logging
wandb==0.16.0
loguru==0.7.2
```

### **3. Configuration (`config.py`)**

```python
# config.py
import os
from dotenv import load_dotenv

load_dotenv()

class RAGConfig:
    # OpenAI Configuration
    OPENAI_API_KEY = os.getenv("OPENAI_API_KEY")
    
    # Model Configuration
    EMBEDDING_MODEL = "sentence-transformers/all-MiniLM-L6-v2"
    LLM_MODEL = "gpt-3.5-turbo"
    
    # Vector Store Configuration
    VECTOR_DB_PATH = "./data/vector_db"
    COLLECTION_NAME = "customer_support_docs"
    
    # Retrieval Configuration
    TOP_K_DOCUMENTS = 5
    SIMILARITY_THRESHOLD = 0.7
    
    # Generation Configuration
    MAX_TOKENS = 500
    TEMPERATURE = 0.1
    
    # Document Processing
    CHUNK_SIZE = 1000
    CHUNK_OVERLAP = 200
    
    # API Configuration
    API_HOST = "0.0.0.0"
    API_PORT = 8000

config = RAGConfig()
```

### **4. Document Processor (`src/document_processor.py`)**

```python
# src/document_processor.py
import os
import re
from typing import List, Dict
from pathlib import Path
import PyPDF2
from docx import Document
from bs4 import BeautifulSoup
from langchain.text_splitter import RecursiveCharacterTextSplitter
from loguru import logger

class DocumentProcessor:
    """Process and chunk documents for RAG system"""
    
    def __init__(self, chunk_size: int = 1000, chunk_overlap: int = 200):
        self.chunk_size = chunk_size
        self.chunk_overlap = chunk_overlap
        self.text_splitter = RecursiveCharacterTextSplitter(
            chunk_size=chunk_size,
            chunk_overlap=chunk_overlap,
            length_function=len,
            separators=["\n\n", "\n", " ", ""]
        )
    
    def extract_text_from_pdf(self, file_path: str) -> str:
        """Extract text from PDF file"""
        try:
            with open(file_path, 'rb') as file:
                pdf_reader = PyPDF2.PdfReader(file)
                text = ""
                for page in pdf_reader.pages:
                    text += page.extract_text() + "\n"
                return text
        except Exception as e:
            logger.error(f"Error extracting text from PDF {file_path}: {e}")
            return ""
    
    def extract_text_from_docx(self, file_path: str) -> str:
        """Extract text from DOCX file"""
        try:
            doc = Document(file_path)
            text = ""
            for paragraph in doc.paragraphs:
                text += paragraph.text + "\n"
            return text
        except Exception as e:
            logger.error(f"Error extracting text from DOCX {file_path}: {e}")
            return ""
    
    def extract_text_from_txt(self, file_path: str) -> str:
        """Extract text from TXT file"""
        try:
            with open(file_path, 'r', encoding='utf-8') as file:
                return file.read()
        except Exception as e:
            logger.error(f"Error extracting text from TXT {file_path}: {e}")
            return ""
    
    def clean_text(self, text: str) -> str:
        """Clean and normalize text"""
        # Remove extra whitespace
        text = re.sub(r'\s+', ' ', text)
        # Remove special characters but keep basic punctuation
        text = re.sub(r'[^\w\s\.\,\!\?\;\:\-\(\)]', '', text)
        # Remove multiple consecutive punctuation
        text = re.sub(r'[\.]{2,}', '.', text)
        return text.strip()
    
    def process_document(self, file_path: str) -> List[Dict[str, str]]:
        """Process a single document and return chunks"""
        file_extension = Path(file_path).suffix.lower()
        
        # Extract text based on file type
        if file_extension == '.pdf':
            text = self.extract_text_from_pdf(file_path)
        elif file_extension == '.docx':
            text = self.extract_text_from_docx(file_path)
        elif file_extension == '.txt':
            text = self.extract_text_from_txt(file_path)
        else:
            logger.warning(f"Unsupported file type: {file_extension}")
            return []
        
        if not text:
            logger.warning(f"No text extracted from {file_path}")
            return []
        
        # Clean text
        cleaned_text = self.clean_text(text)
        
        # Split into chunks
        chunks = self.text_splitter.split_text(cleaned_text)
        
        # Create document chunks with metadata
        document_chunks = []
        for i, chunk in enumerate(chunks):
            document_chunks.append({
                "content": chunk,
                "source": file_path,
                "chunk_id": f"{Path(file_path).stem}_{i}",
                "metadata": {
                    "file_name": Path(file_path).name,
                    "file_type": file_extension,
                    "chunk_index": i,
                    "total_chunks": len(chunks)
                }
            })
        
        logger.info(f"Processed {file_path}: {len(chunks)} chunks created")
        return document_chunks
    
    def process_directory(self, directory_path: str) -> List[Dict[str, str]]:
        """Process all documents in a directory"""
        all_chunks = []
        supported_extensions = ['.pdf', '.docx', '.txt']
        
        for file_path in Path(directory_path).rglob('*'):
            if file_path.suffix.lower() in supported_extensions:
                chunks = self.process_document(str(file_path))
                all_chunks.extend(chunks)
        
        logger.info(f"Processed {len(all_chunks)} total chunks from {directory_path}")
        return all_chunks

# Example usage and sample documents
def create_sample_documents():
    """Create sample customer support documents"""
    sample_docs = {
        "product_guide.txt": """
        Product Installation Guide
        
        Welcome to our software installation guide. This document will help you install and configure our product.
        
        System Requirements:
        - Windows 10 or later / macOS 10.15 or later / Linux Ubuntu 18.04+
        - 8GB RAM minimum, 16GB recommended
        - 10GB free disk space
        - Internet connection for activation
        
        Installation Steps:
        1. Download the installer from our website
        2. Run the installer as administrator
        3. Follow the setup wizard
        4. Enter your license key when prompted
        5. Complete the installation
        
        Troubleshooting:
        If installation fails, check that you have sufficient permissions and disk space.
        """,
        
        "api_documentation.txt": """
        API Documentation
        
        Our REST API allows you to integrate with our platform programmatically.
        
        Authentication:
        All API requests require an API key in the header:
        Authorization: Bearer YOUR_API_KEY
        
        Base URL: https://api.ourcompany.com/v1
        
        Endpoints:
        
        GET /users - Retrieve user list
        POST /users - Create new user
        PUT /users/{id} - Update user
        DELETE /users/{id} - Delete user
        
        Rate Limiting:
        API calls are limited to 1000 requests per hour per API key.
        
        Error Codes:
        400 - Bad Request
        401 - Unauthorized
        403 - Forbidden
        404 - Not Found
        429 - Rate Limit Exceeded
        500 - Internal Server Error
        """,
        
        "billing_faq.txt": """
        Billing and Pricing FAQ
        
        Q: How is billing calculated?
        A: Billing is based on your subscription plan and usage. We offer monthly and annual plans.
        
        Q: When am I charged?
        A: Charges occur on your billing date each month for monthly plans, or annually for yearly plans.
        
        Q: Can I change my plan?
        A: Yes, you can upgrade or downgrade your plan at any time. Changes take effect on your next billing cycle.
        
        Q: What payment methods do you accept?
        A: We accept all major credit cards, PayPal, and bank transfers for enterprise customers.
        
        Q: How do I cancel my subscription?
        A: You can cancel your subscription from your account settings or by contacting support.
        
        Q: Do you offer refunds?
        A: We offer a 30-day money-back guarantee for new customers.
        
        Q: What happens if my payment fails?
        A: We'll retry the payment and send you notifications. Your account may be suspended after multiple failures.
        """
    }
    
    # Create data directory
    os.makedirs("./data/documents", exist_ok=True)
    
    # Write sample documents
    for filename, content in sample_docs.items():
        with open(f"./data/documents/{filename}", 'w') as f:
            f.write(content)
    
    logger.info("Sample documents created successfully")

if __name__ == "__main__":
    # Create sample documents
    create_sample_documents()
    
    # Process documents
    processor = DocumentProcessor()
    chunks = processor.process_directory("./data/documents")
    
    print(f"Processed {len(chunks)} chunks")
    for chunk in chunks[:2]:  # Show first 2 chunks
        print(f"\nChunk ID: {chunk['chunk_id']}")
        print(f"Content: {chunk['content'][:200]}...")
```

### **5. Vector Store (`src/vector_store.py`)**

```python
# src/vector_store.py
import chromadb
import numpy as np
from typing import List, Dict, Optional
from sentence_transformers import SentenceTransformer
from loguru import logger
from config import config

class VectorStore:
    """Vector store for document embeddings using ChromaDB"""
    
    def __init__(self, persist_directory: str = None, collection_name: str = None):
        self.persist_directory = persist_directory or config.VECTOR_DB_PATH
        self.collection_name = collection_name or config.COLLECTION_NAME
        
        # Initialize ChromaDB client
        self.client = chromadb.PersistentClient(path=self.persist_directory)
        
        # Initialize embedding model
        self.embedding_model = SentenceTransformer(config.EMBEDDING_MODEL)
        
        # Get or create collection
        self.collection = self.client.get_or_create_collection(
            name=self.collection_name,
            metadata={"description": "Customer support documents"}
        )
        
        logger.info(f"Vector store initialized with collection: {self.collection_name}")
    
    def generate_embeddings(self, texts: List[str]) -> List[List[float]]:
        """Generate embeddings for a list of texts"""
        try:
            embeddings = self.embedding_model.encode(texts, convert_to_tensor=False)
            return embeddings.tolist()
        except Exception as e:
            logger.error(f"Error generating embeddings: {e}")
            return []
    
    def add_documents(self, documents: List[Dict[str, str]]) -> bool:
        """Add documents to the vector store"""
        try:
            # Prepare data for ChromaDB
            ids = [doc["chunk_id"] for doc in documents]
            texts = [doc["content"] for doc in documents]
            metadatas = [doc["metadata"] for doc in documents]
            
            # Generate embeddings
            logger.info(f"Generating embeddings for {len(texts)} documents...")
            embeddings = self.generate_embeddings(texts)
            
            if not embeddings:
                logger.error("Failed to generate embeddings")
                return False
            
            # Add to collection
            self.collection.add(
                ids=ids,
                documents=texts,
                metadatas=metadatas,
                embeddings=embeddings
            )
            
            logger.info(f"Successfully added {len(documents)} documents to vector store")
            return True
            
        except Exception as e:
            logger.error(f"Error adding documents to vector store: {e}")
            return False
    
    def search(self, query: str, top_k: int = None) -> List[Dict]:
        """Search for similar documents"""
        top_k = top_k or config.TOP_K_DOCUMENTS
        
        try:
            # Generate query embedding
            query_embedding = self.generate_embeddings([query])[0]
            
            # Search in collection
            results = self.collection.query(
                query_embeddings=[query_embedding],
                n_results=top_k,
                include=["documents", "metadatas", "distances"]
            )
            
            # Format results
            formatted_results = []
            for i in range(len(results["documents"][0])):
                formatted_results.append({
                    "content": results["documents"][0][i],
                    ```python
                    "metadata": results["metadatas"][0][i],
                    "similarity_score": 1 - results["distances"][0][i],  # Convert distance to similarity
                    "distance": results["distances"][0][i]
                })
            
            # Filter by similarity threshold
            filtered_results = [
                result for result in formatted_results 
                if result["similarity_score"] >= config.SIMILARITY_THRESHOLD
            ]
            
            logger.info(f"Found {len(filtered_results)} relevant documents for query")
            return filtered_results
            
        except Exception as e:
            logger.error(f"Error searching vector store: {e}")
            return []
    
    def get_collection_stats(self) -> Dict:
        """Get statistics about the collection"""
        try:
            count = self.collection.count()
            return {
                "total_documents": count,
                "collection_name": self.collection_name,
                "embedding_model": config.EMBEDDING_MODEL
            }
        except Exception as e:
            logger.error(f"Error getting collection stats: {e}")
            return {}
    
    def delete_collection(self):
        """Delete the entire collection"""
        try:
            self.client.delete_collection(self.collection_name)
            logger.info(f"Deleted collection: {self.collection_name}")
        except Exception as e:
            logger.error(f"Error deleting collection: {e}")

if __name__ == "__main__":
    # Test vector store
    from document_processor import DocumentProcessor, create_sample_documents
    
    # Create sample documents and process them
    create_sample_documents()
    processor = DocumentProcessor()
    chunks = processor.process_directory("./data/documents")
    
    # Initialize vector store and add documents
    vector_store = VectorStore()
    vector_store.add_documents(chunks)
    
    # Test search
    query = "How do I install the software?"
    results = vector_store.search(query)
    
    print(f"\nSearch results for: '{query}'")
    for i, result in enumerate(results):
        print(f"\n{i+1}. Similarity: {result['similarity_score']:.3f}")
        print(f"Content: {result['content'][:200]}...")
```

### **6. Retriever (`src/retriever.py`)**

```python
# src/retriever.py
from typing import List, Dict, Optional
from vector_store import VectorStore
from loguru import logger
from config import config

class DocumentRetriever:
    """Advanced document retrieval with multiple strategies"""
    
    def __init__(self, vector_store: VectorStore):
        self.vector_store = vector_store
    
    def retrieve_documents(self, query: str, top_k: int = None, strategy: str = "similarity") -> List[Dict]:
        """Retrieve relevant documents using specified strategy"""
        top_k = top_k or config.TOP_K_DOCUMENTS
        
        if strategy == "similarity":
            return self._similarity_retrieval(query, top_k)
        elif strategy == "hybrid":
            return self._hybrid_retrieval(query, top_k)
        elif strategy == "rerank":
            return self._rerank_retrieval(query, top_k)
        else:
            logger.warning(f"Unknown strategy: {strategy}, using similarity")
            return self._similarity_retrieval(query, top_k)
    
    def _similarity_retrieval(self, query: str, top_k: int) -> List[Dict]:
        """Basic similarity-based retrieval"""
        return self.vector_store.search(query, top_k)
    
    def _hybrid_retrieval(self, query: str, top_k: int) -> List[Dict]:
        """Hybrid retrieval combining semantic and keyword search"""
        # Get semantic results
        semantic_results = self.vector_store.search(query, top_k * 2)
        
        # Simple keyword scoring (in production, use proper BM25)
        query_words = set(query.lower().split())
        
        for result in semantic_results:
            content_words = set(result["content"].lower().split())
            keyword_overlap = len(query_words.intersection(content_words)) / len(query_words)
            
            # Combine semantic and keyword scores
            result["hybrid_score"] = (
                result["similarity_score"] * 0.7 + 
                keyword_overlap * 0.3
            )
        
        # Sort by hybrid score and return top_k
        semantic_results.sort(key=lambda x: x["hybrid_score"], reverse=True)
        return semantic_results[:top_k]
    
    def _rerank_retrieval(self, query: str, top_k: int) -> List[Dict]:
        """Retrieval with reranking based on query relevance"""
        # Get initial results
        initial_results = self.vector_store.search(query, top_k * 3)
        
        # Simple reranking based on content length and query word frequency
        query_words = query.lower().split()
        
        for result in initial_results:
            content = result["content"].lower()
            
            # Calculate query word frequency in content
            word_freq_score = sum(content.count(word) for word in query_words) / len(content)
            
            # Penalize very short or very long content
            length_penalty = 1.0
            content_length = len(result["content"])
            if content_length < 100:
                length_penalty = 0.8
            elif content_length > 2000:
                length_penalty = 0.9
            
            # Calculate rerank score
            result["rerank_score"] = (
                result["similarity_score"] * 0.6 + 
                word_freq_score * 0.3 + 
                length_penalty * 0.1
            )
        
        # Sort by rerank score
        initial_results.sort(key=lambda x: x["rerank_score"], reverse=True)
        return initial_results[:top_k]
    
    def get_context_for_generation(self, retrieved_docs: List[Dict], max_context_length: int = 3000) -> str:
        """Prepare context string for LLM generation"""
        context_parts = []
        current_length = 0
        
        for doc in retrieved_docs:
            content = doc["content"]
            source = doc["metadata"].get("file_name", "Unknown")
            
            # Add source attribution
            formatted_content = f"[Source: {source}]\n{content}\n"
            
            if current_length + len(formatted_content) <= max_context_length:
                context_parts.append(formatted_content)
                current_length += len(formatted_content)
            else:
                # Truncate the last document if needed
                remaining_space = max_context_length - current_length
                if remaining_space > 100:  # Only add if meaningful space left
                    truncated_content = formatted_content[:remaining_space-3] + "..."
                    context_parts.append(truncated_content)
                break
        
        return "\n---\n".join(context_parts)

if __name__ == "__main__":
    # Test retriever
    from document_processor import DocumentProcessor, create_sample_documents
    
    # Setup
    create_sample_documents()
    processor = DocumentProcessor()
    chunks = processor.process_directory("./data/documents")
    
    vector_store = VectorStore()
    vector_store.add_documents(chunks)
    
    retriever = DocumentRetriever(vector_store)
    
    # Test different retrieval strategies
    query = "What are the system requirements for installation?"
    
    print("=== Similarity Retrieval ===")
    sim_results = retriever.retrieve_documents(query, strategy="similarity")
    for i, result in enumerate(sim_results[:2]):
        print(f"{i+1}. Score: {result['similarity_score']:.3f}")
        print(f"Content: {result['content'][:150]}...\n")
    
    print("=== Hybrid Retrieval ===")
    hybrid_results = retriever.retrieve_documents(query, strategy="hybrid")
    for i, result in enumerate(hybrid_results[:2]):
        print(f"{i+1}. Hybrid Score: {result['hybrid_score']:.3f}")
        print(f"Content: {result['content'][:150]}...\n")
    
    print("=== Context for Generation ===")
    context = retriever.get_context_for_generation(sim_results)
    print(f"Context length: {len(context)} characters")
    print(f"Context preview: {context[:300]}...")
```

### **7. Generator (`src/generator.py`)**

```python
# src/generator.py
import openai
from typing import List, Dict, Optional
from loguru import logger
from config import config

class ResponseGenerator:
    """Generate responses using LLM with retrieved context"""
    
    def __init__(self, api_key: str = None, model: str = None):
        self.api_key = api_key or config.OPENAI_API_KEY
        self.model = model or config.LLM_MODEL
        
        # Initialize OpenAI client
        openai.api_key = self.api_key
        
        # System prompts for different scenarios
        self.system_prompts = {
            "customer_support": """You are a helpful customer support assistant. Use the provided context to answer user questions accurately and professionally. 

Guidelines:
- Always base your answers on the provided context
- If the context doesn't contain relevant information, say so clearly
- Provide step-by-step instructions when applicable
- Include source references when possible
- Be concise but comprehensive
- Use a friendly, professional tone""",
            
            "technical": """You are a technical documentation assistant. Provide precise, technical answers based on the context.

Guidelines:
- Focus on accuracy and technical details
- Include code examples or configuration details when available
- Explain technical concepts clearly
- Reference specific documentation sections
- If information is incomplete, specify what's missing""",
            
            "general": """You are a knowledgeable assistant. Answer questions based on the provided context.

Guidelines:
- Be accurate and helpful
- Use the context as your primary information source
- Acknowledge limitations in the available information
- Provide clear, well-structured responses"""
        }
    
    def generate_response(
        self, 
        query: str, 
        context: str, 
        prompt_type: str = "customer_support",
        temperature: float = None,
        max_tokens: int = None
    ) -> Dict[str, str]:
        """Generate response using LLM"""
        
        temperature = temperature or config.TEMPERATURE
        max_tokens = max_tokens or config.MAX_TOKENS
        
        try:
            # Get system prompt
            system_prompt = self.system_prompts.get(prompt_type, self.system_prompts["general"])
            
            # Construct user message with context
            user_message = f"""Context:
{context}

Question: {query}

Please provide a helpful answer based on the context above."""

            # Generate response using OpenAI
            response = openai.ChatCompletion.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": system_prompt},
                    {"role": "user", "content": user_message}
                ],
                temperature=temperature,
                max_tokens=max_tokens,
                top_p=0.9,
                frequency_penalty=0.1,
                presence_penalty=0.1
            )
            
            generated_text = response.choices[0].message.content.strip()
            
            return {
                "response": generated_text,
                "model": self.model,
                "prompt_tokens": response.usage.prompt_tokens,
                "completion_tokens": response.usage.completion_tokens,
                "total_tokens": response.usage.total_tokens,
                "status": "success"
            }
            
        except Exception as e:
            logger.error(f"Error generating response: {e}")
            return {
                "response": "I apologize, but I'm unable to generate a response at the moment. Please try again later.",
                "error": str(e),
                "status": "error"
            }
    
    def generate_with_citations(
        self, 
        query: str, 
        retrieved_docs: List[Dict], 
        prompt_type: str = "customer_support"
    ) -> Dict[str, str]:
        """Generate response with source citations"""
        
        # Prepare context with numbered sources
        context_parts = []
        sources = []
        
        for i, doc in enumerate(retrieved_docs, 1):
            source_info = {
                "number": i,
                "file_name": doc["metadata"].get("file_name", "Unknown"),
                "similarity_score": doc.get("similarity_score", 0)
            }
            sources.append(source_info)
            
            context_parts.append(f"[{i}] {doc['content']}")
        
        context = "\n\n".join(context_parts)
        
        # Modified system prompt for citations
        citation_prompt = self.system_prompts[prompt_type] + """

IMPORTANT: When referencing information from the context, include citation numbers in square brackets [1], [2], etc. corresponding to the source numbers provided."""
        
        try:
            user_message = f"""Context with sources:
{context}

Question: {query}

Please provide a helpful answer with appropriate citations [1], [2], etc."""

            response = openai.ChatCompletion.create(
                model=self.model,
                messages=[
                    {"role": "system", "content": citation_prompt},
                    {"role": "user", "content": user_message}
                ],
                temperature=config.TEMPERATURE,
                max_tokens=config.MAX_TOKENS
            )
            
            generated_text = response.choices[0].message.content.strip()
            
            return {
                "response": generated_text,
                "sources": sources,
                "model": self.model,
                "total_tokens": response.usage.total_tokens,
                "status": "success"
            }
            
        except Exception as e:
            logger.error(f"Error generating response with citations: {e}")
            return {
                "response": "I apologize, but I'm unable to generate a response at the moment.",
                "sources": sources,
                "error": str(e),
                "status": "error"
            }
    
    def evaluate_response_quality(self, query: str, response: str, context: str) -> Dict[str, float]:
        """Simple response quality evaluation"""
        try:
            # Basic quality metrics
            metrics = {}
            
            # Relevance: Check if response addresses the query
            query_words = set(query.lower().split())
            response_words = set(response.lower().split())
            relevance_score = len(query_words.intersection(response_words)) / len(query_words)
            metrics["relevance"] = min(relevance_score, 1.0)
            
            # Groundedness: Check if response is based on context
            context_words = set(context.lower().split())
            groundedness_score = len(response_words.intersection(context_words)) / len(response_words)
            metrics["groundedness"] = min(groundedness_score, 1.0)
            
            # Completeness: Response length relative to context
            completeness_score = min(len(response) / (len(context) * 0.3), 1.0)
            metrics["completeness"] = completeness_score
            
            # Overall quality score
            metrics["overall"] = (
                metrics["relevance"] * 0.4 + 
                metrics["groundedness"] * 0.4 + 
                metrics["completeness"] * 0.2
            )
            
            return metrics
            
        except Exception as e:
            logger.error(f"Error evaluating response quality: {e}")
            return {"overall": 0.0, "error": str(e)}

if __name__ == "__main__":
    # Test generator (requires OpenAI API key)
    import os
    
    if not os.getenv("OPENAI_API_KEY"):
        print("Please set OPENAI_API_KEY environment variable to test the generator")
        exit(1)
    
    generator = ResponseGenerator()
    
    # Test basic generation
    test_context = """
