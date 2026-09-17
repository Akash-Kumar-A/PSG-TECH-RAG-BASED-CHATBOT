# PSG Tech RAG-Based Chatbot

A Retrieval-Augmented Generation (RAG) chatbot that answers natural-language questions about **PSG College of Technology** (Coimbatore) — departments, programmes, faculty, campus facilities, and general information — using a Qdrant vector database and Google's Gemini API, served through a Streamlit web interface.

## Overview

The project scrapes/curates information about PSG College of Technology into a set of plain-text documents, embeds them into a Qdrant vector database, and answers user queries by retrieving the most relevant chunks and generating a natural-language response with Gemini.

Two retrieval strategies are implemented as separate app variants:
- **`app_fair.py`** — straightforward semantic (vector) search against Qdrant.
- **`app_rrf.py`** — hybrid retrieval using **Reciprocal Rank Fusion (RRF)** across two retriever passes for improved result ranking.

## Project Structure

```
PSG-TECH-RAG-BASED-CHATBOT/
├── Data set/              # 84 plain-text knowledge base documents
│   ├── about the college.txt
│   ├── Programmes.txt
│   ├── Campus Highlights.txt
│   ├── Founders.txt
│   ├── Dean.txt / Principal.txt / Managing Trustee.txt / The Management.txt
│   ├── B.E / B.Tech / M.Sc / MCA <department>.txt   # per-programme descriptions
│   └── Dr./Mr./Mrs./Ms. <Name>.txt                  # individual faculty profiles
├── data-ingestion.py      # Reads the Data set files, chunks/embeds them, and uploads to Qdrant
├── app_fair.py             # Streamlit app: plain vector search + Gemini response generation
├── app_rrf.py               # Streamlit app: hybrid RRF retrieval + Gemini response generation
└── requirements.txt         # Python dependencies
```

### Knowledge base contents

The `Data set/` directory contains **84 text files** covering:
- General information about the college (history, founders, management, campus)
- All Undergraduate and Postgraduate programmes offered (B.E./B.Tech/M.Sc./MCA across departments such as CSE, ECE, EEE, Mechanical, Civil, Biomedical, Robotics & Automation, Fashion Technology, etc.)
- Individual faculty member profiles (name, designation, department)
- Campus highlights and administrative roles (Principal, Dean, Managing Trustee)

## How It Works

1. **Ingestion** (`data-ingestion.py`):
   - Reads all `.txt` files from `Data set/`.
   - Embeds each document using a `sentence-transformers` model (`all-MiniLM-L6-v2`, 384-dim vectors).
   - Creates a Qdrant collection and uploads the embedded chunks with their filenames as metadata.

2. **Query time** (`app_fair.py` / `app_rrf.py`):
   - The user submits a query via the Streamlit UI.
   - The query is embedded and used to search Qdrant for the most relevant document chunks (via plain vector search, or hybrid RRF search across two retrievers in `app_rrf.py`).
   - The retrieved chunks are passed as context to Google's **Gemini** model (`gemini-1.5-flash`), which generates a concise, natural-language answer grounded in that context.

## Requirements

- Python 3.9+
- Accounts/API keys for:
  - [Qdrant](https://qdrant.tech/) (cloud instance or self-hosted)
  - [Google Generative AI (Gemini)](https://ai.google.dev/)

### Install dependencies

```bash
pip install -r requirements.txt
```

> `requirements.txt` pins fairly old versions (`streamlit==1.15.0`, `langchain==0.0.157`, `qdrant-client==1.2.0`, `google-generativeai==0.2.0`, `pydantic==1.10.7`). `app_rrf.py` uses `langchain_community` and `langchain_core`, which are **not included** in `requirements.txt` — install a current `langchain` (with `langchain-community`) if running that variant, and note that `google-generativeai==0.2.0` will likely need upgrading to support the `gemini-1.5-flash` model used in the app code.

## ⚠️ Security Notice — Hardcoded API Keys

**This codebase contains hardcoded API keys and secrets committed directly in source**, including Qdrant and Gemini credentials in `data-ingestion.py`, `app_fair.py`, and `app_rrf.py`. Before using or publishing this repository:

1. **Rotate/revoke all exposed keys immediately** in your Qdrant and Google AI Studio dashboards, since they are visible in the source code as committed.
2. **Move all credentials to environment variables** (e.g. via a `.env` file loaded with `python-dotenv`, or `os.environ`) instead of hardcoding them.
3. Add `.env` (and any credentials file) to `.gitignore`.

## Getting Started

> After rotating and externalizing the API keys as described above:

1. **Update the hardcoded ingestion path**: `data-ingestion.py` currently reads from a Windows-specific absolute path (`A:\PROJECTS\...\Data set`). Change this to a relative path, e.g.:
   ```python
   documents = read_txt_files("Data set")
   ```
2. **Ingest the knowledge base** into Qdrant:
   ```bash
   python data-ingestion.py
   ```
3. **Run the chatbot** (choose one variant):
   ```bash
   streamlit run app_fair.py   # plain vector search
   # or
   streamlit run app_rrf.py    # hybrid RRF retrieval
   ```
4. Open the Streamlit URL shown in the terminal and ask questions such as *"Who is the Dean of Placements?"* or *"What undergraduate programmes does PSG Tech offer?"*

## Notes

- `app_fair.py` and `app_rrf.py` both connect to the same Qdrant collection (`psg_dataset`) but differ in retrieval strategy — RRF combines rankings from two retriever passes for potentially more robust results.
- The knowledge base reflects a snapshot of publicly available information about PSG College of Technology and may become outdated over time; re-run ingestion after updating the `Data set/` files.
