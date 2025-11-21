# Pipeline de Procesamiento de Datos - Transporte Santiago

Análisis, limpieza, etiquetado y validación de reseñas del sistema de transporte público de Santiago.

---

## 📂 Dataset

**Entrada:** `resenas_transporte_santiago_1000.csv` (1,030 registros)  
**Salida:** `transporte_santiago_clean.csv` (1,002 registros procesados)

---

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

---

## 📁 Estructura del Proyecto

```
00_main.py                              # Pipeline principal
01_data_loader.py                       # Exploración inicial
02_data_cleaning.py                     # Limpieza de datos
03_data_imputation.py                   # Etiquetado de sentimiento
04_data_new_features.py                 # Validación cruzada
06_data_saving.py                       # Exportación
resenas_transporte_santiago_1000.csv    # Dataset original
requirements.txt                        # Dependencias
```

---

## 📊 Pipeline de Procesamiento

### 1. Exploración Inicial (`01_data_loader.py`)

Análisis del dataset original para identificar características y problemas.

**Análisis realizado:**
- **Dimensiones:** 1,030 registros × 12 columnas
- **Tipos de datos:** Identificación de variables numéricas y categóricas
- **Valores nulos:** 1,539 nulos detectados (15.4% del dataset)
  - `empresa_bus`: 624 nulos (60.58%)
  - `linea_metro`: 826 nulos (80.19%)
  - `rating`: 49 nulos (4.76%)
  - `barrio`: 20 nulos (1.94%)
  - `review_text`: 20 nulos (1.94%)
- **Duplicados:** 28 registros duplicados (2.72%)
- **Inconsistencias:** Detección de variaciones en categorías (mayúsculas/minúsculas)

**Estadísticas descriptivas:**
- Variables numéricas: media, mediana, desviación estándar, min/max
- Detección de outliers mediante método IQR (Rango Intercuartílico):
  - `duracion_viaje_min`: 8 outliers (0.8%)
  - `likes`: 7 outliers (0.7%)
  - `respuestas`: 17 outliers (1.7%)

---

### 2. Limpieza de Datos (`02_data_cleaning.py`)

Transformación del dataset para eliminar problemas y estandarizar valores.

**Transformaciones aplicadas:**

#### a) Eliminación de duplicados
- **Acción:** Eliminados 28 registros duplicados
- **Resultado:** 1,002 registros únicos (reducción del 2.72%)

#### b) Imputación de valores nulos
Estrategia por tipo de variable:

| Variable | Nulos | Estrategia | Justificación |
|----------|-------|------------|---------------|
| `rating` | 48 | Mediana (3.0) | Robusto ante outliers, valor central |
| `empresa_bus` | 607 | 'Desconocido' | Depende del medio de transporte |
| `linea_metro` | 803 | 'Desconocido' | Solo aplica para Metro |
| `barrio` | 20 | 'Desconocido' | Información incompleta |
| `review_text` | 20 | 'Desconocido' | Sin comentario del usuario |

**Total:** 1,539 nulos imputados → 100% completitud

#### c) Normalización de categorías
- **Acción:** Aplicar Title Case (primera letra mayúscula, resto minúsculas)
- **Objetivo:** Unificar variaciones (ej: metro → Metro, METRO → Metro)
- **Variables:** `medio_transporte`, `empresa_bus`, `barrio`

#### d) Conversión de tipos de datos
- **Fecha:** Conversión a `datetime64`
  - Rango: 2024-01-01 a 2025-07-04
- **Numéricos:** Conversión y validación de tipos correctos
  - `rating`: float64
  - `tiempo_espera_min`, `duracion_viaje_min`, `likes`, `respuestas`: int64

#### e) Métricas de calidad

| Métrica | Antes | Después | Mejora |
|---------|-------|---------|--------|
| Registros | 1,030 | 1,002 | -28 duplicados |
| Nulos | 1,539 (15.4%) | 0 (0%) | +15.4% completitud |
| Duplicados | 28 (2.72%) | 0 (0%) | 100% únicos |
| Calidad general | 84.6% | 100% | +15.4% |

---

### 3. Etiquetado de Sentimiento (`03_data_imputation.py`)

Creación de variable objetivo (`satisfaccion`) para clasificación de sentimientos.

**Criterios de etiquetado:**

```python
if rating >= 4:
    satisfaccion = 'Positivo'    # Ratings 4-5
elif rating == 3:
    satisfaccion = 'Neutro'      # Rating 3
else:
    satisfaccion = 'Negativo'    # Ratings 1-2
```

**Distribución de clases:**

| Clase | Cantidad | Porcentaje | Ratings |
|-------|----------|------------|---------|
| Positivo | 385 | 38.4% | 4-5 |
| Negativo | 352 | 35.1% | 1-2 |
| Neutro | 265 | 26.4% | 3 |

