# RAG Medical Chatbot

A Retrieval-Augmented Generation (RAG) chat application that answers medical questions grounded in a Medical Encyclopedia PDF. The system retrieves relevant context from the source document and passes it to an LLM to generate accurate, context-aware answers.

```
PDF → documents → chunks → embeddings → vector store → retrieval → LLM → answer
```

## Overview

Instead of relying purely on an LLM's parametric knowledge, this app builds a searchable knowledge base from a medical encyclopedia and retrieves the most relevant passages for each user question before generating a response. This reduces hallucination and grounds answers in a trusted source.

## Architecture

The project is organized into two pipelines plus a delivery layer.

### 1. Knowledge layer (offline / preparation pipeline)

```
Medical Encyclopedia PDF
        ↓
    PDF Loader (PyPDF)
        ↓
     Documents
        ↓
      Chunks (LangChain text splitter)
        ↓
  Embedding Model (Hugging Face)
        ↓
    Embeddings
        ↓
      FAISS (vector store)
```

### 2. Intelligence layer (online / question-answering pipeline)

```
User question
     ↓
  Retriever (FAISS similarity search)
     ↓
Relevant chunks
     ↓
Mistral LLM (Hugging Face) + retrieved context
     ↓
Answer
```

### 3. Application layer

```
Browser
   ↓
HTML/CSS (frontend)
   ↓
Flask (backend)
   ↓
Retrieval + LLM
   ↓
Flask
   ↓
Browser
```

### 4. Delivery layer (CI/CD)

```
Developer → GitHub → Jenkins → Docker build → Trivy security scan → AWS ECR → AWS runtime
```

## Tech Stack

| Component        | Technology                              |
|-------------------|------------------------------------------|
| LLM               | Mistral (via Hugging Face)               |
| Embeddings        | Hugging Face embedding model             |
| Orchestration     | LangChain                                |
| Vector store      | FAISS (local, CPU)                       |
| PDF parsing       | PyPDF                                    |
| Backend           | Flask                                    |
| Frontend          | HTML / CSS                               |
| Secrets           | python-dotenv                            |
| Containerization  | Docker                                   |
| Security scanning | Trivy (Aqua Trivy)                       |
| Source control    | GitHub                                   |
| CI/CD             | Jenkins                                  |
| Container registry| AWS ECR                                  |
| Runtime           | AWS                                      |

## Project Structure

```
RAG-Chatbot/
│
├── .env                        # Hugging Face API token (not committed)
├── requirements.txt
├── setup.py
│
├── data/
│   └── medical_encyclopedia.pdf
│
├── app/
│   ├── __init__.py
│   │
│   ├── common/
│   │   ├── __init__.py
│   │   ├── logger.py            # centralized logging
│   │   └── custom_exception.py  # custom exception classes
│   │
│   ├── components/
│   │   ├── pdf_loader/          # reads PDF into LangChain documents
│   │   ├── data_loader/         # chunking of documents
│   │   ├── llm/                 # Mistral LLM integration
│   │   └── retriever/           # FAISS similarity search
│   │
│   ├── config/                  # API keys, model settings
│   │
│   └── templates/                # Flask HTML templates
│
├── Dockerfile
├── Jenkinsfile
└── venv/
```

## Setup

### Prerequisites

- Python 3.10+
- A Hugging Face account and API token

### Installation

```bash
# Clone the repository
git clone git@github.com:sitnikovdev/RAG-Chatbot.git
cd RAG-Chatbot

# Create and activate a virtual environment
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# Install the project and its dependencies
pip install -e .
```

### Environment variables

Create a `.env` file in the project root:

```
HUGGINGFACEHUB_API_TOKEN=your_hugging_face_token_here
```

### Add source data

Place your medical encyclopedia PDF at:

```
data/medical_encyclopedia.pdf
```

(Any PDF knowledge base can be used instead.)

## Running the app

```bash
flask run
# or
python app/app.py
```

The app will be available at `http://localhost:5000`.

## How it works

1. **Ingestion (offline)**: the PDF is loaded and split into chunks, each chunk is embedded and stored in a local FAISS index.
2. **Retrieval (online)**: when a user asks a question, the retriever embeds the question and performs a similarity search against the FAISS index to find the most relevant chunks.
3. **Generation**: the retrieved chunks are passed to the Mistral LLM as context, which generates the final answer.
4. **Delivery**: Flask serves the request/response cycle between the browser and the RAG pipeline.

## Docker

Build and run the containerized app:

```bash
docker build -t rag-medical-chatbot .
docker run -p 5000:5000 --env-file .env rag-medical-chatbot
```

Before publishing, the image is scanned for vulnerabilities with **Trivy**.

```bash
trivy image rag-medical-chatbot
```

## CI/CD

A Jenkins pipeline automates the deployment flow:

```
GitHub → checkout code → build Docker image → scan with Trivy → push to AWS ECR → deploy → AWS
```

See [`Jenkinsfile`](./Jenkinsfile) for the full pipeline definition.

## Roadmap

Development is tracked via GitHub milestones:

1. Project Setup & Infrastructure
2. Knowledge Ingestion Pipeline
3. Retrieval & LLM Integration
4. Web Application
5. Logging & Error Handling
6. Containerization & Security
7. CI/CD & Deployment

## License

TBD
