# ia-nalista — Pipeline de Datos con Agentes IA

Pipeline de procesamiento de datos basado en agentes secuenciales. Transforma datos crudos en datasets limpios y validados mediante 10 etapas orquestadas por IA.

## Arquitectura

```
Load → Schema → Explore → Nulls → Dedupe → Types → Categories → Validate → Quality → Export
```

| Etapa | Agente | Responsabilidad |
|-------|--------|-----------------|
| 1 | `dataset-pipeline-load` | Resolver URI, cargar datos, estimar tamaño |
| 2 | `dataset-pipeline-schema` | Inferir tipos y estructura |
| 3 | `dataset-pipeline-explore` | Análisis exploratorio (perfilado) |
| 4 | `dataset-pipeline-nulls` | Auditoría e imputación de nulos |
| 5 | `dataset-pipeline-dedupe` | Deduplicación de registros |
| 6 | `dataset-pipeline-types` | Normalización de tipos |
| 7 | `dataset-pipeline-categories` | Codificación de categorías |
| 8 | `dataset-pipeline-validate-ranges` | Validación de rangos esperados |
| 9 | `dataset-pipeline-quality` | Score de calidad, fail-fast |
| 10 | `dataset-pipeline-export` | Escritura final (CSV o Delta Lake) |

## Componentes

```
agents/
├── rules/
│   └── dataset-pipeline-orchestrator.mdc  # Orquestador central
├── agents/                                 # Un agente por etapa
│   └── dataset-pipeline-<stage>.md
└── skills/                                # Contratos y código base
    ├── dataset-pipeline-io/
    ├── dataset-pipeline-clean/
    ├── dataset-pipeline-schema/
    ├── dataset-pipeline-profile/
    ├── dataset-pipeline-types-categories/
    └── dataset-pipeline-validate-quality/
```

## Stack Tecnológico

- **Agentes IA**: Prompts estructurados en Markdown (`.mdc`)
- **Procesamiento**: Python con pandas (datasets <1M filas) o PySpark (≥1M)
- **Formato interno**: Parquet (pipeline), CSV o Delta Lake (export)
- **Orquestación**: Agente orquestador delega cada etapa a sub-agentes

## Convenios Clave

- **Variables de entorno**: `DATA_INPUT_URI`, `DATA_OUTPUT_URI`; opcionales `DATABASE_URL`, `API_KEY`
- **Artefactos por run**: `run_id/`, `manifest.json` (URIs, schema_hash, políticas), `metrics.json`
- **Modo chunked**: `manifest.execution_profile=chunked` para datasets grandes
- **Fail-fast**: Se detiene en etapa 9 si no pasa validaciones de calidad (usuario puede forzar export degradado)

## Para Entrevistadores

### ¿Cómo funciona el pipeline?
El orquestador recibe una instrucción de procesamiento y delega secuencialmente cada etapa al sub-agente correspondiente. Cada sub-agente:
1. Lee el manifest actual y el artefacto de la etapa anterior
2. Ejecuta su transformación específica
3. Actualiza manifest + metrics
4. Pasa el relevo al siguiente agente

### ¿Qué hace cada skill?
Cada skill define **contratos** (interfaces de funciones) y **código base** (implementación de referencia) para un dominio específico. Los agentes consultan las skills como documentación, no como código a ejecutar.

### ¿Cómo se escala?
- El modo `chunked` particiona lectura/escritura para datasets que no caben en memoria
- PySpark distribuye procesamiento en clusters para >1M filas
- El manifest permite policy-driven processing (políticas de imputación, rangos esperados)

### ¿Cómo se valida calidad?
Etapa 9 genera un `quality_score` basado en nulidad, duplicados, consistencia de tipos y validación de rangos. Si falla, el pipeline se detiene (fail-fast) a menos que el usuario solicite continuar con datos degradados.

### ¿Dónde está la lógica de negocio?
Distribuida en los agentes y skills. Los agentes son "dumb coordinators" — la lógica real está en:
- `skills/dataset-pipeline-clean/` → nulidad y deduplicación
- `skills/dataset-pipeline-validate-quality/` → scoring y políticas

## Extender el Pipeline

Para agregar una nueva etapa:
1. Crear `agents/agents/dataset-pipeline-<nombre>.md` con nombre único
2. Definir skills en `agents/skills/` si hay lógica nueva
3. Agregar al orquestador en la secuencia

Para nuevos conectores (S3, DB, API):
```python
# Consultar dataset-pipeline-io/SKILL.md
# Implementar resolve_uri + read_tabular para el protocolo
```