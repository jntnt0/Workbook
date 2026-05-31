"""
Convert a PDF file to JSONL format for LLM fine-tuning or RAG pipelines.
Each page becomes one JSONL entry with a prompt and completion field.

Usage:
    python pdf_to_jsonl.py <input.pdf> <output.jsonl>

Requirements:
    - PyMuPDF (pip install pymupdf)
"""

import sys
import json
import fitz

def pdf_to_jsonl(pdf_path, jsonl_path):
    doc = fitz.open(pdf_path)
    with open(jsonl_path, "w", encoding="utf-8") as f:
        for page in doc:
            text = page.get_text("text").strip()
            if text:
                entry = {"prompt": "Summarize this page:", "completion": text}
                f.write(json.dumps(entry) + "\n")
    print(f"Exported {len(doc)} pages to {jsonl_path}")

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print("Usage: python pdf_to_jsonl.py <input.pdf> <output.jsonl>")
        sys.exit(1)
    pdf_to_jsonl(sys.argv[1], sys.argv[2])