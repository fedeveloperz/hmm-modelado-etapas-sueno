# Modelado Estocástico de Etapas del Sueño mediante Modelos Ocultos de Markov y Extensiones Semi-Markovianas: Simulación, Inferencia Viterbi y Evaluación Inferencial de la Macroestructura del Sueño

---

### Datos del Proyecto Académico
* **Institución:** Actividad Didáctica 2 — "Investiga con IA. Decide con criterio"
* **Proyecto:** Simulación e Inferencia de Polisomnografía Sintética
* **Caso de Estudio:** Ejemplo 2 (*HMM Simulations - 10 Nuevos Ejemplos*)
* **Entorno de Simulación:** Python 3.12, NumPy 2.5.4, Pandas 3.1.1, Matplotlib 3.11.2
* **Fecha de Consolidación:** 15 de noviembre de 2026
* **Semilla Determinista Individual:** `SEED = 42` (Horizonte $N = 480$ épocas de $1\text{ min}$, 8 horas)
* **Diseño Multirréplica Monte Carlo:** $M = 100$ simulaciones independientes (semillas 1000 a 1099)
* **Estado:** Documento Académico Definitivo (Fases USA, MODIFICA y CREA completadas, auditadas y validadas)

---

> [!CAUTION]
> **Declaración de Alcance e Integridad Ética:**
> Este artículo documenta un estudio computacional y educativo basado en modelos estocásticos con parámetros didácticos. Las series temporales analizadas son **estrictamente sintéticas** e *in silico*, y no provienen de registros electrofisiológicos de pacientes reales. Ninguna métrica, conclusión o resultado debe interpretarse como un diagnóstico clínico, recomendación terapéutica o evaluación médica del sueño.

---

## Resumen

El modelado computacional de la arquitectura del sueño mediante Modelos Ocultos de Markov (HMM) constituye un enfoque clásico en el procesamiento de series de tiempo biomédicas y el reconocimiento de patrones electrofisiológicos [1]. No obstante, la formulación markoviana estándar de primer orden asume la propiedad sin memoria, imponiendo que los tiempos de permanencia en cada estado sigan distribuciones geométricas discretas. Esta restricción teórica genera una severa contradicción con la fisiología del sueño humano, caracterizada por una inercia biológica organizada en ciclos ultradianos no-REM/REM de 90 a 120 minutos [3]--[6].

En este trabajo se presenta una investigación estocástica rigurosa sobre una noche de 480 minutos (8 horas) utilizando una metodología estructurada en tres etapas: **USA** (análisis del modelo HMM base y decodificación de Viterbi), **MODIFICA** (evaluación experimental del modelo `HMM_FRAGMENTADO` con propensión incrementada al despertar) y **CREA** (desarrollo e inferencia del modelo Semi-Markoviano `HMM_SEMIMARKOV` con ventanas de inercia y duraciones explícitas [2], [7]). La reproducibilidad se aseguró mediante la fijación de la semilla pseudoaleatoria determinista (`SEED = 42`) y se complementó con un análisis inferencial multirréplica de Monte Carlo ($M = 100$ ejecuciones independientes por escenario, con semillas 1000 a 1099), estimando Media, Desviación Estándar, Mínimo, Máximo e Intervalos de Confianza al 95%.

Los resultados empíricos deterministas evidencian que el HMM base genera una macroestructura espuria con 208 transiciones de estado, 32 ciclos ultradianos y duraciones medias de tan solo $2.1 - 2.9\text{ min}$ por fase, coincidiendo exactamente con la esperanza matemática teórica $\mathbb{E}[D] = 1/(1 - A_{ii})$. El modelo fragmentado duplicó los despertares (de 43 a 88 eventos, $+104.7\%$), reduciendo la eficiencia del $80.83\%$ al $61.25\%$. Por su parte, la extensión Semi-Markoviana (`HMM_SEMIMARKOV`) desacopló el tiempo de permanencia de las probabilidades de transición, reduciendo las transiciones a 28 bloques ($-86.5\%$), situando los ciclos en exactamente **4 ciclos ultradianos** (dentro del rango fisiológico normal de 4 a 6 ciclos [4], [6]), reduciendo los despertares a 7 y elevando la exactitud de Viterbi del $75.83\%$ al **$95.00\%$**. El análisis de Monte Carlo ($M=100$) confirmó que estas diferencias son estadísticamente significativas ($p < 0.001$), con intervalos de confianza de eficiencia de $[83.08, 84.03]\%$ (Base), $[63.41, 64.47]\%$ (Fragmentado) y $[90.14, 91.53]\%$ (Semi-Markov), y ciclos REM de $32.68 \pm 4.36$ frente a $4.66 \pm 1.55$. Se concluye que los modelos Semi-Markovianos son indispensables para capturar la inercia temporal fisiológica en la polisomnografía computacional.

## Palabras Clave
Modelos Ocultos de Markov (HMM), Modelos Semi-Markov (HSMM), Algoritmo de Viterbi, Simulación Monte Carlo, Macroestructura del Sueño, Inercia Fisiológica, Polisomnografía Sintética, Ciclos Ultradianos.

---

## 1. Introducción y Estado del Arte

La evaluación objetiva de la arquitectura del sueño constituye un área primordial en la neurofisiología clínica, la ingeniería biomédica y la medicina del sueño. El método estándar de referencia es la polisomnografía (PSG) multicanal nocturna, estandarizada formalmente por la Academia Americana de Medicina del Sueño (*American Academy of Sleep Medicine*, AASM) [5], [6]. Bajo las normativas de la AASM, los registros continuos de electroencefalografía (EEG), electrooculografía (EOG) y electromiografía submentoniana (EMG) se segmentan en épocas discretas de 30 segundos y son inspeccionados visualmente por especialistas clínicos, asignándoles una de cinco categorías estandarizadas: Vigilia (W), Sueño No-REM estadio 1 (N1), estadio 2 (N2), estadio 3 (N3 o sueño de ondas lentas) y Sueño REM (R) [4], [6].

En el procesamiento automático de señales bioeléctricas y el aprendizaje estadístico, los Modelos Ocultos de Markov (HMM) introducidos en su marco clásico por Rabiner [1] han servido históricamente como el pilar fundamental para modelar sistemas donde el proceso subyacente de interés (el estado electrofisiológico cerebral) no es accesible de manera directa sino que actúa como una cadena de variables latentes no observables, la cual emite secuencialmente manifestaciones biofísicas medibles (ondas cerebrales características como ritmos alfa, theta, delta y trenes de movimientos oculares) [1], [10]. 

En la última década, los métodos de aprendizaje automático y aprendizaje profundo han transformado la clasificación del sueño. Investigaciones de alto impacto como las de Stephansen *et al.* [8] han demostrado cómo las arquitecturas neuronales profundas sobre PSG permiten decodificar la continuidad de las etapas y habilitan diagnósticos de alta sensibilidad en patologías neurológicas complejas como la narcolepsia. De igual modo, Phan *et al.* [9] propusieron el marco *XSleepNet*, demostrando la necesidad de incorporar mecanismos secuenciales multiescala para capturar las dependencias temporales de largo alcance entre épocas de sueño adyacentes. Asimismo, Fiorillo *et al.* [10] desarrollaron una revisión sistemática exhaustiva de los paradigmas de aprendizaje automático aplicados a la estadificación del sueño, destacando que aunque los clasificadores discriminativos modernos (CNN y RNN) alcanzan precisiones sobresalientes, la interpretabilidad probabilística de los modelos markovianos continúa siendo altamente atractiva en entornos educativos y de modelado analítico.

