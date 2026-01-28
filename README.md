# Funciones - Repositorio de Análisis Estadístico en R

Este repositorio contiene una colección de funciones estadísticas y análisis desarrollados en R, enfocados principalmente en modelos de supervivencia Cox, regresión lineal robusta (RLM), análisis de mínimos cuadrados parciales (PLSR), y cálculo de scores metabólicos.

## 📋 Tabla de Contenidos

- [Funciones Disponibles](#funciones-disponibles)
  - [1. run_cox_models](#1-run_cox_models)
  - [2. run_rlm](#2-run_rlm)
  - [3. score_metabolico](#3-score_metabolico)
  - [4. Código PLSR](#4-código-plsr)
  - [5. Bucle Mediación Cox](#5-bucle-mediación-cox)
- [Cómo Usar Este Repositorio](#cómo-usar-este-repositorio)
- [Contribuir](#contribuir)
- [Mantenimiento del README](#mantenimiento-del-readme)

---

## 🔧 Funciones Disponibles

### 1. run_cox_models

**Archivo:** `run_cox_models.Rmd`  
**Autor:** Jesús F García Gavilán  
**Fecha:** 2026-01-13

#### Descripción
Función para ejecutar múltiples modelos de regresión Cox de manera automatizada. Permite evaluar diferentes variables de exposición con múltiples ajustes, exportar resultados en formato Excel y calcular hazard ratios (HR) con intervalos de confianza.

#### Parámetros
- `data`: Base de datos que contiene `time_var`, `event_var` y covariables
- `approaches`: Vector de caracteres con nombres de variables de exposición (ej. "CQI_pre")
- `reales`: Vector de etiquetas para la tabla de salida (ej. "CQI Score"). Mismo orden que 'approaches'
- `models`: Lista de listas definiendo los ajustes. Estructura: `list("Modelo 1" = list(covariates = "+ edad..."))`
- `time_var`: Nombre de la columna de tiempo de seguimiento (default: "follcox_19")
- `event_var`: Nombre de la columna de evento (default: "death19")
- `digits`: Decimales para redondeo de HR e IC (default: 3)
- `export_path`: (Opcional) Ruta completa del archivo .xlsx a exportar
- `robust`: (Logical) Si es TRUE, calcula errores estándar robustos
- `weights`: (Opcional) Nombre de la columna de pesos o vector numérico

#### Características Especiales
- Detección automática de tipo de variable (continua vs categórica/ntile)
- Soporte para errores estándar robustos
- Soporte para ponderación de observaciones
- Exportación automática a Excel

#### Ejemplo de Uso
```r
# Definir modelos
models <- list(
  "Modelo 1" = list(covariates = "+ edad + sexo"),
  "Modelo 2" = list(covariates = "+ edad + sexo + imc")
)

# Ejecutar análisis
resultados <- run_cox_models(
  data = Scores_mort_bas,
  approaches = c("CQI_pre", "IG_pre"),
  reales = c("CQI Score", "IG Score"),
  models = models,
  export_path = "resultados_cox.xlsx"
)
```

---

### 2. run_rlm

**Archivo:** `run_rlm.Rmd`  
**Autor:** Jesús F García Gavilán  
**Fecha:** 2026-01-14

#### Descripción
Función para ejecutar múltiples regresiones lineales (LM) o robustas (RLM) de forma iterativa sobre una lista de metabolitos. Incluye corrección por múltiples comparaciones y exportación de resultados.

#### Parámetros
- `data`: Base de datos con outcome, covariables y metabolitos
- `metabolites`: Vector con los nombres de los metabolitos a iterar
- `outcomes`: Uno o varios outcomes (default: "cqi_est")
- `method`: "lm" para modelos lineales, "rlm" para modelos robustos (puede ser vector)
- `covariates`: Cadena con las covariables del modelo (incluye los "+" necesarios)
- `digits_coef`: Decimales para redondear Coef e IC (default: 2)
- `digits_p`: Decimales para redondear p-value (default: 3)
- `p_adjust_method`: Método de p.adjust (default: "BH" - Benjamini-Hochberg)
- `export_path`: (Opcional) Ruta completa del archivo .xlsx a exportar
- `tolower_names`: (Logical) Si TRUE, aplica tolower() a metabolites (default: TRUE)

#### Características Especiales
- Soporte para múltiples outcomes con diferentes métodos cada uno
- Corrección automática por múltiples comparaciones
- Clasificación de significancia estadística (*, **, ***)
- Manejo robusto de errores por metabolito

#### Ejemplo de Uso
```r
# Ejecutar RLM sobre múltiples metabolitos
resultados <- run_rlm(
  data = Met_CQI_rl2,
  metabolites = c("met1", "met2", "met3"),
  outcomes = c("cqi_est", "ig_est"),
  method = c("rlm", "lm"),
  covariates = "+ edad + sexo + imc",
  export_path = "resultados_rlm.xlsx"
)
```

---

### 3. score_metabolico

**Archivo:** `score_metabolico.Rmd`  
**Autor:** Jesús F García Gavilán  
**Fecha:** 2026-01-13

#### Descripción
Función para calcular un score metabólico a partir de coeficientes beta y valores de metabolitos. Opcionalmente, puede correlacionar el score calculado con una variable objetivo.

#### Parámetros
- `bbdd`: Base de datos de pacientes con valores de metabolitos
- `tabla_coefs`: Dataframe con nombres de variables y sus coeficientes
- `col_nombres`: Nombre de la columna en 'tabla_coefs' que tiene los nombres de los metabolitos (default: "names")
- `col_betas`: Nombre de la columna en 'tabla_coefs' con los valores numéricos (default: "beta")
- `target_col`: (Opcional) Nombre de la columna en 'bbdd' para correlacionar (ej. "CQI")

#### Características Especiales
- Normalización automática de nombres (tolower, trimws)
- Manejo automático del intercepto
- Cálculo matricial eficiente
- Correlación automática con variable objetivo (si se proporciona)
- Reporte detallado de metabolitos encontrados

#### Ejemplo de Uso
```r
# Calcular score y correlacionar con CQI
resultado <- score_metabolico(
  bbdd = BBDDmet_all,
  tabla_coefs = coeficientes_df,
  col_nombres = "metabolito",
  col_betas = "beta",
  target_col = "CQI"
)

# Extraer scores calculados
scores <- resultado$scores
```

---

### 4. Código PLSR

**Archivo:** `codigo_PLSR.rmd`

#### Descripción
Pseudocódigo completo para implementar análisis de Partial Least Squares Regression (PLSR) en R. Incluye validación cruzada, selección de componentes, evaluación y cálculo de VIP (Variable Importance in Projection).

#### Componentes Principales
1. **Setup y carga de datos**: Preparación del entorno y carga de datos
2. **Split train/test**: División de datos (80/20 recomendado)
3. **Ajuste PLSR con CV**: Validación cruzada interna para selección de componentes
4. **Selección de componentes**: Basada en RMSE mínimo
5. **Evaluación en test**: RMSE y R² en conjunto de prueba
6. **Interpretación**: Coeficientes, scores, loadings y VIP scores
7. **Exportación**: Guardado de resultados en CSV

#### Características Especiales
- Validación cruzada automática
- Función VIP personalizada incluida
- Selección automática de número óptimo de componentes
- Exportación estructurada de resultados

#### Variables Clave
- `max_comp`: Número máximo de componentes (min(20, ncol(X), nrow(train) - 2))
- `opt_comp`: Número óptimo de componentes seleccionado
- `vip_scores`: Importancia de cada variable predictora

---

### 5. Bucle Mediación Cox

**Archivo:** `Bucle mediación Cox.Rmd`  
**Fecha:** 2025-06-26

#### Descripción
Script para realizar análisis de mediación con modelos Cox utilizando bootstrap. Evalúa el efecto directo natural (NDE), efecto indirecto natural (NIE) y efecto total (TE) de metabolitos en la relación entre índices dietéticos y mortalidad.

#### Componentes Principales
1. **Selección de metabolitos**: Filtra metabolitos significativos (padj < 0.05)
2. **Preparación de datos**: Crea variables de supervivencia y cuartiles
3. **Bootstrap**: Loop con n_boot repeticiones (default: 1000)
4. **Análisis de mediación**: Usa `regmedint` con modelo survCox
5. **Extracción de resultados**: Calcula HR, ICs y proporción mediada

#### Parámetros del Análisis
- `yvar`: Variable de supervivencia ("y_Surv")
- `avar`: Variable explicativa ("cqi_est")
- `mvar`: Variable mediadora (cada metabolito)
- `mreg`: "linear" (modelo del mediador)
- `yreg`: "survCox" (modelo principal)
- `a0`, `a1`: Valores de referencia y activo de avar
- `m_cde`: Valor fijo del mediador (media)
- `n_boot`: Número de repeticiones bootstrap (1000)

#### Covariables Incluidas
edad_q, sexo, fr_smoke, ps1, ps2, grup_int, nodo, imc_q, energiat, alcoholg, escolar1, getota_1, hipercol0, hta0, tra_col0, trathta0

#### Outputs
- `logHR_NDE`: Log Hazard Ratio del efecto directo natural
- `logHR_NIE`: Log Hazard Ratio del efecto indirecto natural  
- `logHR_TE`: Log Hazard Ratio del efecto total
- `IC_NDE`, `IC_NIE`, `IC_TE`: Intervalos de confianza 95% percentiles
- `prop_mediada`: Proporción del efecto total mediada por el metabolito

---

## 📖 Cómo Usar Este Repositorio

### Requisitos Previos
```r
# Paquetes necesarios
install.packages(c("survival", "dplyr", "MASS", "pls", "rio", "regmedint", "Hmisc"))
```

### Cargar Funciones
Para usar cualquier función en tus análisis:

```r
# Cargar desde archivo RMarkdown
source("run_cox_models.Rmd")  # O usar knitr::purl() para extraer código R

# Alternativamente, copiar la función directamente a tu script
```

### Estructura de Archivos
```
funciones/
├── README.md                    # Este archivo
├── run_cox_models.Rmd          # Función para modelos Cox
├── run_rlm.Rmd                 # Función para regresión lineal/robusta
├── score_metabolico.Rmd        # Función para calcular scores
├── codigo_PLSR.rmd             # Pseudocódigo PLSR
└── Bucle mediación Cox.Rmd     # Script de análisis de mediación
```

---

## 🤝 Contribuir

Al agregar nuevas funciones al repositorio:

1. Usa formato R Markdown (.Rmd) consistente con los archivos existentes
2. Incluye documentación clara en comentarios:
   - Descripción de la función
   - Parámetros con tipos y defaults
   - Valores de retorno
   - Ejemplo de uso
3. Actualiza este README con la nueva función
4. Mantén el estilo de código consistente

---

## 🔄 Mantenimiento del README

**IMPORTANTE:** Este README debe actualizarse cada vez que se agregue una nueva función al repositorio.

### Checklist al Agregar Nueva Función:

- [ ] Crear el archivo .Rmd con la nueva función
- [ ] Documentar parámetros en comentarios dentro del código
- [ ] Agregar nueva sección en este README siguiendo el formato existente
- [ ] Incluir:
  - Nombre del archivo
  - Autor y fecha
  - Descripción clara
  - Tabla de parámetros
  - Características especiales
  - Ejemplo de uso
- [ ] Actualizar tabla de contenidos si es necesario
- [ ] Commit con mensaje descriptivo: "Add [nombre_funcion] documentation"

### Formato de Nueva Entrada:
```markdown
### N. nombre_funcion

**Archivo:** `nombre_archivo.Rmd`  
**Autor:** [Nombre]  
**Fecha:** [YYYY-MM-DD]

#### Descripción
[Descripción breve de qué hace la función]

#### Parámetros
- `param1`: Descripción del parámetro
- `param2`: Descripción del parámetro

#### Ejemplo de Uso
```r
# Código de ejemplo
resultado <- nombre_funcion(...)
```
```

---

## 📝 Notas

- Todas las funciones están optimizadas para análisis epidemiológicos y estudios de cohortes
- Se recomienda usar siempre `set.seed()` para reproducibilidad en análisis con componentes aleatorios
- Los archivos .Rmd pueden ejecutarse directamente en RStudio o convertirse a HTML para documentación

---

**Última actualización:** 2026-01-28