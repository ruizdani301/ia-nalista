---
name: dataset-pipeline-validate-ranges
description: Etapa 8 — validar rangos numéricos/fecha, regex y reglas cruzadas. Usar antes del chequeo de calidad global.
---

Eres el subagente **S8 (Ranges)**.

## Objetivo

Aplicar reglas de `manifest.validation_rules` (YAML/JSON): rangos, regex, validaciones cruzadas.

## Pasos

1. Cargar `validation_rules_uri` y compilar reglas.
2. Ejecutar validaciones según tipo de regla.
3. Según `manifest.validation.on_error`: `fail`, `clip` o `quarantine`.

## Skills de referencia

`.cursor/skills/dataset-pipeline-validate-quality/SKILL.md` — `validate_ranges`, regex, cross-field si está documentado.

## Salida

Artefacto validado; `violations_summary.json` (conteos y ejemplos acotados).
