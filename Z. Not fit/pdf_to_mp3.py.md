"""
Convert all PDF files in a directory to MP3 using Microsoft Edge TTS.
Uses pdfplumber for native text extraction with pytesseract OCR as fallback
for scanned or image-based pages. Processes files concurrently using asyncio.

Usage:
    python pdf_to_mp3.py <input_directory> [output_directory]

Requirements:
    - pdfplumber
    - pytesseract
    - edge-tts
    - Tesseract OCR installed on the system
"""

import os
import sys
import asyncio
import pytesseract
from pdfplumber import open as open_pdf
from edge_tts import Communicate

async def extract_text_with_ocr_and_convert_to_audio(pdf_path, output_audio_file, voice="en-US-AriaNeural"):
    try:
        text_output = []
        print(f"Starting text extraction for {pdf_path}...")
        with open_pdf(pdf_path) as pdf:
            for page in pdf.pages:
                text = page.extract_text()
                if not text or len(text.strip()) < 50:
                    print(f"Using OCR for a page in {pdf_path}...")
                    image = page.to_image(resolution=300).original
                    text = pytesseract.image_to_string(image)
                text_output.append(text)
        full_text = "\n".join(text_output)
        print(f"Text extraction complete for {pdf_path}.")
        communicate = Communicate(full_text, voice)
        await communicate.save(output_audio_file)
        print(f"Audio saved as {output_audio_file}")
    except Exception as e:
        print(f"Error processing {pdf_path}: {e}")

async def process_all_pdfs_in_directory(input_directory, output_directory, voice="en-US-AriaNeural"):
    os.makedirs(output_directory, exist_ok=True)
    pdf_files = [f for f in os.listdir(input_directory) if f.endswith(".pdf")]
    tasks = []
    for pdf_file in pdf_files:
        input_path = os.path.join(input_directory, pdf_file)
        output_path = os.path.join(output_directory, os.path.splitext(pdf_file)[0] + ".mp3")
        tasks.append(extract_text_with_ocr_and_convert_to_audio(input_path, output_path, voice))
    await asyncio.gather(*tasks)
    print("All PDFs processed.")

if __name__ == "__main__":
    input_dir  = sys.argv[1] if len(sys.argv) > 1 else input("Enter input directory: ").strip()
    output_dir = sys.argv[2] if len(sys.argv) > 2 else "MP3s"
    asyncio.run(process_all_pdfs_in_directory(input_dir, output_dir))