# Modo: batch — Procesamiento Masivo de Ofertas

Dos modos de uso: **conductor interactivo** (una sesion de Codex con herramientas de navegador cuando existan) o **standalone** (script para URLs ya recolectadas).

## Arquitectura

```
Codex conductor interactivo
  │
  │  Navega portales o revisa resultados en tiempo real
  │  El usuario ve y controla la sesion
  │
  ├─ Oferta 1: lee JD del DOM o de la URL
  │    └─► codex exec worker → report .md + PDF + tracker-line
  │
  ├─ Oferta 2: siguiente oferta
  │    └─► codex exec worker → report .md + PDF + tracker-line
  │
  └─ Fin: merge tracker-additions → applications.md + resumen
```

Cada worker es un `codex exec` hijo con contexto limpio. El conductor solo orquesta.

## Archivos

```
batch/
  batch-input.tsv               # URLs (por conductor o manual)
  batch-state.tsv               # Progreso (auto-generado, gitignored)
  batch-runner.sh               # Script orquestador standalone
  batch-prompt.md               # Prompt template para workers
  logs/                         # Un log por oferta (gitignored)
  tracker-additions/            # Líneas de tracker (gitignored)
```

## Modo A: Conductor interactivo

1. Leer `batch/batch-state.tsv` para saber qué ya se procesó.
2. Navegar el portal en una sesion interactiva de Codex con navegador si el entorno lo permite.
3. Extraer URLs y guardarlas en `batch-input.tsv`.
4. Para cada URL pendiente:
   - leer JD del DOM o guardarlo en `/tmp/batch-jd-{id}.txt`
   - calcular el siguiente `REPORT_NUM`
   - delegar a `batch/batch-runner.sh` o lanzar un worker equivalente con `codex exec`
   - revisar `batch-state.tsv`, logs y tracker additions
5. Paginacion: siguiente pagina, repetir.
6. Fin: merge `tracker-additions/` a `applications.md` y resumir.

## Modo B: Script standalone

```bash
batch/batch-runner.sh [OPTIONS]
```

Opciones:
- `--dry-run` — lista pendientes sin ejecutar
- `--retry-failed` — solo reintenta fallidas
- `--start-from N` — empieza desde ID N
- `--parallel N` — N workers en paralelo
- `--max-retries N` — intentos por oferta (default: 2)

El runner usa `codex exec -C <repo> --full-auto --search -o <result-file> -`.

## Formato batch-state.tsv

```
id	url	status	started_at	completed_at	report_num	score	error	retries
1	https://...	completed	2026-...	2026-...	002	4.2	-	0
2	https://...	failed	2026-...	2026-...	-	-	Error msg	1
3	https://...	pending	-	-	-	-	-	0
```

## Resumabilidad

- Si muere → re-ejecutar → lee `batch-state.tsv` → skip completadas
- Lock file (`batch-runner.pid`) previene ejecución doble
- Cada worker es independiente: fallo en oferta #47 no afecta a las demás

## Workers (`codex exec`)

Cada worker recibe `batch-prompt.md` con placeholders resueltos. Es self-contained.

El worker produce:
1. Report `.md` en `reports/`
2. PDF en `output/`
3. Línea de tracker en `batch/tracker-additions/{id}.tsv`
4. JSON final en la ultima respuesta del agente, capturado con `codex exec -o`

## Gestión de errores

| Error | Recovery |
|-------|----------|
| URL inaccesible | Worker falla → runner marca `failed`, siguiente |
| JD detrás de login | Conductor intenta leer DOM. Si falla → `failed` |
| Portal cambia layout | El conductor adapta el flujo o cae al modo standalone |
| Worker crashea | Runner marca `failed`, siguiente. Retry con `--retry-failed` |
| Conductor muere | Re-ejecutar → lee state → skip completadas |
| PDF falla | Report .md se guarda. PDF queda pendiente |
