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
                fused_features = torch.stack(fused_features, dim=1)
        
        # Generate logits
        logits = self.output_proj(fused_features)
        
        # Calculate loss if labels are provided
        loss = None
        if labels is not None:
            shift_logits = logits[..., :-1, :].contiguous()
            shift_labels = labels[..., 1:].contiguous()
            loss_fct = nn.CrossEntropyLoss(ignore_index=-100)
            loss = loss_fct(shift_logits.view(-1, shift_logits.size(-1)), shift_labels.view(-1))
        
        return {
            'logits': logits,
            'loss': loss,
            'hidden_states': fused_features
        }
    
    def generate(self, 
                 input_ids: torch.Tensor,
                 attention_mask: torch.Tensor,
                 image: Optional[torch.Tensor] = None,
                 audio: Optional[torch.Tensor] = None,
                 video: Optional[torch.Tensor] = None,
                 max_length: int = 100,
                 temperature: float = 1.0,
                 top_p: float = 0.9) -> torch.Tensor:
        """Generate text given multimodal inputs"""
        
        self.eval()
        with torch.no_grad():
            batch_size = input_ids.size(0)
            device = input_ids.device
            
            # Prepare modality flags
            has_image = torch.tensor(1.0 if image is not None else 0.0, device=device).expand(batch_size)
            has_audio = torch.tensor(1.0 if audio is not None else 0.0, device=device).expand(batch_size)
            has_video = torch.tensor(1.0 if video is not None else 0.0, device=device).expand(batch_size)
            
            generated = input_ids.clone()
            
            for _ in range(max_length):
                outputs = self.forward(
                    input_ids=generated,
                    attention_mask=torch.ones_like(generated),
                    image=image,
                    audio=audio,
                    video=video,
                    has_image=has_image,
                    has_audio=has_audio,
                    has_video=has_video
                )
                
                logits = outputs['logits'][:, -1, :] / temperature
                
                # Apply top-p sampling
                sorted_logits, sorted_indices = torch.sort(logits, descending=True)
                cumulative_probs = torch.cumsum(F.softmax(sorted_logits, dim=-1), dim=-1)
                
                sorted_indices_to_remove = cumulative_probs > top_p
                sorted_indices_to_remove[..., 1:] = sorted_indices_to_remove[..., :-1].clone()
                sorted_indices_to_remove[..., 0] = 0
                
                indices_to_remove = sorted_indices_to_remove.scatter(1, sorted_indices, sorted_indices_to_remove)
                logits[indices_to_remove] = float('-inf')
                
                probs = F.softmax(logits, dim=-1)
                next_token = torch.multinomial(probs, num_samples=1)
                
                generated = torch.cat([generated, next_token], dim=1)
                
                # Check for end token
                if next_token.item() == 50256:  # GPT-2 end token
                    break
            
            return generated

🏋️ train.py

import torch
import torch.nn as nn
from torch.optim import AdamW
from torch.optim.lr_scheduler import CosineAnnealingLR
import yaml
import os
import wandb
from tqdm import tqdm
from typing import Dict, Any
import json
from datetime import datetime

from model import MultimodalLLM
from dataset import create_dataloaders, CollateFunction
from tokenizer import MultimodalTokenizer

