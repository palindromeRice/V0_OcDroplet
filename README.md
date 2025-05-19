# 🧠 CB Chatbot

A unified, multi-modal chatbot for code assistance, supporting both **text and audio input**, with **language detection**, **code verification**, and **correction**.

**Backend:** FastAPI + Ollama + Whisper + Redis  
**Frontend:** Streamlit  
**Docker Image:** `palindromerice/chatbot-app:ocV0.1-inplace`  
**Live:** [http://206.189.134.128:8501/](http://206.189.134.128:8501/)

---

## 📦 Tutorials

### 🚀 Getting Started

This guide helps you run the chatbot using a prebuilt Docker image.

#### Prerequisites

- Docker & Docker Buildx (for cross-platform compatibility)
- Redis server running locally or on the host machine
- Ollama server with required models running on the host machine
- (Optional) GPU for Whisper and LLM acceleration

---

## 🐳 Quick Start (Docker Image)

### 🧱 Pull the Image

You do not need to build the image manually — just pull and run the container using the steps below.

```bash
docker pull palindromerice/chatbot-app:ocV0.1-inplace
```




---

### 🚀 Run the Container

```bash
docker run -d --name chatbot_demo \
  --network host \
  -v /root/model:/model \
  -v /root/whisper_cache:/root/.cache/whisper \
  -e REDIS_HOST=127.0.0.1 \
  -e REDIS_PORT=6379 \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434/api \
  -e LANG_MODEL_PATH=/model/programming-language-identification \
  -e OLLAMA_MODEL=qwen2.5-coder:3b \
  -e OLLAMA_CORRECTION_MODEL=smollm2:135m \
  -e CHATBOT_CHAT_URL="http://127.0.0.1:8000/chat/" \
  -e CHATBOT_CLEAR_URL="http://127.0.0.1:8000/clear_session/" \
  -e USE_STRUCTURED_MODEL=false \
  palindromerice/chatbot-app:ocV0.1-inplace \
  bash /aws_chatbot/start_services.sh
```


Access the services:

- **Frontend UI:** [http://206.189.134.128:8501/](http://206.189.134.128:8501/)
- **Backend API:** [http://206.189.134.128:8000/](http://206.189.134.128:8000/)

---

## 🛠️ How-To Guides

### 🔁 Clear a Chat Session

- Enter your **User ID** in the Streamlit sidebar
- Click **"🔁 Reset Session"**
- Chat history will be cleared

### 🧠 Detect Programming Language

You can use this endpoint to detect the programming language of any code snippet, which is especially useful for auto-formatting code blocks in the Streamlit frontend.

---

#### 🔁 Endpoint
`POST /detect_language/`

#### 📤 Request Body
```json
{
  "code": "def hello(): print('hi')"
}
```

#### 📥 Response
```json
{
  "language": "python"
}
```

---

### 🖼️ Usage in Streamlit for Auto Formatting

To auto-format code blocks using the detected language, call this endpoint before rendering the code with `st.code`.

### ➕ Add a New Language Detection Model

1. Place your model inside `/model/` or mount it via Docker
2. Set the `LANG_MODEL_PATH` environment variable
3. Restart the container

### 🎤 Use Audio Input

- Enable **"🎤 Chat using Audio"** in the UI
- Record and send your message
- Backend uses Whisper to transcribe and respond

---

## 🧬 System Architecture

### Frontend: `streamlit_audio_struct.py`

- Handles user interaction via Streamlit
- Supports both text and audio inputs
- Calls the FastAPI backend for processing

### Backend: `audio_struct_backend.py`

- Built on FastAPI
- Handles chat logic, transcription, language detection
- Serves:
  - `/chat/` endpoint for unified chat (text/audio)
  - `/clear/` to reset session
  - `/detect_language/` to infer code language

### Redis

- Stores user chat history per session for continuity

### Ollama

- LLMs for generating, verifying, and correcting code
- `OLLAMA_MODEL` is used for generation
- `OLLAMA_CORRECTION_MODEL` for improving responses

---

## ⚙️ Design Details

### 📤 Streaming Responses

- Supports FastAPI’s `StreamingResponse` for markdown output
- Streamlit does **not** support streaming structured (JSON) responses — structured outputs are shown only after full response is received

### 🧹 Session Management

- Each session is user-specific and stored in Redis with expiry

### 🔊 Audio Transcription

- Uses OpenAI Whisper for transcribing `.mp3` and `.wav` audio files

### 🧠 Language Detection

- Uses a HuggingFace model for identifying programming languages from code snippets.
- **Current model:** [`philomath-1209/programming-language-identification`](https://huggingface.co/philomath-1209/programming-language-identification)
- You can replace this with any other compatible `AutoModelForSequenceClassification` model by updating the `LANG_MODEL_PATH` environment variable.
- This detection helps apply correct syntax highlighting and formatting in the frontend.


### 🧾 Structured Output

- LLMs format replies into separate explanation and code blocks
- Enabled by setting `USE_STRUCTURED_MODEL=true`

---

## 🔗 API Endpoints

### `POST /chat/`

Handles both text and audio input.

**Form Parameters:**

- `user_id`: User identifier (required)
- `message`: Text input (optional)
- `file`: Audio file (optional, `.mp3` / `.wav`)
- `stream`: Boolean to enable streaming (optional)

Returns: Markdown (if streamed) or structured JSON (`explanation` and `code`)

---

### `POST /clear/`

Clears the user's session.

```json
{
  "message": "Chat session cleared successfully!"
}
```

---

### `POST /detect_language/`

Detects the programming language.

Request:

```json
{
  "code": "print('hello')"
}
```

Response:

```json
{
  "language": "python"
}
```

---

## 🌍 Environment Variables

| Variable                  | Description                                        | Value in Deployment                         |
|---------------------------|----------------------------------------------------|----------------------------------------------|
| `OLLAMA_BASE_URL`         | Ollama API URL                                     | `http://172.17.0.1:11434/api`               |
| `REDIS_HOST`              | Redis host address                                 | `172.17.0.1`                                |
| `REDIS_PORT`              | Redis port                                         | `6379`                                      |
| `LANG_MODEL_PATH`         | Language model path                                | `/model/programming-language-identification` |
| `CHATBOT_CHAT_URL`        | Chat endpoint URL                                  | `http://127.0.0.1:8000/chat/`               |
| `CHATBOT_CLEAR_URL`       | Clear session endpoint                             | `http://127.0.0.1:8000/clear_session/"`              |
| `USE_STRUCTURED_MODEL`    | Enable structured output (`true` / `false`)        | `true`                                      |
| `OLLAMA_MODEL`            | Generation model                                   | `qwen2.5-coder:3b`                          |
| `Structured Output Model` | Correction model                                   | `smollm2:135m`                              |

---

## 📁 File Structure

```
.
├── audio_struct_backend.py       # FastAPI backend logic
├── streamlit_audio_struct.py     # Streamlit UI
├── requirements.txt              # Python dependencies
├── Dockerfile                    # Docker build config
├── start_services.sh             # Entrypoint script
└── /model/                       # Mounted language models
```

---

## 📦 Dependencies

Key packages from `requirements.txt`:

- `fastapi`, `uvicorn`, `gunicorn`, `httpx`
- `streamlit`, `streamlit-mic-recorder`
- `redis`, `transformers`, `torch`, `sentence-transformers`
- `httpx-sse`, `python-multipart`, `opentelemetry`

---

## 📎 Streaming & Output Note

- Markdown response is streamed in real time
- Structured responses are shown only **after** the full reply is received (Streamlit limitation)
- For interactive structured output, use a frontend that supports streaming JSON

---

## 🙋 Support

See inline code comments and docstrings for guidance.  
For issues or questions, reach out to me.

---
