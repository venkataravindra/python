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
    ```python
    # Test basic generation
    test_context = """
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
    """
    
    test_query = "What are the system requirements for installation?"
    
    result = generator.generate_response(test_query, test_context)
    print("Generated Response:")
    print(result["response"])
    print(f"\nTokens used: {result.get('total_tokens', 'N/A')}")
    
    # Test quality evaluation
    quality = generator.evaluate_response_quality(test_query, result["response"], test_context)
    print(f"\nQuality Metrics: {quality}")
```

### **8. RAG Pipeline (`src/rag_pipeline.py`)**

```python
# src/rag_pipeline.py
import time
from typing import Dict, List, Optional
from loguru import logger

from document_processor import DocumentProcessor
from vector_store import VectorStore
from retriever import DocumentRetriever
from generator import ResponseGenerator
from config import config

class RAGPipeline:
    """Complete RAG pipeline orchestrating all components"""
    
    def __init__(self, initialize_components: bool = True):
        self.document_processor = None
        self.vector_store = None
        self.retriever = None
        self.generator = None
        
        if initialize_components:
            self._initialize_components()
    
    def _initialize_components(self):
        """Initialize all RAG components"""
        try:
            logger.info("Initializing RAG pipeline components...")
            
            # Initialize document processor
            self.document_processor = DocumentProcessor(
                chunk_size=config.CHUNK_SIZE,
                chunk_overlap=config.CHUNK_OVERLAP
            )
            
            # Initialize vector store
            self.vector_store = VectorStore()
            
            # Initialize retriever
            self.retriever = DocumentRetriever(self.vector_store)
            
            # Initialize generator
            self.generator = ResponseGenerator()
            
            logger.info("RAG pipeline components initialized successfully")
            
        except Exception as e:
            logger.error(f"Error initializing RAG pipeline: {e}")
            raise
    
    def ingest_documents(self, document_path: str, is_directory: bool = True) -> Dict[str, any]:
        """Ingest documents into the RAG system"""
        start_time = time.time()
        
        try:
            logger.info(f"Starting document ingestion from: {document_path}")
            
            # Process documents
            if is_directory:
                chunks = self.document_processor.process_directory(document_path)
            else:
                chunks = self.document_processor.process_document(document_path)
            
            if not chunks:
                return {
                    "status": "error",
                    "message": "No documents were processed",
                    "chunks_processed": 0,
                    "processing_time": time.time() - start_time
                }
            
            # Add to vector store
            success = self.vector_store.add_documents(chunks)
            
            processing_time = time.time() - start_time
            
            if success:
                logger.info(f"Successfully ingested {len(chunks)} chunks in {processing_time:.2f} seconds")
                return {
                    "status": "success",
                    "message": f"Successfully processed {len(chunks)} document chunks",
                    "chunks_processed": len(chunks),
                    "processing_time": processing_time
                }
            else:
                return {
                    "status": "error",
                    "message": "Failed to add documents to vector store",
                    "chunks_processed": len(chunks),
                    "processing_time": processing_time
                }
                
        except Exception as e:
            logger.error(f"Error during document ingestion: {e}")
            return {
                "status": "error",
                "message": f"Document ingestion failed: {str(e)}",
                "chunks_processed": 0,
                "processing_time": time.time() - start_time
            }
    
    def query(
        self, 
        question: str, 
        retrieval_strategy: str = "similarity",
        prompt_type: str = "customer_support",
        include_citations: bool = True,
        top_k: int = None
    ) -> Dict[str, any]:
        """Process a query through the complete RAG pipeline"""
        start_time = time.time()
        
        try:
            logger.info(f"Processing query: {question}")
            
            # Step 1: Retrieve relevant documents
            retrieved_docs = self.retriever.retrieve_documents(
                question, 
                top_k=top_k or config.TOP_K_DOCUMENTS,
                strategy=retrieval_strategy
            )
            
            if not retrieved_docs:
                return {
                    "status": "no_results",
                    "question": question,
                    "answer": "I couldn't find any relevant information to answer your question. Please try rephrasing your query or contact support for assistance.",
                    "sources": [],
                    "retrieval_time": time.time() - start_time,
                    "generation_time": 0,
                    "total_time": time.time() - start_time
                }
            
            retrieval_time = time.time() - start_time
            generation_start = time.time()
            
            # Step 2: Generate response
            if include_citations:
                result = self.generator.generate_with_citations(
                    question, 
                    retrieved_docs, 
                    prompt_type
                )
            else:
                context = self.retriever.get_context_for_generation(retrieved_docs)
                result = self.generator.generate_response(
                    question, 
                    context, 
                    prompt_type
                )
                result["sources"] = [
                    {
                        "number": i+1,
                        "file_name": doc["metadata"].get("file_name", "Unknown"),
                        "similarity_score": doc.get("similarity_score", 0)
                    }
                    for i, doc in enumerate(retrieved_docs)
                ]
            
            generation_time = time.time() - generation_start
            total_time = time.time() - start_time
            
            # Step 3: Evaluate response quality
            if result["status"] == "success":
                context = self.retriever.get_context_for_generation(retrieved_docs)
                quality_metrics = self.generator.evaluate_response_quality(
                    question, 
                    result["response"], 
                    context
                )
            else:
                quality_metrics = {"overall": 0.0}
            
            # Compile final response
            response = {
                "status": result["status"],
                "question": question,
                "answer": result["response"],
                "sources": result.get("sources", []),
                "quality_metrics": quality_metrics,
                "retrieval_strategy": retrieval_strategy,
                "documents_retrieved": len(retrieved_docs),
                "retrieval_time": retrieval_time,
                "generation_time": generation_time,
                "total_time": total_time,
                "tokens_used": result.get("total_tokens", 0)
            }
            
            if result["status"] == "error":
                response["error"] = result.get("error", "Unknown error")
            
            logger.info(f"Query processed successfully in {total_time:.2f} seconds")
            return response
            
        except Exception as e:
            logger.error(f"Error processing query: {e}")
            return {
                "status": "error",
                "question": question,
                "answer": "I apologize, but I encountered an error while processing your question. Please try again later.",
                "error": str(e),
                "total_time": time.time() - start_time
            }
    
    def get_system_stats(self) -> Dict[str, any]:
        """Get system statistics and health information"""
        try:
            stats = {
                "vector_store": self.vector_store.get_collection_stats() if self.vector_store else {},
                "config": {
                    "embedding_model": config.EMBEDDING_MODEL,
                    "llm_model": config.LLM_MODEL,
                    "chunk_size": config.CHUNK_SIZE,
                    "top_k_documents": config.TOP_K_DOCUMENTS,
                    "similarity_threshold": config.SIMILARITY_THRESHOLD
                },
                "status": "healthy"
            }
            return stats
        except Exception as e:
            logger.error(f"Error getting system stats: {e}")
            return {"status": "error", "error": str(e)}
    
    def reset_system(self) -> Dict[str, str]:
        """Reset the entire RAG system (delete all data)"""
        try:
            if self.vector_store:
                self.vector_store.delete_collection()
            
            # Reinitialize components
            self._initialize_components()
            
            logger.info("RAG system reset successfully")
            return {"status": "success", "message": "System reset successfully"}
            
        except Exception as e:
            logger.error(f"Error resetting system: {e}")
            return {"status": "error", "message": f"Reset failed: {str(e)}"}

# Example usage and testing
if __name__ == "__main__":
    import os
    from document_processor import create_sample_documents
    
    # Check for OpenAI API key
    if not os.getenv("OPENAI_API_KEY"):
        print("Please set OPENAI_API_KEY environment variable")
        exit(1)
    
    # Initialize RAG pipeline
    rag = RAGPipeline()
    
    # Create and ingest sample documents
    create_sample_documents()
    ingestion_result = rag.ingest_documents("./data/documents")
    print("Ingestion Result:", ingestion_result)
    
    # Test queries
    test_queries = [
        "How do I install the software?",
        "What are the system requirements?",
        "How is billing calculated?",
        "What API endpoints are available?",
        "How do I cancel my subscription?"
    ]
    
    print("\n" + "="*50)
    print("TESTING RAG PIPELINE")
    print("="*50)
    
    for query in test_queries:
        print(f"\nQuery: {query}")
        print("-" * 40)
        
        result = rag.query(query)
        
        print(f"Status: {result['status']}")
        print(f"Answer: {result['answer']}")
        print(f"Sources: {len(result.get('sources', []))} documents")
        print(f"Quality Score: {result.get('quality_metrics', {}).get('overall', 0):.3f}")
        print(f"Processing Time: {result['total_time']:.2f}s")
        
        if result.get('sources'):
            print("Source Files:")
            for source in result['sources'][:3]:  # Show top 3 sources
                print(f"  - {source['file_name']} (similarity: {source['similarity_score']:.3f})")
    
    # System stats
    print(f"\n{'='*50}")
    print("SYSTEM STATISTICS")
    print("="*50)
    stats = rag.get_system_stats()
    print(f"Total Documents: {stats['vector_store'].get('total_documents', 0)}")
    print(f"Embedding Model: {stats['config']['embedding_model']}")
    print(f"LLM Model: {stats['config']['llm_model']}")
    print(f"System Status: {stats['status']}")
```

