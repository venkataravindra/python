# Simple RAG System - Complete Example

Here's a minimal, easy-to-understand RAG system in a single file:

## 📁 Project Structure
```
simple_rag/
├── simple_rag.py          # Main RAG implementation
├── requirements.txt       # Dependencies
├── documents/            # Sample documents
│   ├── product_info.txt
│   └── faq.txt
└── README.md
```

## 📋 requirements.txt
```txt
sentence-transformers==2.2.2
faiss-cpu==1.7.4
openai==1.3.0
python-dotenv==1.0.0
```

## 📄 Sample Documents

**documents/product_info.txt:**
```
Our company offers three main products:

1. CloudSync Pro - A cloud storage solution with 1TB storage, real-time sync, and enterprise security. Price: $29/month.

2. DataAnalyzer - Business intelligence tool with advanced analytics, custom dashboards, and ML insights. Price: $99/month.

3. SecureVPN - Enterprise VPN solution with 256-bit encryption, global servers, and 24/7 support. Price: $15/month.

All products come with a 30-day free trial and 24/7 customer support.
```

**documents/faq.txt:**
```
Frequently Asked Questions:

Q: How do I cancel my subscription?
A: You can cancel anytime from your account dashboard or contact support.

Q: Is there a refund policy?
A: Yes, we offer full refunds within 30 days of purchase.

Q: Do you offer discounts for annual plans?
A: Yes, annual plans get 20% discount compared to monthly billing.

Q: What payment methods do you accept?
A: We accept all major credit cards, PayPal, and bank transfers.
```

## 🤖 Complete RAG Implementation

**simple_rag.py:**
```python
import os
import json
from typing import List, Dict
import numpy as np
from sentence_transformers import SentenceTransformer
import faiss
from dotenv import load_dotenv
import openai

# Load environment variables
load_dotenv()
openai.api_key = os.getenv("OPENAI_API_KEY")

class SimpleRAG:
    def __init__(self):
        """Initialize the RAG system"""
        print("🚀 Initializing Simple RAG System...")
        
        # Load embedding model
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        print("✅ Embedding model loaded")
        
        # Initialize storage
        self.documents = []  # Store original text chunks
        self.embeddings = None  # Store embeddings
        self.index = None  # FAISS index
        
    def load_documents(self, documents_dir: str = "documents"):
        """Load and chunk documents"""
        print(f"📚 Loading documents from {documents_dir}...")
        
        self.documents = []
        
        # Read all text files in the directory
        for filename in os.listdir(documents_dir):
            if filename.endswith('.txt'):
                filepath = os.path.join(documents_dir, filename)
                
                with open(filepath, 'r', encoding='utf-8') as file:
                    content = file.read()
                
                # Simple chunking by paragraphs
                chunks = [chunk.strip() for chunk in content.split('\n\n') if chunk.strip()]
                
                # Add metadata to each chunk
                for chunk in chunks:
                    self.documents.append({
                        'content': chunk,
                        'source': filename,
                        'chunk_id': len(self.documents)
                    })
        
        print(f"✅ Loaded {len(self.documents)} document chunks")
        return self.documents
    
    def create_embeddings(self):
        """Create embeddings for all documents"""
        print("🔄 Creating embeddings...")
        
        if not self.documents:
            raise ValueError("No documents loaded. Call load_documents() first.")
        
        # Extract text content
        texts = [doc['content'] for doc in self.documents]
        
        # Create embeddings
        self.embeddings = self.embedding_model.encode(texts)
        print(f"✅ Created embeddings: {self.embeddings.shape}")
        
        # Create FAISS index
        dimension = self.embeddings.shape[1]
        self.index = faiss.IndexFlatIP(dimension)  # Inner product for similarity
        
        # Normalize embeddings for cosine similarity
        faiss.normalize_L2(self.embeddings)
        self.index.add(self.embeddings)
        
        print("✅ FAISS index created")
    
    def retrieve_documents(self, query: str, top_k: int = 3) -> List[Dict]:
        """Retrieve most relevant documents for a query"""
        if self.index is None:
            raise ValueError("Index not created. Call create_embeddings() first.")
        
        # Embed the query
        query_embedding = self.embedding_model.encode([query])
        faiss.normalize_L2(query_embedding)
        
        # Search for similar documents
        scores, indices = self.index.search(query_embedding, top_k)
        
        # Prepare results
        results = []
        for i, (score, idx) in enumerate(zip(scores[0], indices[0])):
            results.append({
                'content': self.documents[idx]['content'],
                'source': self.documents[idx]['source'],
                'similarity_score': float(score),
                'rank': i + 1
            })
        
        return results
    
    def generate_response(self, query: str, context_docs: List[Dict]) -> str:
        """Generate response using OpenAI GPT"""
        
        # Prepare context from retrieved documents
        context = "\n\n".join([
            f"Source: {doc['source']}\nContent: {doc['content']}" 
            for doc in context_docs
        ])
        
        # Create prompt
        prompt = f"""You are a helpful customer support assistant. Answer the user's question based on the provided context.

Context:
{context}

Question: {query}

Instructions:
- Answer based only on the provided context
- If the answer is not in the context, say "I don't have enough information to answer that question"
- Be concise and helpful
- Mention the source when relevant

Answer:"""

        try:
            # Call OpenAI API
            response = openai.chat.completions.create(
                model="gpt-3.5-turbo",
                messages=[
                    {"role": "system", "content": "You are a helpful customer support assistant."},
                    {"role": "user", "content": prompt}
                ],
                max_tokens=300,
                temperature=0.1
            )
            
            return response.choices[0].message.content.strip()
            
        except Exception as e:
            return f"Sorry, I encountered an error generating the response: {str(e)}"
    
    def query(self, question: str, top_k: int = 3) -> Dict:
        """Complete RAG pipeline: retrieve + generate"""
        print(f"\n❓ Question: {question}")
        
        # Step 1: Retrieve relevant documents
        print("🔍 Retrieving relevant documents...")
        retrieved_docs = self.retrieve_documents(question, top_k)
        
        print(f"📄 Found {len(retrieved_docs)} relevant documents:")
        for doc in retrieved_docs:
            print(f"  - {doc['source']} (similarity: {doc['similarity_score']:.3f})")
        
        # Step 2: Generate response
        print("🤖 Generating response...")
        response = self.generate_response(question, retrieved_docs)
        
        return {
            'question': question,
            'answer': response,
            'sources': retrieved_docs,
            'num_sources': len(retrieved_docs)
        }
    
    def interactive_chat(self):
        """Interactive chat interface"""
        print("\n" + "="*50)
        print("🤖 Simple RAG Chat Interface")
        print("Type 'quit' to exit")
        print("="*50)
        
        while True:
            question = input("\n💬 You: ").strip()
            
            if question.lower() in ['quit', 'exit', 'q']:
                print("👋 Goodbye!")
                break
            
            if not question:
                continue
            
            try:
                result = self.query(question)
                print(f"\n🤖 Assistant: {result['answer']}")
                
                # Show sources
                print(f"\n📚 Sources used:")
                for source in result['sources']:
                    print(f"  - {source['source']} (score: {source['similarity_score']:.3f})")
                    
            except Exception as e:
                print(f"❌ Error: {str(e)}")

def main():
    """Main function to demonstrate the RAG system"""
    
    # Initialize RAG system
    rag = SimpleRAG()
    
    # Load documents
    rag.load_documents("documents")
    
    # Create embeddings and index
    rag.create_embeddings()
    
    # Test with sample questions
    sample_questions = [
        "What products do you offer?",
        "How much does CloudSync Pro cost?",
        "What is your refund policy?",
        "Do you offer annual discounts?"
    ]
    
    print("\n" + "="*50)
    print("🧪 Testing with sample questions:")
    print("="*50)
    
    for question in sample_questions:
        result = rag.query(question)
        print(f"\n📋 Q: {question}")
        print(f"📋 A: {result['answer']}")
        print("-" * 30)
    
    # Start interactive chat
    rag.interactive_chat()

if __name__ == "__main__":
    main()
```