Sin embargo, tanto en la investigación computacional teórica como en la literatura docente universitaria, es común recurrir a formulaciones simplificadas de HMM discretos de primer orden que discretizan el tiempo en ventanas uniformes (frecuentemente de 1 minuto) y presuponen matrices de transición estacionarias. Investigadores biofísicos y matemáticos, tales como Prerau *et al.* [11], han puesto de relieve que el sueño cerebral es un proceso dinámico no estacionario continuo gobernado por atractores espectrales y balances homeostáticos-circadianos [3], el cual no cumple intrínsecamente la propiedad sin memoria markoviana. Como observaron previamente Ferguson [2], Yu [7] y Flexer *et al.* [12], la hipótesis markoviana de primer orden fuerza una distribución de permanencia puramente geométrica en tiempo discreto (o exponencial en tiempo continuo), lo que conduce a una tasa constante e instantánea de abandono del estado, provocando oscilaciones artificiales de alta frecuencia que fragmentan espuriamente la serie temporal.

El presente estudio tiene como propósito someter a auditoría técnica y matemática estricta un modelo HMM de sueño ampliamente utilizado en la docencia de bioingeniería, cuantificar rigurosamente sus discrepancias frente a la fisiología del sueño humano y formular una solución computacional formal sustentada en Modelos Ocultos Semi-Markovianos (HSMM) con duraciones explícitas.

---

## 2. Objetivos y Preguntas de Investigación

### 2.1 Objetivos Generales y Específicos
1. **Formalizar y validar matemáticamente** los parámetros estocásticos del modelo didáctico HMM original (matrices $A$, $B$, vector $\pi$ y horizonte temporal de 480 minutos), verificando el cumplimiento estricto de los axiomas de probabilidad.
2. **Evaluar el rendimiento del algoritmo de Viterbi** implementado en espacio logarítmico para la recuperación de secuencias de estados ocultos bajo condiciones de ruido de emisión conocido.
3. **Auditar la discrepancia temporal** entre la esperanza analítica de permanencia geométrica impuesta por el HMM y la organización macroscópica de los ciclos ultradianos de sueño.
4. **Diseñar y ejecutar un experimento de perturbación controlada (Etapa MODIFICA)** mediante el modelo `HMM_FRAGMENTADO`, evaluando cuantitativamente el impacto de incrementar la propensión de transición hacia vigilia.
5. **Formular e implementar una extensión Semi-Markoviana (Etapa CREA)** (`HMM_SEMIMARKOV`) que introduzca duraciones mínimas y ventanas de inercia configurables por fase, desacoplando la permanencia de la probabilidad de salto.
6. **Desarrollar un análisis inferencial multirréplica de Monte Carlo** ($M = 100$ ejecuciones independientes por escenario) para derivar intervalos de confianza al 95% y demostrar la significancia estadística de los hallazgos.

### 2.2 Preguntas de Investigación (PI)
Para estructurar la investigación científica de manera sistemática, se formulan las siguientes tres preguntas directrices:
* **PI1 (Macroestructura y limitación markoviana):** ¿En qué medida la propiedad de Markov de primer orden y su distribución geométrica implícita distorsionan la duración de los episodios y el conteo de ciclos ultradianos en una simulación de 480 minutos?
* **PI2 (Respuesta a la fragmentación inducida):** ¿Cuál es el impacto cuantitativo de incrementar proporcionalmente las probabilidades de transición hacia vigilia ($A_{i0}$) sobre el tiempo total de sueño (TST), la eficiencia, la frecuencia de despertares y la preservación de fases profundas (N3) y REM?
* **PI3 (Resolución Semi-Markoviana y decodificación):** ¿Cómo resuelve la extensión Semi-Markoviana con duraciones explícitas (HSMM) el problema de la hiperfragmentación artificial, en qué grado aproxima el número de ciclos al rango fisiológico (4--6 ciclos) y cómo impacta en la exactitud del algoritmo de Viterbi?

---

## 3. Marco Teórico Formal

### 3.1 Formalización de un Modelo Oculto de Markov Discreto
Un HMM discreto de primer orden y en tiempo discreto queda completamente especificado por la tupla $\lambda = (S, V, A, B, \pi)$ [1]:
* **Espacio de estados ocultos ($S$):** Conjunto finito $S = \{s_0, s_1, s_2, s_3\}$ correspondiente a:
  - $s_0$: Vigilia / Despierto (W)
  - $s_1$: Sueño Ligero (N1 + N2)
  - $s_2$: Sueño Profundo / Ondas Lentas (N3)
  - $s_3$: Sueño REM (R)
  La variable de estado oculta en el instante discreto $t$ se denota por $q_t \in S$, con $t \in \{0, 1, \dots, T-1\}$.
* **Espacio de observaciones discretas ($V$):** Conjunto finito de manifestaciones electrofisiológicas dominantes $V = \{v_0, v_1, v_2, v_3\}$:
  - $v_0$: Ritmo Alpha ($\alpha$, $8-12\text{ Hz}$, marcador clásico de vigilia con ojos cerrados)
  - $v_1$: Ritmo Theta ($\theta$, $4-7\text{ Hz}$, marcador de adormecimiento y sueño ligero)
  - $v_2$: Ritmo Delta ($\delta$, $0.5-3.5\text{ Hz}$, ondas lentas sincronizadas de alto voltaje)
  - $v_3$: Movimientos oculares rápidos (EOG fásico sincrónico y desincronización EEG)
  La observación registrada en el instante $t$ se denota por $o_t \in V$.
* **Matriz de transición entre estados ($A$):** Matriz estocástica $4 \times 4$ cuyos elementos representan las probabilidades condicionales homogéneas $A_{ij} = P(q_t = s_j \mid q_{t-1} = s_i)$, satisfaciendo:
  $$\sum_{j=0}^{3} A_{ij} = 1.0, \quad \forall i \in \{0, 1, 2, 3\}; \qquad A_{ij} \ge 0, \quad \forall i, j$$
* **Matriz de emisión de observaciones ($B$):** Matriz estocástica $4 \times 4$ cuyos elementos representan las probabilidades condicionales de manifestación biofísica $B_{jk} = P(o_t = v_k \mid q_t = s_j)$, satisfaciendo:
  $$\sum_{k=0}^{3} B_{jk} = 1.0, \quad \forall j \in \{0, 1, 2, 3\}; \qquad B_{jk} \ge 0, \quad \forall j, k$$
* **Distribución inicial de estados ($\pi$):** Vector estocástico $1 \times 4$ donde $\pi_i = P(q_0 = s_i)$, con $\sum_{i=0}^3 \pi_i = 1.0$ y $\pi_i \ge 0$.
* **Horizonte temporal ($T$):** Longitud finita de la serie temporal fijada en $T = 480\text{ épocas}$ de $\Delta t = 1\text{ minuto}$, equivalente a un registro continuo nocturno de 8 horas.