### **9. API Server (`src/api.py`)**

```python
# src/api.py
from fastapi import FastAPI, HTTPException, BackgroundTasks
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel, Field
from typing import Optional, List, Dict
import uvicorn
from loguru import logger

from rag_pipeline import RAGPipeline
from config import config

# Initialize FastAPI app
app = FastAPI(
    title="RAG System API",
    description="Customer Support RAG System API",
    version="1.0.0"
)

# Add CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Initialize RAG pipeline
rag_pipeline = RAGPipeline()

# Pydantic models for API
class QueryRequest(BaseModel):
    question: str = Field(..., description="The question to ask")
    retrieval_strategy: str = Field("similarity", description="Retrieval strategy: similarity, hybrid, or rerank")
    prompt_type: str = Field("customer_support", description="Prompt type: customer_support, technical, or general")
    include_citations: bool = Field(True, description="Whether to include source citations")
    top_k: Optional[int] = Field(None, description="Number of documents to retrieve")

class QueryResponse(BaseModel):
    status: str
    question: str
    answer: str
    sources: List[Dict]
    quality_metrics: Dict
    retrieval_strategy: str
    documents_retrieved: int
    retrieval_time: float
    generation_time: float
    total_time: float
    tokens_used: int
    error: Optional[str] = None

class IngestionRequest(BaseModel):
    document_path: str = Field(..., description="Path to document or directory")
    is_directory: bool = Field(True, description="Whether the path is a directory")

class IngestionResponse(BaseModel):
    status: str
    message: str
    chunks_processed: int
    processing_time: float

class SystemStats(BaseModel):
    vector_store: Dict
    config: Dict
    status: str

# API Routes
@app.get("/")
async def root():
    """Root endpoint with API information"""
    return {
        "message": "RAG System API",
        "version": "1.0.0",
        "status": "running",
        "endpoints": {
            "query": "/query",
            "ingest": "/ingest",
            "stats": "/stats",
            "health": "/health"
        }
    }

@app.post("/query", response_model=QueryResponse)
async def query_documents(request: QueryRequest):
    """Query the RAG system"""
    try:
        logger.info(f"Received query: {request.question}")
        
        result = rag_pipeline.query(
            question=request.question,
            retrieval_strategy=request.retrieval_strategy,
            prompt_type=request.prompt_type,
            include_citations=request.include_citations,
            top_k=request.top_k
        )
        
        return QueryResponse(**result)
        
    except Exception as e:
        logger.error(f"Error processing query: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.post("/ingest", response_model=IngestionResponse)
async def ingest_documents(request: IngestionRequest, background_tasks: BackgroundTasks):
    """Ingest documents into the RAG system"""
    try:
        logger.info(f"Starting document ingestion: {request.document_path}")
        
        # For large document sets, you might want to run this in background
        result = rag_pipeline.ingest_documents(
            document_path=request.document_path,
            is_directory=request.is_directory
        )
        
        return IngestionResponse(**result)
        
    except Exception as e:
        logger.error(f"Error during ingestion: {e}")
        raise HTTPException(status_code=500, detail=str(e))

```python
@app.get("/stats", response_model=SystemStats)
async def get_system_stats():
    """Get system statistics"""
    try:
        stats = rag_pipeline.get_system_stats()
        return SystemStats(**stats)
        
    except Exception as e:
        logger.error(f"Error getting system stats: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/health")
