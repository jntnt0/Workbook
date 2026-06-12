"""
Local RAG assistant using Ollama and a JSONL knowledge base.
Searches a dataset for relevant context and passes it to a local LLM.

Usage:
    python rag_assistant.py

Requirements:
    - Ollama running locally with the target model pulled
    - A JSONL file with 'question' and 'answer' keys

Example JSONL format:
    {"question": "What is term life insurance?", "answer": "..."}
"""

import json
import ollama

# Load JSONL data
def load_jsonl(file_path):
    data = []
    with open(file_path, 'r', encoding='utf-8') as file:
        for line in file:
            data.append(json.loads(line.strip()))
    return data

# Search for relevant answers
def find_relevant_answer(query, data):
    for entry in data:
        if query.lower() in entry["question"].lower():  # Assuming "question" key exists
            return entry["answer"]  # Assuming "answer" key exists
    return "No relevant answer found in dataset."

# Load dataset
data = load_jsonl("LifeandHealthInsurance.jsonl")

# Get user query
query = input("Enter your question: ")
relevant_answer = find_relevant_answer(query, data)

# Run Ollama with enhanced prompt
response = ollama.chat(model="llama3", messages=[
    {"role": "system", "content": "You are an expert in life and health insurance."},
    {"role": "user", "content": query},
    {"role": "assistant", "content": relevant_answer}
])

print(response["message"]["content"])