class MultimodalTrainer:
    def __init__(self, config_path: str = "config/config.yaml"):
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
        
        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        print(f"Using device: {self.device}")
        
        # Initialize model
        self.model = MultimodalLLM(config_path).to(self.device)
        self.tokenizer = MultimodalTokenizer(config_path)
        
        # Update model vocab size to match tokenizer
        vocab_size = self.tokenizer.get_vocab_size()
        self.model.text_model.resize_token_embeddings(vocab_size)
        self.model.output_proj = nn.Linear(
            self.config['model']['hidden_size'], 
            vocab_size
        ).to(self.device)
        
        # Initialize optimizer and scheduler
        self.optimizer = AdamW(
            self.model.parameters(),
            lr=self.config['training']['learning_rate'],
            weight_decay=self.config['training']['weight_decay']
        )
        
        self.scheduler = CosineAnnealingLR(
            self.optimizer,
            T_max=self.config['training']['epochs']
        )
        
        # Create dataloaders
        dataloaders = create_dataloaders(config_path)
        self.train_loader = dataloaders['train']
        self.val_loader = dataloaders['val']
        self.test_loader = dataloaders['test']
        
        # Set custom collate function
        collate_fn = CollateFunction()
        self.train_loader.collate_fn = collate_fn
        self.val_loader.collate_fn = collate_fn
        self.test_loader.collate_fn = collate_fn
        
        # Initialize wandb
        wandb.init(
            project="multimodal-llm",
            config=self.config,
            name=f"multimodal_llm_{datetime.now().strftime('%Y%m%d_%H%M%S')}"
        )
        
        # Create directories
        os.makedirs(self.config['paths']['model_dir'], exist_ok=True)
        os.makedirs(self.config['paths']['logs_dir'], exist_ok=True)
        
        self.global_step = 0
        self.best_val_loss = float('inf')
    
    def train_epoch(self) -> Dict[str, float]:
        """Train for one epoch"""
        self.model.train()
        total_loss = 0
        num_batches = 0
        
        progress_bar = tqdm(self.train_loader, desc="Training")
        
        for batch in progress_bar:
            # Move batch to device
            batch = {k: v.to(self.device) if isinstance(v, torch.Tensor) else v 
                    for k, v in batch.items()}
            
            # Forward pass
            outputs = self.model(**batch)
            loss = outputs['loss']
            
            # Backward pass
            self.optimizer.zero_grad()
            loss.backward()
            
            # Gradient clipping
            torch.nn.utils.clip_grad_norm_(self.model.parameters(), 1.0)
            
            self.optimizer.step()
            
            # Update metrics
            total_loss += loss.item()
            num_batches += 1
            self.global_step += 1
            
            # Update progress bar
            progress_bar.set_postfix({
                'loss': f"{loss.item():.4f}",
                'avg_loss': f"{total_loss/num_batches:.4f}"
            })
            
            # Log to wandb
            if self.global_step % 100 == 0:
                wandb.log({
                    'train/loss': loss.item(),
                    'train/learning_rate': self.optimizer.param_groups[0]['lr'],
                    'global_step': self.global_step
                })
            
            # Save checkpoint
            if self.global_step % self.config['training']['save_steps'] == 0:
                self.save_checkpoint(f"checkpoint_step_{self.global_step}")
        
        return {'train_loss': total_loss / num_batches}
    
    def validate(self) -> Dict[str, float]:
        """Validate the model"""
        self.model.eval()
        total_loss = 0
        num_batches = 0
        
        with torch.no_grad():
            for batch in tqdm(self.val_loader, desc="Validation"):
                # Move batch to device
                batch = {k: v.to(self.device) if isinstance(v, torch.Tensor) else v 
                        for k, v in batch.items()}
                
                # Forward pass
                outputs = self.model(**batch)
                loss = outputs['loss']
                
                total_loss += loss.item()
                num_batches += 1
        
        avg_loss = total_loss / num_batches
        return {'val_loss': avg_loss}
    
    def train(self):
        """Main training loop"""
        print("Starting training...")
        
        for epoch in range(self.config['training']['epochs']):
            print(f"\nEpoch {epoch + 1}/{self.config['training']['epochs']}")
            
            # Train
            train_metrics = self.train_epoch()
            
            # Validate
            val_metrics = self.validate()
            
            # Update scheduler
            self.scheduler.step()
            
            # Log metrics
            metrics = {**train_metrics, **val_metrics, 'epoch': epoch + 1}
            wandb.log(metrics)
            
            print(f"Train Loss: {train_metrics['train_loss']:.4f}")
            print(f"Val Loss: {val_metrics['val_loss']:.4f}")
            
            # Save best model
            if val_metrics['val_loss'] < self.best_val_loss:
                self.best_val_loss = val_metrics['val_loss']
                self.save_checkpoint("best_model")
                print("Saved new best model!")
            
            # Save epoch checkpoint
            self.save_checkpoint(f"epoch_{epoch + 1}")
        
        print("Training completed!")
    
    def save_checkpoint(self, name: str):
        """Save model checkpoint"""
        checkpoint = {
            'model_state_dict': self.model.state_dict(),
            'optimizer_state_dict': self.optimizer.state_dict(),
            'scheduler_state_dict': self.scheduler.state_dict(),
            'global_step': self.global_step,
            'best_val_loss': self.best_val_loss,
            'config': self.config
        }
        
        checkpoint_path = os.path.join(self.config['paths']['model_dir'], f"{name}.pt")
        torch.save(checkpoint, checkpoint_path)
        print(f"Checkpoint saved: {checkpoint_path}")
    
    def load_checkpoint(self, checkpoint_path: str):
        """Load model checkpoint"""
        checkpoint = torch.load(checkpoint_path, map_location=self.device)
        
        self.model.load_state_dict(checkpoint['model_state_dict'])
        self.optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
        self.scheduler.load_state_dict(checkpoint['scheduler_state_dict'])
        self.global_step = checkpoint['global_step']
        self.best_val_loss = checkpoint['best_val_loss']
        
        print(f"Checkpoint loaded: {checkpoint_path}")

def main():
    """Main training function"""
    trainer = MultimodalTrainer()
    trainer.train()

