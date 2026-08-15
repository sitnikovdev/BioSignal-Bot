# BioSignal Bot

A Retrieval-Augmented Generation (RAG) chat application that answers questions about your **personal Garmin health data** — sleep, HRV, recovery, Body Battery, resting heart rate, and training load. The system retrieves relevant context from your own Garmin health report and passes it to an LLM to generate grounded, personalized answers instead of generic advice.

```
Garmin health report (PDF) → documents → chunks → embeddings → vector store → retrieval → LLM → answer
```

## Overview

Instead of relying on an LLM's general knowledge, this app builds a searchable knowledge base from your exported Garmin health report and retrieves the most relevant passages for each question before generating a response. This keeps answers grounded in *your own data* — e.g. "why was my recovery low this week" is answered from your actual HRV/sleep/training numbers, not generic wellness advice.

## Architecture

The project is organized into two pipelines plus a delivery layer.

### 1. Knowledge layer (offline / preparation pipeline)

```
Garmin Health Report (PDF export: sleep, HRV, recovery,
Body Battery, resting HR, training load)
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
User question (e.g. "How did I sleep last night?",
"Should I train today?", "Why is my recovery low?")
     ↓
  Retriever (FAISS similarity search)
     ↓
Relevant chunks from the health report
     ↓
Mistral LLM (Hugging Face) + retrieved context
     ↓
Answer grounded in your Garmin data
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
BioSignal-Bot/
│
├── .env                        # Hugging Face API token (not committed)
├── requirements.txt
├── setup.py
│
├── data/
│   └── garmin_health_report.pdf   # your exported Garmin health data
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
│   │   ├── pdf_loader/          # reads Garmin health report into LangChain documents
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
- A Garmin Connect account with an exportable health report

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

### Add your Garmin data

Export your health report from Garmin Connect (sleep, HRV, recovery, Body Battery, resting HR, training load) as a PDF and place it at:

```
data/garmin_health_report.pdf
```

## Running the app

```bash
flask run
# or
python app/app.py
```

The app will be available at `http://localhost:5000`.

## Example questions

- How did I sleep last night?
- Should I train today?
- Why is my recovery low this week?
- Compare this week with last week.
- What affected my HRV?
- Analyze my last 30 days.

## How it works

1. **Ingestion (offline)**: your Garmin health report PDF is loaded and split into chunks, each chunk is embedded and stored in a local FAISS index.
2. **Retrieval (online)**: when you ask a question, the retriever embeds it and performs a similarity search against the FAISS index to find the most relevant chunks of your health data.
3. **Generation**: the retrieved chunks are passed to the Mistral LLM as context, which generates an answer grounded in your actual metrics.
4. **Delivery**: Flask serves the request/response cycle between the browser and the RAG pipeline.

## Docker

Build and run the containerized app:

```bash
docker build -t biosignal-bot .
docker run -p 5000:5000 --env-file .env biosignal-bot
```

Before publishing, the image is scanned for vulnerabilities with **Trivy**.

```bash
trivy image biosignal-bot
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

## Privacy note

Your Garmin health report contains personal biometric data. Keep `data/garmin_health_report.pdf` out of version control (see `.gitignore`) and treat it like any other sensitive personal data.

## License

TBD