**Análisis de balance:**
- Ratio de desbalance: 1.45:1 (clase mayoritaria vs minoritaria)
- Interpretación: Dataset relativamente balanceado
- Clase mayoritaria: Positivo (385 registros)
- Clase minoritaria: Neutro (265 registros)
- Diferencia: 120 registros (11.9%)

**Mapeo detallado rating → satisfacción:**

| Rating | Registros | % | Satisfacción |
|--------|-----------|---|--------------|
| 5 | 158 | 15.8% | Positivo |
| 4 | 227 | 22.7% | Positivo |
| 3 | 265 | 26.4% | Neutro |
| 2 | 201 | 20.1% | Negativo |
| 1 | 151 | 15.1% | Negativo |

---

### 4. Validación Cruzada (`04_data_new_features.py`)

Evaluación del rendimiento mediante técnicas de validación estadística.

#### a) Features utilizadas

Variables predictoras:
- `tiempo_espera_min`: Tiempo de espera del transporte
- `duracion_viaje_min`: Duración del viaje
- `likes`: Número de likes de la reseña
- `respuestas`: Número de respuestas a la reseña

Variable objetivo:
- `satisfaccion`: Positivo, Neutro o Negativo

#### b) División Train/Test (80/20)

- **Train:** 801 registros (79.9%)
- **Test:** 201 registros (20.1%)
- **Estratificación:** Mantiene proporciones de clases en ambos conjuntos

Distribución estratificada por clase:

| Clase | Train | Test |
|-------|-------|------|
| Positivo | 38.5% | 38.3% |
| Negativo | 35.1% | 35.3% |
| Neutro | 26.5% | 26.4% |

#### c) K-Fold Cross Validation (k=5)

**¿Qué es K-Fold?**
- Técnica que divide el dataset de entrenamiento en 5 partes iguales (folds)
- En cada iteración: 4 folds para entrenamiento, 1 fold para validación
- Se realizan 5 iteraciones rotando el fold de validación
- Se promedian los resultados de las 5 iteraciones

**Ventajas de K-Fold:**
1. Uso eficiente de todos los datos (cada registro se valida una vez)
2. Estimación más robusta del rendimiento (reduce varianza)
3. Evita sesgo de una única división train/test
4. Detecta overfitting comparando train vs validación
5. Técnica estándar para datasets medianos (1000-10000 registros)

#### d) Modelo utilizado: Logistic Regression

Modelo baseline para clasificación multiclase:
- Algoritmo: Regresión Logística (sklearn)
- Iteraciones: 1000
- Random state: 42 (reproducibilidad)

#### e) Resultados

**K-Fold Cross Validation (5 folds):**

| Fold | Accuracy |
|------|----------|
| 1 | 50.31% |
| 2 | 65.62% |
| 3 | 63.12% |
| 4 | 60.00% |
| 5 | 64.38% |
| **Promedio** | **60.69% ± 5.51%** |

**Evaluación en conjunto de test:**
- **Accuracy:** 60.70%
- **F1-Score (macro):** 0.49
- **F1-Score (weighted):** 0.53

**Matriz de confusión:**

```
                 Predicho
           Neg   Neu   Pos
Real  Neg   58     4     9    (81.7% recall)
      Neu   20     1    32    (1.9% recall)
      Pos    9     5    63    (81.8% recall)
```

**Classification report:**

```
              precision  recall  f1-score  support
    Negativo       0.67    0.82      0.73       71
      Neutro       0.10    0.02      0.03       53
    Positivo       0.61    0.82      0.70       77

    accuracy                         0.61      201
   macro avg       0.46    0.55      0.49      201
weighted avg       0.49    0.61      0.53      201
```

**Observaciones:**
- Buena clasificación de sentimientos extremos (Positivo y Negativo: ~82% recall)
- Dificultad con clase Neutro (solo 1.9% recall)
- Causa: Features numéricas simples no capturan matices del sentimiento neutro
- Las variables tiempo de espera, duración y engagement (likes/respuestas) son insuficientes para discriminar opiniones neutras

---

### 5. Exportación (`06_data_saving.py`)

Guardado y validación del dataset procesado.

**Archivo generado:** `transporte_santiago_clean.csv`

**Validaciones aplicadas:**
- ✅ Sin valores nulos (100% completitud)
- ✅ Sin duplicados
- ✅ Categorías normalizadas (Title Case)
- ✅ Columna `satisfaccion` presente y correcta
- ✅ 3 clases únicas: Positivo, Neutro, Negativo
- ✅ Tipos de datos correctos (datetime, int, float)
- ✅ Encoding UTF-8
- ✅ 1,002 registros × 13 columnas

**Distribución final:**
- Positivo: 385 (38.4%)
- Negativo: 352 (35.1%)
- Neutro: 265 (26.4%)

