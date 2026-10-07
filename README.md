# Multimodal Document Intelligence

A Phase 5 document-ingestion workspace. It validates and stores PDFs, renders every page, extracts native text, applies local OCR to scanned pages, and detects structured tables with source-addressable cells. Retrieval, embeddings, and reasoning remain out of scope.

## Requirements

- Python 3.10+
- Node.js 20+

## Run locally

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
pip install -r requirements.txt
uvicorn backend.main:app --reload
```

In another terminal:

```powershell
cd frontend
npm install
npm run dev
```

The API is available at `http://localhost:8000` and its interactive docs at `/docs`. The Vite development server runs at `http://localhost:5173`.

Set `VITE_API_URL` to change the frontend API base URL. Backend settings can be configured through `.env`; see `.env.example`.

## Current scope

Original PDFs are stored in `data/documents/`, rendered pages in `data/pages/<document_id>/`, page text/block JSON in `data/processed/<document_id>/pages/`, and document metadata in `storage/metadata/`. Text blocks retain page/document/block IDs, PDF-point bounding boxes, block type, confidence, and source references. Heading and layout labels are heuristic candidates, not guaranteed classifications. Empty-text pages are recorded.

Detected tables are included in the page JSON and stored separately under `data/processed/<document_id>/tables/`. Tables preserve boundaries, multi-row headers, rows, columns, missing cells, raw values, optional parsed numeric values, and cell-level document/page/table/row/column references and PDF-point bounds. Detection uses PyMuPDF and is best-effort; scanned-image tables are not reconstructed from OCR in this phase.

Scanned or image-heavy pages with little usable native text are processed locally using RapidOCR with ONNX Runtime on CPU. Its model files are downloaded and cached on the first OCR run. Page JSON records `TEXT_EXTRACTED`, `OCR_REQUIRED`, `OCR_PROCESSED`, and `OCR_FAILED` in the extraction status/history; OCR confidence and source boxes are retained. High-quality native-text pages bypass OCR.

The document API supports `GET /api/documents`, `POST /api/documents/upload`, `GET /api/documents/{document_id}`, `DELETE /api/documents/{document_id}`, `GET /api/documents/{document_id}/pages/{page_number}/image`, and `GET /api/documents/{document_id}/pages/{page_number}/text`. The UI opens a document to browse rendered pages, text blocks, and structured tables; selected blocks, tables, and cells are outlined on the page image.

Question answering and evidence generation are not implemented. No API keys are required or stored; future integrations should read secrets from environment variables.

## Tests

```powershell
pytest
```