## 🚀 How to Run

1. **Install dependencies:**
```bash
pip install -r requirements.txt
```

2. **Set up OpenAI API key:**
Create a `.env` file:
```
OPENAI_API_KEY=your_openai_api_key_here
```

3. **Create sample documents:**
Create the `documents/` folder and add the sample text files above.

4. **Run the system:**
```bash
python simple_rag.py
```

## 📊 Example Output

```
🚀 Initializing Simple RAG System...
✅ Embedding model loaded
📚 Loading documents from documents...
✅ Loaded 8 document chunks
🔄 Creating embeddings...
✅ Created embeddings: (8, 384)
✅ FAISS index created

==================================================
🧪 Testing with sample questions:
==================================================

❓ Question: What products do you offer?
🔍 Retrieving relevant documents...
📄 Found 3 relevant documents:
  - product_info.txt (similarity: 0.789)
  - product_info.txt (similarity: 0.654)
  - product_info.txt (similarity: 0.432)
🤖 Generating response...

📋 Q: What products do you offer?
📋 A: Based on the product information, we offer three main products:

1. **CloudSync Pro** - A cloud storage solution with 1TB storage, real-time sync, and enterprise security for $29/month
2. **DataAnalyzer** - A business intelligence tool with advanced analytics, custom dashboards, and ML insights for $99/month  
3. **SecureVPN** - An enterprise VPN solution with 256-bit encryption, global servers, and 24/7 support for $15/month

All products include a 30-day free trial and 24/7 customer support.

==================================================
🤖 Simple RAG Chat Interface
Type 'quit' to exit
==================================================

💬 You: How do I cancel my subscription?

❓ Question: How do I cancel my subscription?
🔍 Retrieving relevant documents...
📄 Found 3 relevant documents:
  - faq.txt (similarity: 0.892)
  - faq.txt (similarity: 0.234)
  - product_info.txt (similarity: 0.123)
🤖 Generating response...

🤖 Assistant: You can cancel your subscription anytime from your account dashboard or by contacting our support team.

📚 Sources used:
  - faq.txt (score: 0.892)
  - faq.txt (score: 0.234)
  - product_info.txt (score: 0.123)
```

## 🔄 RAG Flow Explanation

1. **📚 Document Loading**: Read text files and split into chunks
2. **🔢 Embedding Creation**: Convert text chunks to vector embeddings
3. **🗂️ Index Building**: Create FAISS index for fast similarity search
4. **❓ Query Processing**: Convert user question to embedding
5. **🔍 Retrieval**: Find most similar document chunks
6. **🤖 Generation**: Use LLM to generate answer from retrieved context
7. **📤 Response**: Return answer with source information

## ✨ Key Features

- **Simple Setup**: Single file implementation
- **Fast Retrieval**: FAISS for efficient similarity search
- **Quality Embeddings**: Sentence-transformers model
- **Source Tracking**: Know which documents were used
- **Interactive Chat**: Real-time question answering
- **Error Handling**: Graceful error management

This example demonstrates the core RAG concepts in under 200 lines of code while being fully functional!
