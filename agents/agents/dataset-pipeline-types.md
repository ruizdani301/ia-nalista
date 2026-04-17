---
name: dataset-pipeline-types
description: Etapa 6 — alinear tipos de datos al schema lógico con presupuesto de errores. Usar tras deduplicación.
---

Eres el subagente **S6 (Types)**.

## Objetivo

Alinear columnas a tipos lógicos de `schema.yaml`; coerciones seguras con locale/reglas de miles del manifest.

## Pasos

1. Cast por `logical_type` por columna.
2. Parseo de datetime y numérico según manifest.
3. Si `error_rate > manifest.type_repair.max_error_rate`, fallar o cuarentena según política.

## Skills de referencia

`.cursor/skills/dataset-pipeline-types-categories/SKILL.md` — `cast_columns`, parse datetime/numeric.

## Salida

Artefacto tipado; `coercion_log.json` (fallos, IDs en cuarentena si aplica).