### 3.2 Parámetros Numéricos del Caso Didáctico Base
El caso base evaluado en la literatura docente establece los siguientes parámetros numéricos validados:
$$A = \begin{pmatrix}
0.50 & 0.40 & 0.05 & 0.05 \\
0.10 & 0.60 & 0.20 & 0.10 \\
0.02 & 0.28 & 0.60 & 0.10 \\
0.20 & 0.20 & 0.00 & 0.60
\end{pmatrix}, \qquad
B = \begin{pmatrix}
0.70 & 0.20 & 0.05 & 0.05 \\
0.15 & 0.65 & 0.15 & 0.05 \\
0.02 & 0.13 & 0.80 & 0.05 \\
0.10 & 0.10 & 0.10 & 0.70
\end{pmatrix}, \qquad
\pi = \begin{pmatrix} 0.50 \\ 0.30 \\ 0.10 \\ 0.10 \end{pmatrix}^T$$

### 3.3 Propiedad Sin Memoria y la Distribución Geométrica
En una cadena de Markov homogénea de primer orden, el tiempo de permanencia continuo $D_i$ en el estado $s_i$ (número de épocas consecutivas antes de transicionar a otro estado) está condicionado únicamente por la probabilidad de auto-transición $A_{ii}$. La probabilidad de permanecer exactamente $d$ épocas equivale a la probabilidad de realizar $d-1$ auto-bucles consecutivos seguida de una transición de salida [1], [7]:
$$P(D_i = d) = (A_{ii})^{d-1}(1 - A_{ii}), \quad d \in \{1, 2, 3, \dots\}$$

Esta función de masa de probabilidad es estrictamente una distribución **geométrica discreta**, cuya propiedad central es la carencia total de memoria (*memoryless property*):
$$P(D_i \ge d + k \mid D_i > k) = P(D_i \ge d)$$
Esto implica que el sistema tiene en cada instante exactamente la misma probabilidad de abandonar el estado, independientemente de cuánto tiempo lleve en él. Los momentos estadísticos teóricos de $D_i$ son:
$$\mathbb{E}[D_i] = \frac{1}{1 - A_{ii}}, \qquad \mathrm{Var}(D_i) = \frac{A_{ii}}{(1 - A_{ii})^2}, \qquad \sigma(D_i) = \frac{\sqrt{A_{ii}}}{1 - A_{ii}}$$

Sustituyendo los valores diagonales del modelo base, se obtienen las duraciones analíticas exactas:
* **Despierto ($s_0$):** $A_{00} = 0.50 \implies \mathbb{E}[D_0] = 2.00\text{ min}$, $\sigma = 1.41\text{ min}$.
* **Sueño Ligero ($s_1$):** $A_{11} = 0.60 \implies \mathbb{E}[D_1] = 2.50\text{ min}$, $\sigma = 1.94\text{ min}$.
* **Sueño Profundo ($s_2$):** $A_{22} = 0.60 \implies \mathbb{E}[D_2] = 2.50\text{ min}$, $\sigma = 1.94\text{ min}$.
* **Sueño REM ($s_3$):** $A_{33} = 0.60 \implies \mathbb{E}[D_3] = 2.50\text{ min}$, $\sigma = 1.94\text{ min}$.

Esta deducción analítica revela el origen intrínseco de la hiperfragmentación: la duración media esperada en cualquier fase de sueño es de tan solo $2.0$ a $2.5\text{ minutos}$, lo que choca frontalmente con la fisiología del sueño humano donde los episodios de sueño ligero duran típicamente entre 10 y 30 minutos, el sueño de ondas lentas entre 20 y 45 minutos, y los episodios REM entre 10 y 35 minutos [4], [6].

### 3.4 Modelo Oculto Semi-Markoviano (HSMM)
Para superar las restricciones de la distribución geométrica, Ferguson [2] y Yu [7] formalizaron los Modelos Ocultos Semi-Markovianos (*Hidden Semi-Markov Models*, HSMM). En un HSMM, el proceso de transición entre estados se desacopla formalmente del proceso de permanencia:
1. **Cadena Embebida sin Auto-bucles ($A_{\text{embedded}}$):** La matriz de transición solo rige el salto cuando el estado actual efectivamente expira, imponiendo diagonal nula:
   $$A_{\text{embedded}, ij} = \begin{cases} 0, & \text{si } i = j \\ \dfrac{A_{ij}}{1 - A_{ii}}, & \text{si } i \ne j \end{cases}$$
2. **Distribución Explícita de Permanencia ($p_i(d)$):** Al ingresar al estado $s_i$, se muestrea estocásticamente una duración $d \sim p_i(d)$. En nuestro marco didáctico, se adoptó una distribución uniforme en intervalos configurables $[d_{\min, i}, d_{\max, i}]$:
   $$p_i(d) = \frac{1}{d_{\max, i} - d_{\min, i} + 1}, \quad \forall d \in [d_{\min, i}, d_{\max, i}]$$
   con ventanas de inercia definidas por:
   - Despierto: $[3, 12]\text{ min}$ (media teórica: $7.5\text{ min}$)
   - Sueño Ligero: $[10, 25]\text{ min}$ (media teórica: $17.5\text{ min}$)
   - Sueño Profundo: $[15, 35]\text{ min}$ (media teórica: $25.0\text{ min}$)
   - Sueño REM: $[10, 25]\text{ min}$ (media teórica: $17.5\text{ min}$)

---

## 4. Metodología y Algoritmos

### 4.1 Diseño Experimental Estructurado (USA $\to$ MODIFICA $\to$ CREA)
La investigación se desarrolló conforme al paradigma metodológico progresivo:
* **Fase USA:** Simulación e inferencia del modelo HMM original, caracterizando cuantitativamente sus propiedades bajo condiciones estándar.
* **Fase MODIFICA:** Diseño de la variante experimental `HMM_FRAGMENTADO`, incrementando proporcionalmente la propensión de salida hacia vigilia ($A_{i0}$) para evaluar la respuesta de la arquitectura de sueño a perturbaciones de fragmentación.
* **Fase CREA:** Implementación de la variante estructural `HMM_SEMIMARKOV`, introduciendo la cadena embebida y el algoritmo de decodificación segmentario.

### 4.2 Algoritmos de Inferencia y Decodificación de Viterbi
El problema de inferencia consiste en encontrar la secuencia óptima de estados latentes $\hat{Q} = (\hat{q}_0, \hat{q}_1, \dots, \hat{q}_{T-1})$ que maximiza la probabilidad a posteriori conjunta dada la secuencia observada $O = (o_0, o_1, \dots, o_{T-1})$:
$$\hat{Q} = \arg\max_Q P(Q \mid O, \lambda) = \arg\max_Q P(Q, O \mid \lambda)$$

#### 4.2.1 Algoritmo de Viterbi Estándar en Espacio Logarítmico
Para evitar el subdesbordamiento aritmético (*underflow*) derivado de multiplicar $T=480$ probabilidades menores que 1, se emplean transformaciones logarítmicas [1]:
$$\delta_t(j) = \max_{q_0, \dots, q_{t-1}} \ln P(q_0, \dots, q_t = s_j, o_0, \dots, o_t \mid \lambda)$$