if __name__ == "__main__":
    main()

🔮 predict.py

import torch
import yaml
import os
import argparse
from typing import Dict, Any, Optional, Union
import json
from PIL import Image
import numpy as np

from model import MultimodalLLM
from tokenizer import MultimodalTokenizer
from utils.data_processors import MultimodalDataProcessor

class MultimodalPredictor:
    def __init__(self, 
                 model_path: str,
                 config_path: str = "config/config.yaml"):
        
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
        
        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        
        # Initialize components
        self.tokenizer = MultimodalTokenizer(config_path)
        self.processor = MultimodalDataProcessor(self.config)
        self.model = MultimodalLLM(config_path).to(self.device)
        
        # Load trained model
        self.load_model(model_path)
        self.model.eval()
    
    def load_model(self, model_path: str):
        """Load trained model weights"""
        checkpoint = torch.load(model_path, map_location=self.device)
        
        if 'model_state_dict' in checkpoint:
            self.model.load_state_dict(checkpoint['model_state_dict'])
        else:
            self.model.load_state_dict(checkpoint)
        
        print(f"Model loaded from: {model_path}")
    
    def predict_from_file(self, 
                         file_path: str,
                         prompt: str = "",
                         max_length: int = 100,
                         temperature: float = 0.7,
                         top_p: float = 0.9) -> str:
        """Generate text from a file input"""
        
        # Process the file
        processed_data = self.processor.process_file(file_path)
        
        # Combine with prompt
        if prompt:
            processed_data['text'] = f"{prompt} {processed_data['text']}"
        
        return self.predict_from_data(processed_data, max_length, temperature, top_p)
    
    def predict_from_data(self,
                         data: Dict[str, Any],
                         max_length: int = 100,
                         temperature: float = 0.7,
                         top_p: float = 0.9) -> str:
        """Generate text from processed data"""
        
        # Tokenize input
        encoded_data = self.tokenizer.encode_multimodal(data)
        
        # Move to device and add batch dimension
        batch = {}
        for key, value in encoded_data.items():
            if isinstance(value, torch.Tensor):
                batch[key] = value.unsqueeze(0).to(self.device)
        
        # Generate
        with torch.no_grad():
            generated_ids = self.model.generate(
                input_ids=batch['input_ids'],
                attention_mask=batch['attention_mask'],
                image=batch.get('image'),
                audio=batch.get('audio'),
                video=batch.get('video'),
                max_length=max_length,
                temperature=temperature,
                top_p=top_p
            )
        
        # Decode generated text
        generated_text = self.tokenizer.decode_text(generated_ids[0])
        
        # Remove input text from output
        input_text = self.tokenizer.decode_text(batch['input_ids'][0])
        if generated_text.startswith(input_text):
            generated_text = generated_text[len(input_text):].strip()
        
        return generated_text
    
    def predict_multimodal(self,
                          text: str = "",
                          image_path: Optional[str] = None,
                          audio_path: Optional[str] = None,
                          video_path: Optional[str] = None,
                          max_length: int = 100,
                          temperature: float = 0.7,
                          top_p: float = 0.9) -> str:
        """Generate text from multimodal inputs"""
        
                data = {'text': text, 'image': None, 'audio': None, 'video': None}
        
        # Process each modality
        if image_path and os.path.exists(image_path):
            image_data = self.processor.process_file(image_path, 'image')
            data['image'] = image_data['image']
        
        if audio_path and os.path.exists(audio_path):
            audio_data = self.processor.process_file(audio_path, 'audio')
            data['audio'] = audio_data['audio']
        
        if video_path and os.path.exists(video_path):
            video_data = self.processor.process_file(video_path, 'video')
            data['video'] = video_data['video']
        
        return self.predict_from_data(data, max_length, temperature, top_p)
    
    def batch_predict(self, 
                     file_paths: list,
                     prompts: Optional[list] = None,
                     max_length: int = 100,
                     temperature: float = 0.7,
                     top_p: float = 0.9) -> list:
        """Predict on multiple files"""
        
        if prompts is None:
            prompts = [""] * len(file_paths)
        
        results = []
        for file_path, prompt in zip(file_paths, prompts):
            try:
                result = self.predict_from_file(
                    file_path, prompt, max_length, temperature, top_p
                )
                results.append({
                    'file_path': file_path,
                    'prompt': prompt,
                    'prediction': result,
                    'status': 'success'
                })
            except Exception as e:
                results.append({
                    'file_path': file_path,
                    'prompt': prompt,
                    'prediction': None,
                    'status': 'error',
                    'error': str(e)
                })
        
        return results

