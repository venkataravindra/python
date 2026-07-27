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

## 📊 dataset.py (continued)

```python
import torch
from torch.utils.data import Dataset, DataLoader
import os
import json
import yaml
from typing import Dict, List, Any
from utils.data_processors import MultimodalDataProcessor
from tokenizer import MultimodalTokenizer

class MultimodalDataset(Dataset):
    def __init__(self, data_dir: str, config_path: str = "config/config.yaml", split: str = "train"):
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
        
        self.data_dir = data_dir
        self.split = split
        self.processor = MultimodalDataProcessor(self.config)
        self.tokenizer = MultimodalTokenizer(config_path)
        
        # Load data index
        self.data_index = self._build_data_index()
        
    def _build_data_index(self) -> List[Dict[str, Any]]:
        """Build index of all data files"""
        data_index = []
        
        # Scan all subdirectories for files
        for root, dirs, files in os.walk(self.data_dir):
            for file in files:
                if file.startswith('.'):
                    continue
                    
                file_path = os.path.join(root, file)
                file_type = self.processor.detect_file_type(file_path)
                
                data_index.append({
                    'file_path': file_path,
                    'file_type': file_type,
                    'relative_path': os.path.relpath(file_path, self.data_dir)
                })
        
        # Split data
        total_size = len(data_index)
        train_size = int(total_size * self.config['data']['train_split'])
        val_size = int(total_size * self.config['data']['val_split'])
        
        if self.split == "train":
            return data_index[:train_size]
        elif self.split == "val":
            return data_index[train_size:train_size + val_size]
        else:  # test
            return data_index[train_size + val_size:]
    
    def __len__(self) -> int:
        return len(self.data_index)
    
    def __getitem__(self, idx: int) -> Dict[str, torch.Tensor]:
        """Get a single data sample"""
        item = self.data_index[idx]
        
        # Process the file
        processed_data = self.processor.process_file(
            item['file_path'], 
            item['file_type']
        )
        
        # Tokenize and encode
        encoded_data = self.tokenizer.encode_multimodal(processed_data)
        
        # Add labels (for training, we use the text as target)
        if processed_data['text']:
            labels = self.tokenizer.tokenize_text(processed_data['text'])['input_ids']
        else:
            labels = torch.zeros_like(encoded_data['input_ids'])
        
        encoded_data['labels'] = labels
        
        return encoded_data

def create_dataloaders(config_path: str = "config/config.yaml") -> Dict[str, DataLoader]:
    """Create train, validation, and test dataloaders"""
    with open(config_path, 'r') as f:
        config = yaml.safe_load(f)
    
    data_dir = config['paths']['data_dir']
    batch_size = config['data']['batch_size']
    num_workers = config['data']['max_workers']
    
    # Create datasets
    train_dataset = MultimodalDataset(data_dir, config_path, "train")
    val_dataset = MultimodalDataset(data_dir, config_path, "val")
    test_dataset = MultimodalDataset(data_dir, config_path, "test")
    
    # Create dataloaders
    train_loader = DataLoader(
        train_dataset,
        batch_size=batch_size,
        shuffle=True,
        num_workers=num_workers,
        pin_memory=True
    )
    
    val_loader = DataLoader(
        val_dataset,
        batch_size=batch_size,
        shuffle=False,
        num_workers=num_workers,
        pin_memory=True
    )
    
    test_loader = DataLoader(
        test_dataset,
        batch_size=batch_size,
        shuffle=False,
        num_workers=num_workers,
        pin_memory=True
    )
    
    return {
        'train': train_loader,
        'val': val_loader,
        'test': test_loader
    }

class CollateFunction:
    """Custom collate function for batching multimodal data"""
    
    def __call__(self, batch: List[Dict[str, torch.Tensor]]) -> Dict[str, torch.Tensor]:
        """Collate batch of multimodal samples"""
        collated = {}
        
        # Stack tensors
        for key in batch[0].keys():
            if key in ['input_ids', 'attention_mask', 'labels']:
                collated[key] = torch.stack([item[key] for item in batch])
            elif key in ['image', 'audio', 'video']:
                collated[key] = torch.stack([item[key] for item in batch])
            elif key in ['has_image', 'has_audio', 'has_video']:
                collated[key] = torch.stack([item[key] for item in batch])
        
        return collated
```

