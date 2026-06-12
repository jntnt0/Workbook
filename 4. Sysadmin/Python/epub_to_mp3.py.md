"""
Convert all EPUB files in a directory to MP3 using Microsoft Edge TTS.
Supports CLI arguments or interactive directory selection via file dialog.

Usage:
    python epub_to_mp3_v2.py -i ./epubs -o ./mp3s
    python epub_to_mp3_v2.py -i ./epubs -o ./mp3s -v en-GB-SoniaNeural

Requirements:
    - ebooklib
    - beautifulsoup4
    - edge-tts
"""


import argparse
import asyncio
import logging
from pathlib import Path
from ebooklib import epub, ITEM_DOCUMENT
from bs4 import BeautifulSoup
from edge_tts import Communicate
import tkinter as tk
from tkinter import filedialog

# Configure logging
logging.basicConfig(
    format="%(asctime)s %(levelname)s: %(message)s",
    level=logging.INFO,
    datefmt="%Y-%m-%d %H:%M:%S"
)

def choose_directory(title: str) -> Path:
    root = tk.Tk()
    root.withdraw()
    path = filedialog.askdirectory(title=title)
    root.destroy()
    return Path(path) if path else None

async def extract_text_from_epub(epub_path: Path) -> str:
    try:
        book = epub.read_epub(str(epub_path), options={"ignore_ncx": True})
    except Exception as e:
        logging.error(f"Failed to open EPUB {epub_path.name}: {e}")
        return ""
    texts = []
    # Use ITEM_DOCUMENT to catch all HTML/XHTML content
    for item in book.get_items_of_type(ITEM_DOCUMENT):
        try:
            soup = BeautifulSoup(item.get_body_content(), 'html.parser')
            text = soup.get_text(separator="\n").strip()
            if text:
                texts.append(text)
        except Exception as e:
            logging.warning(f"Error parsing {item.id} in {epub_path.name}: {e}")
    return "\n".join(texts)

async def tts_save(text: str, output_file: Path, voice: str):
    try:
        communicate = Communicate(text, voice)
        await communicate.save(str(output_file))
    except Exception as e:
        logging.error(f"TTS conversion failed for {output_file.name}: {e}")

async def process_epub(
    epub_path: Path,
    output_path: Path,
    voice: str,
    min_size: int = 1024
):
    logging.info(f"Processing: {epub_path.name}")
    full_text = await extract_text_from_epub(epub_path)
    if not full_text:
        logging.warning(f"No extractable text in {epub_path.name}; skipping.")
        return

    await tts_save(full_text, output_path, voice)

    try:
        size = output_path.stat().st_size
        if size < min_size:
            logging.error(f"File {output_path.name} too small ({size} bytes); deleting.")
            output_path.unlink(missing_ok=True)
        else:
            logging.info(f"Saved: {output_path.name} ({size} bytes)")
    except FileNotFoundError:
        logging.error(f"Output file {output_path.name} not found after TTS.")

async def main(args):
    input_dir = Path(args.input).expanduser() if args.input else choose_directory("Select EPUB directory")
    output_dir = Path(args.output).expanduser() if args.output else choose_directory("Select output directory")

    if not input_dir or not input_dir.exists():
        logging.error("No valid input directory selected.")
        return
    output_dir.mkdir(parents=True, exist_ok=True)

    epub_paths = list(input_dir.glob("*.epub"))
    if not epub_paths:
        logging.error(f"No EPUB files in {input_dir}.")
        return

    for epub in epub_paths:
        out_file = output_dir / f"{epub.stem}.mp3"
        await process_epub(epub, out_file, args.voice, args.min_size)

    logging.info("All tasks completed.")

if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Convert EPUBs in a directory to MP3 with edge-tts (ignores NCX)."
    )
    parser.add_argument("-i", "--input", help="Directory containing .epub files")
    parser.add_argument("-o", "--output", help="Directory to save .mp3 files")
    parser.add_argument(
        "-v", "--voice",
        default="en-US-AriaNeural",
        help="Azure TTS voice"
    )
    parser.add_argument(
        "--min-size",
        type=int,
        default=1024,
        help="Minimum MP3 size in bytes"
    )
    args = parser.parse_args()
    asyncio.run(main(args))