async def health_check():
    """Health check endpoint"""
    try:
        stats = rag_pipeline.get_system_stats()
        return {
            "status": "healthy",
            "timestamp": time.time(),
            "components": {
                "vector_store": "operational",
                "generator": "operational",
                "retriever": "operational"
            },
            "document_count": stats.get("vector_store", {}).get("total_documents", 0)
        }
    except Exception as e:
        logger.error(f"Health check failed: {e}")
        return {
            "status": "unhealthy",
            "error": str(e),
            "timestamp": time.time()
        }

@app.post("/reset")
async def reset_system():
    """Reset the entire RAG system"""
    try:
        result = rag_pipeline.reset_system()
        return result
    except Exception as e:
        logger.error(f"Error resetting system: {e}")
        raise HTTPException(status_code=500, detail=str(e))

@app.get("/search")
async def search_documents(query: str, top_k: int = 5):
    """Search documents without generating response"""
    try:
        if not rag_pipeline.retriever:
            raise HTTPException(status_code=503, detail="RAG system not initialized")
        
        results = rag_pipeline.retriever.retrieve_documents(query, top_k)
        
        return {
            "query": query,
            "results": results,
            "count": len(results)
        }
        
    except Exception as e:
        logger.error(f"Error searching documents: {e}")
        raise HTTPException(status_code=500, detail=str(e))

