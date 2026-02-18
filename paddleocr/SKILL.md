---
name: paddleocr
description: >
  Build and operate PaddleOCR pipelines for multilingual document understanding with model
  selection, tuning, and deployment optimization. Use for PP-OCR model strategy, detector/recognizer
  configuration, preprocessing, finetuning plans, and production rollout guardrails.
---

# PaddleOCR

## Workflow
1. Confirm document categories, language coverage, and latency targets.
2. Choose OCR pipeline version and model family (mobile/server).
3. Configure detection/recognition and optional orientation/unwarping modules.
4. Tune preprocessing and threshold parameters using validation sets.
5. Profile runtime performance across target hardware.
6. Validate output quality with domain-specific acceptance checks.
7. Deploy with versioned models and rollback controls.

## Preflight (Ask / Check First)
- PaddleOCR version and runtime backend.
- Supported languages and mixed-language frequency.
- Hardware profile (CPU/GPU/NPU/edge).
- Throughput and batch requirements.
- Existing quality pain points (missed text, false positives, layout drift).

## Model Selection Strategy
- Choose PP-OCR series based on accuracy/latency needs.
- Use server models for accuracy-critical workloads.
- Use mobile models for edge or high-throughput constraints.
- Validate `ocr_version` and `lang` compatibility before rollout.
- Pin model artifacts for reproducibility.

## Pipeline Configuration
- Enable orientation and unwarping when document skew/warp is common.
- Keep optional modules disabled when they add latency without quality gain.
- Route by document class if one global pipeline underperforms.
- Keep deterministic preprocessing across environments.

### Example Pipeline
```python
from paddleocr import PaddleOCR

ocr = PaddleOCR(
    text_detection_model_name="PP-OCRv5_server_det",
    text_recognition_model_name="PP-OCRv5_server_rec",
    device="gpu:0",
    use_doc_orientation_classify=True,
    use_doc_unwarping=True,
)
```

## Multilingual Handling
- Use multilingual models where supported by workload.
- Route rare languages to specialized models when needed.
- Validate script-specific confusion (Latin/Cyrillic/CJK mix).
- Keep language metadata in downstream quality reporting.

## Performance and Deployment
- Benchmark CPU vs GPU for real document mix, not synthetic tests.
- Tune batch size and concurrency for stable latency percentiles.
- Measure memory footprint and startup time for autoscaling.
- Package model files with checksum verification.
- Keep warmup path for serving workloads.

## Quality and Post-Processing
- Track word/line-level accuracy by document class.
- Add business validators (fields, totals, identifiers).
- Use confidence thresholds for human review routing.
- Capture drift metrics per source channel.

## Security and Operations
- Protect input/output data with encryption and access controls.
- Avoid logging raw PII-bearing OCR text.
- Keep data retention and deletion policies enforceable.
- Maintain incident runbook for model regressions.

## Validation Commands
```bash
python -c "from paddleocr import PaddleOCR; print('ok')"
```

## Common Failure Modes
- Mismatched model version and language set.
- Over-enabled modules creating latency without quality gains.
- Ignoring document-type routing in mixed workloads.
- No rollback path for model quality regressions.
- Unbounded resource usage in high-volume batches.

## Definition of Done
- Model/version selection is justified and pinned.
- Accuracy and latency are benchmarked on representative data.
- Deployment supports rollback and observability.
- Confidence and post-processing controls are in place.
- Security controls protect OCR data lifecycle.

## References
- `references/paddleocr-2026-02-18.md`

## Reference Index
- `rg -n "PP-OCR|ocr_version|lang" references/paddleocr-2026-02-18.md`
- `rg -n "mobile|server|device" references/paddleocr-2026-02-18.md`
- `rg -n "orientation|unwarping|pipeline" references/paddleocr-2026-02-18.md`
- `rg -n "deployment|quality|operations" references/paddleocr-2026-02-18.md`
