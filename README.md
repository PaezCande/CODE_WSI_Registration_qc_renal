# Comparación de pipelines de registro de WSI y selección automática de tiles confiables en tejido renal (Masson)

Proyecto **COSIECI933 — "Sistemas inteligentes de asistencia al diagnóstico médico"** (UTN).

Este repositorio compara tres pipelines de registro de imágenes histológicas -**Sift+Demons** (desarrollo propio), **DeeperHistReg** y **VALIS**— sobre Whole Slide Images (WSI) de tejido renal teñido con tricrómico de Masson, con el objetivo de establecer un criterio automático (sin intervención de un experto) para seleccionar el subconjunto de tiles alineados de forma confiable. Ese subconjunto se utilizará para construir el conjunto de entrenamiento del modelo de predicción de fibrosis renal de la siguiente etapa del proyecto.

## Contenido del repositorio

```
.
├── pipelines/
│   ├── PipelinesSift_Demons.ipynb      # Pipeline propio: SIFT + FLANN + RANSAC + ECC + Demons
│   ├── PipelineDeeperhistreg.ipynb     # Pipeline con DeeperHistReg (default_nonrigid_high_resolution)
│   └── PipelineValis.ipynb             # Pipeline con VALIS (DISK + LightGlue + SimpleElastix)
├── metricas/
│   └── ComparacionMetricas.ipynb         # Cálculo de las métricas de calidad sin referencia por tile
├── resultados/
│   └── metricas_por_tile.csv           # Output consolidado de las tres corridas (una fila por tile)
└── README.md
```

## Pipelines evaluados

Los tres pipelines comparten las mismas etapas de separación de fragmentos, detección de tejido (GrandQC) y extracción/filtrado de tiles (grilla de 512 px, stride 256 px, umbral de tejido ≥40 %, umbral de informatividad ≥15,0 varianza del Laplaciano), y difieren únicamente en el motor de registro:

| Notebook | Motor de registro | Resumen |
|---|---|---|
| `PipelinesSiftDemons.ipynb` | SIFT + FLANN + RANSAC (afín global) + ECC + Demons (SimpleITK) | Desarrollo propio. Refinamiento no rígido aplicado por tile, sobre un canvas ampliado con contexto (768×768 px, padding de 128 px) para evitar artefactos de borde. |
| `PipelineDeeperhistreg.ipynb` | [DeeperHistReg](https://arxiv.org/abs/2404.14434) — `default_nonrigid_high_resolution` | Registro rígido con descriptores profundos + optimización de instancia no rígida, resuelto una única vez sobre la imagen completa. |
| `PipelineValis.ipynb` | [VALIS](https://doi.org/10.1038/s41467-023-40218-9) — DISK + LightGlue + SimpleElastix | Detector de features profundo (DISK) con matcher LightGlue y refinamiento rígido adicional (MicroRigidRegistrar), seguido de registro no rígido con SimpleElastix. |

## Métricas de calidad (sin referencia)

Ante la ausencia de anotaciones manuales, la calidad de cada tile registrado se evalúa con un conjunto de métricas sin referencia, calculadas en `metricas/metricas_por_tile.ipynb`:

- **PCC-TRE** — desplazamiento residual (px) por correlación cruzada de fase.
- **NCC** — correlación cruzada normalizada.
- **SSIM enmascarado** — similitud estructural restringida a la región de tejido.
- **Mask IoU (Otsu)** — solapamiento geométrico de máscaras de tejido, invariante a la tinción.
- **Folding ratio (%)** — proporción de píxeles con determinante Jacobiano ≤ 0 (proxy vía flujo óptico Farneback), como control de plausibilidad física de la deformación no rígida.

Los umbrales de aceptación (Mask IoU ≥ 0,64; Folding ratio < 1,5 %) siguen el criterio de [URQA (2026)](https://arxiv.org/abs/2602.04046).

## Resultado principal

Bajo el criterio combinado de retención (Mask IoU ≥ 0,64 **y** Folding ratio < 1,5 %), VALIS retiene la mayor proporción de tiles confiables:

| Pipeline | n total | n retenido | % retenido |
|---|---|---|---|
| Sift+Demons | 1.678 | 192 | 11,4 % |
| DeeperHistReg | 2.868 | 201 | 7,0 % |
| VALIS | 1.820 | 451 | **24,8 %** |

Ver el paper para el detalle completo de resultados, discusión y limitaciones (la ventaja de VALIS no es uniforme entre las dos muestras evaluadas).

## Requisitos

```
opencv-python
numpy
pandas
tqdm
scikit-image
scikit-learn
SimpleITK
pyvips
tiatoolbox
deeperhistreg
valis-wsi
```

## Disponibilidad de datos

El notebook de Kaggle con los resultados está disponible en: https://www.kaggle.com/code/tobiasdelgado/comparaci-n-de-pipelines?scriptVersionId=349911939

Las imágenes originales (WSI de tejido renal, provistas por el Centro de Microscopía Electrónica de la UNC) **no se incluyen en este repositorio** por tratarse de datos sensibles de pacientes sujetos a acuerdos de confidencialidad con la institución proveedora. Solo se publican el código y las métricas agregadas por tile (`metricas_por_tile.csv`), sin rutas ni identificadores que permitan reconstruir la procedencia de las imágenes.


## Equipo

Proyecto "Sistemas inteligentes de asistencia al diagnóstico médico" (COSIECI933), Universidad Tecnológica Nacional — Facultad Regional Córdoba.