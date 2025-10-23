
# Multimodal Document Semantic Search (Local, No API Keys)

## What this does
- Parse **PDF** and **DOCX** to extract **text**, **tables**, and **images** (with captions/nearby text)
- Build **embeddings** (text: `all-MiniLM-L6-v2`, images: `clip-ViT-B-32`)
- Store vectors in **FAISS** (separate text/table and image indexes)
- **Hybrid semantic search** over text/tables and images via CLIP
- Show **5 demo queries** with retrieved chunks + **short relevance explanation**

## How to run (Colab or local)
1. Open `multimodal_doc_pipeline.ipynb` in Google Colab or Jupyter.
2. Put your sample files into `./sample_docs` (in Colab, upload into `/content/sample_docs`). Supported: `.pdf`, `.docx`.
3. Run the **Setup** cell. It will print which optional components are available (Camelot, Tesseract).
   - For PDF tables, Camelot works best with high-quality vector PDFs. For image-based tables, OCR is needed.
   - If Camelot fails, you can try `tabula-py` or skip tables.
4. Run all cells in order:
   - **Config**
   - **Extraction** (PDF, DOCX)
   - **Ingestion & Embeddings & FAISS**
   - **Search** + **Demo queries**
5. Inspect results in console; extracted images will be under `./artifacts/images/`.

## Notes / Assumptions
- This is **keyless** (no OpenAI API). It uses small, good-enough local models for evaluation.
- Images are indexed via CLIP **image embeddings**; CLIP **text encoder** is used for text→image search.
- Tables are converted to markdown and embedded as text.
- Captions for images in PDFs are approximated from nearby text; for DOCX we use nearest previous paragraph. OCR (if `pytesseract` available) augments captions with detected text inside images.
- You can extend with Weaviate/Pinecone/Milvus/Chroma by swapping the FAISS section.

## Test Queries Examples
1. `Give me the section discussing quarterly revenue trends`
2. `table: Find the table with country-wise sales breakdown`
3. `image: Show charts about growth rate or bar chart of 2023 vs 2024`
4. `Where is the methodology or experimental setup described?`
5. `Locate any image or caption mentioning 'confusion matrix' or 'ROC'`

## Folder structure
```
sample_docs/           # place your PDFs and DOCX here
artifacts/
  corpus_meta.json     # metadata for indexed chunks
  images/              # extracted images
```

## Extending / Creativity ideas
- Add `layoutparser` to segment PDF pages before deciding captions.
- Add `KDE/BERTScore`-based re-ranker for tighter hits.
- Add a lightweight **RAG** step that cites chunk sources and synthesizes answers.
- Store chunk offsets to render page snippets (bounding boxes) in UI.
