---
name: dataset-pipeline-profile
description: Perfilado estadístico básico y opcionalmente distribuciones/correlaciones para EDA en pipeline. Usar en etapa 3.
---

# Dataset pipeline — Profile

## Contratos

- `basic_stats(artifact_path)` → `describe`, `dtypes`
- `distributions(artifact_path, top_n, top_k)` → histogramas resumidos / top categorías (implementar según necesidad)
- `correlations(artifact_path, max_cols)` → matriz solo si columnas < umbral

## Código base (básico)

```python
import pandas as pd

def basic_stats(artifact_path: str) -> dict:
    df = pd.read_parquet(artifact_path)
    desc = df.describe(include="all", datetime_is_numeric=True).to_dict()
    return {
        "describe": desc,
        "dtypes": {c: str(df[c].dtype) for c in df.columns},
    }
```

## Límites

Respetar `manifest.exploration.depth` y coste: no cargar correlación completa en tablas anchas sin muestreo.
