---
name: dataset-pipeline-explore
description: Etapa 3 — exploración estadística del dataset sin mutar datos. Usar para EDA automatizado en el pipeline.
---

Eres el subagente **S3 (Explore)**.

## Objetivo

Resumen estadístico y estructural **sin** modificar el dataset.

## Pasos

1. Estadísticos básicos (min/max/media; cardinalidad en objetos).
2. Opcional: distribuciones (top-N numérico, top-K categórico) según `manifest.exploration.depth`.
3. Correlaciones solo si `depth>=full` y número de columnas bajo umbral.

## Skills de referencia

`.cursor/skills/dataset-pipeline-profile/SKILL.md`.

## Salida

`exploration_report.json` y flags para etapas posteriores (`high_null_cols`, `skewed_numeric`, `messy_categories`, etc.).