def main():
    parser = argparse.ArgumentParser(description="Multimodal LLM Prediction")
    parser.add_argument("--model_path", required=True, help="Path to trained model")
    parser.add_argument("--config_path", default="config/config.yaml", help="Path to config file")
    parser.add_argument("--input_file", help="Input file path")
    parser.add_argument("--text", help="Input text")
    parser.add_argument("--image", help="Input image path")
    parser.add_argument("--audio", help="Input audio path")
    parser.add_argument("--video", help="Input video path")
    parser.add_argument("--prompt", default="", help="Text prompt")
    parser.add_argument("--max_length", type=int, default=100, help="Maximum generation length")
    parser.add_argument("--temperature", type=float, default=0.7, help="Generation temperature")
    parser.add_argument("--top_p", type=float, default=0.9, help="Top-p sampling parameter")
    parser.add_argument("--output_file", help="Output file path")
    
    args = parser.parse_args()
    
    # Initialize predictor
    predictor = MultimodalPredictor(args.model_path, args.config_path)
    
    # Generate prediction
    if args.input_file:
        result = predictor.predict_from_file(
            args.input_file,
            args.prompt,
            args.max_length,
            args.temperature,
            args.top_p
        )
    else:
        result = predictor.predict_multimodal(
            text=args.text or "",
            image_path=args.image,
            audio_path=args.audio,
            video_path=args.video,
            max_length=args.max_length,
            temperature=args.temperature,
            top_p=args.top_p
        )
    
    # Output result
    print("Generated Text:")
    print(result)
    
    if args.output_file:
        with open(args.output_file, 'w') as f:
            f.write(result)
        print(f"Result saved to: {args.output_file}")

if __name__ == "__main__":
    main()

🧪 evaluate.py

import torch
import yaml
import os
import json
from tqdm import tqdm
from typing import Dict, List, Any
import numpy as np
from sklearn.metrics import accuracy_score, f1_score
import matplotlib.pyplot as plt
import seaborn as sns

from model import MultimodalLLM
from dataset import create_dataloaders
from tokenizer import MultimodalTokenizer
from predict import MultimodalPredictor

