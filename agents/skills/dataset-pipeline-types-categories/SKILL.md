---
name: dataset-pipeline-types-categories
description: Coerción de tipos y normalización de columnas categóricas en el pipeline de datos. Usar en etapas 6 y 7.
---

# Dataset pipeline — Types & categories

## Tipos (S6)

```python
import pandas as pd

def cast_columns(
    artifact_path: str,
    casts: dict[str, str],
    out_path: str,
    max_error_rate: float,
) -> dict:
    df = pd.read_parquet(artifact_path)
    n = len(df)
    coerced_nulls = 0
    for col, typ in casts.items():
        if col not in df.columns:
            continue
        if typ == "datetime64[ns]":
            df[col] = pd.to_datetime(df[col], errors="coerce")
        elif typ in ("int64", "float64"):
            df[col] = pd.to_numeric(df[col], errors="coerce")
        else:
            df[col] = df[col].astype(typ)
        coerced_nulls += int(df[col].isna().sum())
    rate = coerced_nulls / max(n, 1)
    if rate > max_error_rate:
        raise ValueError(f"coercion issues: rate {rate} > {max_error_rate}")
    df.to_parquet(out_path, index=False)
    return {"artifact_path": out_path, "error_rate_hint": rate}
```

Ajustar medición de `error_rate` por fila en producción.

## Categorías (S7)

```python
def normalize_categories(
    artifact_path: str,
    columns: list[str],
    case: str,
    out_path: str,
) -> dict:
    df = pd.read_parquet(artifact_path)
    for c in columns:
        if c not in df.columns:
            continue
        s = df[c].astype(str).str.strip()
        if case == "lower":
            s = s.str.lower()
        elif case == "upper":
            s = s.str.upper()
        df[c] = s
    df.to_parquet(out_path, index=False)
    return {"artifact_path": out_path}
```

## Sinónimos

Cargar `dict[str, str]` desde `manifest.category_maps_uri` y aplicar `series.replace(map)` antes de normalizar case si el negocio lo exige.
