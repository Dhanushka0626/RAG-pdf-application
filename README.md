# PDF RAG Application

A local Retrieval-Augmented Generation (RAG) application that lets you upload PDF documents, ingest their contents into a vector database, and ask questions about the uploaded PDFs through a Streamlit interface.

The application uses:

- **Streamlit** for the web interface
- **FastAPI** for the API and Inngest function server
- **Inngest** for durable ingestion and query workflows
- **LlamaIndex** for PDF loading and text chunking
- **OpenAI** embeddings and chat models
- **Qdrant** for vector storage and semantic search

## How It Works

1. Upload a PDF in Streamlit.
2. The PDF is saved in the `uploads/` directory and an ingestion event is sent to Inngest.
3. The PDF is parsed and split into overlapping text chunks.
4. Each chunk is converted into a 3072-dimensional embedding using OpenAI's `text-embedding-3-large` model.
5. Chunk vectors and source text are stored in the local Qdrant `docs` collection.
6. A question is embedded and used to retrieve the most relevant chunks from Qdrant.
7. The retrieved context is sent to `gpt-4o-mini`, which generates a concise answer and returns the matched source filenames.

## Requirements

- Python 3.14 or newer
- [uv](https://docs.astral.sh/uv/) for dependency and environment management
- Docker or a local Qdrant server
- An OpenAI API key
- Inngest Dev Server for local workflow execution

## Installation

Clone the repository:

```bash
git clone https://github.com/Dhanushka0626/RAG-pdf-application.git
cd RAG-pdf-application
```

Install dependencies:

```bash
uv sync
```

Create an environment file:

```bash
cp .env.example .env
```

Add your OpenAI API key:

```dotenv
OPENAI_API_KEY=your_openai_api_key
```

> The repository does not currently include `.env.example`. Create a `.env` file in the project root with the variable above.

## Start Qdrant

Run Qdrant locally with Docker:

```bash
docker run -d --name qdrant \
  -p 6333:6333 \
  -v qdrant_storage:/qdrant/storage \
  qdrant/qdrant
```

The application connects to `http://localhost:6333` by default and creates a collection named `docs` with cosine distance.

## Start the Application

Start the FastAPI and Inngest function server:

```bash
uv run uvicorn main:app --reload
```

Start the Inngest Dev Server in a second terminal:

```bash
npx inngest-cli@latest dev
```

Start the Streamlit interface in a third terminal:

```bash
uv run streamlit run streamlit_app.py
```

Open the Streamlit URL shown in the terminal, normally:

```text
http://localhost:8501
```

## Usage

### Ingest a PDF

1. Use the **Upload a PDF to Ingest** file uploader.
2. Select a PDF file.
3. Wait for the ingestion success message.
4. Uploaded files are stored in `uploads/` and tracked by their filename.

### Ask a Question

1. Enter a question in the question field.
2. Choose how many relevant chunks should be retrieved. The default is `5`, with a maximum of `20`.
3. Submit the form.
4. The app displays the generated answer and its matched PDF sources.

## Project Structure

```text
.
├── custom_types.py       # Pydantic models for RAG events and responses
├── data_loader.py        # PDF parsing, chunking, and embedding
├── main.py               # FastAPI app and Inngest RAG functions
├── streamlit_app.py      # Streamlit upload and question interface
├── vector_db.py          # Qdrant collection and search operations
├── pyproject.toml        # Project dependencies and metadata
├── uploads/              # Uploaded PDF files (created at runtime)
└── README.md
```

## Configuration

| Variable | Purpose | Default |
| --- | --- | --- |
| `OPENAI_API_KEY` | Authenticates OpenAI embeddings and chat requests | Required |
| `INNGEST_API_BASE` | Base URL used by Streamlit to retrieve Inngest run output | `http://127.0.0.1:8288/v1` |

Qdrant defaults can currently be changed in `vector_db.py`:

- URL: `http://localhost:6333`
- Collection: `docs`
- Vector dimension: `3072`

## Inngest Events

| Event | Purpose |
| --- | --- |
| `rag/ingest_pdf` | Loads and chunks a PDF, creates embeddings, and upserts vectors into Qdrant |
| `rag/query_pdf_ai` | Searches relevant chunks and generates an answer with an OpenAI model |

## API

FastAPI exposes the Inngest integration through `main.py`. Inngest manages the workflow endpoints automatically when `uvicorn main:app --reload` is running.

## Limitations

- No authentication or user isolation is configured.
- The Qdrant collection is local and shared by all users.
- PDF ingestion and querying require network access to OpenAI.
- Uploaded PDFs are stored on the local filesystem in `uploads/`.
- Re-ingesting the same source filename updates existing chunk IDs rather than creating duplicate chunks.
- The Streamlit app depends on the local Inngest Dev Server API for run output.

## License

This project is licensed under the terms of the [MIT License](LICENSE).
