Here's the complete FastAPI application with audio generation functionality integrated:

```python
from openai import OpenAI
from dotenv import load_dotenv
import os 
from fastapi import FastAPI, HTTPException
from fastapi.responses import FileResponse
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
import tempfile
import uuid

load_dotenv()

client = OpenAI(api_key=os.getenv("OPEN_API_KEY"))

app = FastAPI()

app.add_middleware(CORSMiddleware, 
                   allow_origins=["*"],
                   allow_credentials=True,
                   allow_methods=["*"],
                   allow_headers=["*"])

# Pydantic models for request bodies
class AudioRequest(BaseModel):
    text: str
    voice: str = "alloy"  # default voice

class ChatRequest(BaseModel):
    question: str

@app.get("/")
def home():
    return {"message": "Welcome to FastAPI with OpenAI Integration"}

@app.post("/generate-audio")
def generate_audio(request: AudioRequest):
    """
    Generate audio from text using OpenAI TTS
    """
    try:
        if not request.text.strip():
            raise HTTPException(status_code=400, detail="Text cannot be empty")
        
        # Create a unique filename
        audio_filename = f"audio_{uuid.uuid4().hex}.mp3"
        audio_path = os.path.join(tempfile.gettempdir(), audio_filename)
        
        # Generate audio using OpenAI TTS
        with client.audio.speech.with_streaming_response.create(
            model="tts-1",  # Use correct model name
            voice=request.voice,
            input=request.text,
        ) as response:
            response.stream_to_file(audio_path)
        
        # Return the audio file
        return FileResponse(
            path=audio_path,
            media_type="audio/mpeg",
            filename=audio_filename,
            headers={"Content-Disposition": f"attachment; filename={audio_filename}"}
        )
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error generating audio: {str(e)}")

@app.post("/ask")
def ask_ai(request: ChatRequest):
    """
    Ask AI a question using OpenAI Chat Completions
    """
    try:
        if not request.question.strip():
            raise HTTPException(status_code=400, detail="Question cannot be empty")
        
        response = client.chat.completions.create(
            model="gpt-3.5-turbo",  # Use correct model name
            messages=[
                {"role": "user", "content": request.question}
            ]
        )
        
        return {"answer": response.choices[0].message.content}
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error getting AI response: {str(e)}")

@app.get("/voices")
def get_available_voices():
    """
    Get list of available voices for TTS
    """
    return {
        "voices": ["alloy", "echo", "fable", "onyx", "nova", "shimmer"]
    }

# Alternative GET endpoint for simple audio generation (for testing)
@app.get("/generate-audio-simple")
def generate_audio_simple(text: str, voice: str = "alloy"):
    """
    Simple GET endpoint for audio generation (for easy testing)
    """
    try:
        if not text.strip():
            raise HTTPException(status_code=400, detail="Text parameter cannot be empty")
        
        # Create a unique filename
        audio_filename = f"audio_{uuid.uuid4().hex}.mp3"
        audio_path = os.path.join(tempfile.gettempdir(), audio_filename)
        
        # Generate audio using OpenAI TTS
        with client.audio.speech.with_streaming_response.create(
            model="tts-1",
            voice=voice,
            input=text,
        ) as response:
            response.stream_to_file(audio_path)
        
        # Return the audio file
        return FileResponse(
            path=audio_path,
            media_type="audio/mpeg",
            filename=audio_filename
        )
        
    except Exception as e:
        raise HTTPException(status_code=500, detail=f"Error generating audio: {str(e)}")

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Key Changes Made:

1. **Fixed Model Names**: 
   - Changed `gpt-4o-mini-tts` to `tts-1` (correct TTS model)
   - Changed `gpt-5` to `gpt-3.5-turbo` (correct chat model)

2. **Fixed API Calls**:
   - Used `client.audio.speech.with_streaming_response.create()` for TTS
   - Used `client.chat.completions.create()` for chat completions

3. **Added Pydantic Models**: For request validation

4. **File Handling**: 
   - Uses temporary directory for audio files
   - Generates unique filenames using UUID
   - Returns audio files as FileResponse

5. **Error Handling**: Added proper exception handling with HTTPException

6. **Multiple Endpoints**:
   - `POST /generate-audio`: Main audio generation endpoint
   - `GET /generate-audio-simple`: Simple GET endpoint for testing
   - `POST /ask`: Chat completion endpoint
   - `GET /voices`: Lists available voices

## How to Use:

1. **Start the server**:
```bash
uvicorn main:app --reload
```

2. **Generate Audio (POST)**:
```bash
curl -X POST "http://localhost:8000/generate-audio" \
     -H "Content-Type: application/json" \
     -d '{"text": "guru brahma guru vishnu guru devo maheswara", "voice": "alloy"}' \
     --output audio.mp3
```

3. **Generate Audio (GET - for testing)**:
```bash
curl "http://localhost:8000/generate-audio-simple?text=Hello%20World&voice=alloy" \
     --output audio.mp3
```

4. **Ask AI**:
```bash
curl -X POST "http://localhost:8000/ask" \
     -H "Content-Type: application/json" \
     -d '{"question": "What is FastAPI?"}'
```

5. **Get Available Voices**:
```bash
curl "http://localhost:8000/voices"
```

The API documentation will be available at `http://localhost:8000/docs` when you run the server.