## 🧠 model.py

```python
import torch
import torch.nn as nn
import torch.nn.functional as F
from transformers import GPT2LMHeadModel, GPT2Config
import yaml
from typing import Dict, Optional, Tuple

class ImageEncoder(nn.Module):
    """Encode images to feature vectors"""
    
    def __init__(self, hidden_size: int):
        super().__init__()
        self.conv_layers = nn.Sequential(
            nn.Conv2d(3, 64, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(64, 128, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool2d(2),
            nn.Conv2d(128, 256, 3, padding=1),
            nn.ReLU(),
            nn.AdaptiveAvgPool2d((7, 7))
        )
        self.fc = nn.Linear(256 * 7 * 7, hidden_size)
        self.dropout = nn.Dropout(0.1)
    
    def forward(self, images: torch.Tensor) -> torch.Tensor:
        # images: (batch_size, 3, 224, 224)
        x = self.conv_layers(images)
        x = x.view(x.size(0), -1)
        x = self.fc(x)
        x = self.dropout(x)
        return x

class AudioEncoder(nn.Module):
    """Encode audio to feature vectors"""
    
    def __init__(self, hidden_size: int):
        super().__init__()
        self.conv_layers = nn.Sequential(
            nn.Conv1d(1, 64, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool1d(2),
            nn.Conv1d(64, 128, 3, padding=1),
            nn.ReLU(),
            nn.MaxPool1d(2),
            nn.Conv1d(128, 256, 3, padding=1),
            nn.ReLU(),
            nn.AdaptiveAvgPool1d(100)
        )
        self.fc = nn.Linear(256 * 100, hidden_size)
        self.dropout = nn.Dropout(0.1)
    
    def forward(self, audio: torch.Tensor) -> torch.Tensor:
        # audio: (batch_size, seq_len)
        x = audio.unsqueeze(1)  # Add channel dimension
        x = self.conv_layers(x)
        x = x.view(x.size(0), -1)
        x = self.fc(x)
        x = self.dropout(x)
        return x

class VideoEncoder(nn.Module):
    """Encode video to feature vectors"""
    
    def __init__(self, hidden_size: int):
        super().__init__()
        self.frame_encoder = ImageEncoder(hidden_size // 2)
        self.temporal_encoder = nn.LSTM(
            hidden_size // 2, 
            hidden_size // 2, 
            batch_first=True
        )
        self.fc = nn.Linear(hidden_size // 2, hidden_size)
        self.dropout = nn.Dropout(0.1)
    
    def forward(self, video: torch.Tensor) -> torch.Tensor:
        # video: (batch_size, num_frames, 3, 224, 224)
        batch_size, num_frames = video.size(0), video.size(1)
        
        # Encode each frame
        video_flat = video.view(-1, 3, 224, 224)
        frame_features = self.frame_encoder(video_flat)
        frame_features = frame_features.view(batch_size, num_frames, -1)
        
        # Encode temporal information
        temporal_features, _ = self.temporal_encoder(frame_features)
        
        # Use last frame's features
        video_features = temporal_features[:, -1, :]
        video_features = self.fc(video_features)
        video_features = self.dropout(video_features)
        
        return video_features

class MultimodalFusion(nn.Module):
    """Fuse multimodal features"""
    
    def __init__(self, hidden_size: int):
        super().__init__()
        self.hidden_size = hidden_size
        
        # Projection layers for each modality
        self.text_proj = nn.Linear(hidden_size, hidden_size)
        self.image_proj = nn.Linear(hidden_size, hidden_size)
        self.audio_proj = nn.Linear(hidden_size, hidden_size)
        self.video_proj = nn.Linear(hidden_size, hidden_size)
        
        # Attention mechanism for fusion
        self.attention = nn.MultiheadAttention(hidden_size, 8, batch_first=True)
        self.norm = nn.LayerNorm(hidden_size)
        self.dropout = nn.Dropout(0.1)
    
    def forward(self, 
                text_features: torch.Tensor,
                image_features: Optional[torch.Tensor] = None,
                audio_features: Optional[torch.Tensor] = None,
                video_features: Optional[torch.Tensor] = None,
                modality_flags: Optional[Dict[str, torch.Tensor]] = None) -> torch.Tensor:
        
        features = [self.text_proj(text_features)]
        
        if image_features is not None and modality_flags.get('has_image', torch.tensor(0.0)).sum() > 0:
            features.append(self.image_proj(image_features))
        
        if audio_features is not None and modality_flags.get('has_audio', torch.tensor(0.0)).sum() > 0:
            features.append(self.audio_proj(audio_features))
        
        if video_features is not None and modality_flags.get('has_video', torch.tensor(0.0)).sum() > 0:
            features.append(self.video_proj(video_features))
        
        if len(features) == 1:
            return features[0]
        
        # Stack features for attention
        stacked_features = torch.stack(features, dim=1)
        
        # Apply attention
        attended_features, _ = self.attention(
            stacked_features, stacked_features, stacked_features
        )
        
        # Average pool attended features
        fused_features = attended_features.mean(dim=1)
        fused_features = self.norm(fused_features + text_features)
        fused_features = self.dropout(fused_features)
        
        return fused_features

class MultimodalLLM(nn.Module):
    """Main multimodal language model"""
    
    def __init__(self, config_path: str = "config/config.yaml"):
        super().__init__()
        
        with open(config_path, 'r') as f:
            config = yaml.safe_load(f)
        
        self.config = config['model']
        hidden_size = self.config['hidden_size']
        
        # Initialize text model (GPT-2 based)
        gpt_config = GPT2Config(
            vocab_size=self.config['vocab_size'],
            n_embd=hidden_size,
            n_layer=self.config['num_layers'],
            n_head=self.config['num_heads'],
            resid_pdrop=self.config['dropout'],
            embd_pdrop=self.config['dropout'],
            attn_pdrop=self.config['dropout'],
        )
        
        self.text_model = GPT2LMHeadModel(gpt_config)
        
        # Initialize modality encoders
        self.image_encoder = ImageEncoder(hidden_size)
        self.audio_encoder = AudioEncoder(hidden_size)
        self.video_encoder = VideoEncoder(hidden_size)
        
        # Initialize fusion module
        self.fusion = MultimodalFusion(hidden_size)
        
        # Output projection
        self.output_proj = nn.Linear(hidden_size, self.config['vocab_size'])
        
    def forward(self, 
                input_ids: torch.Tensor,
                attention_mask: torch.Tensor,
                image: Optional[torch.Tensor] = None,
                audio: Optional[torch.Tensor] = None,
                video: Optional[torch.Tensor] = None,
                has_image: Optional[torch.Tensor] = None,
                has_audio: Optional[torch.Tensor] = None,
                has_video: Optional[torch.Tensor] = None,
                labels: Optional[torch.Tensor] = None) -> Dict[str, torch.Tensor]:
        
        # Get text embeddings
        text_outputs = self.text_model.transformer(
            input_ids=input_ids,
            attention_mask=attention_mask
        )
        text_features = text_outputs.last_hidden_state
        
        # Encode other modalities
        image_features = None
        audio_features = None
        video_features = None
        
        if image is not None:
            image_features = self.image_encoder(image)
            image_features = image_features.unsqueeze(1).expand(-1, text_features.size(1), -1)
        
        if audio is not None:
            audio_features = self.audio_encoder(audio)
            audio_features = audio_features.unsqueeze(1).expand(-1, text_features.size(1), -1)
        
        if video is not None:
            video_features = self.video_encoder(video)
            video_features = video_features.unsqueeze(1).expand(-1, text_features.size(1), -1)
        
        # Fuse modalities
        modality_flags = {
            'has_image': has_image,
            'has_audio': has_audio,
            'has_video': has_video
        }
        
        # Apply fusion for each sequence position
        batch_size, seq_len, hidden_size = text_features.shape
        fused_features = []
        
        for i in range(seq_len):
            fused = self.fusion(
                text_features[:, i, :],
                image_features[:, i, :] if image_features is not None else None,
                audio_features[:, i, :] if audio_features is not None else None,
                video_features[:, i, :] if video_features is not None else None,
                modality_flags
            )
            fused_features.append(fused)
        