class MultimodalEvaluator:
    def __init__(self, 
                 model_path: str,
                 config_path: str = "config/config.yaml"):
        
        with open(config_path, 'r') as f:
            self.config = yaml.safe_load(f)
        
        self.device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
        
        # Initialize components
        self.predictor = MultimodalPredictor(model_path, config_path)
        self.tokenizer = MultimodalTokenizer(config_path)
        
        # Create test dataloader
        dataloaders = create_dataloaders(config_path)
        self.test_loader = dataloaders['test']
    
    def evaluate_perplexity(self) -> float:
        """Calculate perplexity on test set"""
        self.predictor.model.eval()
        total_loss = 0
        total_tokens = 0
        
        with torch.no_grad():
            for batch in tqdm(self.test_loader, desc="Calculating perplexity"):
                # Move batch to device
                batch = {k: v.to(self.device) if isinstance(v, torch.Tensor) else v 
                        for k, v in batch.items()}
                
                outputs = self.predictor.model(**batch)
                loss = outputs['loss']
                
                # Count tokens (excluding padding)
                mask = batch['attention_mask']
                num_tokens = mask.sum().item()
                
                total_loss += loss.item() * num_tokens
                total_tokens += num_tokens
        
        avg_loss = total_loss / total_tokens
        perplexity = torch.exp(torch.tensor(avg_loss)).item()
        
        return perplexity
    
    def evaluate_generation_quality(self, 
                                   num_samples: int = 100,
                                   max_length: int = 50) -> Dict[str, float]:
        """Evaluate generation quality metrics"""
        
        generated_texts = []
        reference_texts = []
        
        # Generate samples
        count = 0
        for batch in tqdm(self.test_loader, desc="Generating samples"):
            if count >= num_samples:
                break
            
            batch_size = batch['input_ids'].size(0)
            
            for i in range(min(batch_size, num_samples - count)):
                # Prepare input (first half of sequence)
                input_ids = batch['input_ids'][i]
                seq_len = input_ids.size(0)
                split_point = seq_len // 2
                
                input_part = input_ids[:split_point]
                reference_part = input_ids[split_point:]
                
                # Create input data
                input_data = {
                    'text': self.tokenizer.decode_text(input_part),
                    'image': batch.get('image', [None])[i] if batch.get('image') is not None else None,
                    'audio': batch.get('audio', [None])[i] if batch.get('audio') is not None else None,
                    'video': batch.get('video', [None])[i] if batch.get('video') is not None else None
                }
                
                # Generate
                generated_text = self.predictor.predict_from_data(
                    input_data, max_length=max_length
                )
                reference_text = self.tokenizer.decode_text(reference_part)
                
                generated_texts.append(generated_text)
                reference_texts.append(reference_text)
                
                count += 1
                if count >= num_samples:
                    break
        
        # Calculate metrics
        metrics = self._calculate_text_metrics(generated_texts, reference_texts)
        
        return metrics
    
    def _calculate_text_metrics(self, 
                               generated_texts: List[str], 
                               reference_texts: List[str]) -> Dict[str, float]:
        """Calculate text generation metrics"""
        
        # BLEU score (simplified)
        bleu_scores = []
        for gen, ref in zip(generated_texts, reference_texts):
            gen_tokens = set(gen.lower().split())
            ref_tokens = set(ref.lower().split())
            
            if len(ref_tokens) == 0:
                bleu_scores.append(0.0)
            else:
                overlap = len(gen_tokens.intersection(ref_tokens))
                bleu_scores.append(overlap / len(ref_tokens))
        
        # Length statistics
        gen_lengths = [len(text.split()) for text in generated_texts]
        ref_lengths = [len(text.split()) for text in reference_texts]
        
        # Diversity (unique n-grams)
        all_generated = " ".join(generated_texts)
        tokens = all_generated.lower().split()
        
        unigrams = set(tokens)
        bigrams = set(zip(tokens[:-1], tokens[1:]))
        trigrams = set(zip(tokens[:-2], tokens[1:-1], tokens[2:]))
        
        diversity_1 = len(unigrams) / len(tokens) if len(tokens) > 0 else 0
        diversity_2 = len(bigrams) / len(tokens) if len(tokens) > 1 else 0
        diversity_3 = len(trigrams) / len(tokens) if len(tokens) > 2 else 0
        
        return {
            'bleu_score': np.mean(bleu_scores),
            'avg_gen_length': np.mean(gen_lengths),
            'avg_ref_length': np.mean(ref_lengths),
            'length_ratio': np.mean(gen_lengths) / np.mean(ref_lengths) if np.mean(ref_lengths) > 0 else 0,
            'diversity_1': diversity_1,
            'diversity_2': diversity_2,
            'diversity_3': diversity_3
        }
    
    def evaluate_modality_impact(self, num_samples: int = 50) -> Dict[str, Dict[str, float]]:
        """Evaluate impact of different modalities"""
        
        results = {
            'text_only': [],
            'text_image': [],
            'text_audio': [],
            'text_video': [],
            'all_modalities': []
        }
        
        count = 0
        for batch in tqdm(self.test_loader, desc="Evaluating modality impact"):
            if count >= num_samples:
                break
            
            batch_size = batch['input_ids'].size(0)
            
            for i in range(min(batch_size, num_samples - count)):
                # Prepare base input
                input_ids = batch['input_ids'][i]
                text = self.tokenizer.decode_text(input_ids[:len(input_ids)//2])
                
                image = batch.get('image', [None])[i] if batch.get('image') is not None else None
                audio = batch.get('audio', [None])[i] if batch.get('audio') is not None else None
                video = batch.get('video', [None])[i] if batch.get('video') is not None else None
                
                # Test different modality combinations
                combinations = {
                    'text_only': {'text': text, 'image': None, 'audio': None, 'video': None},
                    'text_image': {'text': text, 'image': image, 'audio': None, 'video': None},
                    'text_audio': {'text': text, 'image': None, 'audio': audio, 'video': None},
                    'text_video': {'text': text, 'image': None, 'audio': None, 'video': video},
                    'all_modalities': {'text': text, 'image': image, 'audio': audio, 'video': video}
                }
                
                for combo_name, combo_data in combinations.items():
                    try:
                        generated = self.predictor.predict_from_data(combo_data, max_length=30)
                        results[combo_name].append(len(generated.split()))
                    except:
                        results[combo_name].append(0)
                
                count += 1
                if count >= num_samples:
                    break
        
        # Calculate statistics
        stats = {}
        for combo_name, lengths in results.items():
            stats[combo_name] = {
                'avg_length': np.mean(lengths),
                'std_length': np.std(lengths),
                'min_length': np.min(lengths),
                'max_length': np.max(lengths)
            }
        
        return stats
    
    def generate_evaluation_report(self, output_dir: str = "evaluation_results"):
        """Generate comprehensive evaluation report"""
        
        os.makedirs(output_dir, exist_ok=True)
        
        print("Starting comprehensive evaluation...")
        
        # 1. Perplexity
        print("Calculating perplexity...")
        perplexity = self.evaluate_perplexity()
        
        # 2. Generation quality
        print("Evaluating generation quality...")
        generation_metrics = self.evaluate_generation_quality()
        
        # 3. Modality impact
        print("Evaluating modality impact...")
        modality_impact = self.evaluate_modality_impact()
        
        # Compile results
        results = {
            'perplexity': perplexity,
            'generation_metrics': generation_metrics,
            'modality_impact': modality_impact,
            'model_config': self.config
        }
        
        # Save results
        with open(os.path.join(output_dir, 'evaluation_results.json'), 'w') as f:
            json.dump(results, f, indent=2)
        
        # Generate plots
        self._generate_plots(results, output_dir)
        
        # Generate summary report
        self._generate_summary_report(results, output_dir)
        
        print(f"Evaluation complete! Results saved to {output_dir}")
        
        return results
    
    def _generate_plots(self, results: Dict, output_dir: str):
        """Generate evaluation plots"""
        
        # Modality impact plot
        modality_data = results['modality_impact']
        modalities = list(modality_data.keys())
        avg_lengths = [modality_data[mod]['avg_length'] for mod in modalities]
        
        plt.figure(figsize=(10, 6))
        bars = plt.bar(modalities, avg_lengths)
        plt.title('Average Generation Length by Modality Combination')
        plt.xlabel('Modality Combination')
        plt.ylabel('Average Length (words)')
        plt.xticks(rotation=45)
        
        # Add value labels on bars
        for bar, length in zip(bars, avg_lengths):
            plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.1,
                    f'{length:.1f}', ha='center', va='bottom')
        
        plt.tight_layout()
        plt.savefig(os.path.join(output_dir, 'modality_impact.png'), dpi=300, bbox_inches='tight')
        plt.close()
        
        # Generation metrics plot
        gen_metrics = results['generation_metrics']
                # Generation metrics plot
        gen_metrics = results['generation_metrics']
        metrics_to_plot = ['bleu_score', 'diversity_1', 'diversity_2', 'diversity_3']
        metric_values = [gen_metrics[metric] for metric in metrics_to_plot]
        
        plt.figure(figsize=(10, 6))
        bars = plt.bar(metrics_to_plot, metric_values, color=['skyblue', 'lightgreen', 'lightcoral', 'gold'])
        plt.title('Text Generation Quality Metrics')
        plt.xlabel('Metrics')
        plt.ylabel('Score')
        plt.xticks(rotation=45)
        
        # Add value labels on bars
        for bar, value in zip(bars, metric_values):
            plt.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 0.01,
                    f'{value:.3f}', ha='center', va='bottom')
        
        plt.tight_layout()
        plt.savefig(os.path.join(output_dir, 'generation_metrics.png'), dpi=300, bbox_inches='tight')
        plt.close()
    
    def _generate_summary_report(self, results: Dict, output_dir: str):
        """Generate text summary report"""
        
        report = f"""
# Multimodal LLM Evaluation Report

## Model Performance Summary

### Perplexity
- **Test Perplexity**: {results['perplexity']:.2f}

### Generation Quality Metrics
- **BLEU Score**: {results['generation_metrics']['bleu_score']:.3f}
- **Average Generated Length**: {results['generation_metrics']['avg_gen_length']:.1f} words
- **Average Reference Length**: {results['generation_metrics']['avg_ref_length']:.1f} words
- **Length Ratio**: {results['generation_metrics']['length_ratio']:.3f}
- **Diversity-1 (Unigrams)**: {results['generation_metrics']['diversity_1']:.3f}
- **Diversity-2 (Bigrams)**: {results['generation_metrics']['diversity_2']:.3f}
- **Diversity-3 (Trigrams)**: {results['generation_metrics']['diversity_3']:.3f}

### Modality Impact Analysis
"""
        
        for modality, stats in results['modality_impact'].items():
            report += f"""
#### {modality.replace('_', ' ').title()}
- Average Length: {stats['avg_length']:.1f} ± {stats['std_length']:.1f} words
- Range: {stats['min_length']:.0f} - {stats['max_length']:.0f} words
"""
        
        report += f"""
## Model Configuration
- Hidden Size: {results['model_config']['model']['hidden_size']}
- Number of Attention Heads: {results['model_config']['model']['num_attention_heads']}
- Number of Layers: {results['model_config']['model']['num_hidden_layers']}
- Vision Model: {results['model_config']['model']['vision_model']}
- Audio Model: {results['model_config']['model']['audio_model']}

## Recommendations

### Performance Analysis
"""
        
        # Add performance analysis
        perplexity = results['perplexity']
        if perplexity < 20:
            report += "- ✅ **Excellent** perplexity score indicates strong language modeling capability\n"
        elif perplexity < 50:
            report += "- ⚠️ **Good** perplexity score with room for improvement\n"
        else:
            report += "- ❌ **Poor** perplexity score suggests need for more training or model adjustments\n"
        
        bleu = results['generation_metrics']['bleu_score']
        if bleu > 0.3:
            report += "- ✅ **Good** BLEU score indicates quality text generation\n"
        elif bleu > 0.15:
            report += "- ⚠️ **Moderate** BLEU score suggests acceptable generation quality\n"
        else:
            report += "- ❌ **Low** BLEU score indicates poor generation quality\n"
        
        diversity = results['generation_metrics']['diversity_1']
        if diversity > 0.7:
            report += "- ✅ **High** diversity indicates varied and creative text generation\n"
        elif diversity > 0.5:
            report += "- ⚠️ **Moderate** diversity with some repetition in generated text\n"
        else:
            report += "- ❌ **Low** diversity suggests repetitive text generation\n"
        
        # Save report
        with open(os.path.join(output_dir, 'evaluation_report.md'), 'w') as f:
            f.write(report)

