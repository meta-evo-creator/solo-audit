---
name: babata-superocr
version: 1.0.0
description: |
  Babata SuperOCR v1.0.0 — Dual-engine OCR (PaddleOCR document + RapidOCR handwriting). Printed Chinese docs, handwriting recognition, PDF/image text extraction. Local, zero API dependency.
  Use when: OCR, image text extraction, PDF to text, handwriting recognition, scanned documents.
  NOT for: video, audio, real-time camera feed.
metadata:
  openclaw:
    emoji: 📝
    requires:
      bins: [python]
      env: []
---

# Babata SuperOCR 📝 v1.0

> Dual-engine. PaddleOCR for printed documents, RapidOCR for handwriting. Auto-routing. Zero API.

## Dual-Engine Architecture

```
Input: PDF / Image / Screenshot
         │
         ▼
    ┌─────────────┐
    │ Auto-detect  │ ← Printed? Handwriting?
    └──────┬──────┘
           │
      ┌────┴────┐
      ↓         ↓
  ┌────────┐┌────────┐
  │Paddle  ││ Rapid  │
  │OCR     ││ OCR    │
  │Printed ││Handwrite│
  └───┬────┘└───┬────┘
      │         │
      └────┬────┘
           ↓
       text output
```

## Engine Selection

| Scenario | Engine | Why |
|:---------|:------:|:-----|
| 印刷文档/合同/报告 | PaddleOCR PP-OCRv4 | 99% accuracy on Chinese |
| PDF扫描件 | PaddleOCR | Document OCR specialist |
| 手写体/签名/便签 | RapidOCR | 60-85% handwriting accuracy |
| 混合文档(印刷+手写) | both engine | Merge results |

## Fallback Chain

```
PaddleOCR (primary, printed docs)
  ↓ incompatible / fails
RapidOCR (handwriting/lightweight)
  ↓ all fail
「OCR不可达」— 人工处理
```

## Scripts

| Script | Engine | Use |
|:-------|:------:|:----|
| `ocr_paddle.py` | PaddleOCR | Printed Chinese documents |
| `ocr_rapid.py` | RapidOCR | Handwriting/quick OCR |
| `ocr_batch.py` | Both | Batch processing |
| `ocr_benchmark.py` | Both | Accuracy benchmarking |

## LEARNED PATTERNS

### Dual-engine local-first
No API. No network. No quota. PaddleOCR for docs, RapidOCR for handwriting. Simple, reliable, always available.
