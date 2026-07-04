# Marker PDF to Markdown Conversion Guide

## Overview

[marker](https://github.com/datalab-to/marker) converts PDFs to markdown (or JSON/HTML) using vision models. It can use LLM backends for higher quality output.

**Token cost efficiency:** Skip `--use_llm` for simple scanned PDFs (just text/tables). Use it only for complex layouts, equations, or figures where LLM significantly improves output quality. When using LLMs, prefer multimodal models (e.g., `deepseek-v4-flash`, `minimax-m2.7`) — they handle visual content far better. Non-multimodal models work but are significantly worse for anything with figures or complex layout.

## Installation

marker-pdf has a Pillow dependency that requires **Python 3.12** (Python 3.14 has no compatible binary wheel).

```bash
# Install Python 3.12 via uv
uv python install 3.12

# Create venv with Python 3.12
uv venv .venv312 --python 3.12

# Install marker-pdf
uv pip install --python .venv312/bin/python marker-pdf

# Fix missing psutil if needed
uv pip install --python .venv312/bin/python psutil
```

## Important: Page Range is 0-Indexed

marker uses **0-based** page indexing internally. To convert human pages 23-37, use:

```bash
--page_range "22-36"
```

Range syntax: comma-separated numbers and ranges, e.g. `"0,5-10,20"` — end is **inclusive**.

## Basic Usage

### Single PDF, single-file converter

```bash
marker_single input.pdf --output_format markdown
```

### With page range

```bash
marker_single input.pdf \
  --page_range "22-36" \
  --output_format markdown \
  --output_dir ./output
```

### With LLM backend (OpenAI-compatible)

```bash
marker_single input.pdf \
  --page_range "22-36" \
  --output_format markdown \
  --output_dir ./output \
  --use_llm \
  --llm_service marker.services.openai.OpenAIService \
  --openai_api_key "sk-..." \
  --openai_base_url "https://your-endpoint/v1" \
  --openai_model "your-model"
```

## LLM Backend Configuration

marker does **not** use `DOCLING_TEXT_*` env vars — those are docling-specific. marker uses `OpenAIService` for OpenAI-compatible endpoints.

### CLI flags

| Flag | Description |
|------|-------------|
| `--llm_service` | Service class, e.g. `marker.services.openai.OpenAIService` |
| `--openai_api_key` | API key |
| `--openai_base_url` | Base URL (no trailing slash) |
| `--openai_model` | Model name |

### Environment variables (pydantic-settings)

| Env Var | Description |
|---------|-------------|
| `OPENAI_API_KEY` | API key |
| `OPENAI_BASE_URL` | Base URL |
| `OPENAI_MODEL` | Model name |

### Other available services

| Service | Class |
|---------|-------|
| Google Gemini | `marker.services.gemini.GoogleGeminiService` |
| Claude | `marker.services.claude.ClaudeService` |
| Azure OpenAI | `marker.services.azure_openai.AzureOpenAIService` |
| Ollama | `marker.services.ollama.OllamaService` |
| Google Vertex | `marker.services.vertex.GoogleVertexService` |

## JSON Config File

```bash
marker_single input.pdf --config_json ./marker_config.json
```

```json
{
  "output_format": "markdown",
  "page_range": "22-36",
  "use_llm": true,
  "llm_service": "marker.services.openai.OpenAIService",
  "openai_api_key": "sk-...",
  "openai_base_url": "https://your-endpoint/v1",
  "openai_model": "your-model"
}
```

## Output

Output goes to `{output_dir}/{pdf_basename}/{pdf_basename}.md`

Images extracted from the PDF are saved in the same folder with names like `_page_29_Figure_2.jpeg`.

## Common Issues

### FileNotFoundError with Unicode filenames

If the PDF path has Unicode characters (like smart quotes), use Python to invoke marker:

```bash
python3 -c "
import subprocess, os
pdf_path = next(f for f in os.listdir('/path/to/dir') if 'keyword' in f)
subprocess.run(['marker_single', pdf_path, '--page_range', '22-36', ...])
"
```

### Unicode output directory listing

```bash
python3 -c "
from pathlib import Path
md_file = next(Path('./output').glob('*/*.md'))
print(md_file.read_text()[:500])
"
```

### Pillow build failure on Python 3.14

Use Python 3.12 — see Installation section above.

## Full Example

```bash
uv python install 3.12
uv venv .venv312 --python 3.12
uv pip install --python .venv312/bin/python marker-pdf

# Activate venv
source .venv312/bin/activate

# Convert pages 23-37 with LLM
marker_single "/path/to/book.pdf" \
  --page_range "22-36" \
  --output_format markdown \
  --output_dir ./output \
  --use_llm \
  --llm_service marker.services.openai.OpenAIService \
  --openai_api_key "sk-your-key" \
  --openai_base_url "https://api.example.com/v1" \
  --openai_model "gpt-4o-mini"
```

## Key Reference Links

- marker GitHub: https://github.com/datalab-to/marker
- Page range parsing: `marker/util.py` — `parse_range_str()`
- OpenAI service: `marker/services/openai.py`
- Config parser: `marker/config/parser.py`