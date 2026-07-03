---
name: rclip-development
description: >
  Guide for using the rclip CLI to search a local photo library with natural-language or image queries.
  Use this skill when an agent needs to search for images on disk using rclip.
license: MIT
version: 2.0.0
author: Jelloeater
tags:
  - cli
  - image-search
  - ai
  - clip
  - computer-vision
platforms:
  - linux
  - macos
  - windows
---

## What is rclip?

**rclip** is a semantic photo-search CLI tool. It lets you search a local image library using natural-language descriptions, similar images, or a mix of both — entirely on-device with no cloud uploads.

## Installation

```bash
# Linux (snap)
sudo snap install rclip

# macOS (Homebrew, Apple Silicon only)
brew install yurijmikhalevich/tap/rclip

# Any platform (pip)
pip install rclip
```

On the first run in a directory rclip builds a search index. Subsequent runs only reprocess new or changed images.

## Basic Usage

Run rclip from the directory that contains your images (or pass a path with `--dir`):

```bash
cd /path/to/photos
rclip "search query"
```

Default output (score + filepath, top 10 results):

```
score  filepath
0.297  /photos/sunrise-beach.jpg
0.286  /photos/dawn-walk.png
0.274  /photos/morning-hike.heic
```

Higher score = closer match.

## Image-to-Image Search

Pass a local file path (must start with `./`) or an image URL as the query:

```bash
rclip ./cat.jpg
rclip https://example.com/cat.jpg
```

## Combined & Arithmetic Queries

Mix and weight text and image queries with `+` / `-`:

```bash
rclip horse + stripes               # horses that have stripes
rclip apple - fruit                 # apple-like things that aren't fruit
rclip "./city.jpg" + night          # similar to city.jpg but at night
rclip "2:golden retriever" + "./pool.jpg"   # weight a term with a multiplier
rclip "./racing-car.jpg" - "2:sports car" + "2:snow"
```

Multipliers (`2:`, `0.5:`) scale how strongly a term influences the result. Image queries are typically weighted higher than text ones, so a `0.5:` prefix can help balance mixed queries.

## Key Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--top N` | `-t N` | Return top N results (default: 10) |
| `--filepath-only` | `-f` | Print only file paths, no scores or header |
| `--no-indexing` | `-n` | Skip re-indexing (faster when you know no images changed) |
| `--exclude-dir DIR` | | Exclude a directory from search (repeatable; overrides defaults: `@eaDir`, `node_modules`, `.git`) |
| `--experimental-raw-support` | | Enable RAW format support (`arw`, `cr2`, `dng`) |

## Supported Image Formats

Always indexed: `jpg`, `jpeg`, `png`, `webp`, `heic`, `tiff`, `tif`, `bmp`, `gif`, `jp2`, `pnm`, `pbm`, `pgm`, `ppm`.

RAW formats (`arw`, `cr2`, `dng`) require `--experimental-raw-support`.

## Automation & Piping

Use `-f` to emit plain file paths for piping into other tools:

```bash
# Open top 5 results in the default viewer (Linux)
rclip -f -t 5 "sunset over the ocean" | xargs -d '\n' -n 1 xdg-open

# Copy top 3 matches to another directory
rclip -f -t 3 "golden retriever" | xargs -I {} cp {} /path/to/destination

# List matching files for further processing
rclip -f -t 20 "birthday party" > matches.txt
```

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `RCLIP_DATADIR` | Override the directory where the search index (SQLite DB) is stored |
| `RCLIP_MODEL_CACHE_DIR` | Override the directory where the ONNX model is cached |
