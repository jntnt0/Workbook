#!/bin/bash
# Concatenate all MP3 files in the current directory into a single output file
# Requires: ffmpeg
# Note: Filenames with spaces may cause issues — rename them before running

output="output.mp3"

# Generate sorted file list
ls *.mp3 | sort -n > filelist.txt

# Convert to ffmpeg concat format
sed "s/^/file '/;s/$/'/" filelist.txt > input.txt

# Concatenate
ffmpeg -f concat -safe 0 -i input.txt -c copy "$output"

# Cleanup
rm filelist.txt input.txt

echo "Done. All files concatenated into: $output"