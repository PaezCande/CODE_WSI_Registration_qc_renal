# Alineacion de Tiles de Nefrologia 🔬
Evaluación comparativa de pipelines de registro de imágenes histológicas (GrandQC vs VALIS) mediante Correlación de Fase y NMI.

![Python](https://img.shields.io/badge/Python-3.12-blue?logo=python)
![OpenCV](https://img.shields.io/badge/OpenCV-Image_Processing-green?logo=opencv)
![Pandas](https://img.shields.io/badge/Pandas-Data_Analysis-red?logo=pandas)

Repositorio dedicado a la evaluación comparativa y cuantificación del error de registro entre dos pipelines de alineación de imágenes histológicas de tejido renal.

Este proyecto procesa lotes de *tiles* (recortes de imágenes de alta resolución) para determinar qué método logra una mejor convergencia y precisión submicrométrica al alinear cortes seriales con tinciones diferentes.

## Pipelines Evaluados

1. **Pipeline VALIS:** Separación por método de Otsu + alineación rígida/no rígida mediante librería VALIS.
2. **Pipeline GrandQC:** Extracción de características (SIFT/RANSAC) seguido de una micro-alineación fina no lineal (ECC + Demons).

## Metodología de Evaluación

Para garantizar que la evaluación sea independiente de las variaciones de color y contraste propias de las distintas tinciones, el sistema utiliza dos métricas matemáticamente independientes:

* **Métrica Principal - Correlación de Fase (Dominio Frecuencial):** 
  Calcula el desplazamiento residual en micrones (µm). Aplica ecualización adaptativa (CLAHE) y operadores Sobel para evaluar la coincidencia estructural basada exclusivamente en gradientes (bordes anatómicos).
* **Métrica de Validación - Información Mutua Normalizada (NMI):** 
  Métrica de la familia teórico-informacional que evalúa la entropía conjunta entre ambas imágenes. Valores más altos indican mayor coincidencia espacial de las estructuras biológicas.
* **Análisis Estadístico:** 
  Implementación del test no paramétrico de Mann-Whitney U para determinar la significancia estadística de las diferencias en el error residual entre ambos métodos.

## Estructura del Notebook

El archivo principal (`evaluacion_pipelines_alineacion.ipynb`) está modularizado en las siguientes etapas:

1. **Configuración del Entorno:** Validación de rutas locales y definición de la resolución espacial (`MPP_BASE`).
2. **Motor de Métricas:** Funciones core de procesamiento de imágenes con OpenCV.
3. **Procesamiento en Lote (Batch):** Evaluación masiva de pares de imágenes para cada método.
4. **Análisis Estadístico Descriptivo:** Cálculo de percentiles de error, boxplots e histogramas de densidad.
5. **Validación Visual:** Mapeo de coordenadas orgánicas $(x, y)$ del portaobjetos original para inspeccionar visualmente el acople de estructuras en paralelo.
6. **Concordancia de Métricas:** Análisis de correlación de rangos de Spearman ($\rho$) entre el error físico (µm) y el NMI.

## Requisitos e Instalación

Para ejecutar este entorno localmente, se requiere **Python 3** y las siguientes dependencias:

```bash
pip install opencv-python numpy pandas matplotlib scipy tqdm