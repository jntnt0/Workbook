"""
PDF Combiner with Index
Sorts all PDFs in a folder alphabetically, then combines them into
chunked output files (~500 MB each) with a table of contents at the front.

No PDF gets split across files - if it won't fit, it goes in the next chunk.

Usage:
    python pdf_combiner.py <input_folder> [output_folder] [--max-mb 500]

Examples:
    python pdf_combiner.py ./meraki_pdfs
    python pdf_combiner.py ./meraki_pdfs ./combined_output --max-mb 400
    python pdf_combiner.py ./cisco_exam_pdfs/Troubleshooting_Guides ./combined_tsg
"""

import os
import sys
import argparse
import tempfile
from pathlib import Path

from PyPDF2 import PdfReader, PdfWriter
from reportlab.lib.pagesizes import letter
from reportlab.lib.units import inch
from reportlab.platypus import SimpleDocTemplate, Paragraph, Spacer, PageBreak
from reportlab.lib.styles import getSampleStyleSheet, ParagraphStyle
from reportlab.lib.enums import TA_LEFT, TA_CENTER
from io import BytesIO


def get_pdf_files(folder):
    """Get all PDF files in folder, sorted alphabetically (case-insensitive)."""
    pdfs = []
    for f in os.listdir(folder):
        if f.lower().endswith('.pdf'):
            full = os.path.join(folder, f)
            size = os.path.getsize(full)
            if size > 0:
                pdfs.append((f, full, size))
    pdfs.sort(key=lambda x: x[0].lower())
    return pdfs


def build_index_pdf(entries, chunk_num, total_chunks):
    """
    Build an index/TOC PDF in memory.
    entries: list of (filename, start_page) tuples
    Returns bytes of a PDF.
    """
    buf = BytesIO()
    doc = SimpleDocTemplate(buf, pagesize=letter,
                            topMargin=0.75*inch, bottomMargin=0.75*inch,
                            leftMargin=0.75*inch, rightMargin=0.75*inch)

    styles = getSampleStyleSheet()
    title_style = ParagraphStyle(
        'IndexTitle', parent=styles['Title'],
        fontSize=18, spaceAfter=20, alignment=TA_CENTER
    )
    subtitle_style = ParagraphStyle(
        'IndexSubtitle', parent=styles['Normal'],
        fontSize=10, spaceAfter=30, alignment=TA_CENTER,
        textColor='#666666'
    )
    entry_style = ParagraphStyle(
        'IndexEntry', parent=styles['Normal'],
        fontSize=8, leading=11, spaceAfter=2
    )
    header_style = ParagraphStyle(
        'SectionHeader', parent=styles['Normal'],
        fontSize=10, leading=14, spaceAfter=6, spaceBefore=12,
        textColor='#333333', bold=True
    )

    story = []
    label = f"Volume {chunk_num} of {total_chunks}" if total_chunks > 1 else "Complete Index"
    story.append(Paragraph("TABLE OF CONTENTS", title_style))
    story.append(Paragraph(f"{label} &mdash; {len(entries)} documents", subtitle_style))

    current_letter = None
    for i, (filename, start_page) in enumerate(entries, 1):
        first = filename[0].upper()
        if first != current_letter:
            current_letter = first
            story.append(Paragraph(f"<b>— {current_letter} —</b>", header_style))

        clean = filename.replace('.pdf', '').replace('_', ' ')
        # Truncate long names for the index
        if len(clean) > 100:
            clean = clean[:97] + "..."
        line = f"{i}. {clean} <font color='#888888'>.... p.{{PAGE}}</font>"
        # We'll fix page numbers after we know index length
        line = f"{i}. {clean}"
        if start_page is not None:
            line += f" <font color='#888888'>&mdash; page {start_page}</font>"
        story.append(Paragraph(line, entry_style))

    doc.build(story)
    return buf.getvalue()


def count_pages(filepath):
    """Count pages in a PDF file. Returns 0 on error."""
    try:
        reader = PdfReader(filepath)
        return len(reader.pages)
    except Exception:
        return 0


