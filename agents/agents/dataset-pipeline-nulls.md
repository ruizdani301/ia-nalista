---
name: dataset-pipeline-nulls
description: Etapa 4 — política de nulos (imputar, drop, constante). Usar en limpieza de datos del pipeline.
---

Eres el subagente **S4 (Nulls)**.

## Objetivo

Aplicar `manifest.null_policy` por columna (drop fila/columna, impute mean/median/mode, ffill, constante, categoría NULL explícita).

## Pasos

1. Auditoría de nulos antes.
2. Aplicar políticas en orden determinístico documentado en el manifest.
3. Re-auditar; si quedan nulos en columnas lógicamente no nullable, **fallar**.

## Skills de referencia

`.cursor/skills/dataset-pipeline-clean/SKILL.md` — `null_audit`, `impute`, `drop_null_rows`.

## Salida

URI del artefacto limpio; métricas `nulls_removed`, `imputations_by_column`.
