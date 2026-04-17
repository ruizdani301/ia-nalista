---
name: dataset-pipeline-io
description: Contratos y código base para I/O tabular del pipeline de datasets (resolver URI, leer, escribir, verificar). Usar al implementar etapas 1 y 10 o conectores file/s3/db.
---

# Dataset pipeline — I/O

## Variables de entorno

- Obligatorias según origen: `DATA_INPUT_URI`, `DATA_OUTPUT_URI`
- Opcional: `DATABASE_URL` (lectura/escritura SQL), `API_KEY` (API tabular), credenciales cloud / `STORAGE_CREDENTIALS`

## Contratos

| Función | Entrada | Salida |
|--------|---------|--------|
| `resolve_uri` | `uri`, `run_id` | `normalized_uri`, `scheme`, `estimated_bytes`, `source_hash_hint` |
| `read_tabular` | `uri`, `options` (incl. `out_path`) | `artifact_path`, `row_count` |
| `probe_size` | `artifact_path` | `byte_size` |
| `write_tabular` | `df_path`, `output_uri`, `fmt` | `written_uri` |
| `verify_write` | `output_uri`, `expected_rows` | `ok`, `row_count` |
| `publish_metadata` | `output_dir`, `files` | `metadata_index` |

## Código base (Python / pandas)

```python
from dataclasses import dataclass
from typing import Any
import os
import pandas as pd

@dataclass
class ResolveInput:
    uri: str
    run_id: str

@dataclass
class ResolveOutput:
    normalized_uri: str
    scheme: str
    estimated_bytes: int | None
    source_hash_hint: str | None

def resolve_uri(inp: ResolveInput) -> ResolveOutput:
    return ResolveOutput(
        normalized_uri=inp.uri.strip(),
        scheme=inp.uri.split(":", 1)[0],
        estimated_bytes=None,
        source_hash_hint=None,
    )

def read_tabular(uri: str, options: dict[str, Any]) -> dict[str, Any]:
    path = uri.replace("file://", "") if uri.startswith("file://") else uri
    if path.endswith(".csv"):
        kw = {k: v for k, v in options.items() if k in ("sep", "encoding", "dtype")}
        df = pd.read_csv(path, **kw)
    elif path.endswith(".parquet"):
        df = pd.read_parquet(path)
    else:
        raise ValueError("unsupported or remote URI — implement DB/API")
    out_path = options["out_path"]
    df.to_parquet(out_path, index=False)
    return {"artifact_path": out_path, "row_count": len(df)}

def probe_size(artifact_path: str) -> dict[str, Any]:
    return {"byte_size": os.path.getsize(artifact_path)}

def write_tabular(df_path: str, output_uri: str, fmt: str) -> dict[str, Any]:
    df = pd.read_parquet(df_path)
    if fmt == "parquet":
        df.to_parquet(output_uri, index=False)
    elif fmt == "csv":
        df.to_csv(output_uri, index=False)
    else:
        raise ValueError(fmt)
    return {"written_uri": output_uri}

def verify_write(output_uri: str, expected_rows: int) -> dict[str, Any]:
    df = (
        pd.read_parquet(output_uri)
        if output_uri.endswith(".parquet")
        else pd.read_csv(output_uri)
    )
    return {"ok": len(df) == expected_rows, "row_count": len(df)}

def publish_metadata(output_dir: str, files: list[str]) -> dict[str, Any]:
    return {"metadata_index": f"{output_dir}/index.json"}
```

## Modo chunked

Si `manifest.execution_profile == chunked`, particionar lectura/escritura por `chunk_key` y consolidar en la etapa de export según destino.