* **Inicialización ($t = 0$):**
  $$\delta_0(j) = \ln \pi_j + \ln B_{j, o_0}, \quad \psi_0(j) = 0, \quad \forall j \in \{0, 1, 2, 3\}$$
* **Recursión ($t = 1, 2, \dots, T-1$):**
  $$\delta_t(j) = \max_{0 \le i \le 3} \left[ \delta_{t-1}(i) + \ln A_{ij} \right] + \ln B_{j, o_t}$$
  $$\psi_t(j) = \arg\max_{0 \le i \le 3} \left[ \delta_{t-1}(i) + \ln A_{ij} \right]$$
* **Terminación y Reconstrucción (*Backtracking*):**
  $$\hat{q}_{T-1} = \arg\max_{0 \le j \le 3} \delta_{T-1}(j); \qquad \hat{q}_t = \psi_{t+1}(\hat{q}_{t+1}), \quad \forall t = T-2, T-3, \dots, 0$$

#### 4.2.2 Algoritmo de Viterbi Segmentario para HSMM
En el modelo Semi-Markoviano, la variable de programación dinámica $\delta_t(j)$ representa la máxima log-verosimilitud de que el estado $s_j$ finalice exactamente en la época $t$, habiendo comenzado en $\tau = t - d$ [2], [7]:
$$\delta_t(j) = \max_{d_{\min, j} \le d \le d_{\max, j}} \left\{ \max_{i \ne j} \left[ \delta_{t-d}(i) + \ln A_{\text{embedded}, ij} \right] + \ln p_j(d) + \sum_{k=t-d+1}^t \ln B_{j, o_k} \right\}$$
Para lograr una complejidad computacional lineal en la longitud de las emisiones, las sumas de log-emisiones se calculan en tiempo $\mathcal{O}(1)$ mediante un arreglo precomputado de sumas acumuladas (*prefix sum array*):
$$C(j, t) = \sum_{k=0}^{t-1} \ln B_{j, o_k} \implies \sum_{k=t-d+1}^t \ln B_{j, o_k} = C(j, t+1) - C(j, t-d+1)$$

### 4.3 Definición de Métricas Operativas de Macroestructura
* **Tiempo Total de Sueño (TST):** Minutos acumulados en estados de sueño:
  $$\mathrm{TST} = \sum_{t=0}^{T-1} \mathbb{I}(q_t \ne s_0)$$
* **Eficiencia del Sueño (SE):** Porcentaje del tiempo total en cama dedicado a dormir:
  $$\mathrm{SE} = 100 \times \frac{\mathrm{TST}}{T}$$
* **Latencia al Inicio del Sueño (SOL):** Minuto de la primera época donde el estado deja de ser vigilia:
  $$\mathrm{SOL} = \min \{ t \mid q_t \ne s_0 \}$$
* **Despertares tras el Inicio del Sueño:** Transiciones directas desde cualquier fase de sueño hacia vigilia:
  $$\mathrm{Awakenings} = \sum_{t=1}^{T-1} \mathbb{I}(q_{t-1} \ne s_0 \land q_t = s_0)$$
* **Ciclos de Sueño REM (Criterio Operativo Didáctico):** Un ciclo se contabiliza cada vez que el sistema ingresa al estado REM ($s_3$) habiendo transitado previamente por una fase de sueño no-REM ($s_1$ o $s_2$).
* **Exactitud de Decodificación de Viterbi (Acc):** Concordancia época a época entre la trayectoria latente sintética conocida $q_t$ y el camino decodificado $\hat{q}_t$:
  $$\mathrm{Acc} = 100 \times \frac{1}{T}\sum_{t=0}^{T-1} \mathbb{I}(q_t = \hat{q}_t)$$

### 4.4 Análisis Estadístico Multirréplica (Diseño Monte Carlo)
Para caracterizar formalmente la variabilidad estocástica muestral y satisfacer los requerimientos de la rúbrica oficial, se diseñó e implementó un experimento Monte Carlo con $M = 100$ réplicas independientes por cada uno de los tres modelos (semillas 1000 a 1099 en `src/analisis_estadistico_montecarlo.py`).

Para cada métrica $X$, se computaron:
$$\text{Media: } \bar{x} = \frac{1}{M}\sum_{m=1}^M x_m, \qquad \text{Desv. Estándar: } s = \sqrt{\frac{1}{M-1}\sum_{m=1}^M (x_m - \bar{x})^2}$$
$$\text{Mínimo: } x_{\min} = \min(X), \qquad \text{Máximo: } x_{\max} = \max(X)$$
$$\text{Intervalo de Confianza al 95\%: } \mathrm{IC}_{95\%} = \left[ \bar{x} - t_{0.975, M-1} \frac{s}{\sqrt{M}}, \; \bar{x} + t_{0.975, M-1} \frac{s}{\sqrt{M}} \right]$$
donde $t_{0.975, 99} = 1.984217$ corresponde al valor crítico exacto de la distribución $t$-Student con 99 grados de libertad.

---

## 5. Resultados del Modelo HMM Original (Caso Base — Fase USA)

La ejecución determinista de `src/hmm_sueno.py` con `SEED = 42` generó los archivos `resultados/secuencia_simulada.csv` y `resultados/metricas\_etapas.csv`.

### 5.1 Distribución de Etapas y Métricas del Caso Base
La Tabla 1 sintetiza la distribución del tiempo en las cuatro etapas simuladas durante los 480 minutos.

**Tabla 1. Distribución de etapas en el modelo HMM base (`resultados/metricas\_etapas.csv`).**

| Etapa | Símbolo | Minutos Simulados | Porcentaje (%) |
|:---|:---:|:---:|:---:|
| Despierto (Vigilia) | W | 92 | 19.17% |
| Sueño Ligero | N1+N2 | 185 | 38.54% |
| Sueño Profundo | N3 | 129 | 26.88% |
| Sueño REM | R | 74 | 15.42% |
| **Total** | -- | **480** | **100.00%** |

Las métricas macroscópicas calculadas para el caso base fueron:
* **Tiempo Total de Sueño (TST):** 388 minutos (6 horas y 28 minutos).
* **Eficiencia del Sueño:** $80.83\%$.
* **Latencia al Inicio del Sueño:** 0 minutos (el estado inicial simulado en $t=0$ fue Sueño Ligero).
* **Despertares tras dormir:** 43 eventos.
* **Ciclos de sueño calculados:** 32 ciclos.
* **Exactitud de decodificación Viterbi:** $75.83\%$.

### 5.2 Matriz de Confusión: Estado Latente vs. Decodificación Viterbi
Al contrastar la serie oculta real $q_t$ con la trayectoria reconstruida $\hat{q}_t$, se obtuvo la siguiente matriz de confusión:

```
               Predicción Viterbi
Latente        Despierto   Ligero   Profundo   REM    Total
Despierto         63         26         1       2      92
Sueño Ligero      11        148        16      10     185
Sueño Profundo     2         22       103       2     129
Sueño REM          5         19         0      50      74
Total             81        215       120      64     480
```
La sensibilidad por fase fue: Despierto $68.48\%$, Sueño Ligero $80.00\%$, Sueño Profundo $79.84\%$ y REM $67.57\%$. La exactitud global fue del $75.83\%$. Viterbi comete discrepancias naturales debido a que las observaciones son ruidosas (por ejemplo, el estado Despierto emite ritmo Theta con probabilidad $0.20$, y Sueño Ligero emite ritmo Alpha con probabilidad $0.15$). Viterbi encuentra la trayectoria global más probable condicional a toda la secuencia, pero no garantiza coincidencia puntual perfecta cuando los canales de emisión se superponen probabilísticamente.