def main():
    import argparse
    
    parser = argparse.ArgumentParser(description="Evaluate Multimodal LLM")
    parser.add_argument("--model_path", required=True, help="Path to trained model")
    parser.add_argument("--config_path", default="config/config.yaml", help="Path to config file")
    parser.add_argument("--output_dir", default="evaluation_results", help="Output directory")
    parser.add_argument("--num_samples", type=int, default=100, help="Number of samples for evaluation")
    
    args = parser.parse_args()
    
    evaluator = MultimodalEvaluator(args.model_path, args.config_path)
    results = evaluator.generate_evaluation_report(args.output_dir)
    
    print("\n" + "="*50)
    print("EVALUATION SUMMARY")
    print("="*50)
    print(f"Perplexity: {results['perplexity']:.2f}")
    print(f"BLEU Score: {results['generation_metrics']['bleu_score']:.3f}")
    print(f"Diversity-1: {results['generation_metrics']['diversity_1']:.3f}")
    print("="*50)

if __name__ == "__main__":
    main()

🚀 run.py
#!/usr/bin/env python3
"""
Main runner script for the Multimodal LLM project
Provides a unified interface for training, evaluation, and inference
"""

import argparse
import os
import sys
import yaml
from typing import Dict, Any

def setup_environment():
    """Setup environment and check dependencies"""
    try:
        import torch
        import transformers
        import datasets
        import wandb
        print("✅ All dependencies available")
        print(f"PyTorch version: {torch.__version__}")
        print(f"CUDA available: {torch.cuda.is_available()}")
        if torch.cuda.is_available():
            print(f"CUDA devices: {torch.cuda.device_count()}")
    except ImportError as e:
        print(f"❌ Missing dependency: {e}")
        sys.exit(1)

