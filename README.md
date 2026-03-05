# LAB 2 – Estimación del nivel de estrés basada en GSR (EDA)

Este repositorio desarrolla el Laboratorio 2, cuyo objetivo fue diseñar e implementar un biosensor capaz de medir y analizar la respuesta galvánica cutánea (GSR/EDA), con el fin de estimar un nivel de estrés en tiempo real. Los datos adquiridos se transmiten de forma inalámbrica hacia MATLAB para su visualización y procesamiento.

## Integrantes
- Jhonathan David Guevara
- Juan Pablo Díaz 
- María Paula Fernández

## 1. Resumen
Se construyó un sensor de GSR usando dos electrodos metálicos (monedas) en contacto con la piel, un circuito de acondicionamiento básico con resistencia limitadora y un ESP32 para adquisición. La señal se muestrea con el ADC del ESP32 (ADC1), se transmite de forma inalámbrica a MATLAB y se visualiza en tiempo real. La estimación del nivel de estrés se plantea mediante umbrales calibrados por sujeto a partir de una prueba de inspiración profunda.

## 2. Fundamento teórico (EDA / GSR)
La actividad electrodérmica (EDA) describe variaciones en las propiedades eléctricas de la piel relacionadas con la activación del sistema nervioso simpático, principalmente por cambios en la sudoración. Esto cambia la conductancia de la piel, conocida como respuesta galvánica cutánea (GSR).

Componentes típicas:
- SCL (Skin Conductance Level): componente lenta o basal (tendencia).
- SCR (Skin Conductance Response): respuestas transitorias asociadas a estímulos (incremento rápido con retorno más lento).

En general, la activación simpática incrementa la conductancia de la piel y puede reflejarse como aumentos en SCL y/o aparición de SCR.

## 3. Objetivos

### 3.1 Objetivo general
Implementar un sistema continuo de medición de GSR para estimar el nivel de estrés.

### 3.2 Objetivos específicos
- Adquirir la señal GSR con un sistema vestible estable.
- Garantizar seguridad del usuario limitando la corriente a través de la piel.
- Transmitir la señal de forma inalámbrica y visualizarla en MATLAB en tiempo real.
- Proponer un criterio de clasificación (bajo/medio/alto) con umbrales por sujeto.
- Comparar el comportamiento de la señal en reposo, respiración profunda, movimiento y tarea cognitiva.

## 4. Materiales y herramientas

### 4.1 Hardware
- ESP32 (ADC1 para adquisición)
- 2 electrodos metálicos (monedas)
- Resistencia: 100 kΩ
- Capacitor: 5 µF
- Cables, protoboard o montaje equivalente
- Elementos de sujeción (cinta/velcro/elástico)

### 4.2 Software
- MATLAB (lectura del flujo de datos, graficación y análisis)

## 5. Seguridad eléctrica
La seguridad se evalúa con el caso extremo de resistencia de piel cercana a 0 Ω, donde la corriente máxima queda limitada por la resistencia serie:

$$
I_{\text{max}}=\frac{V}{R}=\frac{3.3\,\text{V}}{100\,\text{k}\Omega}=3.3\times10^{-5}\,\text{A}=0.033\,\text{mA}
$$

Este valor es inferior a 1 mA, cumpliendo el criterio de seguridad planteado para el laboratorio.

## 6. Diseño del circuito y acondicionamiento

### 6.1 Conversión de resistencia de piel a voltaje
El sistema se basa en un divisor resistivo donde los cambios de \(R_{skin}\) modifican el voltaje en el nodo de medición leído por el ADC del ESP32.

Esquema conceptual:
- 3.3 V → R (100 kΩ) → nodo (ADC1) → R_skin (piel entre electrodos) → GND
- C (5 µF) desde el nodo (ADC1) a GND

### 6.2 Comportamiento del filtro RC
Con \(R = 100\,\text{k}\Omega\) y \(C = 5\,\mu\text{F}\) \(\left(=5\times10^{-6}\,\text{F}\right)\):

$$
\tau = R\,C = (100\,000)\,(5\times10^{-6}) = 5\times10^{-1}\,\text{s} = 0.5\,\text{s}
$$

$$
f_c = \frac{1}{2\pi \tau} \approx \frac{1}{2\pi(0.5)} \approx 0.318\,\text{Hz}
$$

Con estos valores, el capacitor implementa un **filtro pasa-bajas** que ayuda a suavizar ruido y variaciones rápidas, dejando principalmente la componente lenta típica de GSR (SCL). La separación SCL/SCR puede realizarse adicionalmente mediante filtrado digital en MATLAB (promedio móvil o filtros pasa-bajas/pasa-altas suaves).

