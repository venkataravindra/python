# Universal Multimodal LLM Project Structure

Here's a comprehensive project structure for building a multimodal LLM that can process text, video, audio, documents, Excel, images, and web pages.

## 📁 Project Structure

```
multimodal_llm/
├── 📄 data/
│   ├── raw/
│   │   ├── text/
│   │   ├── images/
│   │   ├── audio/
│   │   ├── video/
│   │   ├── documents/
│   │   └── web/
│   └── processed/
├── 🔤 tokenizer.py
├── 📊 dataset.py
├── 🧠 model.py
├── 🏋️ train.py
├── 🔮 predict.py
├── utils/
│   ├── data_processors.py
│   ├── feature_extractors.py
│   └── web_scraper.py
├── config/
│   └── config.yaml
├── requirements.txt
└── main.py
```

## 📋 requirements.txt

```txt
torch>=2.0.0
transformers>=4.30.0
datasets>=2.12.0
tokenizers>=0.13.0
Pillow>=9.5.0
opencv-python>=4.7.0
librosa>=0.10.0
soundfile>=0.12.0
PyPDF2>=3.0.0
python-docx>=0.8.11
openpyxl>=3.1.0
pandas>=2.0.0
numpy>=1.24.0
requests>=2.31.0
beautifulsoup4>=4.12.0
selenium>=4.10.0
accelerate>=0.20.0
wandb>=0.15.0
tqdm>=4.65.0
```

## ⚙️ config/config.yaml

```yaml
model:
  name: "multimodal_llm"
  max_length: 2048
  vocab_size: 50000
  hidden_size: 768
  num_layers: 12
  num_heads: 12
  dropout: 0.1

data:
  batch_size: 8
  max_workers: 4
  train_split: 0.8
  val_split: 0.1
  test_split: 0.1

training:
  epochs: 10
  learning_rate: 5e-5
  weight_decay: 0.01
  warmup_steps: 1000
  save_steps: 1000
  eval_steps: 500

paths:
  data_dir: "./data"
  model_dir: "./models"
  logs_dir: "./logs"

modalities:
  text:
    enabled: true
    max_length: 512
  image:
    enabled: true
    size: [224, 224]
    channels: 3
  audio:
    enabled: true
    sample_rate: 16000
    max_duration: 30
  video:
    enabled: true
    fps: 1
    max_frames: 30
```

## 🛠️ utils/data_processors.py

