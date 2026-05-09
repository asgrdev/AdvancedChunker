
# AdvancedChunker & DataHelper

A powerful text processing utility for chunking Persian/Arabic/English texts with token-aware splitting, plus a collection of helper functions for everyday data operations.

## Features

### AdvancedChunker
- 🧠 **Token-aware chunking** using tiktoken (GPT-4o compatible)
- 📝 **Sentence-level preservation** - never cuts in the middle of a sentence
- 🔄 **Intelligent overlap** - maintains context between chunks
- 🎯 **Configurable token limits** - perfect for LLM context windows
- 🌍 **Multi-language support** - works with Persian, Arabic, English, and more

### DataHelper
- 🧹 **Text cleaning & normalization**
- 🔑 **Simple keyword extraction** (no NLP libraries needed)
- 🌐 **Language detection** (fa/ar/en)
- 🆔 **Unique ID generation** with MD5
- ⏱️ **Retry decorators** with exponential backoff (sync & async)
- 📅 **Date normalization** from multiple formats
- 📊 **Result summary display** for logging

## Installation

```bash
pip install tiktoken loguru

## Quick Start

### Basic Chunking

```python
from advanced_chunker import AdvancedChunker

# Initialize chunker
chunker = AdvancedChunker(
    model_name="gpt-4o",     # tokenizer model
    max_tokens=400,          # maximum tokens per chunk
    overlap_sentences=2,     # sentences to overlap between chunks
    min_chunk_tokens=50      # minimum tokens for a valid chunk
)

# Sample text
text = """
Machine learning is a subset of artificial intelligence. 
It enables systems to learn from data without explicit programming. 
Deep learning, a further subset, uses neural networks with many layers. 
Transformers have revolutionized NLP in recent years. 
Attention mechanisms allow models to focus on relevant parts of input.
"""

# Chunk the text
chunks = chunker.chunk_text(text)

for i, chunk in enumerate(chunks):
    print(f"Chunk {i+1}:")
    print(chunk)
    print("-" * 50)
```

### Batch Processing Multiple Documents

```python
documents = [
    "First document about AI and machine learning...",
    "Second document about natural language processing...",
    "Third document about computer vision..."
]

all_chunks = []
for doc_id, doc_text in enumerate(documents):
    chunks = chunker.chunk_text(doc_text)
    all_chunks.extend({
        "doc_id": doc_id,
        "chunk_index": i,
        "text": chunk,
        "token_count": chunker._get_token_count(chunk)
    } for i, chunk in enumerate(chunks))

print(f"Total chunks created: {len(all_chunks)}")
```

### Advanced Chunking with Overlap

```python
# Create chunks with context preservation
chunker = AdvancedChunker(
    max_tokens=300,
    overlap_sentences=3,     # Keep 3 sentences from previous chunk
    min_chunk_tokens=100
)

long_text = "..." * 1000
chunks = chunker.chunk_text(long_text)

# Each chunk maintains context from previous sentences
for i, chunk in enumerate(chunks):
    print(f"Chunk {i}: {len(chunk)} chars, {chunker._get_token_count(chunk)} tokens")
```

## DataHelper Examples

### Text Processing

```python
from data_helper import DataHelper

# Clean messy text
dirty_text = "This   has   too   many   spaces\n\n\nand blank lines"
clean = DataHelper.clean_text(dirty_text)
print(clean)  # "This has too many spaces and blank lines"

# Extract keywords without NLP
text = "Machine learning and deep learning are subsets of artificial intelligence"
keywords = DataHelper.extract_keywords(text, top_n=5)
print(keywords)  # ['learning', 'machine', 'deep', 'subsets', 'artificial']

# Detect language
lang = DataHelper.detect_language("سلام دنیا")  # "fa"
lang = DataHelper.detect_language("Hello world")  # "en"
```

### Retry Mechanism

```python
from data_helper import DataHelper

# Sync retry
@DataHelper.retry(max_attempts=3, delay=1.0, backoff=2.0)
def unstable_api_call():
    # Your API call here
    return response

