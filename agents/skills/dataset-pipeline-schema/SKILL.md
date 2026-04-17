---
name: dataset-pipeline-schema
description: Inferir schema, comparar con schema esperado y documentar schema.yaml para el pipeline de datos. Usar en etapa 2.
---

# Dataset pipeline — Schema

## Contratos

| Función | Rol |
|--------|-----|
| `infer(artifact_path)` | Columnas, dtype pandas, nullable |
| `compare_expected(artifact_path, expected_path)` | `match`, `missing_expected` |
| `document(schema, out_yaml)` | Escribe YAML canónico |

## Código base

```python
from typing import Any
import pandas as pd
import yaml

def infer(artifact_path: str) -> dict[str, Any]:
    df = pd.read_parquet(artifact_path)
    cols = []
    for c in df.columns:
        cols.append({
            "name": c,
            "dtype": str(df[c].dtype),
            "nullable": bool(df[c].isna().any()),
        })
    return {"columns": cols}

def compare_expected(artifact_path: str, expected_path: str) -> dict[str, Any]:
    actual = infer(artifact_path)
    with open(expected_path) as f:
        expected = yaml.safe_load(f)
    actual_names = {x["name"] for x in actual["columns"]}
    missing = [
        c["name"] for c in expected["columns"]
        if c["name"] not in actual_names
    ]
    return {"match": len(missing) == 0, "missing_expected": missing}

def document(schema: dict[str, Any], out_yaml: str) -> dict[str, Any]:
    with open(out_yaml, "w") as f:
        yaml.safe_dump(schema, f, sort_keys=False)
    return {"schema_uri": out_yaml}
```

## Tipos lógicos

Extiende el YAML con `logical_type` por columna (`date`, `currency`, `category`, etc.) para consumo en S6–S8.
