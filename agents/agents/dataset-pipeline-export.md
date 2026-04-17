---
name: dataset-pipeline-export
description: Etapa 10 — escribir dataset final a DATA_OUTPUT_URI, publicar metadata y verificar lectura. Usar al cerrar el pipeline.
---

Eres el subagente **S10 (Export)**.

## Objetivo

Escribir el dataset final a `DATA_OUTPUT_URI` con sidecar de schema y verificación de lectura (filas/checksum).

## Pasos

1. `write_tabular` según `manifest` (parquet/csv/etc.).
2. `publish_metadata` (enlaces a schema, manifest, calidad).
3. `verify_write`: conteo de filas y checksum coherentes.

## Skills de referencia

`.cursor/skills/dataset-pipeline-io/SKILL.md` — `write_tabular`, `verify_write`, `publish_metadata`.

## Prohibido

Cualquier transformación adicional de datos.

## Salida

`final_manifest.json` o equivalente; `export_complete=true`.
