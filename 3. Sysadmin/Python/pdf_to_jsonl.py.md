"""
pdf_to_jsonl.py
Convert a PDF file to JSONL format for use in LLM fine-tuning and RAG pipelines.

Overview:
    Large language model workflows — whether fine-tuning a model or building a
    retrieval-augmented generation (RAG) pipeline — commonly require a structured
    text corpus in JSONL format, where each line is a self-contained JSON object.
    This script extracts text from a PDF on a per-page basis and writes each page
    as a JSONL entry with a configurable prompt field and the page text as the
    completion field.

    Typical use cases include converting technical documentation, runbooks, or
    reference PDFs into a format suitable for ingestion by tools such as LlamaIndex,
    LangChain, or a custom fine-tuning pipeline.

Usage:
    python pdf_to_jsonl.py <input.pdf> <output.jsonl>
    python pdf_to_jsonl.py <input.pdf> <output.jsonl> --prompt "Summarize this content:"
    python pdf_to_jsonl.py <input.pdf> <output.jsonl> --min-chars 100

Parameters:
    input.pdf       Path to the source PDF file.
    output.jsonl    Path for the output JSONL file. Created if it does not exist.
    --prompt        Prompt string written to each entry's 'prompt' field.
                    Default: "Summarize this page:"
    --min-chars     Minimum character count for a page to be included.
                    Pages below this threshold are skipped (e.g. blank pages,
                    page numbers, headers). Default: 50

Requirements:
    pip install pymupdf
"""

import sys
import json
import argparse
import fitz  # PyMuPDF


def extract_pages(pdf_path: str, min_chars: int) -> list[dict]:
    """
    Open a PDF and extract text from each page.

    Returns a list of dicts with 'page' (1-based) and 'text' keys.
    Pages whose text falls below min_chars are excluded.
    """
    doc = fitz.open(pdf_path)
    pages = []

    for page in doc:
        text = page.get_text("text").strip()
        if len(text) >= min_chars:
            pages.append({
                "page": page.number + 1,
                "text": text
            })

    doc.close()
    return pages


def write_jsonl(pages: list[dict], output_path: str, prompt: str) -> int:
    """
    Write extracted page data to a JSONL file.

    Each line is a JSON object with 'prompt', 'completion', and 'source_page' fields.
    Returns the number of entries written.
    """
    written = 0

    with open(output_path, "w", encoding="utf-8") as f:
        for entry in pages:
            record = {
                "prompt": prompt,
                "completion": entry["text"],
                "source_page": entry["page"]
            }
            f.write(json.dumps(record, ensure_ascii=False) + "\n")
            written += 1

    return written


def main():
    parser = argparse.ArgumentParser(
        description="Convert a PDF to JSONL for LLM fine-tuning or RAG pipelines."
    )
    parser.add_argument("input",  help="Path to the source PDF file")
    parser.add_argument("output", help="Path for the output JSONL file")
    parser.add_argument(
        "--prompt",
        default="Summarize this page:",
        help="Prompt string for each JSONL entry (default: 'Summarize this page:')"
    )
    parser.add_argument(
        "--min-chars",
        type=int,
        default=50,
        help="Minimum character count to include a page (default: 50)"
    )
    args = parser.parse_args()

    try:
        pages = extract_pages(args.input, args.min_chars)
    except FileNotFoundError:
        print(f"Error: Input file not found: {args.input}", file=sys.stderr)
        sys.exit(1)
    except Exception as e:
        print(f"Error reading PDF: {e}", file=sys.stderr)
        sys.exit(1)

    if not pages:
        print(f"No pages met the minimum character threshold ({args.min_chars}). Nothing written.")
        sys.exit(0)

    try:
        written = write_jsonl(pages, args.output, args.prompt)
    except Exception as e:
        print(f"Error writing output: {e}", file=sys.stderr)
        sys.exit(1)

    total_pages = written + (0)  # placeholder if skipped count is added later
    print(f"Done. {written} entries written to {args.output}.")


if __name__ == "__main__":
    main()
