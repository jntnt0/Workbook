|   |
|---|
|#!/bin/bash|
|# Capture selected area and copy OCR text to clipboard|
|gnome-screenshot -a -f /tmp/ocr_screenshot.png|
|tesseract /tmp/ocr_screenshot.png stdout \| xclip -selection clipboard|
|rm /tmp/ocr_screenshot.png|