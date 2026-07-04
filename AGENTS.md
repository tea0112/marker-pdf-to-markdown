# PROJECT KNOWLEDGE BASE

**Generated:** 2026-07-04
**Commit:** b594c20
**Branch:** main

## OVERVIEW

Documentation wrapper for [marker-pdf](https://github.com/datalab-to/marker) — converts PDFs to markdown/JSON/HTML using vision models. This repo contains usage docs only; marker itself is installed in `.venv312/`.

## STRUCTURE

```
marker-pdf-to-markdown/
├── README.md         # All documentation (178 lines)
├── .gitignore        # Standard Python + .omo/
├── .venv312/        # Python 3.12 venv with marker-pdf installed
├── .venv/           # Python 3.14 venv (incomplete)
├── .venv2/          # Secondary venv
└── .omo/            # OpenCode session state
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| marker-pdf usage docs | README.md | Full guide with examples |
| Page range parsing | marker source in .venv312 | `marker/util.py` — `parse_range_str()` |
| OpenAI service impl | marker source in .venv312 | `marker/services/openai.py` |
| Config parser | marker source in .venv312 | `marker/config/parser.py` |

## CONVENTIONS

- **Page indexing**: marker uses 0-based page numbers internally
- **Python version**: Requires Python 3.12 (Pillow has no 3.14 wheel)
- **LLM flag**: `--use_llm` only for complex docs; skip for simple text/tables
- **Output path**: `{output_dir}/{pdf_basename}/{pdf_basename}.md`
- **Image naming**: `_page_{N}_Figure_{M}.jpeg`

## ANTI-PATTERNS (THIS PROJECT)

- Do NOT use Python 3.14 — Pillow build fails
- Do NOT use non-multimodal LLMs for figure-heavy documents
- Do NOT track `.omo/run-continuation/` in git (already ignored)

## COMMANDS

```bash
# Install marker (Python 3.12 required)
uv python install 3.12
uv venv .venv312 --python 3.12
uv pip install --python .venv312/bin/python marker-pdf
uv pip install --python .venv312/bin/python psutil

# Convert PDF (basic)
marker_single input.pdf --output_format markdown

# Convert with page range (0-based: human pages 23-37 → "22-36")
marker_single input.pdf --page_range "22-36" --output_format markdown

# Convert with LLM backend
marker_single input.pdf --page_range "22-36" --output_format markdown \
  --use_llm \
  --llm_service marker.services.openai.OpenAIService \
  --openai_api_key "sk-..." \
  --openai_base_url "https://your-endpoint/v1" \
  --openai_model "your-model"
```

## NOTES

- This is a documentation-only repo — no source code here
- Actual marker code lives in `.venv312/lib/python3.12/site-packages/marker/`
- Multiple venvs exist from experimentation; `.venv312` is the recommended one
- Git history: only 2 commits (initial + README update)