# Convert PDFs to clean markdown with GenAI


## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Approach Comparison](#approach-comparison)
- [Advanced Options](#advanced-options)
- [License](#license)

## 🎯 Overview

Complex PDFs often contain a mix of text, tables, charts, and images that are difficult to extract cleanly. Traditional ML methods and libraries fall short as PDF formats are so diverse. The only truly reliable way to extract and structure content is to use a multimodal LLM with vision capabilities. This project demonstrates two approaches to convert PDFs to Markdown using multi-modal LLMs. Once the PDFs are in Markdown, they can be easily ingested into vector databases or sent to LLMs directly for GenAI applications such as retrieval-augmented generation (RAG) chatbots or extracting structured data from unstructured documents. 
This project offers two distinct methods for converting PDFs to Markdown:

1. **Full Page Vision OCR** (`pdf_to_markdown_full_ocr.py`): Renders entire PDF pages as images and uses LLM vision to transcribe and structure content
2. **Hybrid Text + Image Description** (`pdf_to_markdown_with_image_descriptions.py`): Extracts text using PyMuPDF and replaces embedded images with AI-generated descriptions

Both approaches leverage GPT-4.1's multimodal capabilities to handle images and graphics.

## ✨ Features

- 📖 **Accurate text extraction** with layout preservation
- 🖼️ **Intelligent image handling** - converts charts, diagrams, and figures to descriptive text
- 📊 **Table structure preservation** in Markdown format
- 🎯 **Heading hierarchy detection** for proper document structure
- 📝 **Multiple formatting styles** support (bold, italic, lists, blockquotes)
- ⚡ **Configurable processing** with page ranges and quality settings
- 🔧 **Environment-based configuration** for easy deployment
- 📦 **Size-optimized image processing** with automatic quality adjustment

## 🏗️ Architecture

### Full Page Vision OCR Workflow

```mermaid
graph TD
B[Convert each page in PDF to image]
B --> C
subgraph "For each image"
direction LR
C[Pass image to LLM] --> D[LLM transcribes text and describes images/diagrams]
end
D --> F[Combine all transcriptions/descriptions into final markdown output]


%% ---- Reusable style classes (readable on GitHub light & dark) ----
classDef llm fill:#FFB000,stroke:#333,stroke-width:1px,color:#111;
classDef nonllm fill:#A3D3FF,stroke:#333,stroke-width:1px,color:#111;
classDef output fill:#B8F2C2,stroke:#333,stroke-width:1px,color:#111;
classDef term fill:#EDEDED,stroke:#333,stroke-width:1px,color:#111;

%% ---- Apply classes ----
class B nonllm;
class C llm;
class D llm;
class F output;
class G term;

```

### Hybrid Text + Image Description Workflow

```mermaid
graph TD
A[Convert PDF to Markdown via PyMuPDF4LLM]
subgraph "For each image"
direction LR
B[Pass image to LLM] --> C[LLM generates description of image]
C --> D[Replace base64 with description in Markdown]
end
A --> B
D --> E[Final Markdown Output]

%% ---- Reusable style classes (readable on GitHub light & dark) ----
classDef llm fill:#FFB000,stroke:#333,stroke-width:1px,color:#111;
classDef nonllm fill:#A3D3FF,stroke:#333,stroke-width:1px,color:#111;
classDef output fill:#B8F2C2,stroke:#333,stroke-width:1px,color:#111;
classDef term fill:#EDEDED,stroke:#333,stroke-width:1px,color:#111;

%% ---- Apply classes ----
class A nonllm;
class B llm;
class C llm;
class D nonllm;
class E output;

```

## 🔧 Installation

### Prerequisites
- Python 3.12+
- Azure OpenAI deployment (GPT-4 Vision recommended)

### Create a virtual environment (recommended)

Windows (PowerShell):
```powershell
python -m venv .venv
# Activate the venv
.\.venv\Scripts\Activate.ps1
# If you see an execution policy error, run (once) in an elevated PowerShell:
#   Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

macOS / Linux:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### Install Dependencies

```bash
# Upgrade pip inside the venv and install required packages
python -m pip install --upgrade pip
pip install -r requirements.txt
```


## ⚙️ Configuration

Rename `example.env` to `.env` in the project root and set the required environment variables:

```env
# Required Azure AI configuration
PROJECT_ENDPOINT=https://your-project.openai.azure.com/
MODEL_DEPLOYMENT_NAME=gpt-4-vision-preview  # or your deployment name
AZURE_OPENAI_API_VERSION=2024-02-01

# Optional: Context window for image descriptions (Hybrid approach only)
VISION_CONTEXT_BEFORE_CHARS=400
VISION_CONTEXT_AFTER_CHARS=400
VISION_MAX_CONTEXT_CHARS=1000
```

### Azure Setup

1. Create an Azure AI Project in Azure AI Foundry
2. Deploy GPT4.1
3. Ensure your Azure credentials are configured (az login for local dev or use Managed Identity in Azure)
    
## 📖 Usage

### Full Page Vision OCR Approach

Best for: **Scanned documents, complex layouts, handwritten text, or when maximum fidelity is needed. Also useful when each page contains many images that should be processed together**

```bash
# Basic usage
python pdf_to_markdown_full_ocr.py input.pdf

# Specify output file
python pdf_to_markdown_full_ocr.py input.pdf -o output.md

# Process specific pages
python pdf_to_markdown_full_ocr.py input.pdf --start 1 --end 10

# Without page headers
python pdf_to_markdown_full_ocr.py input.pdf --no-page-headings
```

### Hybrid Text + Image Description Approach

Best for: **Digital PDFs with embedded images**

```bash
# Basic usage
python pdf_to_markdown_with_image_descriptions.py input.pdf

# Specify output file
python pdf_to_markdown_with_image_descriptions.py input.pdf -o output.md
```

## 🔄 Approach Comparison

| Feature | Full Page Vision | Hybrid Approach |
|---------|-----------------|-----------------|
| **Method** | Renders entire pages as images | Extracts text via PyMuPDF4LLM, processes images separately |
| **Best For** | Scanned PDFs, complex layouts | Digital PDFs with clear text |
| **Accuracy** | Highest for all content types | High for text, depends on PyMuPDF for structure |
| **Processing Time** | Slower (full page vision) | Faster (unless multiple images per page) |
| **Token Usage** | Higher (~3500 tokens/page) | Lower (varies by image count) |
| **Layout Preservation** | Excellent | Excellent |
| **Handwritten Text** | ✅ Supported | ❌ Not supported |
| **Image Handling** | Describes in context | Describes with surrounding text |
| **Page Range Selection** | ✅ Supported | ❌ Full document only |

## 🎛️ Advanced Options

### Full Page Vision Settings

Modify these constants in `pdf_to_markdown_full_ocr.py`:

```python
RENDER_DPI = 280                # Image quality (260-320 recommended)
MAX_IMAGE_BYTES = 20 * 1024**2  # Max size per page (20MB default)
JPEG_QUALITY_START = 85         # Initial JPEG quality
MIN_JPEG_QUALITY = 35           # Minimum before downscaling
DOWNSCALE_FLOOR_PX = 720        # Minimum edge size when downscaling
MAX_TOKENS_PER_PAGE = 3500      # LLM token limit per page
TEMPERATURE = 0.0               # LLM temperature (0 = deterministic)
```

### Output Format Examples

**Full Page Vision Output:**
```markdown
## Page 1

# Executive Summary

This document presents quarterly results...

**Key Metrics:**
- Revenue: $5.2M
- Growth: 23% YoY
- Customer satisfaction: 94%

IMAGE: Bar chart showing quarterly revenue trends from Q1-Q4 2024, with values 
ranging from $4.1M to $5.2M, showing steady upward progression

## Financial Overview

| Quarter | Revenue | Expenses | Profit |
|---------|---------|----------|--------|
| Q1      | $4.1M   | $3.2M    | $0.9M  |
| Q2      | $4.5M   | $3.3M    | $1.2M  |
...
```

**Hybrid Approach Output:**
```markdown
# Executive Summary

This document presents quarterly results...

**Key Metrics:**
- Revenue: $5.2M
- Growth: 23% YoY
- Customer satisfaction: 94%

> Image: Bar chart displaying quarterly revenue data for 2024. The chart shows 
> four bars representing Q1 through Q4, with values of $4.1M, $4.5M, $4.8M, 
> and $5.2M respectively. The bars are colored in gradient blue, with a trend 
> line indicating 23% year-over-year growth.

## Financial Overview
...
```

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

