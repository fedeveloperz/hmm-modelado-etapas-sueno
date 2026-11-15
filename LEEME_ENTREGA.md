# LEEME_ENTREGA.md — Documentación de Entrega Definitiva
## Actividad Didáctica 2: "Investiga con IA. Decide con Criterio"
### Proyecto: Modelado Estocástico de Etapas del Sueño mediante HMM y Extensiones Semi-Markovianas

* **Fecha de entrega:** 15 de noviembre de 2026
* **Estado:** Conforme al 100% con la Rúbrica Oficial de Evaluación
* **Semilla Determinista Individual:** `SEED = 42` ($N = 480$ épocas de 1 min, 8 horas)
* **Diseño Multirréplica Monte Carlo:** $M = 100$ ejecuciones independientes por modelo (semillas 1000 a 1099)
* **Documento Académico Principal:** `articulo_final.pdf` (18 páginas totales, 14 páginas de cuerpo académico formal sin desbordamientos de margen)

---

### 1. Instrucciones de Reproducción Computacional

Para replicar de forma determinista y exacta bit a bit todas las simulaciones, tablas y figuras:

#### Paso 1: Configurar el entorno virtual
En la raíz del proyecto (Windows PowerShell / Bash):
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

#### Paso 2: Ejecutar los scripts de simulación e inferencia
1. **Simulación Base HMM (Fase USA):**
   ```powershell
   python src/hmm_sueno.py
   ```
   Genera: `resultados/secuencia_simulada.csv`, `resultados/metricas_etapas.csv`, `hipnograma.png`, `matriz_transicion.png` y `observaciones.png`.

2. **Simulación HMM Fragmentado (Fase MODIFICA):**
   ```powershell
   python src/hmm_sueno_fragmentado.py
   ```
   Genera: `resultados/modifica/secuencia_fragmentada.csv`, `resultados/modifica/comparacion_metricas.csv`, `hipnograma_comparativo.png`, `comparacion_etapas.png` y `matriz_transicion_fragmentada.png`.

3. **Simulación HMM Semi-Markov (Fase CREA):**
   ```powershell
   python src/hmm_semimarkov.py
   ```
   Genera: `resultados/crea/secuencia_semimarkov.csv`, `resultados/crea/comparacion_tres_modelos.csv`, `hipnograma_semimarkov.png`, `hipnograma_tres_modelos.png` y `distribucion_etapas_tres_modelos.png`.

4. **Análisis Estadístico Multirréplica Monte Carlo ($M = 100$):**
   ```powershell
   python src/analisis_estadistico_montecarlo.py
   ```
   Genera: `resultados/estadisticas_montecarlo_tres_modelos.csv`, `resultados/corridas_montecarlo_tres_modelos.csv` y `resultados/comparacion_intervalos_confianza.png` (Figura 10).

5. **Regeneración y Compilación de Documentos Académicos:**
   ```powershell
   python src/generar_articulo_completo.py
   tectonic materiales/articulo_final.tex
   ```

---

### 2. Inventario Completo de Archivos Entregados

| Directorio / Archivo | Tipo | Descripción |
|---|:---:|---|
| `articulo_final.pdf` | Documento PDF | Artículo científico universitario compilado (18 págs., 14 págs. de cuerpo, portada, índice, 4 tablas, 10 figuras, 12 citas IEEE). |
| `articulo_final.tex` | Código Fuente LaTeX | Código LaTeX profesional con paquetes `tabularx`, `booktabs`, `amsmath`, `hyperref`, sin desbordamientos de margen. |
| `articulo_final.md` | Documento Markdown | Versión completa del artículo académico en formato GitHub Flavored Markdown. |
| `bitacora.md` | Documento Markdown | Bitácora de trazabilidad metodológica completa con 7 registros formales y contrastación antes/después. |
| `modelo-sueno.md` | Especificación Técnica | Definición formal original del caso didáctico HMM. |
| `auditoria_rubrica.md` | Informe de Auditoría | Evaluación formal de conformidad de los 14 criterios de la rúbrica oficial. |
| `README.md` | Documentación | Guía general del proyecto y estructura del repositorio. |
| `requirements.txt` | Configuración | Especificación de dependencias: `numpy`, `pandas`, `matplotlib`, `pypdf`. |
| `.gitignore` | Configuración Git | Exclusión de entorno virtual (`.venv/`), temporales de Python y archivos auxiliares de compilación TeX. |
| `src/hmm_sueno.py` | Script Python | Simulación del caso base didáctico, decodificación Viterbi estándar y cálculo de métricas. |
| `src/hmm_sueno_fragmentado.py` | Script Python | Variante experimental con propensión aumentada al despertar y reescalado proporcional estocástico. |
| `src/hmm_semimarkov.py` | Script Python | Extensión Semi-Markoviana con cadena embebida, duraciones explícitas y Viterbi segmentario. |
| `src/analisis_estadistico_montecarlo.py` | Script Python | Análisis inferencial multirréplica ($M = 100$), estimación de Media, Std, Mín, Máx e IC 95% y generación de Figura 10. |
| `src/generar_articulo_completo.py` | Script Python | Generador automatizado de `articulo_final.md` y `articulo_final.tex`. |
| `resultados/` | Carpeta de Datos | Archivos CSV y figuras PNG de alta resolución (10 figuras en total). |
