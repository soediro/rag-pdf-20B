# rag-pdf-20B
Full PDF → Markdown → RAG

# Directory Structure:
```
├── README.md                # Project overview & setup steps
├── requirements.txt         # Python dependencies
├── Dockerfile               # (optional) container for reproducible dev
├── .gitignore
├── src/
│   ├── __init__.py
│   ├── convert.py           # pdf → markdown + image extraction
│   ├── ingest.py            # chunking, embedding, FAISS indexing
│   ├── api.py               # FastAPI + llama.cpp wrapper
│   └── utils.py             # small helpers (e.g., prompt building)
├── data/
│   ├── pdfs/                # 👉  Put your raw PDFs here
│   ├── md/                  # 👉  Generated Markdown files
│   └── images/              # 👉  Extracted images (per‑PDF subfolders)
├── vector_store/             # FAISS index & metadata (auto‑generated)
│   ├── index.faiss
│   └── metadata.json
├── model/                   # 👉  Place your local 20B ggml.bin here
│   └── 20B.ggml.bin
└── logs/                    # Optional: API logs, ingestion logs
```

### File‑by‑file quick‑look

| File | Purpose |
|------|---------|
| **README.md** | High‑level walk‑through, prerequisites, usage. |
| **requirements.txt** | `pymupdf`, `markdownify`, `camelot‑py[cv]`, `tabula‑py`, `pandas`, `fastapi`, `uvicorn`, `sentence‑transformers`, `faiss‑cpu`, `llama‑cpp‑python` (or your own wrapper). |
| **Dockerfile** | Optional container that installs system libs and Python packages. |
| **src/convert.py** | Implements the script from “PDF ➜ Markdown” (extracts text, images, tables). |
| **src/ingest.py** | Loads all Markdown, chunks them, embeds with `sentence‑transformers`, writes FAISS index + metadata. |
| **src/api.py** | FastAPI server that loads the FAISS index, runs `llama.cpp` inference, returns JSON. |
| **src/utils.py** | Helpers: prompt templates, token counting, logging. |
| **data/pdfs/** | Your source PDFs. |
| **data/md/** | Output Markdown. |
| **data/images/** | Extracted image files. |
| **vector_store/** | Persisted FAISS index & metadata. |
| **model/** | `20B.ggml.bin` (or any other local LLM you want). |
| **logs/** | Optional. |

### Quick start (local)

```bash
# 1️⃣  Clone repo & enter
git clone https://github.com/yourname/rag-pdf-20B.git
cd rag-pdf-20B

# 2️⃣  (Optional) create venv
python3 -m venv .venv
source .venv/bin/activate

# 3️⃣  Install deps
pip install -r requirements.txt

# 4️⃣  Put your PDFs in data/pdfs/

# 5️⃣  Convert PDFs → Markdown + images
python -m src.convert

# 6️⃣  Build the FAISS index
python -m src.ingest

# 7️⃣  Start the API
uvicorn src.api:app --host 0.0.0.0 --port 8000
```

You can now query the service:

```bash
curl -X POST http://localhost:8000/ask \
     -H "Content-Type: application/json" \
     -d '{"text":"Explain the mean‑reversion strategy."}'
```

### Docker (optional)

```dockerfile
# Dockerfile
FROM python:3.10-slim

# System deps
RUN apt-get update && apt-get install -y \
    poppler-utils tesseract-ocr ghostscript openjdk-11-jdk \
    && rm -rf /var/lib/apt/lists/*

# Workdir
WORKDIR /app

# Copy repo
COPY . .

# Python deps
RUN pip install --no-cache-dir -r requirements.txt

# Expose
EXPOSE 8000

# Entry
CMD ["uvicorn", "src.api:app", "--host", "0.0.0.0", "--port", "8000"]
```

Build & run:

```bash
docker build -t rag-pdf-20b .
docker run -p 8000:8000 rag-pdf-20b
```

---

That’s the full directory layout you need to get the pipeline up and running.  Happy RAG-ing!