if __name__ == "__main__":
    import time
    
    # Run the API server
    uvicorn.run(
        "api:app",
        host="0.0.0.0",
        port=8000,
        reload=True,
        log_level="info"
    )
```

### **10. Web Interface (`src/web_interface.py`)**

```python
# src/web_interface.py
import streamlit as st
import requests
import json
import time
from typing import Dict, List

# Configure Streamlit page
st.set_page_config(
    page_title="RAG Customer Support System",
    page_icon="🤖",
    layout="wide",
    initial_sidebar_state="expanded"
)

# API Configuration
API_BASE_URL = "http://localhost:8000"

def call_api(endpoint: str, method: str = "GET", data: Dict = None) -> Dict:
    """Make API calls to the RAG system"""
    try:
        url = f"{API_BASE_URL}{endpoint}"
        
        if method == "GET":
            response = requests.get(url)
        elif method == "POST":
            response = requests.post(url, json=data)
        else:
            raise ValueError(f"Unsupported method: {method}")
        
        response.raise_for_status()
        return response.json()
        
    except requests.exceptions.ConnectionError:
        st.error("❌ Cannot connect to RAG API. Please ensure the API server is running on port 8000.")
        return {"error": "Connection failed"}
    except requests.exceptions.RequestException as e:
        st.error(f"❌ API Error: {str(e)}")
        return {"error": str(e)}

def display_sources(sources: List[Dict]):
    """Display source documents in a formatted way"""
    if not sources:
        return
    
    st.subheader("📚 Sources")
    
    for i, source in enumerate(sources):
        with st.expander(f"Source {source['number']}: {source['file_name']}"):
            col1, col2 = st.columns([3, 1])
            
            with col1:
                st.write(f"**File:** {source['file_name']}")
            
            with col2:
                similarity = source.get('similarity_score', 0)
                st.metric("Similarity", f"{similarity:.3f}")

def display_metrics(metrics: Dict, response_data: Dict):
    """Display response quality metrics"""
    st.subheader("📊 Response Metrics")
    
    col1, col2, col3, col4 = st.columns(4)
    
    with col1:
        quality_score = metrics.get('overall', 0)
        st.metric(
            "Quality Score", 
            f"{quality_score:.3f}",
            delta=None,
            help="Overall response quality (0-1)"
        )
    
    with col2:
        st.metric(
            "Documents Used", 
            response_data.get('documents_retrieved', 0),
            help="Number of documents retrieved"
        )
    
    with col3:
        st.metric(
            "Response Time", 
            f"{response_data.get('total_time', 0):.2f}s",
            help="Total processing time"
        )
    
    with col4:
        st.metric(
            "Tokens Used", 
            response_data.get('tokens_used', 0),
            help="LLM tokens consumed"
        )
    
    # Detailed metrics
    with st.expander("Detailed Metrics"):
        col1, col2, col3 = st.columns(3)
        
        with col1:
            st.metric("Relevance", f"{metrics.get('relevance', 0):.3f}")
        with col2:
            st.metric("Groundedness", f"{metrics.get('groundedness', 0):.3f}")
        with col3:
            st.metric("Completeness", f"{metrics.get('completeness', 0):.3f}")

