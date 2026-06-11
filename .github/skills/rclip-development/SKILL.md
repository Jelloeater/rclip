---
name: rclip-development
description: >
  Guide for developing, testing, and maintaining the rclip codebase.
  Use this skill whenever working on rclip source code, tests, or CI workflows.
license: MIT
version: 1.0.0
author: Jelloeater
tags:
  - python
  - ai
  - image-search
  - clip
  - computer-vision
platforms:
  - linux
  - macos
  - windows
---

## Project Overview

**rclip** is a semantic photo-search CLI tool powered by OpenCLIP's ViT-B/32 model. It builds a local SQLite index of image feature vectors and searches them with natural-language text or image queries — entirely on-device.

Key source files:

| File | Purpose |
|------|---------|
| `rclip/main.py` | Core `RClip` class (indexing + search) and CLI entry point |
| `rclip/model.py` | CLIP/ONNX model wrapper — computes image & text feature vectors |
| `rclip/db.py` | SQLite database layer — stores image paths, metadata, and vectors |
| `rclip/fs.py` | Filesystem walker that discovers image files |
| `rclip/const.py` | Supported image extensions (`IMAGE_EXT`, `IMAGE_RAW_EXT`) |
| `rclip/utils/helpers.py` | CLI argument parser, path utilities |
| `rclip/utils/preview.py` | In-terminal image previews (iTerm2, Konsole, wezterm, …) |

## Environment Setup

This project uses [uv](https://docs.astral.sh/uv/) for dependency management (Python 3.11–3.13 required).

```bash
uv sync          # install all dependencies (dev group included by default)
```

## Linting

```bash
make lint        # runs both style and type checks
make lint-style  # ruff check (style)
make lint-types  # ty check (type checker)
make fix-style   # auto-fix style issues with ruff
```

Configuration lives in `pyproject.toml` under `[tool.ruff]` and `[tool.ty]`. Line length is 120, indent width is 2.

## Testing

```bash
make test        # uv run pytest tests/
```

Tests live in `tests/`. End-to-end tests (`tests/e2e/`) require real image files and are skipped unless `RCLIP_TEST_RUN_SYSTEM_RCLIP=true`.

When adding or changing features, update or add tests in `tests/` accordingly.

## Architecture Notes

- **Feature vectors** are stored as raw `float32` bytes in SQLite. `numpy.frombuffer` / `.tobytes()` are used to serialise/deserialise.
- **Incremental indexing**: images are re-indexed only when `mtime` or `size` changes. A "flagging" approach marks images as being indexed, then clears the flag on re-visit, so stale entries can be removed.
- **Query arithmetic**: queries like `"2:golden retriever" + "./pool.jpg" - fruit` are parsed in `rclip/utils/helpers.py` and resolved to weighted positive/negative feature vectors.
- **RAW support** is opt-in via `--experimental-raw-support`; RAW files are skipped if a processed sidecar exists alongside them.
- **Model download**: the ONNX model is fetched from Hugging Face Hub on first use (`rclip/model_download.py`).

## Conventional Commits

All commits in this repository follow the [Conventional Commits](https://www.conventionalcommits.org/) specification. Use prefixes like `feat:`, `fix:`, `refactor:`, `test:`, `chore:`, `docs:` in commit messages.

## CI

The CI workflow (`.github/workflows/validate.yaml`) runs on every PR and push to `main`:

1. `lint` — runs `.github/actions/lint`
2. `test` — matrix across Python 3.11/3.12/3.13 × Linux/macOS/Windows

Always ensure both `make lint` and `make test` pass locally before committing.
