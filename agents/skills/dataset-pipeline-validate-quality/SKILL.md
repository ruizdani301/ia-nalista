---
name: dataset-pipeline-validate-quality
description: Validación por rangos/regex y score de calidad compuesto para el pipeline de datos. Usar en etapas 8 y 9.
---

# Dataset pipeline — Validate & quality

## Rangos (S8)

```python
import pandas as pd

def validate_ranges(
    artifact_path: str,
    rules: list[dict],
    out_path: str,
    on_error: str,
) -> dict:
    df = pd.read_parquet(artifact_path)
    violations = 0
    for r in rules:
        col = r["column"]
        lo, hi = r.get("min"), r.get("max")
        if col not in df.columns:
            continue
        mask = pd.Series(True, index=df.index)
        if lo is not None:
            mask &= df[col] >= lo
        if hi is not None:
            mask &= df[col] <= hi
        violations += int((~mask).sum())
        if on_error == "clip" and lo is not None and hi is not None:
            df[col] = df[col].clip(lower=lo, upper=hi)
    if on_error == "fail" and violations:
        raise ValueError(f"validation violations: {violations}")
    df.to_parquet(out_path, index=False)
    return {"violations": violations, "artifact_path": out_path}
```

Regex y reglas cruzadas: mismo patrón (máscara booleana + conteo de violaciones).

## Calidad compuesta (S9)

```python
def score_composite(metrics: dict[str, float], weights: dict[str, float]) -> dict:
    wsum = sum(weights.values()) or 1e-9
    score = sum(metrics.get(k, 0) * weights.get(k, 0) for k in set(metrics) | set(weights)) / wsum
    return {"score": score, "pass": score >= 0.85, "breakdown": metrics}
```

Define `metrics` (completitud, unicidad en clave, etc.) desde agregados del manifest y reportes previos.

## Drift vs baseline

Comparar distribuciones de columnas clave contra artefacto `baseline_uri` (KS, proporciones categóricas, o reglas de negocio); umbral en manifest.
