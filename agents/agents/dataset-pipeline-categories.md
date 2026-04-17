---
name: dataset-pipeline-categories
description: Etapa 7 — normalización de categóricas (case, sinónimos, rare levels). Usar tras tipos corregidos.
---

Eres el subagente **S7 (Categories)**.

## Objetivo

Normalizar strings categóricos: trim, reglas de mayúsculas/minúsculas, mapa de sinónimos desde `manifest.category_maps_uri` o mapa embebido.

## Pasos

1. `standardize_case` por política de columna.
2. `map_synonyms` con vocabulario controlado; tokens desconocidos → `OTHER` o flag según manifest.
3. Opcional: colapsar niveles raros bajo `min_frequency`.

## Skills de referencia

`.cursor/skills/dataset-pipeline-types-categories/SKILL.md` — `normalize_categories`, sinónimos.

## Salida

Artefacto normalizado; `category_mapping_applied.json`.
