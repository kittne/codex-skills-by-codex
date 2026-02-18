# Tesseract OCR Reference (2026-02-18)

## Context7 Sources
- Library: `/tesseract-ocr/tessdoc`
- Tesseract command-line usage and quality-improvement docs.

## Practical Guidance
- Choose PSM by layout type.
- Use preprocessing (deskew, binarization, denoise) as first lever for accuracy.
- Use `preserve_interword_spaces=1` when layout spacing matters.
- Use character whitelist and dictionary disable switches for constrained text domains.
- Use `tessedit_write_images=true` for internal preprocessing diagnostics.

## Source URLs
- <https://github.com/tesseract-ocr/tessdoc>
- <https://github.com/tesseract-ocr/tesseract>