def train_model(config_path: str, resume_from: str = None):
    """Train the multimodal model"""
    from train import MultimodalTrainer
    
    print("🚀 Starting training...")
    trainer = MultimodalTrainer(config_path)
    
    if resume_from:
        print(f"📂 Resuming from checkpoint: {resume_from}")
        trainer.load_checkpoint(resume_from)
    
    trainer.train()
    print("✅ Training completed!")

def evaluate_model(model_path: str, config_path: str, output_dir: str):
    """Evaluate the trained model"""
    from evaluate import MultimodalEvaluator
    
    print("📊 Starting evaluation...")
    evaluator = MultimodalEvaluator(model_path, config_path)
    results = evaluator.generate_evaluation_report(output_dir)
    print("✅ Evaluation completed!")
    return results

def run_inference(model_path: str, 
                 config_path: str,
                 input_data: Dict[str, Any],
                 output_file: str = None):
    """Run inference on input data"""
    from predict import MultimodalPredictor
    
    print("🔮 Running inference...")
    predictor = MultimodalPredictor(model_path, config_path)
    
    if 'file_path' in input_data:
        result = predictor.predict_from_file(
            input_data['file_path'],
            input_data.get('prompt', ''),
            input_data.get('max_length', 100),
            input_data.get('temperature', 0.7),
            input_data.get('top_p', 0.9)
        )
    else:
        result = predictor.predict_multimodal(
            text=input_data.get('text', ''),
            image_path=input_data.get('image_path'),
            audio_path=input_data.get('audio_path'),
            video_path=input_data.get('video_path'),
            max_length=input_data.get('max_length', 100),
            temperature=input_data.get('temperature', 0.7),
            top_p=input_data.get('top_p', 0.9)
        )
    
    print("Generated text:")
    print("-" * 50)
    print(result)
    print("-" * 50)
    
    if output_file:
        with open(output_file, 'w') as f:
            f.write(result)
        print(f"💾 Result saved to: {output_file}")
    
    return result

