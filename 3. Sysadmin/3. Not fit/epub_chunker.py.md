"""
Extract and chunk text from an EPUB file into numbered .txt files.
Useful for feeding large ebooks into tools with character limits (LLMs, TTS, etc).

Usage:
    python epub_chunker.py <path-to-epub> [--max-chars 9950]

Requirements:
    - ebooklib
    - beautifulsoup4
"""

import os
import sys
import argparse
from ebooklib import epub, ITEM_DOCUMENT
from bs4 import BeautifulSoup

def clean_epub_text(epub_path):
    book = epub.read_epub(epub_path)
    full_text = []
    for item in book.get_items_of_type(ITEM_DOCUMENT):
        soup = BeautifulSoup(item.get_content(), 'html.parser')
        for tag in soup(['script', 'style', 'nav', 'header', 'footer']):
            tag.decompose()
        full_text.append(soup.get_text(separator=' ', strip=True))
    return '\n\n'.join(full_text)

def split_text(text, max_chars):
    """Split text into chunks at word boundaries."""
    words = text.split()
    chunks, current, count = [], [], 0
    for w in words:
        if count + len(w) + 1 > max_chars:
            chunks.append(' '.join(current))
            current, count = [w], len(w)
        else:
            current.append(w)
            count += len(w) + 1
    if current:
        chunks.append(' '.join(current))
    return chunks

def main():
    parser = argparse.ArgumentParser(description='Chunk EPUB text into numbered .txt files')
    parser.add_argument('epub_path', help='Path to the EPUB file')
    parser.add_argument('--max-chars', type=int, default=9950, help='Max characters per chunk (default: 9950)')
    args = parser.parse_args()

    if not os.path.isfile(args.epub_path):
        print(f"Error: File not found: {args.epub_path}")
        sys.exit(1)

    base = os.path.splitext(os.path.basename(args.epub_path))[0]
    out_dir = os.path.join(os.getcwd(), f"{base}_chunks")
    os.makedirs(out_dir, exist_ok=True)

    text = clean_epub_text(args.epub_path)
    chunks = split_text(text, args.max_chars)

    for idx, chunk in enumerate(chunks, 1):
        with open(os.path.join(out_dir, f"{idx:03}.txt"), 'w', encoding='utf-8') as f:
            f.write(chunk)

    print(f"Exported {len(chunks)} chunks to {out_dir}")

if __name__ == '__main__':
    main()