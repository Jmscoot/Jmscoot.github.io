---
layout: post
title: "TI C2000에서 float, double type에 대해"
date: 2026-10-07 00:55:00 +0900
categories: [임베디드, TI]
tags: [TI, double, float]
---


## Double/Float type<br>

#### 기본사양
| 항목 | float | double |
|---|---|---|
| IEEE 754 명칭 | Single precision (binary32) | Double precision (binary64) |
| 크기 | 32비트 (4바이트) | 64비트 (8바이트) |
| 비트 구성 (부호/지수/가수) | 1 / 8 / 23 | 1 / 11 / 52 |
| 지수 바이어스 | 127 | 1023 |
| 유효 정밀도 (hidden bit 포함) | 24비트 | 53비트 |
| 십진 유효숫자 | 약 7자리 | 약 15~16자리 |
| 최소 정규수 (`FLT_MIN` / `DBL_MIN`) | ≈ 1.18 × 10⁻³⁸ | ≈ 2.23 × 10⁻³⁰⁸ |
| 최대값 (`FLT_MAX` / `DBL_MAX`) | ≈ 3.40 × 10³⁸ | ≈ 1.80 × 10³⁰⁸ |
| 머신 엡실론 (`FLT_EPSILON` / `DBL_EPSILON`) | ≈ 1.19 × 10⁻⁷ | ≈ 2.22 × 10⁻¹⁶ |
