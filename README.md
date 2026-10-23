# Análisis descriptivo del precio de viviendas – King County (`reg_cat_house_sales`)

Proyecto Integrador III – Analítica de Datos · IU Digital · Grupo 1 Canvas

Proyecto de analítica descriptiva que obtiene el dataset de ventas de viviendas del Condado de King (Seattle, EE. UU.) desde Hugging Face, lo transforma con pandas y genera automáticamente un resumen en `.txt` mediante GitHub Actions.

## Integrantes

- Jonathan Alvarez Bustamante
- Yusep Alexander Ruiz Agudelo
- Yonier Alexis Quiceno Rodriguez

## Propósito y necesidad

**Escenario (hipotético):** una firma de consultoría y avalúos inmobiliarios quiere tener criterios de referencia basados en datos para tasar viviendas, en lugar de depender solo de la experiencia de cada perito.

**Necesidad:** conocer cómo varía el precio de venta de las viviendas según su tamaño, su calificación de construcción, su ubicación frente al agua y su antigüedad o remodelación.

**Pregunta de análisis:**

> ¿Cómo varía el precio de venta de las viviendas vendidas en el Condado de King entre mayo de 2014 y mayo de 2015 según su tamaño (área habitable, área del lote, habitaciones y baños), su calificación de construcción, su ubicación frente al agua y su antigüedad o remodelación?

El análisis es **descriptivo**: caracteriza y compara grupos de viviendas. No predice precios ni establece relaciones de causa y efecto.

## Dataset

| Campo | Valor |
|---|---|
| Repositorio en Hugging Face | [`inria-soda/tabular-benchmark`](https://huggingface.co/datasets/inria-soda/tabular-benchmark) |
| Configuración | `reg_cat_house_sales` (archivo `reg_cat/house_sales.csv`) |
| Split | `train` (único disponible) |
| Revisión consultada | commit `8d0ff91` (4 de septiembre de 2023) |
| Visor | <https://huggingface.co/datasets/inria-soda/tabular-benchmark/viewer/reg_cat_house_sales> |
| Tamaño | 21.613 filas × 18 columnas (≈ 2 MB) |
| Muestreo | No: se usa el dataset completo |
| Fuente original | *House Sales in King County, USA* (Kaggle, harlfoxem), curado en OpenML (ID 44066) por Grinsztajn et al. (2022) |
| Licencia | La ficha de Hugging Face no la indica; la fuente original en Kaggle y OpenML es **CC0 (dominio público)** |
| Datos personales | No contiene (no hay nombres, direcciones ni identificadores de propietarios) |

**Cada fila representa una vivienda vendida.** Notas importantes sobre las columnas:

- `price` está en **escala logarítmica**: `price = ln(precio_USD + 1)`. Para obtener dólares se usa `np.expm1(price)`.
- `date_year` está codificado: `0` = 2014 y `1` = 2015.
- `waterfront`: `1` = frente al agua, `0` = no.
- `yr_renovated = 0` significa que la vivienda nunca fue remodelada.
- Las áreas están en **pies cuadrados** (1 m² ≈ 10,764 sqft).

## Estructura del repositorio

```
.
├── .github/workflows/ejecutar.yml   # GitHub Actions: ejecuta el notebook y guarda resultados
├── src/
│   ├── clases/
│   │   ├── __init__.py
│   │   ├── hf_loader.py             # HFDatasetLoader: descarga desde Hugging Face → DataFrame
│   │   └── reportes.py              # Reportes: resumen del DataFrame → .txt
│   └── sincronicas/
│       ├── transformaciones.ipynb   # Notebook: carga, revisión, transformaciones y reporte
│       ├── datos/                   # CSV descargado (lo genera el notebook)
│       └── reportes/                # Resumen .txt (lo genera el notebook)
├── requirements.txt
└── README.md
```

> **Para la EA2:** el documento Word de la etapa 1 (`.docx`) se adjunta en Canvas. Si el docente lo pide dentro del repositorio, se sugiere guardarlo en `AE2/documento/`.

## Cómo ejecutarlo

### Opción 1 – Automática (GitHub Actions)

Cada `push` a `main` ejecuta el workflow `.github/workflows/ejecutar.yml`, que:

1. instala Python 3.12 y las dependencias,
2. ejecuta `src/sincronicas/transformaciones.ipynb`,
3. guarda el CSV en `datos/` y el resumen en `reportes/`,
4. hace un commit automático con el usuario `github-actions[bot]`.

### Opción 2 – Local

```bash
git clone <URL-del-repositorio>
cd analisis_reg_cat_house_sales
python -m venv .venv
source .venv/bin/activate          # Windows (PowerShell): .venv\Scripts\Activate.ps1
pip install -r requirements.txt
pip install pyarrow openpyxl       # opcional: exportar a parquet o xlsx
cd src/sincronicas
jupyter notebook transformaciones.ipynb
```

Ejecuta las celdas en orden. Los resultados quedan en:

- `src/sincronicas/datos/reg_cat_house_sales.csv`
- `src/sincronicas/reportes/reg_cat_house_sales.txt`

### Token de Hugging Face (opcional)

El dataset es público y no requiere token. Para evitar el aviso de solicitudes sin autenticación:

- **Local:** `hf auth login`
- **GitHub Actions:** crea el secreto `HF_TOKEN` en *Settings → Secrets and variables → Actions* y agrégalo al paso que ejecuta el notebook:

  ```yaml
  env:
    HF_TOKEN: ${{ secrets.HF_TOKEN }}
  ```

**Nunca escribas el token en el código.**

## Transformaciones aplicadas en el notebook

| # | Transformación | Resultado |
|---|---|---|
| 1 | Eliminación de duplicados | 0 duplicados (21.613 filas) |
| 2 | Filtro de registros inconsistentes (0 habitaciones, menos de 1 baño, más de 15 habitaciones) | 21.521 filas (se descartan 92) |
| 3 | Renombrado de columnas al español | 18 columnas en español |
| 4 | Decodificación (`Vista al agua` → Sí/No; `Año de venta` → 2014/2015) y cálculo de `Precio (USD)` | 19 columnas |

## Dependencias

Ver `requirements.txt`: `numpy`, `pandas`, `matplotlib`, `notebook` y `datasets` (además de `pyarrow` y `openpyxl`, que se instalan en el workflow).

## Referencias

- Grinsztajn, L., Oyallon, E. y Varoquaux, G. (2022). Why do tree-based models still outperform deep learning on typical tabular data? *NeurIPS 2022 Datasets and Benchmarks Track*.
- harlfoxem. (2016). *House Sales in King County, USA* [Conjunto de datos]. Kaggle. <https://www.kaggle.com/datasets/harlfoxem/housesalesprediction>
- INRIA-SODA. (2023). *tabular-benchmark* [Conjunto de datos]. Hugging Face. <https://huggingface.co/datasets/inria-soda/tabular-benchmark>
