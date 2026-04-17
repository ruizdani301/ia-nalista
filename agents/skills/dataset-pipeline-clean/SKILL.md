---
name: dataset-pipeline-clean
description: Auditoría de nulos, imputación, drop y deduplicación para el pipeline de datos. Usar en etapas 4 y 5.
---

# Dataset pipeline — Clean

## Nulos

```python
from typing import Any
import pandas as pd

def null_audit(artifact_path: str) -> dict[str, int]:
    df = pd.read_parquet(artifact_path)
    return {c: int(df[c].isna().sum()) for c in df.columns}

def impute(artifact_path: str, policy: dict[str, str], out_path: str) -> dict[str, Any]:
    df = pd.read_parquet(artifact_path)
    for col, method in policy.items():
        if col not in df.columns:
            continue
        if method == "mean":
            df[col] = df[col].fillna(df[col].mean())
        elif method == "median":
            df[col] = df[col].fillna(df[col].median())
        elif method == "mode":
            m = df[col].mode(dropna=True)
            df[col] = df[col].fillna(m.iloc[0] if len(m) else None)
        elif method == "drop_col":
            df = df.drop(columns=[col])
    df.to_parquet(out_path, index=False)
    return {"artifact_path": out_path}
```

## Dedupe

```python
def dedupe(artifact_path: str, subset: list[str] | None, out_path: str) -> dict[str, Any]:
    df = pd.read_parquet(artifact_path)
    before = len(df)
    df = df.drop_duplicates(subset=subset)
    df.to_parquet(out_path, index=False)
    return {"rows_dropped": before - len(df), "artifact_path": out_path}
```