# Async retry
@DataHelper.async_async_retry(max_attempts=3, delay=1.0)
async def async_api_call():
    # Your async API call
    return await response
```

### ID Generation & Safe Data Access

```python
# Create unique IDs
doc_id = DataHelper.make_id("database", "user_12345")
print(doc_id)  # MD5 hash

# Safely access nested dictionaries
data = {"user": {"profile": {"name": "John"}}}
name = DataHelper.safe_get(data, "user", "profile", "name")
print(name)  # "John"

# Handle missing keys gracefully
city = DataHelper.safe_get(data, "user", "address", "city", default="Unknown")
print(city)  # "Unknown"
```

### Date Normalization

```python
# Convert various date formats
date1 = DataHelper.normalize_date("2024-01-15T10:30:00Z")
date2 = DataHelper.normalize_date("2024-01-15")
date3 = DataHelper.normalize_date(1705319400)  # Unix timestamp

print(date1)  # datetime object
```

## Advanced Use Case: RAG Pipeline

```python
class RAGProcessor:
    def __init__(self):
        self.chunker = AdvancedChunker(
            model_name="gpt-4o",
            max_tokens=500,
            overlap_sentences=2
        )
    
    def process_document(self, document: str) -> List[dict]:
        # Clean text first
        cleaned = DataHelper.clean_text(document)
        
        # Chunk the document
        chunks = self.chunker.chunk_text(cleaned)
        
        # Create chunks with metadata
        processed = []
        for i, chunk in enumerate(chunks):
            processed.append({
                "id": DataHelper.make_id("doc", f"chunk_{i}"),
                "text": chunk,
                "tokens": self.chunker._get_token_count(chunk),
                "keywords": DataHelper.extract_keywords(chunk),
                "language": DataHelper.detect_language(chunk)
            })
        
        return processed

# Usage
processor = RAGProcessor()
chunks = processor.process_document(long_document)
```

## Configuration Options

### AdvancedChunker Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `model_name` | `str` | `"gpt-4o"` | Model for tiktoken tokenizer |
| `max_tokens` | `int` | `400` | Max tokens per chunk |
| `overlap_sentences` | `int` | `2` | Sentences to overlap between chunks |
| `min_chunk_tokens` | `int` | `50` | Minimum tokens for valid chunk |
| `chunk_delimiter` | `str` | `"\n\n"` | Separator between chunks |

### DataHelper Methods

| Method | Description |
|--------|-------------|
| `clean_text()` | Remove extra spaces and blank lines |
| `truncate()` | Cut text to max characters |
| `extract_keywords()` | Simple frequency-based keyword extraction |
| `detect_language()` | Basic language detection (fa/ar/en) |
| `chunk_text()` | Quick text splitting (simple version) |
| `make_id()` | Generate MD5 hash from source + identifier |
| `retry()` | Sync retry decorator with exponential backoff |
| `async_retry()` | Async version of retry |
| `normalize_date()` | Convert multiple date formats to datetime |
| `safe_get()` | Safe nested dictionary access |
| `print_result_summary()` | Pretty print processing results |

## Performance Notes

- **Token counting** uses `tiktoken` for OpenAI-compatible accuracy
- **Overlap preservation** recalculates token counts for precision
- **Single sentences exceeding max_tokens** are handled as separate chunks with warnings
- **Memory efficient** - processes sequentially, no huge intermediate structures

## Requirements

```txt
tiktoken>=0.5.0
loguru>=0.7.0
```

## Use Cases

- 📚 **Document splitting** for LLM context windows
- 🔍 **RAG systems** - prepare documents for retrieval
- 🧠 **Embedding preparation** before vector store insertion
- 📝 **Text preprocessing** for NLP pipelines
- 🌐 **Multi-lingual applications** (Persian, Arabic, English)
- ⚡ **Batch document processing** with progress tracking

---

**Perfect for:** LangChain alternatives, custom RAG implementations, document processing pipelines, and any application needing smart text chunking!
