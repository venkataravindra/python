# Enhanced Multi-Modal Tokenizer

I'll help you create an enhanced tokenizer that can handle multiple file types. Here's a comprehensive solution:

## 📦 **Required Dependencies**

First, install the required packages:

```bash
pip install PyPDF2 pandas python-docx pillow opencv-python pytesseract speechrecognition moviepy
```

## 🔧 **Enhanced tokenizer.py**

```python
"""
enhanced_tokenizer.py

Purpose:
--------
Enhanced tokenizer that can process multiple file types:
- Text files (.txt)
- PDF files (.pdf)
- Word documents (.docx)
- Excel files (.xlsx, .xls)
- Images (.jpg, .png, .jpeg, .bmp) - OCR
- Videos (.mp4, .avi, .mov) - Audio extraction + Speech-to-text

Features:
1. Auto file type detection
2. Text extraction from various formats
3. OCR for images
4. Speech-to-text for videos
5. Unified text processing pipeline
"""

import os
import pandas as pd
from pathlib import Path
import warnings
warnings.filterwarnings("ignore")

# File processing imports
try:
    import PyPDF2
    PDF_AVAILABLE = True
except ImportError:
    PDF_AVAILABLE = False
    print("Warning: PyPDF2 not installed. PDF processing disabled.")

try:
    from docx import Document
    DOCX_AVAILABLE = True
except ImportError:
    DOCX_AVAILABLE = False
    print("Warning: python-docx not installed. Word document processing disabled.")

try:
    from PIL import Image
    import pytesseract
    OCR_AVAILABLE = True
except ImportError:
    OCR_AVAILABLE = False
    print("Warning: PIL/pytesseract not installed. Image OCR disabled.")

try:
    from moviepy.editor import VideoFileClip
    import speech_recognition as sr
    VIDEO_AVAILABLE = True
except ImportError:
    VIDEO_AVAILABLE = False
    print("Warning: moviepy/speech_recognition not installed. Video processing disabled.")


class EnhancedTokenizer:
    """
    Multi-modal tokenizer for various file types
    """
    
    def __init__(self, data_path):
        self.data_path = data_path
        self.supported_extensions = {
            'text': ['.txt'],
            'pdf': ['.pdf'],
            'word': ['.docx', '.doc'],
            'excel': ['.xlsx', '.xls', '.csv'],
            'image': ['.jpg', '.jpeg', '.png', '.bmp', '.tiff'],
            'video': ['.mp4', '.avi', '.mov', '.mkv', '.wmv']
        }
        
        # Initialize variables
        self.raw_text = ""
        self.tokens = []
        self.vocab = {}
        self.reverse_vocab = {}
        self.encoded_tokens = []
        
    def detect_file_type(self, file_path):
        """Detect file type based on extension"""
        extension = Path(file_path).suffix.lower()
        
        for file_type, extensions in self.supported_extensions.items():
            if extension in extensions:
                return file_type
        
        return 'unknown'
    
    def extract_text_from_txt(self, file_path):
        """Extract text from .txt files"""
        try:
            with open(file_path, 'r', encoding='utf-8') as file:
                return file.read()
        except Exception as e:
            print(f"Error reading text file: {e}")
            return ""
    
    def extract_text_from_pdf(self, file_path):
        """Extract text from PDF files"""
        if not PDF_AVAILABLE:
            print("PDF processing not available. Install PyPDF2.")
            return ""
        
        try:
            text = ""
            with open(file_path, 'rb') as file:
                pdf_reader = PyPDF2.PdfReader(file)
                for page in pdf_reader.pages:
                    text += page.extract_text() + "\n"
            return text
        except Exception as e:
            print(f"Error reading PDF file: {e}")
            return ""
    
    def extract_text_from_word(self, file_path):
        """Extract text from Word documents"""
        if not DOCX_AVAILABLE:
            print("Word document processing not available. Install python-docx.")
            return ""
        
        try:
            doc = Document(file_path)
            text = ""
            for paragraph in doc.paragraphs:
                text += paragraph.text + "\n"
            return text
        except Exception as e:
            print(f"Error reading Word document: {e}")
            return ""
    
    def extract_text_from_excel(self, file_path):
        """Extract text from Excel files"""
        try:
            # Read all sheets
            if file_path.endswith('.csv'):
                df = pd.read_csv(file_path)
                text = df.to_string(index=False)
            else:
                excel_file = pd.ExcelFile(file_path)
                text = ""
                for sheet_name in excel_file.sheet_names:
                    df = pd.read_excel(file_path, sheet_name=sheet_name)
                    text += f"Sheet: {sheet_name}\n"
                    text += df.to_string(index=False) + "\n\n"
            
            return text
        except Exception as e:
            print(f"Error reading Excel file: {e}")
            return ""
    
    def extract_text_from_image(self, file_path):
        """Extract text from images using OCR"""
        if not OCR_AVAILABLE:
            print("Image OCR not available. Install PIL and pytesseract.")
            return ""
        
        try:
            image = Image.open(file_path)
            text = pytesseract.image_to_string(image)
            return text
        except Exception as e:
            print(f"Error processing image: {e}")
            return ""
    
    def extract_text_from_video(self, file_path):
        """Extract text from video using speech-to-text"""
        if not VIDEO_AVAILABLE:
            print("Video processing not available. Install moviepy and speech_recognition.")
            return ""
        
        try:
            # Extract audio from video
            video = VideoFileClip(file_path)
            audio_path = "temp_audio.wav"
            video.audio.write_audiofile(audio_path, verbose=False, logger=None)
            
            # Convert speech to text
            recognizer = sr.Recognizer()
            with sr.AudioFile(audio_path) as source:
                audio = recognizer.record(source)
                text = recognizer.recognize_google(audio)
            
            # Clean up temporary file
            if os.path.exists(audio_path):
                os.remove(audio_path)
            
            video.close()
            return text
            
        except Exception as e:
            print(f"Error processing video: {e}")
            return ""
    
    def process_file(self, file_path):
        """Process a single file and extract text"""
        file_type = self.detect_file_type(file_path)
        
        print(f"Processing {file_type} file: {file_path}")
        
        if file_type == 'text':
            return self.extract_text_from_txt(file_path)
        elif file_type == 'pdf':
            return self.extract_text_from_pdf(file_path)
        elif file_type == 'word':
            return self.extract_text_from_word(file_path)
        elif file_type == 'excel':
            return self.extract_text_from_excel(file_path)
        elif file_type == 'image':
            return self.extract_text_from_image(file_path)
        elif file_type == 'video':
            return self.extract_text_from_video(file_path)
        else:
            print(f"Unsupported file type: {file_type}")
            return ""
    
    def process_directory(self, directory_path):
        """Process all supported files in a directory"""
        all_text = ""
        
        for root, dirs, files in os.walk(directory_path):
            for file in files:
                file_path = os.path.join(root, file)
                text = self.process_file(file_path)
                all_text += text + "\n"
        
        return all_text
    
    def build_vocabulary(self):
        """Build vocabulary from extracted text"""
        # Convert to lowercase
        self.raw_text = self.raw_text.lower()
        
        # Tokenization
        self.tokens = self.raw_text.split()
        
        # Remove empty tokens
        self.tokens = [token for token in self.tokens if token.strip()]
        
        # Create vocabulary
        unique_words = sorted(set(self.tokens))
        
        self.vocab = {}
        for index, word in enumerate(unique_words):
            self.vocab[word] = index
        
        # Create reverse vocabulary
        self.reverse_vocab = {}
        for word, index in self.vocab.items():
            self.reverse_vocab[index] = word
        
        # Encode tokens
        self.encoded_tokens = []
        for word in self.tokens:
            self.encoded_tokens.append(self.vocab[word])
    
    def process(self):
        """Main processing function"""
        if os.path.isfile(self.data_path):
            # Single file
            self.raw_text = self.process_file(self.data_path)
        elif os.path.isdir(self.data_path):
            # Directory
            self.raw_text = self.process_directory(self.data_path)
        else:
            print(f"Path not found: {self.data_path}")
            return
        
        # Build vocabulary
        self.build_vocabulary()
        
        # Display results
        self.display_results()
    
    def display_results(self):
        """Display processing results"""
        print("=" * 50)
        print("ENHANCED TOKENIZER RESULTS")
        print("=" * 50)
        
        print(f"Total characters: {len(self.raw_text)}")
        print(f"Total tokens: {len(self.tokens)}")
        print(f"Vocabulary size: {len(self.vocab)}")
        
        print("\n" + "=" * 50)
        print("SAMPLE TEXT (first 500 characters)")
        print("=" * 50)
        print(self.raw_text[:500] + "..." if len(self.raw_text) > 500 else self.raw_text)
        
        print("\n" + "=" * 50)
        print("VOCABULARY (first 20 words)")
        print("=" * 50)
        vocab_items = list(self.vocab.items())[:20]
        for word, idx in vocab_items:
            print(f"{word}: {idx}")
        
        if len(self.vocab) > 20:
            print(f"... and {len(self.vocab) - 20} more words")


# -------------------------
# Usage Example
# -------------------------

if __name__ == "__main__":
    
    # Initialize tokenizer
    # You can pass a single file or directory path
    CURRENT_DIR = os.path.dirname(__file__)
    
    # Option 1: Single file
    # DATA_PATH = os.path.join(CURRENT_DIR, "data.txt")
    
    # Option 2: Directory with multiple files
    DATA_PATH = os.path.join(CURRENT_DIR, "data")  # Create a 'data' folder
    
    # Option 3: Specific file
    # DATA_PATH = "path/to/your/document.pdf"
    
    tokenizer = EnhancedTokenizer(DATA_PATH)
    tokenizer.process()
    
    # Export for use in other modules
    vocab = tokenizer.vocab
    reverse_vocab = tokenizer.reverse_vocab
    encoded_tokens = tokenizer.encoded_tokens
    tokens = tokenizer.tokens
```

