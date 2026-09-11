# transporte-santiago-data-pipeline

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![pandas](https://img.shields.io/badge/pandas-2.0+-yellow?logo=pandas)
![scikit-learn](https://img.shields.io/badge/scikit--learn-1.3+-orange?logo=scikit-learn)
![License: MIT](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

> ⚠️ **Disclaimer educativo**: Este dataset fue provisto con fines académicos.
> Los datos son anonimizados y se usan solo para demostración de técnicas de ML/NLP.
> No representan datos de producción ni deben usarse comercialmente.

---

Análisis, limpieza, etiquetado y validación de reseñas del sistema de transporte público de Santiago.

## 📂 Dataset

**Entrada:** `resenas_transporte_santiago_1000.csv^` (1,030 registros) 
**Salida:** `transporte_santiago_clean.csv` (1,002 registros procesados)

## 🚀 Ejecución

```bash
# Instalar dependencias
pip install -r requirements.txt

# Ejecutar pipeline completo
python 00_main.py
```

**Archivos generados:**
- `transporte_santiago_clean.csv` - Dataset limpio
- `confusion_matrix_logistic.png` - Matriz de confusión

## 📁 Estructura del Proyecto

```
00_main.py                              # Pipeline principal
01_data_loader.py                       # Exploración inicial
02_data_cleaning.py                        # Limpieza de datos
03_data_imputation.py                       # Etiquetado de sentimento
04_data_new_features.py                      # Validación cruzada
05_data_saving.py                        # Exportación
resenas_transporte_santiago_1000.csv     # Dataset original
requirements.txt                          # Dependencias
```

Ver código completo en: https://github.com/TomasFuentealba/transporte-santiago-data-pipeline