def merge_pdfs(file_list, output_path, index_bytes=None):
    """
    Merge a list of PDF files into one output file.
    Optionally prepend index_bytes (a PDF in bytes) at the front.
    """
    writer = PdfWriter()

    # Prepend index if provided
    if index_bytes:
        index_reader = PdfReader(BytesIO(index_bytes))
        for page in index_reader.pages:
            writer.add_page(page)

    for filepath in file_list:
        try:
            reader = PdfReader(filepath)
            for page in reader.pages:
                writer.add_page(page)
        except Exception as e:
            print(f"  WARN: Could not read {filepath}: {e}")

    with open(output_path, 'wb') as f:
        writer.write(f)


def plan_chunks(pdfs, max_bytes):
    """
    Plan which PDFs go into which chunk.
    Returns list of lists: [ [(filename, filepath, size), ...], ... ]
    """
    chunks = []
    current = []
    current_size = 0

    for filename, filepath, size in pdfs:
        # If single file exceeds max, it gets its own chunk
        if size >= max_bytes:
            if current:
                chunks.append(current)
                current = []
                current_size = 0
            chunks.append([(filename, filepath, size)])
            continue

        if current_size + size > max_bytes:
            # Start new chunk
            chunks.append(current)
            current = [(filename, filepath, size)]
            current_size = size
        else:
            current.append((filename, filepath, size))
            current_size += size

    if current:
        chunks.append(current)

    return chunks


def main():
    parser = argparse.ArgumentParser(description='Combine PDFs into indexed volumes')
    parser.add_argument('input_folder', help='Folder containing PDF files')
    parser.add_argument('output_folder', nargs='?', default=None,
                        help='Output folder (default: <input>_combined)')
    parser.add_argument('--max-mb', type=int, default=500,
                        help='Max size per output file in MB (default: 500)')
    args = parser.parse_args()

    input_folder = args.input_folder
    if not os.path.isdir(input_folder):
        print(f"ERROR: {input_folder} is not a directory")
        sys.exit(1)

    output_folder = args.output_folder or (input_folder.rstrip('/\\') + '_combined')
    os.makedirs(output_folder, exist_ok=True)
    max_bytes = args.max_mb * 1024 * 1024

    # Step 1: Gather and sort
    print(f"Scanning {input_folder} for PDFs...")
    pdfs = get_pdf_files(input_folder)
    total_size = sum(s for _, _, s in pdfs)
    print(f"Found {len(pdfs)} PDFs, {total_size / (1024**3):.2f} GB total")

    if not pdfs:
        print("No PDFs found. Exiting.")
        sys.exit(0)

    # Step 2: Plan chunks by raw file size
    chunks = plan_chunks(pdfs, max_bytes)
    print(f"Planned {len(chunks)} output volume(s) at ~{args.max_mb} MB max each\n")

    # Step 3: Build each volume
    for ci, chunk in enumerate(chunks, 1):
        chunk_files = [fp for _, fp, _ in chunk]
        chunk_names = [fn for fn, _, _ in chunk]
        chunk_size = sum(s for _, _, s in chunk)

        print(f"=== Volume {ci}/{len(chunks)} ===")
        print(f"  {len(chunk)} PDFs, {chunk_size / (1024**2):.1f} MB raw")

        # Count pages to build index with page numbers
        print("  Counting pages...")
        page_counts = []
        for fn, fp, _ in chunk:
            pc = count_pages(fp)
            page_counts.append(pc)

        # Build index entries with estimated page offsets
        # We don't know index length yet, so do two passes
        # Pass 1: build index without page numbers to measure its length
        dummy_entries = [(fn, None) for fn in chunk_names]
        dummy_index = build_index_pdf(dummy_entries, ci, len(chunks))
        dummy_reader = PdfReader(BytesIO(dummy_index))
        index_pages = len(dummy_reader.pages)

        # Pass 2: build index with correct page numbers
        entries = []
        running_page = index_pages + 1  # 1-based, after index
        for j, fn in enumerate(chunk_names):
            entries.append((fn, running_page))
            running_page += page_counts[j]

        index_pdf = build_index_pdf(entries, ci, len(chunks))

        # Merge everything
        output_name = f"combined_vol{ci:02d}.pdf"
        output_path = os.path.join(output_folder, output_name)
        print(f"  Merging into {output_name}...")
        merge_pdfs(chunk_files, output_path, index_bytes=index_pdf)

        final_size = os.path.getsize(output_path)
        print(f"  Done: {final_size / (1024**2):.1f} MB\n")

    print("=" * 50)
    print(f"All volumes saved to: {os.path.abspath(output_folder)}")


if __name__ == "__main__":
    main()
