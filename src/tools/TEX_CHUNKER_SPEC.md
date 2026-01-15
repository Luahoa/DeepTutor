# TeX Chunker Specification

## Overview

The `TexChunker` tool is designed to intelligently split LaTeX content into smaller, manageable chunks for processing by Large Language Models (LLMs). It aims to preserve semantic context by prioritizing splits at logical boundaries (sections, paragraphs) while respecting token limits.

## Core Logic

### 1. Initialization
- The chunker is initialized with a specific LLM model name (defaulting to `gpt-4o` or `LLM_MODEL` env var).
- It uses `tiktoken` to create an encoder for accurate token estimation.

### 2. Token Estimation
- **Method:** `estimate_tokens(text)`
- **Preprocessing:** Clean text to remove anomalies like excessively long repeated characters (which can cause token explosion) and truncate extremely long lines.
- **Encoding:** Uses the model's tokenizer to count tokens.
- **Fallback:** If encoding fails, approximates using `len(text) // 4`.

### 3. Chunking Strategy
- **Method:** `split_tex_into_chunks(tex_content, max_tokens, overlap)`
- **Workflow:**
    1.  **Global Check:** If the total token count of the document is within `max_tokens`, return the document as a single chunk.
    2.  **Section Splitting:** The document is first split by LaTeX section markers (`\section`, `\subsection`, `\subsubsection`).
    3.  **Chunk Assembly:**
        - Iterate through the sections.
        - **Case A: Section fits in current chunk:** Add it to the current chunk.
        - **Case B: Section doesn't fit, but fits in a new chunk:** Close current chunk (adding overlap to the next), start new chunk with this section.
        - **Case C: Section is larger than `max_tokens`:**
            - The section is further split by paragraphs (double newlines).
            - If a paragraph is still too large, it is split by sentences.
            - These sub-units are added to chunks, maintaining the `max_tokens` limit and adding overlap where necessary.

### 4. Overlap Handling
- **Method:** `_get_overlap_text(previous_chunk, overlap_tokens)`
- When a new chunk is started, it includes the last `overlap_tokens` from the previous chunk to ensure context continuity.
- Overlap is calculated based on token count, not character count.

## regex Patterns used

- **Section Split:** `r"(\\(?:sub)*section\{[^}]*\})"`
- **Paragraph Split:** `r"\n\n+"`
- **Sentence Split:** `r"(?<=[.!?])\s+"`
- **Repeated Char Cleanup:** `r"(\s)\1{100,}"`

## Configuration
- **max_tokens:** Maximum tokens allowed per chunk (default: 8000).
- **overlap:** Number of tokens to overlap between chunks (default: 500).