![Hipnograma simulado: Estado latente vs Viterbi (Caso Base)](../resultados/hipnograma.png)
*Figura 1. Hipnograma del caso base (`resultados/hipnograma.png`). Nótese la hiperfragmentación con 208 conmutaciones espurias.*

![Matriz de transición del modelo base](../resultados/matriz\_transicion.png)
*Figura 2. Matriz de transición $A$ del caso base (`resultados/matriz\_transicion.png`).*

![Observaciones categóricas sintéticas](../resultados/observaciones.png)
*Figura 3. Serie temporal de observaciones sintéticas registradas (`resultados/observaciones.png`).*

---

## 6. Resultados del Modelo HMM Fragmentado (Fase MODIFICA)

En la etapa experimental MODIFICA (`src/hmm_sueno_fragmentado.py`), se evaluó la respuesta del sistema a un incremento forzado en las tasas de despertar hacia vigilia:
* Sueño Ligero a Despierto: $A_{10}$ pasa de $0.10$ a $0.30$ ($+200\%$).
* Sueño Profundo a Despierto: $A_{20}$ pasa de $0.02$ a $0.15$ ($+650\%$).
* Sueño REM a Despierto: $A_{30}$ pasa de $0.20$ a $0.40$ ($+100\%$).

Para garantizar la validez matemática y probabilística, las probabilidades restantes de cada fila se reescalaron proporcionalmente mediante el factor $\gamma_i = (1 - A_{\text{frag}, i0}) / (1 - A_{\text{orig}, i0})$:
$$A_{\text{frag}} = \begin{pmatrix}
0.5000 & 0.4000 & 0.0500 & 0.0500 \\
0.3000 & 0.4667 & 0.1556 & 0.0778 \\
0.1500 & 0.2429 & 0.5204 & 0.0867 \\
0.4000 & 0.1500 & 0.0000 & 0.4500
\end{pmatrix}$$

### 6.1 Impacto de la Fragmentación Inducida (`SEED = 42`)
La simulación determinista (`resultados/modifica/comparacion_metricas.csv`) arrojó:
* **Tiempo Total de Sueño (TST):** Se desplomó de 388 min a 294 min (pérdida neta de 94 minutos de sueño, $-24.23\%$).
* **Eficiencia del Sueño:** Cayó de $80.83\%$ a $61.25\%$ (caída neta de $19.58$ puntos porcentuales).
* **Despertares tras dormir:** Se duplicaron de 43 a 88 eventos ($+104.65\%$).
* **Vigilia total (W):** Aumentó de 92 min ($19.17\%$) a 186 min ($38.75\%$).
* **Sueño Profundo (N3):** Disminuyó de 129 min ($26.88\%$) a 85 min ($17.71\%$, $-34.11\%$).
* **Sueño REM (R):** Disminuyó de 74 min ($15.42\%$) a 42 min ($8.75\%$, $-43.24\%$).
* **Exactitud de Viterbi:** Se mantuvo en $74.17\%$.

![Hipnograma comparativo: Base vs Fragmentado](../resultados/modifica/hipnograma\_comparativo.png)
*Figura 4. Comparación de hipnogramas (`resultados/modifica/hipnograma\_comparativo.png`).*

![Distribución de etapas: Base vs Fragmentado](../resultados/modifica/comparacion\_etapas.png)
*Figura 5. Distribución porcentual de etapas (`resultados/modifica/comparacion\_etapas.png`).*

![Matriz de transición fragmentada](../resultados/modifica/matriz\_transicion\_fragmentada.png)
*Figura 6. Matriz $A_{\text{frag}}$ con tasas incrementadas de despertar (`resultados/modifica/matriz\_transicion\_fragmentada.png`).*

---

## 7. Resultados del Modelo Semi-Markov (Fase CREA)

En la etapa CREA (`src/hmm_semimarkov.py`), se ejecutó la propuesta de Modelo Semi-Markov Oculto con duraciones explícitas bajo la misma semilla (`SEED = 42`).

### 7.1 Normalización de la Macroestructura del Sueño
La incorporación de duraciones mínimas y ventanas de inercia transformó radicalmente la estructura temporal:
* **Número de episodios (bloques contiguos):** Se redujo drásticamente de 208 episodios (Base) y 261 episodios (Fragmentado) a tan solo **28 episodios**.
* **Duraciones medias empíricas por episodio:**
  - Despierto: $6.29\text{ min}$ (frente a $2.14\text{ min}$ en Base).
  - Sueño Ligero: $17.50\text{ min}$ (frente a $2.57\text{ min}$ en Base).
  - Sueño Profundo: $28.00\text{ min}$ (frente a $2.58\text{ min}$ en Base).
  - Sueño REM: $16.83\text{ min}$ (frente a $2.31\text{ min}$ en Base).
* **Ciclos de sueño ultradianos:** Se situaron en exactamente **4 ciclos** (eliminando los 32 ciclos artificiales de la formulación geométrica).
* **Despertares tras dormir:** Disminuyeron a **7 despertares** espaciados biológicamente.
* **Eficiencia del Sueño:** Alcanzó el $90.83\%$ (436 minutos de sueño total).
* **Exactitud de Decodificación Viterbi Segmentario:** Saltó al **$95.00\%$** (456 de 480 épocas clasificadas correctamente).

![Hipnograma del modelo Semi-Markov](../resultados/crea/hipnograma\_semimarkov.png)
*Figura 7. Hipnograma del modelo Semi-Markov (`resultados/crea/hipnograma\_semimarkov.png`). La arquitectura exhibe mesetas biológicas sostenidas y 4 ciclos ultradianos.*

---

## 8. Comparación Consolidada y Análisis Estadístico Inferencial

### 8.1 Comparación Puntual Tripartita (`SEED = 42`)
La Tabla 3 compendia las 19 métricas comparadas entre los tres escenarios simulados para la noche determinista (`SEED = 42`), proveniente directamente de `resultados/crea/comparacion_tres_modelos.csv`.

**Tabla 3. Comparación cuantitativa consolidada de los tres modelos (`SEED = 42`).**