def main():
    """Main Streamlit application"""
    
    # Header
    st.title("🤖 RAG Customer Support System")
    st.markdown("Ask questions about our products and services!")
    
    # Sidebar
    with st.sidebar:
        st.header("⚙️ Settings")
        
        # Query settings
        st.subheader("Query Configuration")
        
        retrieval_strategy = st.selectbox(
            "Retrieval Strategy",
            ["similarity", "hybrid", "rerank"],
            help="Choose how documents are retrieved"
        )
        
        prompt_type = st.selectbox(
            "Response Type",
            ["customer_support", "technical", "general"],
            help="Choose the response style"
        )
        
        include_citations = st.checkbox(
            "Include Citations",
            value=True,
            help="Include source references in responses"
        )
        
        top_k = st.slider(
            "Documents to Retrieve",
            min_value=1,
            max_value=10,
            value=5,
            help="Number of relevant documents to use"
        )
        
        st.divider()
        
        # System status
        st.subheader("System Status")
        
        if st.button("🔄 Check Status"):
            with st.spinner("Checking system status..."):
                health_data = call_api("/health")
                
                if "error" not in health_data:
                    st.success("✅ System Healthy")
                    st.json(health_data)
                else:
                    st.error("❌ System Unhealthy")
        
        # System stats
        if st.button("📊 View Stats"):
            with st.spinner("Loading system statistics..."):
                stats_data = call_api("/stats")
                
                if "error" not in stats_data:
                    st.success("📈 System Statistics")
                    
                    # Document count
                    doc_count = stats_data.get("vector_store", {}).get("total_documents", 0)
                    st.metric("Total Documents", doc_count)
                    
                    # Configuration
                    with st.expander("Configuration"):
                        config_data = stats_data.get("config", {})
                        for key, value in config_data.items():
                            st.write(f"**{key}:** {value}")
                else:
                    st.error("Failed to load statistics")
        
        st.divider()
        
        # Document ingestion
        st.subheader("📁 Document Management")
        
        with st.expander("Ingest Documents"):
            document_path = st.text_input(
                "Document Path",
                value="./data/documents",
                help="Path to document or directory"
            )
            
            is_directory = st.checkbox("Is Directory", value=True)
            
            if st.button("📤 Ingest Documents"):
                with st.spinner("Ingesting documents..."):
                    ingest_data = call_api(
                        "/ingest",
                        method="POST",
                        data={
                            "document_path": document_path,
                            "is_directory": is_directory
                        }
                    )
                    
                    if "error" not in ingest_data:
                        st.success(f"✅ {ingest_data['message']}")
                        st.write(f"Processed: {ingest_data['chunks_processed']} chunks")
                        st.write(f"Time: {ingest_data['processing_time']:.2f}s")
                    else:
                        st.error("❌ Ingestion failed")
        
        # System reset
        with st.expander("⚠️ System Reset"):
            st.warning("This will delete all documents and reset the system!")
            
            if st.button("🗑️ Reset System", type="secondary"):
                with st.spinner("Resetting system..."):
                    reset_data = call_api("/reset", method="POST")
                    
                    if reset_data.get("status") == "success":
                        st.success("✅ System reset successfully")
                    else:
                        st.error("❌ Reset failed")
    
    # Main content area
    col1, col2 = st.columns([2, 1])
    
    with col1:
        # Chat interface
        st.subheader("💬 Ask a Question")
        
        # Initialize chat history
        if "messages" not in st.session_state:
            st.session_state.messages = []
        
        # Display chat history
        for message in st.session_state.messages:
            with st.chat_message(message["role"]):
                st.markdown(message["content"])
                
                if message["role"] == "assistant" and "metadata" in message:
                    # Display sources and metrics for assistant responses
                    metadata = message["metadata"]
                    
                    if metadata.get("sources"):
                        display_sources(metadata["sources"])
                    
                    if metadata.get("quality_metrics"):
                        display_metrics(metadata["quality_metrics"], metadata)
        
        # Chat input
        if prompt := st.chat_input("Ask your question here..."):
            # Add user message to chat history
            st.session_state.messages.append({"role": "user", "content": prompt})
            
            # Display user message
            with st.chat_message("user"):
                st.markdown(prompt)
            
            # Generate assistant response
            with st.chat_message("assistant"):
                with st.spinner("Thinking..."):
                    # Call RAG API
                    response_data = call_api(
                        "/query",
                        method="POST",
                        data={
                            "question": prompt,
                            "retrieval_strategy": retrieval_strategy,
                            "prompt_type": prompt_type,
                            "include_citations": include_citations,
                            "top_k": top_k
                        }
                    )
                    
                    if "error" not in response_data:
                        # Display response
                        st.markdown(response_data["answer"])
                        
                        # Store assistant message with metadata
                        st.session_state.messages.append({
                            "role": "assistant",
                            "content": response_data["answer"],
                            "metadata": response_data
                        })
                        
                        # Display sources and metrics
                        if response_data.get("sources"):
                            display_sources(response_data["sources"])
                        
                        if response_data.get("quality_metrics"):
                            display_metrics(response_data["quality_metrics"], response_data)
                    
                    else:
                        error_message = "I apologize, but I encountered an error processing your question. Please try again."
                        st.error(error_message)
                        
                        st.session_state.messages.append({
                            "role": "assistant",
                            "content": error_message
                        })
    
    with col2:
        # Quick actions and examples
        st.subheader("🚀 Quick Start")
        
        # Example questions
        st.write("**Try these example questions:**")
        
        example_questions = [
            "How do I install the software?",
            "What are the system requirements?",
            "How is billing calculated?",
            "What API endpoints are available?",
            "How do I reset my password?",
            "What support options are available?"
        ]
        
        for question in example_questions:
            if st.button(question, key=f"example_{hash(question)}"):
                # Add to chat input (simulate user input)
                st.session_state.messages.append({"role": "user", "content": question})
                st.rerun()
        
        st.divider()
        
        # Search functionality
        st.subheader("🔍 Document Search")
        
        search_query = st.text_input("Search documents:", placeholder="Enter search terms...")
        search_top_k = st.slider("Results to show:", 1, 10, 3)
        
        if st.button("🔍 Search") and search_query:
            with st.spinner("Searching..."):
                search_results = call_api(f"/search?query={search_query}&top_k={search_top_k}")
                
                if "error" not in search_results:
                    st.success(f"Found {search_results['count']} results")
                    
                    for i, result in enumerate(search_results["results"]):
                        with st.expander(f"Result {i+1} - {result['metadata']['file_name']}"):
                            st.write(f"**Similarity:** {result['similarity_score']:.3f}")
                            st.write(f"**Content:** {result['content'][:200]}...")
                else:
                    st.error("Search failed")
        
        st.divider()
        
        # Clear chat history
        if st.button("🗑️ Clear Chat History"):
            st.session_state.messages = []
            st.rerun()

if __name__ == "__main__":
    main()
```

### **11. Testing Suite (`tests/test_rag_system.py`)**

```python
# tests/test_rag_system.py
import pytest
import os
import tempfile
import shutil
from unittest.mock import Mock, patch

# Import our modules
import sys
sys.path.append('../src')

from document_processor import DocumentProcessor
from vector_store import VectorStore
from retriever import DocumentRetriever
from generator import ResponseGenerator
from rag_pipeline import RAGPipeline

class TestDocumentProcessor:
    """Test document processing functionality"""
    
    def setup_method(self):
        """Setup test environment"""
        self.temp_dir = tempfile.mkdtemp()
        self.processor = DocumentProcessor()
    
    def teardown_method(self):
        """Cleanup test environment"""
        shutil.rmtree(self.temp_dir)
    
    def test_text_processing(self):
        """Test basic text processing"""
        # Create
        