---

## 📈 Resumen de Resultados

### Transformaciones Aplicadas

| Etapa | Input | Output | Cambio |
|-------|-------|--------|--------|
| Dataset original | 1,030 registros | - | - |
| Eliminación duplicados | 1,030 | 1,002 | -28 |
| Imputación nulos | 1,539 nulos | 0 nulos | -1,539 |
| Normalización | Inconsistencias | Estandarizado | - |
| Etiquetado | Sin 'satisfaccion' | Con 'satisfaccion' | +1 columna |
| **Dataset final** | **12 columnas** | **13 columnas** | **1,002 registros** |

### Calidad del Dataset Final

| Métrica | Valor |
|---------|-------|
| Registros | 1,002 |
| Columnas | 13 |
| Completitud | 100% (0 nulos) |
| Unicidad | 100% (0 duplicados) |
| Balance de clases | Ratio 1.45:1 (balanceado) |

### Rendimiento del Modelo

| Métrica | Valor |
|---------|-------|
| Accuracy (K-Fold CV) | 60.69% ± 5.51% |
| Accuracy (Test) | 60.70% |
| F1-Score (macro) | 0.49 |
| F1-Score (weighted) | 0.53 |
| Mejor fold | 65.62% (Fold 2) |
| Peor fold | 50.31% (Fold 1) |

---

## 🔧 Decisiones Técnicas

### Imputación de Nulos

**Rating (variable crítica):**
- Decisión: Imputar con mediana (3.0)
- Razón: Conserva 48 registros con información valiosa
- Impacto: Asigna etiqueta 'Neutro' a casos sin calificación
- Alternativa descartada: Eliminar registros (pérdida de 4.76% de datos)

**Variables categóricas:**
- Decisión: Imputar con 'Desconocido'
- Razón: Mantiene información del registro sin inventar datos
- Impacto: No introduce sesgo, permite identificar datos faltantes

### Normalización de Categorías

**Método:** Title Case
- Ejemplo: metro → Metro, METRO → Metro, Bus Interurbano → Bus Interurbano
- Ventaja: Mejor legibilidad que lowercase
- Objetivo: Unificar variaciones por capitalización

### Validación Cruzada

**K-Fold con k=5:**
- Balance entre costo computacional y estimación robusta
- Cada fold: ~160 registros de validación
- Varianza aceptable: ±5.51%

**Estratificación:**
- Mantiene proporción de clases en train/test
- Evita desbalance accidental en conjuntos

**Logistic Regression como baseline:**
- Modelo simple e interpretable
- Rápida ejecución
- Establece benchmark inicial

---

## 📦 Dependencias

```
pandas >= 2.0.0          # Manipulación de datos
numpy >= 1.24.0          # Operaciones numéricas
scikit-learn >= 1.3.0    # Validación cruzada, modelos ML
matplotlib >= 3.7.0      # Visualización
seaborn >= 0.12.0        # Gráficos estadísticos
```

Instalar con:
```bash
pip install -r requirements.txt
```

---

## 🎯 Limitaciones y Observaciones

### Limitaciones del Análisis

1. **Features limitadas:** Solo 4 variables numéricas simples (tiempo_espera, duracion_viaje, likes, respuestas)
2. **Texto no utilizado:** La columna `review_text` no se incorpora en el modelo actual
3. **Clase Neutro mal clasificada:** Solo 1.9% recall debido a features insuficientes
4. **Modelo baseline:** Logistic Regression no captura patrones complejos

### Observaciones Importantes

- Alta proporción de nulos en `empresa_bus` (60%) y `linea_metro` (80%) es natural: dependen del tipo de transporte
- Outliers detectados (duracion_viaje, likes, respuestas) se mantienen por ser casos válidos, no errores
- Dataset balanceado (ratio 1.45:1) facilita el entrenamiento sin técnicas especiales de balanceo
- Accuracy de ~60% es razonable para un modelo baseline con solo 4 features numéricas

---

## 📝 Archivos del Proyecto

**Scripts:**
- `00_main.py` - Pipeline principal que ejecuta el flujo completo
- `01_data_loader.py` - Exploración y análisis inicial
- `02_data_cleaning.py` - Limpieza y transformación
- `03_data_imputation.py` - Etiquetado de sentimientos
- `04_data_new_features.py` - Validación cruzada
- `06_data_saving.py` - Exportación y validación final

**Datos:**
- `resenas_transporte_santiago_1000.csv` - Dataset original (entrada)
- `transporte_santiago_clean.csv` - Dataset procesado (salida)

**Visualizaciones:**
- `confusion_matrix_logistic.png` - Matriz de confusión del modelo

**Configuración:**
- `requirements.txt` - Dependencias del proyecto
- `README.md` - Esta documentación