## 7. Región anatómica y colocación de electrodos
Se utilizaron dedos (falanges distales) por su buena respuesta electrodérmica y facilidad de montaje. Para reducir artefactos:
- Asegurar contacto uniforme sin presión excesiva.
- Minimizar movimiento de los cables.
- Mantener la piel limpia y seca antes de la medición.

## 8. Adquisición de datos

### 8.1 Configuración de muestreo
- ADC: ADC1 del ESP32
- Frecuencia de muestreo: 100 Hz
- Transmisión: inalámbrica hacia MATLAB

### 8.2 Datos transmitidos
- Valor ADC crudo o voltaje equivalente

## 9. Procedimiento experimental

### 9.1 Parte A – Preparación y diseño
1. Revisión conceptual: EDA, SCL y SCR.
2. Cálculo de seguridad considerando \(R_{\text{skin}} \approx 0\,\Omega\).
3. Ensamble del circuito y verificación de lectura estable del ADC en reposo.
4. Montaje vestible y prueba de estabilidad mecánica (fijación de electrodos).

### 9.2 Parte B – Adquisición, umbrales y validación
1. Registro en reposo (baseline).
2. Prueba de inspiración profunda y exhalación lenta para observar SCR.
3. Registro durante movimiento para identificar artefactos.
4. Visualización en MATLAB en tiempo real y almacenamiento de la señal.

### 9.3 Parte C – Estrés por tarea cognitiva
1. Registro mientras el sujeto realiza una actividad cognitiva breve.
2. Comparación con reposo: cambios en SCL y presencia/frecuencia de SCR.

## 10. Estimación del nivel de estrés (umbrales)

### 10.1 Variables de calibración
- **Vmin:** valor mínimo típico en reposo  
- **Vmax:** valor máximo durante la respuesta por inspiración profunda  

### 10.2 Regla propuesta por rango
- **Bajo:** 
- **Medio:**
- **Alto:** 

## 11. Visualización y análisis en MATLAB
Flujo de trabajo:
1. Conexión inalámbrica y lectura continua de muestras.
2. Graficación en tiempo real de la señal.
3. Filtrado digital:
   - Promedio móvil (suavizado)
   - Pasa-bajas para resaltar SCL
   - Derivada o pasa-altas suave para resaltar SCR
4. Cálculo del nivel (bajo/medio/alto) por ventanas temporales.

## 12. Resultados
- Señal en reposo (baseline).

- Respuesta durante inspiración profunda (SCR visible).
  
- Señal con movimiento (artefactos y estabilidad del montaje).
  
- Señal durante tarea cognitiva (variación de SCL y/o aumento de SCR).
  
- Señal del nivel de estrés (bajo/medio/alto) versus tiempo.

## 13. Análisis

### 13.1 Análisis 1 – Respiración en reposo vs verbalización
Durante verbalización o una tarea demandante, el patrón respiratorio puede cambiar (variación en ritmo y profundidad) y se puede incrementar la activación simpática. Esto suele reflejarse en GSR como incremento del nivel basal (SCL) y/o mayor ocurrencia/amplitud de respuestas transitorias (SCR).

### 13.2 Análisis 2 – Alcance y limitaciones para detección de patologías
La GSR es un indicador autonómico no invasivo útil para estudiar activación simpática y respuesta a estímulos. Sin embargo, tiene baja especificidad: temperatura ambiente, hidratación, presión de electrodos, movimiento, estrés basal y otros factores pueden alterar la señal. Por sí sola no permite diagnóstico clínico, pero puede ser complementaria en estudios fisiológicos.

## 14. Preguntas propuestas

### 14.1 ¿Por qué una inspiración profunda incrementa la GSR?
Una inspiración profunda puede inducir una activación autonómica transitoria y cambios en la sudoración, aumentando la conductancia de la piel. Esto se observa como una respuesta SCR: aumento rápido y retorno lento hacia el nivel basal.

### 14.2 Ventajas y desventajas de GSR como indicador de estrés
Ventajas:
- Medición no invasiva y de bajo costo.
- Alta sensibilidad a cambios simpáticos.
- Útil para comparar condiciones (reposo vs tarea).

Desventajas:
- No es específica para “estrés” (muchos factores influyen).
- Susceptible a artefactos por movimiento y contacto.
- Requiere calibración por sujeto y control experimental.

## 16. Referencias
- Boucsein, W. Electrodermal Activity. Springer, 2012.
- Figner, B., & Murphy, R. O. (2011). Using Skin Conductance in Judgment and Decision Making Research. En A Handbook of Process Tracing Methods for Decision Research.
- Loggia, M. L., Juneau, M., & Bushnell, C. M. (2011). Autonomic responses and pain/activation relationships. Pain, 152(3), 592–598.
- Breimhorst, M. et al. (2011). Electrodermal measures and stress-related responses. The Journal of Pain, 12(1), 61–70.