def setup_project():
    """Setup project structure and download required models"""
    print("🔧 Setting up project...")
    
    # Create directories
    directories = [
        "data/raw",
        "data/processed", 
        "models/checkpoints",
        "models/pretrained",
        "logs",
        "evaluation_results",
        "outputs"
    ]
    
    for directory in directories:
        os.makedirs(directory, exist_ok=True)
        print(f"📁 Created directory: {directory}")
    
    # Check config file
    if not os.path.exists("config/config.yaml"):
        print("⚠️ Config file not found. Please ensure config/config.yaml exists.")
        return False
    
    print("✅ Project setup completed!")
    return True

def validate_config(config_path: str) -> bool:
    """Validate configuration file"""
    try:
        with open(config_path, 'r') as f:
            config = yaml.safe_load(f)
        
        required_sections = ['model', 'training', 'data', 'paths']
        for section in required_sections:
            if section not in config:
                print(f"❌ Missing required config section: {section}")
                return False
        
        print("✅ Configuration file is valid")
        return True
    
    except Exception as e:
        print(f"❌ Error validating config: {e}")
        return False

def main():
    parser = argparse.ArgumentParser(
        description="Multimodal LLM - Unified Training, Evaluation, and Inference",
        formatter_class=argparse.RawDescriptionHelpFormatter,
        epilog="""
Examples:
  # Setup project
  python run.py setup
  
  # Train model
  python run.py train --config config/config.yaml
  
  # Resume training
  python run.py train --config config/config.yaml --resume models/checkpoints/checkpoint_step_1000.pt
  
  # Evaluate model
  python run.py evaluate --model models/checkpoints/best_model.pt --output evaluation_results
  
  # Run inference with text
  python run.py predict --model models/checkpoints/best_model.pt --text "Describe this image" --image path/to/image.jpg
  
  # Run inference with file
  python run.py predict --model models/checkpoints/best_model.pt --file path/to/multimodal_file.json
        """
    )
    
    subparsers = parser.add_subparsers(dest='command', help='Available commands')
    
    # Setup command
    setup_parser = subparsers.add_parser('setup', help='Setup project structure')
    
    # Train command
    train_parser = subparsers.add_parser('train', help='Train the model')
    train_parser.add_argument('--config', default='config/config.yaml', help='Config file path')
    train_parser.add_argument('--resume', help='Resume from checkpoint')
    
    # Evaluate command
    eval_parser = subparsers.add_parser('evaluate', help='Evaluate the model')
    eval_parser.add_argument('--model', required=True, help='Model checkpoint path')
    eval_parser.add_argument('--config', default='config/config.yaml', help='Config file path')
    eval_parser.add_argument('--output', default='evaluation_results', help='Output directory')
    
    # Predict command
    predict_parser = subparsers.add_parser('predict', help='Run inference')
    predict_parser.add_argument('--model', required=True, help='Model checkpoint path')
    predict_parser.add_argument('--config', default='config/config.yaml', help='Config file path')
    predict_parser.add_argument('--file', help='Input file path')
    predict_parser.add_argument('--text', help='Input text')
    predict_parser.add_argument('--image', help='Input image path')
    predict_parser.add_argument('--audio', help='Input audio path')
    predict_parser.add_argument('--video', help='Input video path')
    predict_parser.add_argument('--prompt', default='', help='Text prompt')
    predict_parser.add_argument('--max_length', type=int, default=100, help='Max generation length')
    predict_parser.add_argument('--temperature', type=float, default=0.7, help='Generation temperature')
    predict_parser.add_argument('--top_p', type=float, default=0.9, help='Top-p sampling')
    predict_parser.add_argument('--output', help='Output file path')
    
    # Validate command
    validate_parser = subparsers.add_parser('validate', help='Validate configuration')
    validate_parser.add_argument('--config', default='config/config.yaml', help='Config file path')
    
    args = parser.parse_args()
    
    if args.command is None:
        parser.print_help()
        return
    
    # Setup environment
    setup_environment()
    
    if args.command == 'setup':
        setup_project()
    
    elif args.command == 'validate':
        validate_config(args.config)
    
    elif args.command == 'train':
        if not validate_config(args.config):
            sys.exit(1)
        train_model(args.config, args.resume)
    
    elif args.command == 'evaluate':
        if not validate_config(args.config):
            sys.exit(1)
        evaluate_model(args.model, args.config, args.output)
    
    elif args.command == 'predict':
        if not validate_config(args.config):
            sys.exit(1)
        
        input_data = {}
        if args.file:
            input_data['file_path'] = args.file
        else:
            input_data.update({
                'text': args.text or '',
                'image_path': args.image,
                'audio_path': args.audio,
                'video_path': args.video,
                'prompt': args.prompt,
                'max_length': args.max_length,
                'temperature': args.temperature,
                'top_p': args.top_p
            })
        
        run_inference(args.model, args.config, input_data, args.output)

if __name__ == "__main__":
    main()



