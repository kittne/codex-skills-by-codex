---
name: tesseract-ocr
description: >
  Build and operate Tesseract OCR pipelines with strong preprocessing, segmentation tuning,
  and output quality controls. Use for OCR CLI/runtime configuration, language pack strategy,
  image normalization, debugging low-accuracy cases, and production deployment workflows.
---

# Tesseract OCR

## Workflow
1. Confirm document classes, language mix, and quality targets.
2. Build preprocessing pipeline (rescale, denoise, binarize, deskew).
3. Select OCR engine mode and page segmentation mode per document type.
4. Tune dictionaries, whitelists, and language model packs.
5. Add post-processing and confidence-based review logic.
6. Benchmark accuracy and latency on representative datasets.
7. Operationalize monitoring and failure triage.

## Preflight (Ask / Check First)
- Tesseract version and installed language data.
- Input quality (resolution, skew, compression artifacts).
- Document classes (receipts, IDs, forms, books).
- Accuracy target and acceptable manual-review rate.
- Runtime constraints (CPU budget, throughput, batch size).

## Preprocessing Strategy
- Normalize resolution before OCR pass.
- Deskew aggressively for scanned pages.
- Apply denoise/binarization tuned to source quality.
- Preserve necessary borders; trim noisy margins.
- Keep preprocessing deterministic and auditable.

## Segmentation and Engine Tuning
- Choose PSM per layout type (`6` block, `7` line, `11` sparse text).
- Use OEM settings aligned with installed model support.
- Preserve interword spacing when layout fidelity matters.
- Disable dictionaries for serials/codes/non-language text.
- Use character whitelists for constrained fields.

### Typical Commands
```bash
tesseract input.png output --psm 6 -l eng
tesseract input.png output --psm 6 -c preserve_interword_spaces=1
tesseract numbers.png output -c tessedit_char_whitelist=0123456789
```

## Language and Model Management
- Install only required language packs for each workload.
- Route documents by language when multilingual accuracy is critical.
- Keep model versioning pinned for reproducible output.
- Revalidate after any language pack upgrade.

## Quality and Post-Processing
- Use confidence thresholds to route low-quality outputs.
- Normalize Unicode and whitespace in post-processing.
- Add domain-specific validators (date, ID format, totals consistency).
- Capture OCR debug artifacts for hard failures.

## Debugging and Operations
- Use `tessedit_write_images=true` to inspect internal preprocessing output.
- Track accuracy by document type and source channel.
- Sample and manually review low-confidence outputs.
- Keep retry/requeue strategy for transient image handling failures.
- Maintain deterministic fallback path for unsupported layouts.

## Security and Compliance
- Redact sensitive OCR output in logs.
- Encrypt source and extracted text at rest/in transit.
- Limit data retention to policy minimums.
- Restrict access to training/validation datasets.

## Validation Commands
```bash
tesseract input.png stdout --psm 6 -l eng
tesseract input.png stdout --psm 11 -l eng quiet
```

## Common Failure Modes
- Wrong PSM choice for document layout.
- No deskew/denoise on poor scans.
- Mixing languages without routing.
- Over-aggressive binarization removing text features.
- Missing confidence gating for noisy inputs.

## Definition of Done
- Preprocessing and OCR configs are documented per document class.
- Accuracy and latency are benchmarked on representative data.
- Confidence gating and manual review policy are in place.
- Debug workflow exists for hard OCR failures.
- Security controls cover extracted text lifecycle.

## References
- `references/tesseract-ocr-2026-02-18.md`

## Reference Index
- `rg -n "psm|oem|whitelist|dictionary" references/tesseract-ocr-2026-02-18.md`
- `rg -n "preprocess|deskew|binar" references/tesseract-ocr-2026-02-18.md`
- `rg -n "tessedit_write_images|debug" references/tesseract-ocr-2026-02-18.md`
- `rg -n "quality|confidence|operations" references/tesseract-ocr-2026-02-18.md`
