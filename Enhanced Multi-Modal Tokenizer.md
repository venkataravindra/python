# Complete Enhanced Multi-Modal Tokenizer with URL Support

Let me complete the code and provide the full file for future reference:

## 🔧 **Complete enhanced_multimodal_tokenizer.py**

```python
"""
enhanced_multimodal_tokenizer.py

Purpose:
--------
Advanced tokenizer that can process multiple data sources:
- URLs (web scraping, YouTube videos, online documents)
- Text files (.txt)
- PDF files (.pdf)
- Word documents (.docx)
- Excel files (.xlsx, .xls, .csv)
- Images (.jpg, .png, .jpeg, .bmp) - OCR
- Videos (.mp4, .avi, .mov) - Audio extraction + Speech-to-text
- Web content (HTML scraping)
- YouTube videos (audio extraction)

Features:
1. Auto source type detection
2. Web scraping capabilities
3. YouTube video processing
4. OCR for images
5. Speech-to-text for videos
6. Unified text processing pipeline

Author: AI Assistant
Date: 2024
Version: 2.0
"""

import os
import re
import json
import requests
import pandas as pd
from pathlib import Path
from urllib.parse import urlparse, urljoin
from collections import Counter
import warnings
warnings.filterwarnings("ignore")

# Web scraping imports
try:
    from bs4 import BeautifulSoup
    import requests
    WEB_SCRAPING_AVAILABLE = True
except ImportError:
    WEB_SCRAPING_AVAILABLE = False
    print("Warning: BeautifulSoup/requests not installed. Web scraping disabled.")

# YouTube processing
try:
    import yt_dlp
    YOUTUBE_AVAILABLE = True
except ImportError:
    YOUTUBE_AVAILABLE = False
    print("Warning: yt-dlp not installed. YouTube processing disabled.")

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


class EnhancedMultiModalTokenizer:
    """
    Advanced multi-modal tokenizer for various data sources
    """
    
    def __init__(self, data_source):
        self.data_source = data_source
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
        self.source_type = ""
        self.processed_sources = []
        
    def detect_source_type(self, source):
        """Detect source type (URL, file, directory)"""
        # Check if it's a URL
        if self.is_url(source):
            if 'youtube.com' in source or 'youtu.be' in source:
                return 'youtube'
            elif any(ext in source.lower() for ext in ['.pdf', '.docx', '.xlsx', '.jpg', '.png', '.mp4']):
                return 'url_file'
            else:
                return 'url_webpage'
        
        # Check if it's a file
        elif os.path.isfile(source):
            extension = Path(source).suffix.lower()
            for file_type, extensions in self.supported_extensions.items():
                if extension in extensions:
                    return file_type
            return 'unknown_file'
        
        # Check if it's a directory
        elif os.path.isdir(source):
            return 'directory'
        
        return 'unknown'
    
    def is_url(self, string):
        """Check if string is a valid URL"""
        try:
            result = urlparse(string)
            return all([result.scheme, result.netloc])
        except:
            return False
    
    # ==========================================
    # URL PROCESSING METHODS
    # ==========================================
    
    def extract_text_from_webpage(self, url):
        """Extract text from webpage"""
        if not WEB_SCRAPING_AVAILABLE:
            print("Web scraping not available. Install beautifulsoup4 and requests.")
            return ""
        
        try:
            headers = {
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
            }
            response = requests.get(url, headers=headers, timeout=30)
            response.raise_for_status()
            
            soup = BeautifulSoup(response.content, 'html.parser')
            
            # Remove script and style elements
            for script in soup(["script", "style"]):
                script.decompose()
            
            # Extract text
            text = soup.get_text()
            
            # Clean up text
            lines = (line.strip() for line in text.splitlines())
            chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
            text = ' '.join(chunk for chunk in chunks if chunk)
            
            return text
            
        except Exception as e:
            print(f"Error scraping webpage {url}: {e}")
            return ""
    
    def extract_text_from_youtube(self, url):
        """Extract text from YouTube video (transcript + audio)"""
        if not YOUTUBE_AVAILABLE:
            print("YouTube processing not available. Install yt-dlp.")
            return ""
        
        try:
            # Configure yt-dlp
            ydl_opts = {
                'format': 'bestaudio/best',
                'outtmpl': 'temp_youtube_audio.%(ext)s',
                'writesubtitles': True,
                'writeautomaticsub': True,
                'subtitleslangs': ['en'],
                'quiet': True
            }
            
            text = ""
            
            with yt_dlp.YoutubeDL(ydl_opts) as ydl:
                # Extract info
                info = ydl.extract_info(url, download=False)
                
                # Get title and description
                title = info.get('title', '')
                description = info.get('description', '')
                text += f"Title: {title}\n"
                text += f"Description: {description}\n"
                
                # Try to get subtitles
                if info.get('subtitles') or info.get('automatic_captions'):
                    ydl_opts['skip_download'] = True
                    with yt_dlp.YoutubeDL(ydl_opts) as ydl_sub:
                        ydl_sub.download([url])
                    
                    # Look for subtitle files
                    for file in os.listdir('.'):
                        if 'temp_youtube_audio' in file and file.endswith('.vtt'):
                            with open(file, 'r', encoding='utf-8') as f:
                                subtitle_content = f.read()
                                # Clean VTT format
                                subtitle_text = self.clean_vtt_subtitles(subtitle_content)
                                text += subtitle_text
                            os.remove(file)
                            break
                
                # If no subtitles, try audio extraction
                if len(text.split()) < 50:  # If we don't have much text
                    try:
                        ydl_opts['skip_download'] = False
                        with yt_dlp.YoutubeDL(ydl_opts) as ydl_audio:
                            ydl_audio.download([url])
                        
                        # Find downloaded audio file
                        for file in os.listdir('.'):
                            if 'temp_youtube_audio' in file:
                                audio_text = self.extract_text_from_audio(file)
                                text += audio_text
                                os.remove(file)
                                break
                    except:
                        pass
            
            return text
            
        except Exception as e:
            print(f"Error processing YouTube video {url}: {e}")
            return ""
    
    def clean_vtt_subtitles(self, vtt_content):
        """Clean VTT subtitle format"""
        lines = vtt_content.split('\n')
        text_lines = []
        
        for line in lines:
            line = line.strip()
            # Skip VTT headers, timestamps, and empty lines
            if (not line or 
                line.startswith('WEBVTT') or 
                '-->' in line or 
                re.match(r'^\d+$', line) or
                line.startswith('NOTE')):
                continue
            text_lines.append(line)
        
        return ' '.join(text_lines)
    
    def extract_text_from_url_file(self, url):
        """Download and process file from URL"""
        try:
            headers = {
                'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
            }
            response = requests.get(url, headers=headers, timeout=30)
            response.raise_for_status()
            
            # Determine file type from URL
            parsed_url = urlparse(url)
            filename = os.path.basename(parsed_url.path)
            
            if not filename:
                # Try to get filename from Content-Disposition header
                content_disposition = response.headers.get('Content-Disposition', '')
                if 'filename=' in content_disposition:
                    filename = content_disposition.split('filename=')[1].strip('"')
                else:
                    filename = 'temp_file'
            
            # Save temporary file
            temp_path = f"temp_{filename}"
            with open(temp_path, 'wb') as f:
                f.write(response.content)
            
            # Process the file
            text = self.process_file(temp_path)
            
            # Clean up
            if os.path.exists(temp_path):
                os.remove(temp_path)
            
            return text
            
        except Exception as e:
            print(f"Error downloading file from URL {url}: {e}")
            return ""
    
    # ==========================================
    # FILE PROCESSING METHODS
    # ==========================================
    
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
        return self.extract_text_from_audio(file_path, is_video=True)
    
    def extract_text_from_audio(self, file_path, is_video=False):
        """Extract text from audio/video using speech-to-text"""
        if not VIDEO_AVAILABLE:
            print("Audio/Video processing not available. Install moviepy and speech_recognition.")
            return ""
        
        try:
            if is_video:
                # Extract audio from video
                video = VideoFileClip(file_path)
                audio_path = "temp_audio.wav"
                video.audio.write_audiofile(audio_path, verbose=False, logger=None)
                video.close()
            else:
                audio_path = file_path
            
            # Convert speech to text
            recognizer = sr.Recognizer()
            
            # Process audio in chunks for longer files
            text = ""
            try:
                with sr.AudioFile(audio_path) as source:
                    # Adjust for ambient noise
                    recognizer.adjust_for_ambient_noise(source)
                    audio = recognizer.record(source)
                    text = recognizer.recognize_google(audio)
            except sr.UnknownValueError:
                print("Could not understand audio")
            except sr.RequestError as e:
                print(f"Error with speech recognition service: {e}")
            
            # Clean up temporary file
            if is_video and os.path.exists(audio_path):
                os.remove(audio_path)
            
            return text
            
        except Exception as e:
            print(f"Error processing audio/video: {e}")
            return ""
    
    # ==========================================
    # MAIN PROCESSING METHODS
    # ==========================================
    
    def process_file(self, file_path):
        """Process a single file and extract text"""
        file_type = self.detect_source_type(file_path)
        
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
        processed_files = []
        
        for root, dirs, files in os.walk(directory_path):
            for file in files:
                file_path = os.path.join(root, file)
                file_extension = Path(file_path).suffix.lower()
                
                # Check if file is supported
                is_supported = False
                for file_type, extensions in self.supported_extensions.items():
                    if file_extension in extensions:
                        is_supported = True
                        break
                
                if is_supported:
                    text = self.process_file(file_path)
                    if text:
                        all_text += f"\n\n--- Content from {file_path} ---\n"
                        all_text += text
                        processed_files.append(file_path)
        
        self.processed_sources = processed_files
        return all_text
    
    def process_source(self):
        """Main method to process the data source"""
        source_type = self.detect_source_type(self.data_source)
        self.source_type = source_type
        
        print(f"Detected source type: {source_type}")
        print(f"Processing: {self.data_source}")
        
        if source_type == 'url_webpage':
            self.raw_text = self.extract_text_from_webpage(self.data_source)
        elif source_type == 'youtube':
            self.raw_text = self.extract_text_from_youtube(self.data_source)
        elif source_type == 'url_file':
            self.raw_text = self.extract_text_from_url_file(self.data_source)
        elif source_type == 'directory':
            self.raw_text = self.process_directory(self.data_source)
        elif source_type in ['text', 'pdf', 'word', 'excel', 'image', 'video']:
            self.raw_text = self.process_file(self.data_source)
        else:
            print(f"Unsupported source type: {source_type}")
            return False
        
        if not self.raw_text:
            print("No text extracted from source")
            return False
        
        print(f"Successfully extracted {len(self.raw_text)} characters")
        return True
    
    # ==========================================
    # TEXT PROCESSING AND TOKENIZATION
    # ==========================================
    
    def clean_text(self, text):
        """Clean and preprocess text"""
        # Remove extra whitespace
        text = re.sub(r'\s+', ' ', text)
        
        # Remove special characters but keep punctuation
        text = re.sub(r'[^\w\s\.\,\!\?\;\:\-\(\)]', '', text)
        
        # Remove extra spaces
        text = text.strip()
        
        return text
    
    def tokenize_text(self, method='word'):
        """Tokenize the extracted text"""
        if not self.raw_text:
            print("No text to tokenize. Process source first.")
            return []
        
        # Clean text
        cleaned_text = self.clean_text(self.raw_text)
        
        if method == 'word':
            # Word-level tokenization
            tokens = re.findall(r'\b\w+\b', cleaned_text.lower())
        elif method == 'sentence':
            # Sentence-level tokenization
            tokens = re.split(r'[.!?]+', cleaned_text)
            tokens = [sent.strip() for sent in tokens if sent.strip()]
        elif method == 'character':
            # Character-level tokenization
            tokens = list(cleaned_text)
        elif method == 'subword':
            # Simple subword tokenization (BPE-like)
            tokens = self.simple_bpe_tokenize(cleaned_text)
        else:
            print(f"Unknown tokenization method: {method}")
            return []
        
        self.tokens = tokens
        return tokens
    
    def simple_bpe_tokenize(self, text, vocab_size=1000):
        """Simple BPE-like subword tokenization"""
        # Start with character-level tokens
        tokens = list(text.lower())
        
        # Count pairs
        for _ in range(vocab_size):
            pairs = {}
            for i in range(len(tokens) - 1):
                pair = (tokens[i], tokens[i + 1])
                pairs[pair] = pairs.get(pair, 0) + 1
            
            if not pairs:
                break
            
            # Find most frequent pair
            best_pair = max(pairs, key=pairs.get)
            
            # Merge the best pair
            new_tokens = []
            i = 0
            while i < len(tokens):
                if i < len(tokens) - 1 and (tokens[i], tokens[i + 1]) == best_pair:
                    new_tokens.append(tokens[i] + tokens[i + 1])
                    i += 2
                else:
                    new_tokens.append(tokens[i])
                    i += 1
            
            tokens = new_tokens
        
        return tokens
    
    def build_vocabulary(self):
        """Build vocabulary from tokens"""
        if not self.tokens:
            print("No tokens available. Tokenize text first.")
            return {}
        
        # Count token frequencies
        token_counts = Counter(self.tokens)
        
        # Create vocabulary with special tokens
        vocab = {
            '<PAD>': 0,
            '<UNK>': 1,
            '<START>': 2,
            '<END>': 3
        }
        
        # Add tokens to vocabulary
        for token, count in token_counts.most_common():
            if token not in vocab:
                vocab[token] = len(vocab)
        
        self.vocab = vocab
        self.reverse_vocab = {v: k for k, v in vocab.items()}
        
        return vocab
    
    def encode_tokens(self):
        """Encode tokens to numerical IDs"""
        if not self.vocab:
            print("No vocabulary available. Build vocabulary first.")
            return []
        
        encoded = []
        for token in self.tokens:
            if token in self.vocab:
                encoded.append(self.vocab[token])
            else:
                encoded.append(self.vocab['<UNK>'])
        
        self.encoded_tokens = encoded
        return encoded
    
    def decode_tokens(self, encoded_tokens):
        """Decode numerical IDs back to tokens"""
        if not self.reverse_vocab:
            print("No vocabulary available for decoding.")
            return []
        
        decoded = []
        for token_id in encoded_tokens:
            if token_id in self.reverse_vocab:
                decoded.append(self.reverse_vocab[token_id])
            else:
                decoded.append('<UNK>')
        
        return decoded
    
    # ==========================================
    # ANALYSIS AND STATISTICS
    # ==========================================
    
    def get_statistics(self):
        """Get statistics about the processed text"""
        stats = {
            'source_type': self.source_type,
            'source_path': self.data_source,
            'raw_text_length': len(self.raw_text) if self.raw_text else 0,
            'num_tokens': len(self.tokens),
            'vocab_size': len(self.vocab),
            'unique_tokens': len(set(self.tokens)) if self.tokens else 0,
            'processed_sources': self.processed_sources
        }
        
        if self.tokens:
            # Token frequency analysis
            token_counts = Counter(self.tokens)
            stats['most_common_tokens'] = token_counts.most_common(10)
            stats['average_token_length'] = sum(len(token) for token in self.tokens) / len(self.tokens)
        
        return stats
    
    def print_statistics(self):
        """Print detailed statistics"""
        stats = self.get_statistics()
        
        print("\n" + "="*50)
        print("TOKENIZATION STATISTICS")
        print("="*50)
        print(f"Source Type: {stats['source_type']}")
        print(f"Source Path: {stats['source_path']}")
        print(f"Raw Text Length: {stats['raw_text_length']:,} characters")
        print(f"Number of Tokens: {stats['num_tokens']:,}")
        print(f"Vocabulary Size: {stats['vocab_size']:,}")
        print(f"Unique Tokens: {stats['unique_tokens']:,}")
        
        if stats.get('most_common_tokens'):
            print(f"Average Token Length: {stats['average_token_length']:.2f}")
            print("\nMost Common Tokens:")
            for token, count in stats['most_common_tokens']:
                print(f"  '{token}': {count}")
        
        if stats['processed_sources']:
            print(f"\nProcessed {len(stats['processed_sources'])} files:")
            for source in stats['processed_sources'][:5]:  # Show first 5
                print(f"  - {source}")
            if len(stats['processed_sources']) > 5:
                print(f"  ... and {len(stats['processed_sources']) - 5} more")
    
    # ==========================================
    # SAVE AND LOAD FUNCTIONALITY
    # ==========================================
    
    def save_results(self, output_dir='tokenizer_output'):
        """Save tokenization results to files"""
        os.makedirs(output_dir, exist_ok=True)
        
        # Save raw text
        with open(os.path.join(output_dir, 'raw_text.txt'), 'w', encoding='utf-8') as f:
            f.write(self.raw_text)
        
        # Save tokens
        with open(os.path.join(output_dir, 'tokens.txt'), 'w', encoding='utf-8') as f:
            for token in self.tokens:
                f.write(f"{token}\n")
        
        # Save vocabulary
        with open(os.path.join(output_dir, 'vocabulary.json'), 'w', encoding='utf-8') as f:
            json.dump(self.vocab, f, indent=2, ensure_ascii=False)
        
        # Save encoded tokens
        with open(os.path.join(output_dir, 'encoded_tokens.json'), 'w') as f:
            json.dump(self.encoded_tokens, f)
        
        # Save statistics
        with open(os.path.join(output_dir, 'statistics.json'), 'w', encoding='utf-8') as f:
            json.dump(self.get_statistics(), f, indent=2, ensure_ascii=False)
        
        print(f"Results saved to {output_dir}/")
    
    def load_results(self, input_dir='tokenizer_output'):
        """Load previously saved tokenization results"""
        try:
            # Load raw text
            with open(os.path.join(input_dir, 'raw_text.txt'), 'r', encoding='utf-8') as f:
                self.raw_text = f.read()
            
            # Load tokens
            with open(os.path.join(input_dir, 'tokens.txt'), 'r', encoding='utf-8') as f:
                self.tokens = [line.strip() for line in f.readlines()]
            
            # Load vocabulary
            with open(os.path.join(input_dir, 'vocabulary.json'), 'r', encoding='utf-8') as f:
                self.vocab = json.load(f)
                self.reverse_vocab = {v: k for k, v in self.vocab.items()}
            
            # Load encoded tokens
            with open(os.path.join(input_dir, 'encoded_tokens.json'), 'r') as f:
                self.encoded_tokens = json.load(f)
            
            print(f"Results loaded from {input_dir}/")
            return True
            
        except Exception as e:
            print(f"Error loading results: {e}")
            return False
    
    # ==========================================
    # UTILITY METHODS
    # ==========================================
    
    def get_sample_text(self, num_chars=500):
        """Get a sample of the raw text"""
        if not self.raw_text:
            return "No text available"
        
        if len(self.raw_text) <= num_chars:
            return self.raw_text
        else:
            return self.raw_text[:num_chars] + "..."
    
    def get_sample_tokens(self, num_tokens=20):
        """Get a sample of tokens"""
        if not self.tokens:
            return []
        
        return self.tokens[:num_tokens]
    
    def search_tokens(self, query, case_sensitive=False):
        """Search for tokens containing a query string"""
        if not self.tokens:
            return []
        
        if not case_sensitive:
            query = query.lower()
            tokens = [token.lower() for token in self.tokens]
        else:
            tokens = self.tokens
        
        matching_indices = []
        for i, token in enumerate(tokens):
            if query in token:
                matching_indices.append(i)
        
        return matching_indices


# ==========================================
# EXAMPLE USAGE AND TESTING
# ==========================================

def main():
    """Example usage of the Enhanced Multi-Modal Tokenizer"""
    
    print("Enhanced Multi-Modal Tokenizer")
    print("=" * 50)
    
    # Example 1: Process a webpage
    print("\n1. Processing a webpage...")
    try:
        tokenizer1 = EnhancedMultiModalTokenizer("https://en.wikipedia.org/wiki/Natural_language_processing")
        if tokenizer1.process_source():
            tokens = tokenizer1.tokenize_text(method='word')
            vocab = tokenizer1.build_vocabulary()
            encoded = tokenizer1.encode_tokens()
            tokenizer1.print_statistics()
            print(f"Sample text: {tokenizer1.get_sample_text(200)}")
            print(f"Sample tokens: {tokenizer1.get_sample_tokens(10)}")
    except Exception as e:
        print(f"Error processing webpage: {e}")
    
    # Example 2: Process a YouTube video
    print("\n2. Processing a YouTube video...")
    try:
        tokenizer2 = EnhancedMultiModalTokenizer("https://www.youtube.com/watch?v=dQw4w9WgXcQ")
        if tokenizer2.process_source():
            tokens = tokenizer2.tokenize_text(method='word')
            vocab = tokenizer2.build_vocabulary()
            tokenizer2.print_statistics()
    except Exception as e:
        print(f"Error processing YouTube video: {e}")
    
    # Example 3: Process a local file
    print("\n3. Processing a local text file...")
    try:
        # Create a sample text file
        sample_text = """
        This is a sample text file for testing the Enhanced Multi-Modal Tokenizer.
        It contains multiple sentences and various words that will be tokenized.
        The tokenizer can handle different file formats including text, PDF, Word documents, and more.
        """
        
        with open('sample.txt', 'w') as f:
            f.write(sample_text)
        
        tokenizer3 = EnhancedMultiModalTokenizer("sample.txt")
        if tokenizer3.process_source():
            tokens = tokenizer3.tokenize_text(method='word')
            vocab = tokenizer3.build_vocabulary()
                        encoded = tokenizer3.encode_tokens()
            tokenizer3.print_statistics()
            
            # Save results
            tokenizer3.save_results('sample_output')
            
            # Demonstrate search functionality
            search_results = tokenizer3.search_tokens('token')
            print(f"Tokens containing 'token': {len(search_results)} found")
        
        # Clean up
        if os.path.exists('sample.txt'):
            os.remove('sample.txt')
            
    except Exception as e:
        print(f"Error processing local file: {e}")
    
    # Example 4: Process a directory
    print("\n4. Processing a directory...")
    try:
        # Create sample directory with multiple files
        os.makedirs('sample_dir', exist_ok=True)
        
        # Create sample files
        files_content = {
            'doc1.txt': 'This is the first document with some text content.',
            'doc2.txt': 'This is the second document with different content.',
            'doc3.txt': 'This is the third document for testing directory processing.'
        }
        
        for filename, content in files_content.items():
            with open(os.path.join('sample_dir', filename), 'w') as f:
                f.write(content)
        
        tokenizer4 = EnhancedMultiModalTokenizer("sample_dir")
        if tokenizer4.process_source():
            tokens = tokenizer4.tokenize_text(method='word')
            vocab = tokenizer4.build_vocabulary()
            tokenizer4.print_statistics()
        
        # Clean up
        import shutil
        if os.path.exists('sample_dir'):
            shutil.rmtree('sample_dir')
            
    except Exception as e:
        print(f"Error processing directory: {e}")
    
    print("\n" + "="*50)
    print("Demo completed!")


# ==========================================
# ADVANCED FEATURES
# ==========================================

class AdvancedTokenizerFeatures:
    """Additional advanced features for the tokenizer"""
    
    @staticmethod
    def batch_process_urls(urls, output_dir='batch_output'):
        """Process multiple URLs in batch"""
        os.makedirs(output_dir, exist_ok=True)
        results = {}
        
        for i, url in enumerate(urls):
            print(f"Processing URL {i+1}/{len(urls)}: {url}")
            try:
                tokenizer = EnhancedMultiModalTokenizer(url)
                if tokenizer.process_source():
                    tokens = tokenizer.tokenize_text()
                    vocab = tokenizer.build_vocabulary()
                    encoded = tokenizer.encode_tokens()
                    
                    # Save individual results
                    url_output_dir = os.path.join(output_dir, f'url_{i+1}')
                    tokenizer.save_results(url_output_dir)
                    
                    results[url] = {
                        'success': True,
                        'num_tokens': len(tokens),
                        'vocab_size': len(vocab),
                        'output_dir': url_output_dir
                    }
                else:
                    results[url] = {'success': False, 'error': 'Failed to process source'}
                    
            except Exception as e:
                results[url] = {'success': False, 'error': str(e)}
        
        # Save batch summary
        with open(os.path.join(output_dir, 'batch_summary.json'), 'w') as f:
            json.dump(results, f, indent=2)
        
        return results
    
    @staticmethod
    def compare_tokenization_methods(text, methods=['word', 'sentence', 'character', 'subword']):
        """Compare different tokenization methods on the same text"""
        # Create temporary file
        temp_file = 'temp_comparison.txt'
        with open(temp_file, 'w', encoding='utf-8') as f:
            f.write(text)
        
        results = {}
        
        for method in methods:
            try:
                tokenizer = EnhancedMultiModalTokenizer(temp_file)
                tokenizer.process_source()
                tokens = tokenizer.tokenize_text(method=method)
                vocab = tokenizer.build_vocabulary()
                
                results[method] = {
                    'num_tokens': len(tokens),
                    'vocab_size': len(vocab),
                    'sample_tokens': tokens[:10] if tokens else [],
                    'unique_tokens': len(set(tokens)) if tokens else 0
                }
                
            except Exception as e:
                results[method] = {'error': str(e)}
        
        # Clean up
        if os.path.exists(temp_file):
            os.remove(temp_file)
        
        return results
    
    @staticmethod
    def extract_keywords(tokenizer, top_k=20):
        """Extract top keywords from tokenized text"""
        if not tokenizer.tokens:
            return []
        
        # Filter out common stop words
        stop_words = {
            'the', 'a', 'an', 'and', 'or', 'but', 'in', 'on', 'at', 'to', 'for', 'of', 'with', 'by',
            'is', 'are', 'was', 'were', 'be', 'been', 'being', 'have', 'has', 'had', 'do', 'does', 'did',
            'will', 'would', 'could', 'should', 'may', 'might', 'can', 'this', 'that', 'these', 'those',
            'i', 'you', 'he', 'she', 'it', 'we', 'they', 'me', 'him', 'her', 'us', 'them'
        }
        
        # Filter tokens
        filtered_tokens = [token for token in tokenizer.tokens 
                          if token.lower() not in stop_words and len(token) > 2]
        
        # Count frequencies
        token_counts = Counter(filtered_tokens)
        
        return token_counts.most_common(top_k)
    
    @staticmethod
    def generate_ngrams(tokens, n=2):
        """Generate n-grams from tokens"""
        if len(tokens) < n:
            return []
        
        ngrams = []
        for i in range(len(tokens) - n + 1):
            ngram = tuple(tokens[i:i+n])
            ngrams.append(ngram)
        
        return ngrams
    
    @staticmethod
    def calculate_text_similarity(tokenizer1, tokenizer2):
        """Calculate similarity between two tokenized texts"""
        if not tokenizer1.tokens or not tokenizer2.tokens:
            return 0.0
        
        # Convert to sets for Jaccard similarity
        set1 = set(tokenizer1.tokens)
        set2 = set(tokenizer2.tokens)
        
        # Jaccard similarity
        intersection = len(set1.intersection(set2))
        union = len(set1.union(set2))
        
        if union == 0:
            return 0.0
        
        return intersection / union


# ==========================================
# COMMAND LINE INTERFACE
# ==========================================

def create_cli():
    """Create command line interface for the tokenizer"""
    import argparse
    
    parser = argparse.ArgumentParser(description='Enhanced Multi-Modal Tokenizer')
    parser.add_argument('source', help='Source to process (URL, file path, or directory)')
    parser.add_argument('--method', choices=['word', 'sentence', 'character', 'subword'], 
                       default='word', help='Tokenization method')
    parser.add_argument('--output', '-o', default='tokenizer_output', 
                       help='Output directory for results')
    parser.add_argument('--stats', action='store_true', 
                       help='Print detailed statistics')
    parser.add_argument('--keywords', type=int, default=0, 
                       help='Extract top N keywords')
    parser.add_argument('--sample', type=int, default=500, 
                       help='Number of characters to show in sample')
    parser.add_argument('--save', action='store_true', 
                       help='Save results to files')
    
    return parser

def run_cli():
    """Run the command line interface"""
    parser = create_cli()
    args = parser.parse_args()
    
    print(f"Processing: {args.source}")
    print(f"Method: {args.method}")
    print("-" * 50)
    
    try:
        # Initialize tokenizer
        tokenizer = EnhancedMultiModalTokenizer(args.source)
        
        # Process source
        if not tokenizer.process_source():
            print("Failed to process source")
            return
        
        # Tokenize
        tokens = tokenizer.tokenize_text(method=args.method)
        if not tokens:
            print("Failed to tokenize text")
            return
        
        # Build vocabulary
        vocab = tokenizer.build_vocabulary()
        encoded = tokenizer.encode_tokens()
        
        # Print sample
        print(f"Sample text ({args.sample} chars):")
        print(tokenizer.get_sample_text(args.sample))
        print()
        
        # Print statistics
        if args.stats:
            tokenizer.print_statistics()
        else:
            stats = tokenizer.get_statistics()
            print(f"Tokens: {stats['num_tokens']:,}")
            print(f"Vocabulary size: {stats['vocab_size']:,}")
            print(f"Unique tokens: {stats['unique_tokens']:,}")
        
        # Extract keywords
        if args.keywords > 0:
            keywords = AdvancedTokenizerFeatures.extract_keywords(tokenizer, args.keywords)
            print(f"\nTop {args.keywords} keywords:")
            for keyword, count in keywords:
                print(f"  {keyword}: {count}")
        
        # Save results
        if args.save:
            tokenizer.save_results(args.output)
            print(f"\nResults saved to: {args.output}")
        
    except Exception as e:
        print(f"Error: {e}")
        import traceback
        traceback.print_exc()


# ==========================================
# INSTALLATION REQUIREMENTS
# ==========================================

def check_dependencies():
    """Check and report on available dependencies"""
    dependencies = {
        'requests': 'Web scraping',
        'beautifulsoup4': 'HTML parsing',
        'yt-dlp': 'YouTube video processing',
        'PyPDF2': 'PDF processing',
        'python-docx': 'Word document processing',
        'pandas': 'Excel/CSV processing',
        'Pillow': 'Image processing',
        'pytesseract': 'OCR (Optical Character Recognition)',
        'moviepy': 'Video processing',
        'SpeechRecognition': 'Speech-to-text conversion'
    }
    
    print("Dependency Status:")
    print("-" * 40)
    
    for package, description in dependencies.items():
        try:
            __import__(package.replace('-', '_').lower())
            status = "✓ Installed"
        except ImportError:
            status = "✗ Not installed"
        
        print(f"{package:20} {status:15} - {description}")
    
    print("\nTo install all dependencies:")
    print("pip install requests beautifulsoup4 yt-dlp PyPDF2 python-docx pandas Pillow pytesseract moviepy SpeechRecognition")


# ==========================================
# MAIN EXECUTION
# ==========================================

if __name__ == "__main__":
    import sys
    
    if len(sys.argv) > 1:
        # Run CLI if arguments provided
        run_cli()
    else:
        # Run demo
        print("Enhanced Multi-Modal Tokenizer")
        print("=" * 50)
        print("No arguments provided. Running demo...")
        print("For CLI usage, run: python enhanced_multimodal_tokenizer.py <source>")
        print()
        
        # Check dependencies
        check_dependencies()
        print()
        
        # Run main demo
        main()


# ==========================================
# ADDITIONAL UTILITY FUNCTIONS
# ==========================================

def quick_tokenize(source, method='word', save_results=False):
    """Quick tokenization function for simple use cases"""
    tokenizer = EnhancedMultiModalTokenizer(source)
    
    if tokenizer.process_source():
        tokens = tokenizer.tokenize_text(method=method)
        vocab = tokenizer.build_vocabulary()
        encoded = tokenizer.encode_tokens()
        
        if save_results:
            tokenizer.save_results()
        
        return {
            'tokens': tokens,
            'vocab': vocab,
            'encoded': encoded,
            'stats': tokenizer.get_statistics(),
            'tokenizer': tokenizer
        }
    
    return None

def batch_tokenize(sources, method='word', output_dir='batch_tokenize_output'):
    """Batch tokenize multiple sources"""
    results = {}
    os.makedirs(output_dir, exist_ok=True)
    
    for i, source in enumerate(sources):
        print(f"Processing {i+1}/{len(sources)}: {source}")
        
        try:
            result = quick_tokenize(source, method=method)
            if result:
                # Save individual results
                source_output_dir = os.path.join(output_dir, f'source_{i+1}')
                result['tokenizer'].save_results(source_output_dir)
                
                results[source] = {
                    'success': True,
                    'stats': result['stats'],
                    'output_dir': source_output_dir
                }
            else:
                results[source] = {'success': False, 'error': 'Processing failed'}
                
        except Exception as e:
            results[source] = {'success': False, 'error': str(e)}
    
    # Save batch summary
    with open(os.path.join(output_dir, 'batch_summary.json'), 'w') as f:
        json.dump(results, f, indent=2, default=str)
    
    return results


# ==========================================
# CONFIGURATION AND SETTINGS
# ==========================================

class TokenizerConfig:
    """Configuration class for tokenizer settings"""
    
    def __init__(self):
        self.max_file_size = 100 * 1024 * 1024  # 100MB
        self.request_timeout = 30
        self.max_retries = 3
        self.chunk_size = 1024 * 1024  # 1MB chunks for large files
        self.supported_languages = ['en', 'es', 'fr', 'de', 'it', 'pt']
        self.ocr_language = 'eng'
        self.speech_recognition_language = 'en-US'
        
        # User agent for web requests
        self.user_agent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36'
        
        # Output settings
        self.save_raw_text = True
        self.save_tokens = True
        self.save_vocabulary = True
        self.save_encoded_tokens = True
        self.save_statistics = True
    
    def to_dict(self):
        """Convert config to dictionary"""
        return {
            'max_file_size': self.max_file_size,
            'request_timeout': self.request_timeout,
            'max_retries': self.max_retries,
            'chunk_size': self.chunk_size,
            'supported_languages': self.supported_languages,
            'ocr_language': self.ocr_language,
            'speech_recognition_language': self.speech_recognition_language,
            'user_agent': self.user_agent,
            'save_raw_text': self.save_raw_text,
            'save_tokens': self.save_tokens,
            'save_vocabulary': self.save_vocabulary,
            'save_encoded_tokens': self.save_
 