```python
import os
import cv2
import librosa
import pandas as pd
import numpy as np
from PIL import Image
import PyPDF2
import docx
from bs4 import BeautifulSoup
import requests
from typing import Dict, Any, List, Union
import base64
import io

class DataProcessor:
    def __init__(self, config):
        self.config = config
    
    def process_text(self, text_path: str) -> str:
        """Process text files"""
        try:
            with open(text_path, 'r', encoding='utf-8') as f:
                return f.read()
        except Exception as e:
            print(f"Error processing text: {e}")
            return ""
    
    def process_image(self, image_path: str) -> np.ndarray:
        """Process image files"""
        try:
            image = Image.open(image_path).convert('RGB')
            size = tuple(self.config['modalities']['image']['size'])
            image = image.resize(size)
            return np.array(image)
        except Exception as e:
            print(f"Error processing image: {e}")
            return np.zeros((224, 224, 3))
    
    def process_audio(self, audio_path: str) -> np.ndarray:
        """Process audio files"""
        try:
            sr = self.config['modalities']['audio']['sample_rate']
            max_duration = self.config['modalities']['audio']['max_duration']
            
            audio, _ = librosa.load(audio_path, sr=sr, duration=max_duration)
            
            # Pad or truncate to fixed length
            target_length = sr * max_duration
            if len(audio) < target_length:
                audio = np.pad(audio, (0, target_length - len(audio)))
            else:
                audio = audio[:target_length]
                
            return audio
        except Exception as e:
            print(f"Error processing audio: {e}")
            return np.zeros(16000 * 30)
    
    def process_video(self, video_path: str) -> np.ndarray:
        """Process video files"""
        try:
            cap = cv2.VideoCapture(video_path)
            frames = []
            fps = self.config['modalities']['video']['fps']
            max_frames = self.config['modalities']['video']['max_frames']
            
            frame_count = 0
            while cap.read()[0] and frame_count < max_frames:
                ret, frame = cap.read()
                if ret and frame_count % fps == 0:
                    frame = cv2.resize(frame, (224, 224))
                    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                    frames.append(frame)
                frame_count += 1
            
            cap.release()
            
            # Pad or truncate frames
            while len(frames) < max_frames:
                frames.append(np.zeros((224, 224, 3)))
            
            return np.array(frames[:max_frames])
        except Exception as e:
            print(f"Error processing video: {e}")
            return np.zeros((30, 224, 224, 3))
    
    def process_pdf(self, pdf_path: str) -> str:
        """Process PDF files"""
        try:
            text = ""
            with open(pdf_path, 'rb') as file:
                pdf_reader = PyPDF2.PdfReader(file)
                for page in pdf_reader.pages:
                    text += page.extract_text()
            return text
        except Exception as e:
            print(f"Error processing PDF: {e}")
            return ""
    
    def process_docx(self, docx_path: str) -> str:
        """Process DOCX files"""
        try:
            doc = docx.Document(docx_path)
            text = ""
            for paragraph in doc.paragraphs:
                text += paragraph.text + "\n"
            return text
        except Exception as e:
            print(f"Error processing DOCX: {e}")
            return ""
    
    def process_excel(self, excel_path: str) -> str:
        """Process Excel files"""
        try:
            df = pd.read_excel(excel_path)
            return df.to_string()
        except Exception as e:
            print(f"Error processing Excel: {e}")
            return ""
    
    def process_webpage(self, url: str) -> str:
        """Process web pages"""
        try:
            response = requests.get(url, timeout=10)
            soup = BeautifulSoup(response.content, 'html.parser')
            
            # Remove script and style elements
            for script in soup(["script", "style"]):
                script.decompose()
            
            text = soup.get_text()
            lines = (line.strip() for line in text.splitlines())
            chunks = (phrase.strip() for line in lines for phrase in line.split("  "))
            text = ' '.join(chunk for chunk in chunks if chunk)
            
            return text
        except Exception as e:
            print(f"Error processing webpage: {e}")
            return ""

class MultimodalDataProcessor:
    def __init__(self, config):
        self.processor = DataProcessor(config)
        self.config = config
    
    def process_file(self, file_path: str, file_type: str = None) -> Dict[str, Any]:
        """Process any file type and return structured data"""
        if file_type is None:
            file_type = self.detect_file_type(file_path)
        
        result = {
            'text': '',
            'image': None,
            'audio': None,
            'video': None,
            'metadata': {'file_path': file_path, 'file_type': file_type}
        }
        
        if file_type == 'text':
            result['text'] = self.processor.process_text(file_path)
        elif file_type == 'image':
            result['image'] = self.processor.process_image(file_path)
        elif file_type == 'audio':
            result['audio'] = self.processor.process_audio(file_path)
        elif file_type == 'video':
            result['video'] = self.processor.process_video(file_path)
        elif file_type == 'pdf':
            result['text'] = self.processor.process_pdf(file_path)
        elif file_type == 'docx':
            result['text'] = self.processor.process_docx(file_path)
        elif file_type == 'excel':
            result['text'] = self.processor.process_excel(file_path)
        elif file_type == 'webpage':
            result['text'] = self.processor.process_webpage(file_path)
        
        return result
    
    def detect_file_type(self, file_path: str) -> str:
        """Detect file type from extension"""
        ext = os.path.splitext(file_path)[1].lower()
        
        type_mapping = {
            '.txt': 'text', '.md': 'text',
            '.jpg': 'image', '.jpeg': 'image', '.png': 'image', '.bmp': 'image',
            '.wav': 'audio', '.mp3': 'audio', '.flac': 'audio',
            '.mp4': 'video', '.avi': 'video', '.mov': 'video',
            '.pdf': 'pdf',
            '.docx': 'docx', '.doc': 'docx',
            '.xlsx': 'excel', '.xls': 'excel'
        }
        
        return type_mapping.get(ext, 'text')
```