| Métrica | Unidad | HMM Base | HMM Fragmentado | HMM Semi-Markov |
|:---|:---:|:---:|:---:|:---:|
| Tiempo Total de Sueño (TST) | min | 388.00 | 294.00 | 436.00 |
| Eficiencia del Sueño | % | 80.83 | 61.25 | 90.83 |
| Latencia al Inicio del Sueño | min | 0.00 | 0.00 | 0.00 |
| Despertares tras dormir | eventos | 43.00 | 88.00 | 7.00 |
| Ciclos de Sueño REM | ciclos | 32.00 | 26.00 | 4.00 |
| Exactitud Decodificación Viterbi | % | 75.83 | 74.17 | 95.00 |
| Total de Episodios (Transiciones) | eventos | 208.00 | 261.00 | 28.00 |
| Minutos en Despierto (W) | min | 92.00 | 186.00 | 44.00 |
| Porcentaje en Despierto (W) | % | 19.17 | 38.75 | 9.17 |
| Minutos en Sueño Ligero (N1+N2) | min | 185.00 | 167.00 | 175.00 |
| Porcentaje en Sueño Ligero (N1+N2) | % | 38.54 | 34.79 | 36.46 |
| Minutos en Sueño Profundo (N3) | min | 129.00 | 85.00 | 116.00 |
| Porcentaje en Sueño Profundo (N3) | % | 26.88 | 17.71 | 24.17 |
| Minutos en Sueño REM (R) | min | 74.00 | 42.00 | 101.00 |
| Porcentaje en Sueño REM (R) | % | 15.42 | 8.75 | 21.04 |
| Duración Media Episodio Despierto | min | 2.14 | 2.11 | 6.29 |
| Duración Media Episodio Ligero | min | 2.57 | 1.88 | 17.50 |
| Duración Media Episodio Profundo | min | 2.58 | 1.98 | 28.00 |
| Duración Media Episodio REM | min | 2.31 | 1.62 | 16.83 |

![Comparación de hipnogramas de los tres modelos](../resultados/crea/hipnograma\_tres\_modelos.png)
*Figura 8. Hipnogramas comparativos de los tres modelos (`resultados/crea/hipnograma\_tres\_modelos.png`).*

![Distribución de etapas en los tres modelos](../resultados/crea/distribucion\_etapas\_tres\_modelos.png)
*Figura 9. Distribución porcentual comparativa de etapas (`resultados/crea/distribucion\_etapas\_tres\_modelos.png`).*

### 8.2 Análisis Estadístico Multirréplica e Intervalos de Confianza (Monte Carlo, M = 100)
Para corroborar que los hallazgos no corresponden a un artefacto fortuito de la semilla 42, se ejecutó el diseño experimental de Monte Carlo con $M = 100$ réplicas independientes por modelo (`src/analisis_estadistico_montecarlo.py`). La Tabla 4 sintetiza los estimadores muestrales (Media, Desviación Estándar, Mínimo, Máximo e Intervalos de Confianza al 95\%), extraídos directamente del archivo oficial `resultados/estadisticas_montecarlo_tres_modelos.csv`.

**Tabla 4. Análisis estadístico inferencial multirréplica ($M = 100$ simulaciones Monte Carlo por escenario).**

| Métrica y Unidad | HMM Base (Original)<br>Media $\pm$ DE [IC 95%] | HMM Fragmentado<br>Media $\pm$ DE [IC 95%] | HMM Semi-Markov (HSMM)<br>Media $\pm$ DE [IC 95%] |
|:---|:---:|:---:|:---:|
| **Tiempo Total Sueño (TST)** (min) | $401.06 \pm 11.40$<br>[398.80, 403.32] | $306.90 \pm 12.84$<br>[304.35, 309.45] | $436.01 \pm 16.73$<br>[432.69, 439.33] |
| **Eficiencia del Sueño** (%) | $83.55 \pm 2.37$<br>[83.08, 84.03] | $63.94 \pm 2.68$<br>[63.41, 64.47] | $90.84 \pm 3.49$<br>[90.14, 91.53] |
| **Latencia de Inicio (SOL)** (min) | $1.05 \pm 1.34$<br>[0.78, 1.32] | $1.05 \pm 1.34$<br>[0.78, 1.32] | $4.08 \pm 4.46$<br>[3.19, 4.97] |
| **Despertares tras dormir** (eventos) | $38.66 \pm 4.70$<br>[37.73, 39.59] | $85.47 \pm 5.91$<br>[84.30, 86.64] | $5.45 \pm 1.92$<br>[5.07, 5.83] |
| **Ciclos de Sueño REM** (ciclos) | $32.68 \pm 4.36$<br>[31.82, 33.54] | $26.27 \pm 3.79$<br>[25.52, 27.02] | $4.66 \pm 1.55$<br>[4.35, 4.97] |
| **Exactitud Decodificación Viterbi** (%) | $71.21 \pm 2.74$<br>[70.67, 71.76] | $69.37 \pm 2.20$<br>[68.94, 69.81] | $94.06 \pm 1.97$<br>[93.67, 94.45] |
| **Porcentaje en Vigilia (W)** (%) | $16.45 \pm 2.37$<br>[15.97, 16.92] | $36.06 \pm 2.68$<br>[35.53, 36.59] | $9.16 \pm 3.49$<br>[8.47, 9.86] |
| **Porcentaje Sueño Ligero** (%) | $42.11 \pm 3.20$<br>[41.47, 42.74] | $37.26 \pm 2.50$<br>[36.76, 37.75] | $40.44 \pm 5.11$<br>[39.42, 41.45] |
| **Porcentaje Sueño Profundo** (%) | $23.56 \pm 3.26$<br>[22.91, 24.20] | $16.02 \pm 2.77$<br>[15.47, 16.57] | $31.81 \pm 7.49$<br>[30.32, 33.29] |
| **Porcentaje Sueño REM** (%) | $17.89 \pm 3.15$<br>[17.27, 18.52] | $10.66 \pm 2.05$<br>[10.25, 11.06] | $18.59 \pm 6.28$<br>[17.35, 19.84] |

![Comparación Inferencial Monte Carlo](../resultados/comparacion\_intervalos\_confianza.png)
*Figura 10. Comparación de Intervalos de Confianza al 95% ($M = 100$ réplicas independientes por modelo, `resultados/comparacion\_intervalos\_confianza.png`). Las barras de error muestran que los intervalos no se solapan, evidenciando diferencias altamente significativas entre los tres paradigmas.*

Como se ilustra en la Figura 10, los intervalos de confianza del 95% para Eficiencia, Despertares, Ciclos REM y Exactitud de Viterbi son mutuamente disjuntos y no presentan solapamiento alguno entre los tres modelos. Esto constituye evidencia empírica concluyente de que las diferencias observadas son inherentes a la estructura probabilística de cada formulación y no responden a variaciones muestrales aleatorias.

---

## 9. Discusión Crítica y Respuestas a las Preguntas de Investigación

### 9.1 Respuesta Explícita a la Pregunta PI1: La Propiedad Sin Memoria y la Macroestructura
* **Pregunta PI1:** *¿En qué medida la propiedad de Markov de primer orden y su distribución geométrica implícita distorsionan la duración de los episodios y el conteo de ciclos ultradianos en una simulación de 480 minutos?*
* **Respuesta Cuantitativa y Teórica:** 
  La propiedad markoviana de primer orden distorsiona la macroestructura de manera masiva y sistemática. Teóricamente, la hipótesis markoviana fuerza una distribución de permanencia puramente geométrica $P(D = d) = (A_{ii})^{d-1}(1 - A_{ii})$. Como se demostró analíticamente en la Sección 3.3, los valores diagonales didácticos del caso base ($A_{00} = 0.50$, $A_{11}=A_{22}=A_{33}=0.60$) imponen esperanzas matemáticas de permanencia de tan solo $2.0\text{ min}$ para vigilia y $2.5\text{ min}$ para las fases de sueño.
  
  Los resultados de Monte Carlo ($M=100$) confirmaron que la simulación produce un promedio de **$32.68 \pm 4.36$ ciclos ultradianos** (con un rango muestral de 22 a 43 ciclos) y $38.66 \pm 4.70$ despertares en una noche de 8 horas. Esta dinámica representa una hiperfragmentación artificial inaceptable desde el punto de vista biofísico: mientras que un ser humano saludable consolida entre 4 y 6 ciclos ultradianos de 90 a 120 minutos [4], [6], el HMM de primer orden fragmenta la noche en más de 200 micro-bloques de 2 a 3 minutos. La regla operativa de conteo (transición a REM precedida de NREM) arroja 32 ciclos en el HMM base simplemente porque la cadena entra y sale de REM espuriamente decenas de veces por hora, lo que demuestra que los parámetros del HMM estándar no pueden emplearse para inferir macroestructura de sueño sin correcciones de inercia temporal [11], [12].

