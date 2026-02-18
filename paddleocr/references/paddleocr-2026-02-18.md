# PaddleOCR Reference (2026-02-18)

## Context7 Sources
- Library: `/paddlepaddle/paddleocr`
- PP-OCR pipeline docs and release notes surfaced via Context7.

## Practical Guidance
- Select `ocr_version` and `lang` as a tested pair.
- Choose server vs mobile model families based on accuracy/latency needs.
- Enable orientation/unwarping only for document classes that benefit.
- Validate multilingual behavior and script confusion explicitly.
- Keep model artifacts pinned for deterministic rollouts.

## Source URLs
- <https://github.com/PaddlePaddle/PaddleOCR>
- <https://www.paddleocr.ai/>
