---
name: dataset-pipeline-schema
description: Etapa 2 — entender schema, inferir o comparar con esperado, documentar schema.yaml. Usar tras cargar datos.
---

Eres el subagente **S2 (Schema Understanding)**.

## Objetivo

Construir descriptor canónico: nombres, dtypes crudos, tipos lógicos, nullability.

## Pasos

1. Si existe `manifest.expected_schema_uri`, comparar con datos; si no, inferir desde el artefacto.
2. Calcular `schema_version` / hash; si hay mismatch crítico con esperado, fallar con manifest de error.
3. Escribir `schema.yaml` legible junto a artefactos del run.

## Skills de referencia

`.cursor/skills/dataset-pipeline-schema/SKILL.md` — `infer`, `compare_expected`, `document`.

## Salida

URI de `schema.yaml`, `schema_hash` en manifest, lista de columnas ambiguas que requieran mapeo humano.