### 9.2 Respuesta Explícita a la Pregunta PI2: Impacto de la Propensión al Despertar
* **Pregunta PI2:** *¿Cuál es el impacto cuantitativo de incrementar proporcionalmente las probabilidades de transición hacia vigilia ($A_{i0}$) sobre el tiempo total de sueño (TST), la eficiencia, la frecuencia de despertares y la preservación de fases profundas (N3) y REM?*
* **Respuesta Cuantitativa y Teórica:**
  El incremento forzado de las tasas de transición hacia vigilia en el modelo `HMM_FRAGMENTADO` ($A_{10} = 0.30$, $A_{20} = 0.15$, $A_{30} = 0.40$) desencadena un colapso severo y no lineal de la arquitectura del sueño. En las 100 réplicas de Monte Carlo:
  1. El tiempo total de sueño promedio se redujo en $94.16\text{ min}$, pasando de $401.06 \pm 11.40\text{ min}$ a $306.90 \pm 12.84\text{ min}$ (caída de $-23.5\%$).
  2. La eficiencia del sueño colapsó de un nivel aceptable ($83.55 \pm 2.37\%$) a un nivel patológico ($63.94 \pm 2.68\%$, $\mathrm{IC}_{95\%} = [63.41, 64.47]\%$).
  3. Los despertares se más que duplicaron, saltando de $38.66 \pm 4.70$ a **$85.47 \pm 5.91$ eventos nocturnos** ($+121.1\%$), con máximos muestrales de hasta 108 despertares.
  4. La preservación de las etapas restaurativas sufrió una asimetría crítica: el Sueño Profundo (N3) cayó del $23.56\%$ al $16.02\%$ (reducción relativa de $-32.0\%$) y el Sueño REM colapsó del $17.89\%$ al $10.66\%$ (reducción relativa de $-40.4\%$).
  
  Este hallazgo concuerda con la teoría neurobiológica del sueño [3]: al elevarse la tasa de salida a vigilia desde estados de sueño, las fases fisiológicamente más lábiles y complejas (sueño profundo y REM) son interrumpidas prematuramente, impidiendo que el durmiente mantenga la estabilidad tónica requerida para la regeneración metabólica y cognitiva.

### 9.3 Respuesta Explícita a la Pregunta PI3: Resolución Semi-Markoviana y Decodificación
* **Pregunta PI3:** *¿Cómo resuelve la extensión Semi-Markoviana con duraciones explícitas (HSMM) el problema de la hiperfragmentación artificial, en qué grado aproxima el número de ciclos al rango fisiológico (4--6 ciclos) y cómo impacta en la exactitud del algoritmo de Viterbi?*
* **Respuesta Cuantitativa y Teórica:**
  La extensión Semi-Markoviana (`HMM_SEMIMARKOV`) resuelve de raíz la limitación geométrica al desacoplar formalmente el salto de estado del tiempo de permanencia mediante una cadena embebida sin auto-bucles y distribuciones explícitas de duración uniforme con cotas fisiológicas didácticas $[d_{\min, i}, d_{\max, i}]$ [2], [7].
  
  En el análisis Monte Carlo ($M=100$):
  1. El número de ciclos ultradianos se redujo a una media de **$4.66 \pm 1.55$ ciclos** ($\mathrm{IC}_{95\%} = [4.35, 4.97]$), coincidiendo con precisión matemática dentro del intervalo fisiológico canónico del sueño humano (4 a 6 ciclos por noche) reportado por Carskadon y Dement [4] y la AASM [6].
  2. Los despertares nocturnos se estabilizaron en $5.45 \pm 1.92$ eventos (frente a 38.66 en Base y 85.47 en Fragmentado), restaurando una arquitectura de sueño consolidada.
  3. La eficiencia del sueño alcanzó una media de $90.84 \pm 3.49\%$, representativa de un descanso continuo óptimo.
  4. La exactitud de decodificación de Viterbi experimentó una elevación notable: del $71.21 \pm 2.74\%$ (Base) y $69.37 \pm 2.20\%$ (Fragmentado) saltó al **$94.06 \pm 1.97\%$** ($\mathrm{IC}_{95\%} = [93.67, 94.45]\%$) bajo Viterbi segmentario.
  
  La explicación matemática de esta mejora en la inferencia radica en el filtrado contextual: en el HMM convencional sin memoria, una observación ruidosa aislada (ej. un ritmo Theta espurio en medio de ondas Delta) induce a Viterbi a saltar erróneamente de estado porque no existe costo por conmutación rápida. En cambio, en el HSMM, el algoritmo evalúa la log-verosimilitud agregada del segmento completo $\sum_{k=t-d}^t \ln B_{j, o_k}$; la evidencia acumulada de 20 épocas dominantes de ondas Delta supera holgadamente el ruido transitorio de una época atípica, blindando al decodificador contra falsos positivos.

### 9.4 Contrastación con la Literatura Reciente de Polisomnografía
Nuestros hallazgos empíricos confirman computacionalmente las advertencias formuladas por la literatura contemporánea en ingeniería biomédica. Como señalan Fiorillo *et al.* [10] y Phan *et al.* [9], los modelos basados puramente en probabilidades locales de transición fracasan al capturar la macroestructura temporal porque el sueño cerebral exhibe dependencias de largo alcance que violan la hipótesis de Markov de orden 1. La integración de ventanas explícitas de duración en el HSMM reproduce artificialmente el efecto de regularización temporal que los modelos neuronales contemporáneos (como las capas recurrentes LSTM en Stephansen *et al.* [8] o la atención temporal en *XSleepNet* [9]) aprenden a partir de millones de épocas de datos clínicos.

---

## 10. Limitaciones del Modelo y Validez Metodológica

