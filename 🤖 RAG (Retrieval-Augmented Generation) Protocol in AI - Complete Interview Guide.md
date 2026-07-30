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
                    "metadata":