## 📁 **Updated File Structure**

```
your_project/
├── data/                    # New data directory
│   ├── text_files/
│   │   ├── data.txt
│   │   └── more_text.txt
│   ├── documents/
│   │   ├── document.pdf
│   │   ├── report.docx
│   │   └── data.xlsx
│   ├── images/
│   │   ├── text_image.jpg
│   │   └── screenshot.png
│   └── videos/
│       └── presentation.mp4
├── enhanced_tokenizer.py   # New enhanced tokenizer
├── dataset.py
├── model.py
├── train.py
└── predict.py
```

## 🔄 **Updated dataset.py**

```python
"""
Updated dataset.py to work with enhanced tokenizer
"""

# Import enhanced tokenizer
from enhanced_tokenizer import encoded_tokens

# Rest of the code remains the same
inputs = []
outputs = []

# Loop through tokens
for i in range(len(encoded_tokens) - 1):
    input_sequence = encoded_tokens[:i + 1]
    target = encoded_tokens[i + 1]
    inputs.append(input_sequence)
    outputs.append(target)

# Print Dataset
print("=" * 50)
print("INPUTS")
print("=" * 50)
for item in inputs[:10]:  # Show first 10 only
    print(item)

print("\n" + "=" * 50)
print("OUTPUTS")
print("=" * 50)
for item in outputs[:10]:  # Show first 10 only
    print(item)

print(f"\nTotal training examples: {len(inputs)}")
```