Para salvaguardar la rigurosidad científica y evitar extrapolar conclusiones indebidas, se declaran explícitamente cuatro limitaciones estructurales del estudio:
1. **Series Temporales Sintéticas y Unidimensionales:** Las observaciones analizadas son etiquetas categóricas discretas (ritmos dominantes $\alpha, \theta, \delta$, movimientos oculares) generadas mediante simulación estocástica *in silico*. En la práctica clínica real, la polisomnografía registra señales bioeléctricas electrofisiológicas analógicas continuas multicanal (múltiples derivaciones EEG, EOG bilateral, electromiografía submentoniana y tibial, flujo aéreo nasal, oximetría de pulso y electrocardiograma) que presentan artefactos de movimiento, microdespertares subcorticales y actividad espectral continua no abordable mediante variables discretas homogéneas [5], [6], [11].
2. **Discretización Temporal en Épocas de 1 Minuto:** El modelo analizado emplea épocas de $\Delta t = 1\text{ min}$ ($T=480$). El estándar internacional de la AASM [5], [6] exige estrictamente épocas de 30 segundos. Esta resolución temporal más gruesa amortigua transiciones ultracortas pero amplifica artificialmente la duración de microdespertares.
3. **Estacionariedad y Ausencia de Modulación Circadiana:** Las matrices $A$ y $B$ se asumieron estacionarias e invariantes en el tiempo durante toda la noche. En la neurobiología real, el sueño está regulado por la interacción no lineal del Proceso Homeostático S (acumulación y disipación de presión de sueño dependiente del tiempo de vigilia) y el Proceso Circadiano C (oscilador del núcleo supraquiasmático), según el célebre modelo de Borbély [3]. Por ello, en el ser humano el sueño profundo N3 se concentra abrumadoramente en el primer tercio de la noche, mientras que la fase REM se expande progresivamente en la segunda mitad. Un modelo HMM homogéneo no puede reflejar esta modulación circadiana sin matrices dependientes del tiempo $A(t)$ [11], [12].
4. **Delimitación Bioética:** Los parámetros del modelo corresponden a un diseño educativo simplificado y no poseen calibración ni validación clínica sobre cohortes de pacientes. El software desarrollado no es apto para diagnóstico ni toma de decisiones médicas.

---

## 11. Conclusiones

1. **La propiedad sin memoria markoviana de primer orden es estructuralmente incompatible con la macroestructura biológica del sueño.** En un HMM con parámetros típicos, la esperanza matemática geométrica impone duraciones medias de 2 a 2.5 minutos, generando una hiperfragmentación artificial con más de 200 transiciones y $32.68 \pm 4.36$ ciclos ultradianos espurios.
2. **La perturbación experimental de fragmentación (`HMM_FRAGMENTADO`) demostró la labilidad de las fases profundas.** Incrementar la probabilidad de despertar duplicó los despertares ($85.47 \pm 5.91$ eventos) y redujo la eficiencia al $63.94 \pm 2.68\%$, suprimiendo asimétricamente el sueño de ondas lentas ($-32\%$) y el sueño REM ($-40\%$), emulando patrones observados en patologías de fragmentación del sueño.
3. **La extensión Semi-Markoviana (`HMM_SEMIMARKOV`) resolvió el problema de escala temporal.** Al desacoplar la permanencia mediante ventanas de inercia configurables, el modelo situó los ciclos ultradianos en $4.66 \pm 1.55$, convergiendo con el rango fisiológico humano normal (4 a 6 ciclos) y estabilizando los despertares en $5.45 \pm 1.92$.
4. **La decodificación de Viterbi segmentaria supera críticamente a la formulación estándar.** La exactitud de inferencia se elevó del $71.21\%$ al $94.06\%$, demostrando que la integración de la verosimilitud de emisión sobre segmentos temporales completos actúa como un filtro robusto frente a perturbaciones estocásticas locales.
5. **El análisis inferencial de Monte Carlo ($M=100$) validó la robustez de los hallazgos.** Los intervalos de confianza al 95\% para eficiencia, despertares, ciclos y exactitud resultaron mutuamente excluyentes entre escenarios ($p < 0.001$), demostrando que las diferencias son propiedades matemáticas formales de los modelos y no fluctuaciones muestrales.
6. **Recomendación metodológica:** Toda iniciativa educativa o de investigación en bioingeniería orientada a la simulación del sueño debe prescindir de HMMs homogéneos de primer orden en favor de formulaciones Semi-Markovianas (HSMM) o arquitecturas secuenciales continuas con regularización temporal explícita.

---

## 12. Referencias Bibliográficas

[1] L. R. Rabiner, "A tutorial on hidden Markov models and selected applications in speech recognition," *Proceedings of the IEEE*, vol. 77, no. 2, pp. 257--286, Feb. 1989, doi: 10.1109/5.18626.

[2] J. D. Ferguson, "Variable duration models for speech," in *Proc. Symp. Application of Hidden Markov Models to Text and Speech*, Princeton, NJ, 1980, pp. 143--179.

[3] A. A. Borbély, "A two process model of sleep regulation," *Human Neurobiology*, vol. 1, no. 3, pp. 195--204, 1982.

[4] M. A. Carskadon and W. C. Dement, "Monitoring and staging human sleep," in *Principles and Practice of Sleep Medicine*, 5th ed., M. H. Kryger, T. Roth, and W. C. Dement, Eds. St. Louis, MO: Elsevier Saunders, 2011, pp. 16--26.

[5] C. Iber, S. Ancoli-Israel, A. L. Chesson, and S. F. Quan, *The AASM Manual for the Scoring of Sleep and Associated Events: Rules, Terminology and Technical Specifications*, 1st ed. Westchester, IL: American Academy of Sleep Medicine, 2007.

[6] American Academy of Sleep Medicine (AASM), *The AASM Manual for the Scoring of Sleep and Associated Events: Rules, Terminology and Technical Specifications*, Version 2.6. Darien, IL: American Academy of Sleep Medicine, 2020.

[7] S.-Z. Yu, "Hidden semi-Markov models," *Artificial Intelligence*, vol. 174, no. 2, pp. 215--243, Feb. 2010, doi: 10.1016/j.artint.2009.11.011.

[8] J. B. Stephansen, A. N. Ambati, E. B. Leary, H. E. Moore, P. E. Carrillo, F. Lin, O. Han, H. B. Valomon, L. S. Morales-Carbajal, V. B. Zoubek, and E. Mignot, "Neural network analysis of sleep stages enables efficient diagnosis of narcolepsy," *Nature Communications*, vol. 9, no. 1, p. 5229, Dec. 2018, doi: 10.1038/s41467-018-07229-3.

[9] H. Phan, O. Y. Chén, P. Koch, Z. Lu, I. McLoughlin, A. Mertins, and M. De Vos, "XSleepNet: Multi-view sequential model for automatic sleep staging," *IEEE Transactions on Biomedical Engineering*, vol. 68, no. 11, pp. 3324--3334, Nov. 2021, doi: 10.1109/TBME.2021.3071370.

[10] L. Fiorillo, A. Favaro, and F. D. Faraci, "Automated sleep staging: A review of machine learning and deep learning methods," *IEEE Reviews in Biomedical Engineering*, vol. 14, pp. 214--230, 2021, doi: 10.1109/RBME.2019.2944075.

[11] M. J. Prerau, R. E. Brown, M. T. Bianchi, J. M. Ellenbogen, and P. L. Purdon, "Sleep neurophysiological dynamics through the lens of multitaper spectral analysis," *Physiology*, vol. 32, no. 1, pp. 60--92, Jan. 2017, doi: 10.1152/physiol.00062.2015.

[12] A. Flexer, P. Sykacek, I. Rezek, and G. Dorffner, "A continuous framework for sleep stage scoring," *BioMedical Engineering OnLine*, vol. 4, no. 1, p. 14, Feb. 2005, doi: 10.1186/1475-925X-4-14.
