# Modelado de Etapas del Sueño con Cadenas Ocultas de Markov (HMM) y Semi-Markovianas (HSMM)

[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)
[![Status](https://img.shields.io/badge/status-completado%20y%20verificado-brightgreen.svg)]()
[![Formato](https://img.shields.io/badge/formato-IEEE%20Transactions-red.svg)]()
[![Licencia](https://img.shields.io/badge/licencia-Educativa%20%2F%20MIT-lightgrey.svg)]()

Repositorio oficial del proyecto de investigación y simulación computacional sobre el **modelado estocástico de la arquitectura del sueño humano**. Se implementa y compara un **Modelo Oculto de Markov estándar (HMM Base)**, una variante de **Sueño Fragmentado**, y un **Modelo Oculto Semi-Markoviano (HSMM)** con duraciones explícitas de permanencia, validado mediante inferencia estadística de **Monte Carlo ($M = 100$ réplicas)** y decodificación global de **Viterbi en espacio logarítmico**.

El artículo académico consolidado en formato IEEE se encuentra disponible en PDF:
👉 **[materiales/articulo_final.pdf](materiales/articulo_final.pdf)**

---

## Tabla de Contenidos
1. [Descripción del Proyecto](#1-descripción-del-proyecto)
2. [Objetivos](#2-objetivos)
3. [Modelo Matemático](#3-modelo-matemático)
4. [Estados y Observaciones](#4-estados-y-observaciones)
5. [Escenarios de Simulación](#5-escenarios-de-simulación)
6. [Estructura del Repositorio](#6-estructura-del-repositorio)
7. [Instalación y Requisitos](#7-instalación-y-requisitos)
8. [Guía de Ejecución](#8-guía-de-ejecución)
9. [Parámetros y Semillas](#9-parámetros-y-semillas)
10. [Resultados y Figuras](#10-resultados-y-figuras)
11. [Limitaciones y Consideraciones Éticas](#11-limitaciones-y-consideraciones-éticas)
12. [Documentación y Enlaces](#12-documentación-y-enlaces)

---

## 1. Descripción del Proyecto

El análisis objetivo de la arquitectura del sueño mediante polisomnografía (PSG) requiere clasificar épocas sucesivas de 30 segundos en fases discretas de vigilia y sueño. Debido a que los estados neurofisiológicos cerebrales subyacentes son latentes y solo se manifiestan a través de señales biomédicas electrofisiológicas (EEG, EOG, EMG), los **Modelos Ocultos de Markov (HMM)** representan un formalismo probabilístico ideal para su estudio.

Este proyecto implementa la metodología didáctica **USA $\to$ MODIFICA $\to$ CREA**:
* **USA:** Simulación y decodificación del modelo HMM clásico con parámetros biofísicos base.
* **MODIFICA:** Inducción experimental de hiperactivación/fragmentación del sueño mediante ajuste estocástico de la matriz de transición.
* **CREA:** Desarrollo de una arquitectura **Semi-Markoviana (HSMM)** que resuelve la limitación intrínseca de la distribución geométrica de permanencia temporal del HMM estándar, incorporando ventanas mínimas y distribuciones explícitas de duración por etapa.
* **VALIDA:** Batería inferencial de Monte Carlo ($M = 100$ semillas independientes) con intervalos de confianza del 95% ($t$-Student).

---

## 2. Objetivos

### 2.1 Objetivo General
Modelar, simular e inferir las fases de la arquitectura del sueño a través de modelos markovianos y semi-markovianos discretos, cuantificando la exactitud de decodificación de Viterbi, la alteración por fragmentación y el beneficio del control explícito de la duración de permanencia.

### 2.2 Preguntas de Investigación
* **P1:** ¿Con qué exactitud puede el algoritmo de Viterbi reconstruir la trayectoria latente del sueño a partir de observaciones polisomnográficas ruidosas o ambiguas?
* **P2:** ¿De qué manera un incremento en la probabilidad de transición a vigilia afecta la eficiencia de sueño (SE), los despertares y la fragmentación estructural?
* **P3:** ¿Cómo supera un modelo Semi-Markoviano (HSMM) la patología de permanencia geométrica irremediable del HMM clásico, produciendo hipnogramas con estabilidad y ciclos fisiológicamente más plausibles?

---

## 3. Modelo Matemático

Cada modelo se formaliza como una tupla estocástica $\lambda = (S, V, A, B, \pi)$:

* **Distribución Inicial ($\pi$):**
  $$\pi = \begin{bmatrix} 1.0 & 0.0 & 0.0 & 0.0 \end{bmatrix}$$
  El individuo inicia el registro en vigilia ($q_1 = \text{Despierto}$).

* **Matriz de Transición ($A \in \mathbb{R}^{4 \times 4}$):**
  $$A_{ij} = P(q_{t+1} = s_j \mid q_t = s_i), \quad \sum_{j=1}^4 A_{ij} = 1.0$$

* **Matriz de Emisión ($B \in \mathbb{R}^{4 \times 5}$):**
  $$B_{jk} = P(o_t = v_k \mid q_t = s_j), \quad \sum_{k=1}^5 B_{jk} = 1.0$$

* **Distribución Geométrica de Permanencia (HMM Clásico):**
  La permanencia continua en un estado $s_i$ sigue analíticamente una distribución geométrica discreta:
  $$P(D_i = d) = (A_{ii})^{d-1}(1 - A_{ii}), \quad \mathbb{E}[D_i] = \frac{1}{1 - A_{ii}} \text{ épocas}$$
  Esto impone la propiedad de "falta de memoria" (*memoryless*), donde la probabilidad máxima de abandonar un estado ocurre siempre en la primera época ($d = 1$), en contradicción con la fisiología del sueño humano.

* **Extensión Semi-Markoviana (HSMM):**
  Desacopla la transición de la permanencia mediante una matriz embebida sin autorrecurrencia ($A_{ii}^{\text{emb}} = 0$) y una distribución de duración explícita:
  $$d \sim \mathcal{U}(d_{\min, i}, d_{\max, i}), \quad \mathbb{E}[D_i] = \frac{d_{\min, i} + d_{\max, i}}{2}$$

* **Decodificación Log-Viterbi:**
  Para evitar subdesbordamiento aritmético (*underflow*) en $T = 480$ épocas:
  $$\delta_t(j) = \max_{1 \le i \le 4} \left[ \delta_{t-1}(i) + \ln A_{ij} \right] + \ln B_j(o_t)$$

Para detalles matemáticos completos, deducciones analíticas y pseudocódigo de Viterbi segmentario, consulte **[docs/modelo_matematico.md](docs/modelo_matematico.md)**.

---

## 4. Estados y Observaciones

### Estados Ocultos ($S$)
| Índice | Estado Latente | Denominación Clínica | Descripción Neurofisiológica |
| :---: | :--- | :--- | :--- |
| **0** | `Despierto` | Vigilia (Wake) | Individuo en vigilia con actividad electroencefalográfica rápida. |
| **1** | `Sueño Ligero` | N1 / N2 (Light Sleep) | Sueño superficial, ralentización de frecuencias cerebrales. |
| **2** | `Sueño Profundo` | N3 / SWS (Slow Wave) | Sueño sincronizado de ondas lentas delta de alto voltaje. |
| **3** | `REM` | REM (Paradoxical) | Sueño desincronizado con atonía muscular y movimientos oculares rápidos. |

### Observaciones Discretas ($V$)
Combinaciones multivariables de señales polisomnográficas:
1. `EEG_Alfa + EOG_Parpadeo + EMG_Alto`: Patrón típico de vigilia con alerta o relajación inicial.
2. `EEG_Theta + EOG_Lento + EMG_Moderado`: Ritmo theta con movimientos oculares lentos (N1).
3. `EEG_Delta + EOG_Nulo + EMG_Bajo`: Ondas lentas sincronizadas delta de alto voltaje (N3).
4. `EEG_Desincronizado + EOG_Rapido + EMG_Atonia`: Sueño paradójico con atonía muscular completa (REM).
5. `EEG_Huso_K + EOG_Nulo + EMG_Bajo`: Husos de sueño y complejos K característicos de N2.

---

## 5. Escenarios de Simulación

El proyecto analiza comparativamente tres modelos principales sobre una ventana de $T = 480$ épocas (240 minutos / 4 horas):

1. **HMM Base (Fase USA):**
   * Parámetros originales con $A_{\text{base}}$.
   * Representa una noche de sueño sintético estándar bajo el supuesto markoviano puro.

2. **HMM Fragmentado (Fase MODIFICA):**
   * Incremento en la probabilidad de transición a vigilia desde sueño ligero ($0.10 \to 0.30$), sueño profundo ($0.05 \to 0.20$) y REM ($0.10 \to 0.25$).
   * Simula un trastorno de mantenimiento de sueño (insomnio/apnea obstructiva).

3. **HMM Semi-Markoviano HSMM (Fase CREA):**
   * Duraciones mínimas configurables: Vigilia (1 época / 0.5 min), Sueño Ligero (5 épocas / 2.5 min), Sueño Profundo (10 épocas / 5.0 min), REM (5 épocas / 2.5 min).
   * Elimina microfragmentaciones espurias y genera estabilidad ultradiana fisiológicamente verosímil.

4. **Inferencia Estadística Monte Carlo:**
   * Evaluación de 100 réplicas estocásticas independientes por cada modelo para estimar medias poblacionales e intervalos de confianza del 95%.

Para la descripción detallada del marco experimental, consulte **[docs/metodologia.md](docs/metodologia.md)**.

---

## 6. Estructura del Repositorio

El proyecto mantiene una estructura modular y organizada:

```text
.
├── .gitignore                          # Exclusiones de Git (Python, .venv, caches, LaTeX aux)
├── README.md                           # Documentación principal en español
├── requirements.txt                    # Dependencias mínimas del entorno Python
├── docs/                               # Documentación técnica complementaria
│   ├── modelo_matematico.md            # Formulación formal de HMM, HSMM y Log-Viterbi
│   └── metodologia.md                  # Protocolo experimental, métricas y diseño Monte Carlo
├── src/                                # Código fuente de simulación y análisis
│   ├── hmm_sueno.py                    # Modelo HMM Base (Fase USA)
│   ├── hmm_sueno_fragmentado.py        # Modelo HMM Fragmentado (Fase MODIFICA)
│   ├── hmm_semimarkov.py               # Modelo HSMM con Duraciones Explícitas (Fase CREA)
│   ├── analisis_estadistico_montecarlo.py # Batería de simulación Monte Carlo (M=100)
│   ├── actualizar_bitacora.py          # Script de actualización de bitácora
│   ├── generar_articulo_completo.py    # Generador de fuentes LaTeX y Markdown
│   ├── organizar_directorios.py        # Script de organización de carpetas
│   └── sincronizar_entrega.py          # Generador de paquetes de entrega ZIP
├── resultados/                         # Archivos generados por las simulaciones
│   ├── csv/                            # Tablas de secuencias y métricas (10 archivos CSV)
│   │   ├── secuencia_simulada.csv
│   │   ├── metricas_etapas.csv
│   │   ├── secuencia_fragmentada.csv
│   │   ├── metricas_etapas_fragmentado.csv
│   │   ├── comparacion_metricas.csv
│   │   ├── secuencia_semimarkov.csv
│   │   ├── metricas_etapas_semimarkov.csv
│   │   ├── comparacion_tres_modelos.csv
│   │   ├── corridas_montecarlo_tres_modelos.csv
│   │   └── estadisticas_montecarlo_tres_modelos.csv
│   └── figuras/                        # Gráficos en alta resolución 300 DPI (10 archivos PNG)
│       ├── hipnograma.png
│       ├── observaciones.png
│       ├── matriz_transicion.png
│       ├── hipnograma_comparativo.png
│       ├── matriz_transicion_fragmentada.png
│       ├── comparacion_etapas.png
│       ├── hipnograma_semimarkov.png
│       ├── hipnograma_tres_modelos.png
│       ├── distribucion_etapas_tres_modelos.png
│       └── comparacion_intervalos_confianza.png
└── materiales/                         # Documentación y manuscrito académico
    ├── articulo_final.pdf              # Manuscrito académico definitivo compilado (18 páginas)
    ├── articulo_final.tex              # Código fuente LaTeX completo en formato IEEE
    ├── articulo_final.md               # Versión Markdown consolidada del artículo
    ├── bitacora.md                     # Bitácora auditada de decisiones humanas
    ├── modelo-sueno.md                 # Transcripción del caso y matrices originales
    └── auditoria_rubrica.md            # Auditoría formal contra la rúbrica académica
```

---

## 7. Instalación y Requisitos

### Requisitos Previos
* **Python:** Versión 3.10 o superior (probado con Python 3.12 y 3.13).
* **Compilador LaTeX opcional:** Tectonic, XeLaTeX o pdfLaTeX (únicamente si se desea recompilar el manuscrito PDF).

### Pasos de Instalación
1. Clonar el repositorio:
   ```bash
   git clone https://github.com/usuario/hmm-sueno-antigravity.git
   cd hmm-sueno-antigravity
   ```

2. Crear y activar un entorno virtual:
   * **Windows (PowerShell):**
     ```powershell
     python -m venv .venv
     .\.venv\Scripts\Activate.ps1
     ```
   * **macOS / Linux:**
     ```bash
     python3 -m venv .venv
     source .venv/bin/activate
     ```

3. Instalar las dependencias:
   ```bash
   pip install -r requirements.txt
   ```

---

## 8. Guía de Ejecución

Todos los scripts pueden ejecutarse de manera individual o secuencial desde la raíz del proyecto.

### 8.1 Ejecución Individual de Modelos

* **1. Modelo HMM Base (Fase USA):**
  ```bash
  python src/hmm_sueno.py
  ```
  *Genera: `resultados/csv/secuencia_simulada.csv`, `resultados/csv/metricas_etapas.csv`, `resultados/figuras/hipnograma.png`, `resultados/figuras/observaciones.png`, `resultados/figuras/matriz_transicion.png`.*

* **2. Modelo HMM Fragmentado (Fase MODIFICA):**
  ```bash
  python src/hmm_sueno_fragmentado.py
  ```
  *Genera: `resultados/csv/secuencia_fragmentada.csv`, `resultados/csv/comparacion_metricas.csv`, `resultados/figuras/hipnograma_comparativo.png`, `resultados/figuras/matriz_transicion_fragmentada.png`, `resultados/figuras/comparacion_etapas.png`.*

* **3. Modelo Semi-Markoviano HSMM (Fase CREA):**
  ```bash
  python src/hmm_semimarkov.py
  ```
  *Genera: `resultados/csv/secuencia_semimarkov.csv`, `resultados/csv/comparacion_tres_modelos.csv`, `resultados/figuras/hipnograma_tres_modelos.png`, `resultados/figuras/distribucion_etapas_tres_modelos.png`.*

* **4. Análisis Estadístico Inferencial de Monte Carlo ($M = 100$):**
  ```bash
  python src/analisis_estadistico_montecarlo.py
  ```
  *Genera: `resultados/csv/corridas_montecarlo_tres_modelos.csv`, `resultados/csv/estadisticas_montecarlo_tres_modelos.csv`, `resultados/figuras/comparacion_intervalos_confianza.png`.*

### 8.2 Ejecución Completa en un Solo Comando
* **Windows (PowerShell):**
  ```powershell
  python src/hmm_sueno.py; python src/hmm_sueno_fragmentado.py; python src/hmm_semimarkov.py; python src/analisis_estadistico_montecarlo.py
  ```
* **Linux / macOS:**
  ```bash
  python3 src/hmm_sueno.py && python3 src/hmm_sueno_fragmentado.py && python3 src/hmm_semimarkov.py && python3 src/analisis_estadistico_montecarlo.py
  ```

---

## 9. Parámetros y Semillas

Para asegurar la reproducibilidad determinista exacta:

| Parámetro | Valor Canónico | Justificación |
| :--- | :---: | :--- |
| **Semilla Principal (`SEED`)** | `42` | Semilla determinista fija para las simulaciones individuales comparativas. |
| **Horizonte Temporal ($N$)** | `480` épocas/min | 480 pasos temporales discretizados ($T = 480$ min / 8 h). |
| **Semillas Monte Carlo** | `1000` a `1099` | 100 réplicas independientes por modelo para análisis inferencial. |
| **Nivel de Confianza** | `95%` | Intervalos con distribución $t$-Student ($t_{0.975, 99} \approx 1.9842$). |
| **Duraciones Mínimas HSMM** | Wake: 1, Ligero: 5, Profundo: 10, REM: 5 min | Evita fluctuaciones espurias menores a 5 min en profundo y 2.5–5 min en ligero/REM. |

---

## 10. Resultados y Figuras

### 10.1 Comparativa de Métricas (Corrida Determinista `SEED = 42`)
*Datos extraídos directamente de `resultados/csv/comparacion_tres_modelos.csv`:*

| Métrica | Unidad | HMM Base | HMM Fragmentado | HMM Semi-Markov (HSMM) |
| :--- | :---: | :---: | :---: | :---: |
| **Tiempo Total de Sueño (TST)** | $\text{min}$ | $388.00$ | $294.00$ | $436.00$ |
| **Eficiencia del Sueño (SE)** | $\%$ | $80.83\%$ | $61.25\%$ | $90.83\%$ |
| **Latencia al Inicio del Sueño (SOL)** | $\text{min}$ | $0.00$ | $0.00$ | $0.00$ |
| **Despertares tras dormir** | $\text{eventos}$ | $43$ | $88$ | $7$ |
| **Ciclos REM computados** | $\text{ciclos}$ | $32$ | $26$ | $4$ |
| **Exactitud Decodificación Viterbi** | $\%$ | $75.83\%$ | $74.17\%$ | $95.00\%$ |
| **Total episodios de transición** | $\text{bouts}$ | $208$ | $368$ | $28$ |
| **Duración media Vigilia** | $\text{min}$ | $2.14$ | $2.11$ | $6.29$ |
| **Duración media Sueño Profundo** | $\text{min}$ | $2.13$ | $1.65$ | $26.17$ |
| **Duración media REM** | $\text{min}$ | $2.88$ | $2.42$ | $22.00$ |

### 10.2 Resultados Inferenciales Monte Carlo ($M = 100$ réplicas, Media $\pm$ Desv. Est. [IC 95%])
*Datos extraídos directamente de `resultados/csv/estadisticas_montecarlo_tres_modelos.csv`:*

| Métrica | HMM Base (Media $\pm$ DE [IC 95%]) | HMM Fragmentado (Media $\pm$ DE [IC 95%]) | HMM Semi-Markov (Media $\pm$ DE [IC 95%]) |
| :--- | :---: | :---: | :---: |
| **TST (minutos)** | $401.06 \pm 11.40$ $[398.80, 403.32]$ | $306.90 \pm 12.84$ $[304.35, 309.45]$ | $436.01 \pm 16.73$ $[432.69, 439.33]$ |
| **Eficiencia de Sueño (SE)** | $83.55 \pm 2.37\%$ $[83.08, 84.03]$ | $63.94 \pm 2.68\%$ $[63.41, 64.47]$ | $90.84 \pm 3.49\%$ $[90.14, 91.53]$ |
| **Latencia de Sueño (SOL)** | $1.05 \pm 1.34\text{ min}$ $[0.78, 1.32]$ | $1.05 \pm 1.34\text{ min}$ $[0.78, 1.32]$ | $4.08 \pm 4.46\text{ min}$ $[3.19, 4.97]$ |
| **Despertares tras dormir** | $38.66 \pm 4.70$ $[37.73, 39.59]$ | $85.47 \pm 5.91$ $[84.30, 86.64]$ | $5.45 \pm 1.92$ $[5.07, 5.83]$ |
| **Ciclos de Sueño REM** | $32.68 \pm 4.36$ $[31.82, 33.54]$ | $26.27 \pm 3.79$ $[25.52, 27.02]$ | $4.66 \pm 1.55$ $[4.35, 4.97]$ |
| **Exactitud Viterbi** | $71.21 \pm 2.74\%$ $[70.67, 71.76]$ | $69.37 \pm 2.20\%$ $[68.94, 69.81]$ | $94.06 \pm 1.97\%$ $[93.67, 94.45]$ |

### 10.3 Galería de Figuras Clave
Las figuras se encuentran en alta resolución (300 DPI) en `resultados/figuras/`:
* `hipnograma_tres_modelos.png`: Comparación vertical de hipnogramas (Base vs. Fragmentado vs. HSMM).
* `comparacion_intervalos_confianza.png`: Gráficos de barras con intervalos de confianza del 95% para las 6 métricas.
* `matriz_transicion.png` y `matriz_transicion_fragmentada.png`: Mapas de calor estocásticos de las dinámicas de transición.
* `distribucion_etapas_tres_modelos.png`: Histogramas comparativos del porcentaje de tiempo en cada fase.

---

## 11. Limitaciones y Consideraciones Éticas

> [!WARNING] **ALCANCE EDUCATIVO Y LIMITACIONES DE VALIDEZ CLÍNICA**
> 1. **Datos Sintéticos:** Este software genera simulaciones computacionales a partir de modelos probabilísticos hipotéticos predefinidos. **No contiene registros polisomnográficos de pacientes reales**.
> 2. **No es una herramienta de diagnóstico:** Ningún resultado, métrica o figura debe emplearse para la toma de decisiones clínicas, diagnóstico o tratamiento de patologías del sueño humano.
> 3. **Hipótesis Markoviana:** En el sueño humano biológico, las transiciones no son homogéneas ni de primer orden; están moduladas por el ritmo circadiano (Proceso C) y la presión homeostática de sueño (Proceso S), aspectos que los modelos de Markov estacionarios no incorporan de forma nativa.

---

## 12. Documentación y Enlaces

* **Manuscrito Académico Definitivo:**
  * [Artículo en formato PDF (18 páginas)](materiales/articulo_final.pdf)
  * [Código fuente LaTeX (IEEE Transactions)](materiales/articulo_final.tex)
  * [Artículo en Markdown](materiales/articulo_final.md)
* **Documentación Técnica y Auditoría:**
  * [docs/modelo_matematico.md](docs/modelo_matematico.md): Ecuaciones matemáticas, deducciones analíticas y pseudocódigo.
  * [docs/metodologia.md](docs/metodologia.md): Protocolo de simulación, métricas y diseño Monte Carlo.
  * [materiales/bitacora.md](materiales/bitacora.md): Registro cronológico de decisiones de diseño y control humano.
  * [materiales/auditoria_rubrica.md](materiales/auditoria_rubrica.md): Matriz de cumplimiento contra la rúbrica oficial.

---

## Licencia y Créditos
Desarrollado como proyecto de investigación aplicada en modelado estocástico. Distribuido bajo licencia libre con fines exclusivamente académicos y de divulgación científica.