## 🔄 **Updated train.py and predict.py**

```python
# In train.py, change the import:
from enhanced_tokenizer import vocab

# In predict.py, change the import:
from enhanced_tokenizer import vocab, reverse_vocab
```

## 🚀 **Usage Examples**

### **1. Process Single File**
```python
# PDF file
tokenizer = EnhancedTokenizer("document.pdf")
tokenizer.process()

# Image with text
tokenizer = EnhancedTokenizer("text_image.jpg")
tokenizer.process()

# Video file
tokenizer = EnhancedTokenizer("presentation.mp4")
tokenizer.process()
```

### **2. Process Directory**
```python
# Process all files in data directory
tokenizer = EnhancedTokenizer("./data")
tokenizer.process()
```

### **3. Process Multiple File Types**
```python
# Create data directory with mixed file types
data_dir = "./mixed_data"
tokenizer = EnhancedTokenizer(data_dir)
tokenizer.process()
```

## 🎯 **Key Features**

| Feature | Description |
|---------|-------------|
| **Auto Detection** | Automatically detects file type by extension |
| **PDF Support** | Extracts text from PDF documents |
| **Word Documents** | Processes .docx files |
| **Excel/CSV** | Extracts data from spreadsheets |
| **Image OCR** | Converts text in images to readable text |
| **Video Processing** | Extracts speech from videos and converts to text |
| **Directory Processing** | Processes all supported files in a folder |
| **Error Handling** | Graceful handling of unsupported files |

## ⚠️ **Important Notes**

1. **OCR Setup**: For image processing, you need to install Tesseract OCR:
   - Windows: Download from [GitHub](https://github.com/UB-Mannheim/tesseract/wiki)
   - Mac: `brew install tesseract`
   - Linux: `sudo apt-get install tesseract-ocr`

2. **Video Processing**: Requires internet connection for Google Speech Recognition

3. **Large Files**: Video and image processing can be slow for large files

4. **Dependencies**: Install only the packages you need based on your file types

This enhanced tokenizer will significantly expand your LLM's ability to learn from diverse data sources! 🚀
