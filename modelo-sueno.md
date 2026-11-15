# Ejemplo 2 – Análisis de Etapas del Sueño (Polisomnografía)

**Fuente de parámetros:** archivo HTML del usuario, *HMM Simulations - 10 Nuevos Ejemplos*, Ejemplo 2.  
**Uso:** transcripción estructurada del prompt de sueño. Los parámetros se mantienen tal como aparecen en el material.

## Contexto
Sistema que identifica etapas del sueño a partir de señales cerebrales observadas.

- **Estados ocultos:** Despierto, Sueño Ligero, Sueño Profundo, REM.
- **Observaciones:** Ondas Alpha, Ondas Theta, Ondas Delta, Movimientos Oculares Rápidos.
- **Simulación:** 480 épocas, 1 minuto por época, 8 horas.

## Matriz de transición A
Filas = etapa actual; columnas = etapa siguiente. Orden: Despierto, Ligero, Profundo, REM.

| Estado | Despierto | Ligero | Profundo | REM |
|---|---:|---:|---:|---:|
| Despierto | 0.50 | 0.40 | 0.05 | 0.05 |
| Sueño Ligero | 0.10 | 0.60 | 0.20 | 0.10 |
| Sueño Profundo | 0.02 | 0.28 | 0.60 | 0.10 |
| REM | 0.20 | 0.20 | 0.00 | 0.60 |

## Matriz de emisión B
Filas = etapa; columnas = Alpha, Theta, Delta, Movimientos Oculares Rápidos.

| Estado | Alpha | Theta | Delta | Mov. oculares rápidos |
|---|---:|---:|---:|---:|
| Despierto | 0.70 | 0.20 | 0.05 | 0.05 |
| Sueño Ligero | 0.15 | 0.65 | 0.15 | 0.05 |
| Sueño Profundo | 0.02 | 0.13 | 0.80 | 0.05 |
| REM | 0.10 | 0.10 | 0.10 | 0.70 |

## Distribución inicial π
[Despierto: 0.50, Sueño Ligero: 0.30, Sueño Profundo: 0.10, REM: 0.10]

## Requisitos del prompt original
1. Implementar clase HMM para análisis del sueño.
2. Simular noche de 480 épocas.
3. Visualizar hipnograma, señales EEG simuladas y fases con colores: azul, verde, morado, rojo respectivamente.
4. Calcular eficiencia del sueño, latencia, tiempo/porcentaje por etapa, ciclos completos y despertares.
5. Implementar Viterbi para clasificar etapas desde señales.
6. Comparar hipnograma latente simulado vs inferido y precisión del clasificador.
7. Detectar alteraciones simuladas: sueño fragmentado o ausencia de REM.
8. Generar informe **educativo de simulación**, no diagnóstico.
9. Visualizar matriz de transición como heatmap.

## Advertencias metodológicas
- Estas matrices son **didácticas e hipotéticas**, no parámetros médicos validados.
- Los símbolos EEG categóricos son observaciones simuladas, no señales EEG físicas ni mediciones de polisomnografía.
- “Ciclos completos” requiere definir operacionalmente cómo se cuentan; el código debe declarar su criterio.
- La latencia y eficiencia calculadas son métricas de esta secuencia simulada, no de una persona.
- No usar “diagnóstico probable” clínico. Sustituir por descripción de patrones simulados y limitaciones.