## 🔤 tokenizer.py

```python
import torch
from transformers import AutoTokenizer, AutoImageProcessor
from typing import Dict, List, Any, Optional
import numpy as np
import yaml

class MultimodalTokenizer:
    def __init__(self, config_path: str = "config/config.yaml"):
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
        
        # Initialize text tokenizer
        self.text_tokenizer = AutoTokenizer.from_pretrained(
            "microsoft/DialoGPT-medium",
            padding_side="right"
        )
        
        # Add special tokens for multimodal data
        special_tokens = {
            "additional_special_tokens": [
                "<|image|>", "<|audio|>", "<|video|>", 
                "<|document|>", "<|webpage|>", "<|excel|>",
                "<|start_modality|>", "<|end_modality|>"
            ]
        }
        self.text_tokenizer.add_special_tokens(special_tokens)
        
        # Initialize image processor
        self.image_processor = AutoImageProcessor.from_pretrained(
            "microsoft/resnet-50"
        )
        
        self.max_length = self.config['model']['max_length']
    
    def tokenize_text(self, text: str) -> Dict[str, torch.Tensor]:
        """Tokenize text input"""
        if not text:
            text = ""
        
        encoded = self.text_tokenizer(
            text,
            max_length=self.max_length,
            padding="max_length",
            truncation=True,
            return_tensors="pt"
        )
        
        return {
            'input_ids': encoded['input_ids'].squeeze(0),
            'attention_mask': encoded['attention_mask'].squeeze(0)
        }
    
    def process_image(self, image: np.ndarray) -> torch.Tensor:
        """Process image for model input"""
        if image is None:
            return torch.zeros(3, 224, 224)
        
        # Normalize image
        image = image.astype(np.float32) / 255.0
        image = torch.from_numpy(image).permute(2, 0, 1)
        
        return image
    
    def process_audio(self, audio: np.ndarray) -> torch.Tensor:
        """Process audio for model input"""
        if audio is None:
            return torch.zeros(16000 * 30)
        
        # Normalize audio
        audio = audio.astype(np.float32)
        if np.max(np.abs(audio)) > 0:
            audio = audio / np.max(np.abs(audio))
        
        return torch.from_numpy(audio)
    
    def process_video(self, video: np.ndarray) -> torch.Tensor:
        """Process video for model input"""
        if video is None:
            return torch.zeros(30, 3, 224, 224)
        
        # Normalize and convert video frames
        video = video.astype(np.float32) / 255.0
        video = torch.from_numpy(video).permute(0, 3, 1, 2)
        
        return video
    
    def encode_multimodal(self, data: Dict[str, Any]) -> Dict[str, torch.Tensor]:
        """Encode multimodal data"""
        result = {}
        
        # Process text
        text = data.get('text', '')
        if data.get('image') is not None:
            text = f"<|image|> {text}"
        if data.get('audio') is not None:
            text = f"<|audio|> {text}"
        if data.get('video') is not None:
            text = f"<|video|> {text}"
        
        text_tokens = self.tokenize_text(text)
        result.update(text_tokens)
        
        # Process other modalities
        result['image'] = self.process_image(data.get('image'))
        result['audio'] = self.process_audio(data.get('audio'))
        result['video'] = self.process_video(data.get('video'))
        
        # Add modality flags
        result['has_image'] = torch.tensor(1.0 if data.get('image') is not None else 0.0)
        result['has_audio'] = torch.tensor(1.0 if data.get('audio') is not None else 0.0)
        result['has_video'] = torch.tensor(1.0 if data.get('video') is not None else 0.0)
        
        return result
    
    def decode_text(self, token_ids: torch.Tensor) -> str:
        """Decode token IDs back to text"""
        return self.text_tokenizer.decode(token_ids, skip_special_tokens=True)
    
    def get_vocab_size(self) -> int:
        """Get vocabulary size"""
        return len(self.text_tokenizer)
```

