# M12 · ¿Este envió internacional llega tarde?

**Integrantes:** Valentin Galeano · Emanuel Xie
**IA declarada:** Gemini
**Pista:** Reglas
**Publicación en LinkedIn:** https://www.linkedin.com/in/valentin-galeano-183064321/

---

## 1. Formulación y PEAS

**La pregunta.** Para cada envió internacional, ¿llegara tarde al cliente final? 
SÍ significa: El envió sufre demoras encadenadas (efecto mariposa) y sobrepasa la fecha prometida de entrega al cliente final
NO significa: El envió llega a tiempo o absorbe los pequeños imprevistos dentro de sus días de holgura
| PEAS | Clasificador de reglas logicas para alertas operativas de envios internacionales |
|---|---|
| Medida de rendimiento | Minimización de falsos negativos (evitar penalizaciones por clientes no avisados) manteniendo un equilibrio con los Falsos Positivos (evitar gastos necesarios de flete urgente |
| Entorno | Sistema de gestión de envíos internacionales (ERP/TMS), tracking de puertos globales y registros de depósito |
| Actuadores | Alerta al área de Operaciones para contratar fletes urgentes o reconfigurar la ruta de transbordo |
| Sensores | Datos de reserva de carga, número de escalas, índice de congestión portuaria en origen/transbordo y marcas de tiempo de salida del depósito |

**Objetivo y métrica.** 
Lo que importa: Predecir si las pequeñas demoras (como escalas o congestión portuaria) se amplificarán y causarán un retraso final, evitando penalizaciones por reclamos de clientes

Métrica principal: Sensibilidad (Recall), para minimizar Falsos Negativos

¿Son lo mismo?: No. La métrica que usa la gerencia (% de envíos despachados a tiempo del depósito) mide solo el inicio, mientras que la métrica de nuestro sistema evalúa si el envío llega tarde al final de toda la cadena internacional. Un paquete puede salir a tiempo del depósito (dato de gerencia = 1) y de todas formas llegar tarde por el efecto mariposa de la congestión y transbordos (resultado real = 1).


## 2. El sistema

**Cómo decide.** El sistema decide aplicando una serie de Reglas Si-Entonces evaluadas en orden secuencial estricto. Si se cumple la condición de una regla, se emite la salida SÍ y finaliza la evaluación; si ninguna regla se cumple, el sistema emite por defecto la salida NO.

REGLA 1: Si Despachado a tiempo = 0 Y Días de holgura ≤ 5 → Salida = SÍ

REGLA 2: Si Escalas ≥ 3 Y Congestión portuaria ≥ 7 → Salida = SÍ

REGLA 3: Si Días de holgura ≤ 2 → Salida = SÍ

REGLA 4 (Por defecto): En cualquier otro caso → Salida = NO

**Variables que usa y por qué.** 
- Despachado a tiempo del depósito (0 o 1): Se conoce al momento de tomar la decisión, cuando el contenedor sale del depósito de origen para iniciar el tramo internacional. Es el detonante inicial de la cadena de retrasos.
- Días de holgura (cantidad de días): Margen de maniobra disponible. Una holgura ≤ 5, combinada con demoras iniciales, reduce la capacidad de absorción del sistema.
- Escalas (cantidad de transbordos) y Congestión portuaria (0 a 10): Variables que amplifican las demoras intermedias (efecto mariposa).
- Variables descartadas: Ninguna. Se utilizaron las 4 variables del dataset para capturar las interacciones clave sin sobredimensionar el modelo.

**Cómo se corre en vivo.** 
1. Se registra la salida del depósito: si el envío no salió a tiempo (Despachado = 0) y tiene Holgura ≤ 5, se emite SÍ inmediatamente para contratar flete de auxilio.
2. Si no cumple la regla 1, se verifica si atraviesa ≥ 3 escalas con congestión portuaria ≥ 7. De ser así, se emite SÍ.
3. Si tampoco cumple la regla 2, se verifica si le quedan ≤ 2 días de holgura. Si cumple, se emite SÍ.
4. Si no se activa ninguna de las condiciones anteriores, la salida es NO.

**Dos casos de ejemplo.**
| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| Caso A | Escalas: 4, Holgura: 1, Congestión: 8, Despachado a tiempo: 1 | SÍ |
| Caso B | Escalas: 2, Holgura: 8, Congestión: 3, Despachado a tiempo: 1 | NO |

## 3. Medición

Caso 1: (Esc=1, Holg=9, Cong=4, Desp=0) -> Predicción: NO | Real: NO (VN)
Caso 2: (Esc=2, Holg=8, Cong=2, Desp=1) -> Predicción: NO | Real: NO (VN)
Caso 3: (Esc=3, Holg=9, Cong=8, Desp=1) -> Predicción: SÍ (Cumple Regla 2: Escalas mayor o igual a 3 y Congestión mayor o igual a 7) | Real: NO (FP)
Caso 4: (Esc=2, Holg=6, Cong=9, Desp=0) -> Predicción: NO (Holgura 6 es mayor a 5, no cumple ninguna regla) | Real: NO (VN)
Caso 5: (Esc=4, Holg=7, Cong=5, Desp=0) -> Predicción: NO (Holgura es 7 y Congestión es 5, no activa reglas) | Real: SÍ (FN)
Caso 6: (Esc=4, Holg=10, Cong=8, Desp=1) -> Predicción: SÍ (Cumple Regla 2) | Real: SÍ (VP)
Caso 7: (Esc=1, Holg=3, Cong=2, Desp=1) -> Predicción: NO | Real: NO (VN)
Caso 8: (Esc=5, Holg=4, Cong=1, Desp=1) -> Predicción: NO (Congestión es 1 y Holgura es 4, no activa reglas) | Real: SÍ (FN)
Caso 9: (Esc=1, Holg=2, Cong=9, Desp=1) -> Predicción: SÍ (Cumple Regla 3: Holgura menor o igual a 2) | Real: SÍ (VP)
Caso 10: (Esc=3, Holg=5, Cong=7, Desp=1) -> Predicción: SÍ (Cumple Regla 2) | Real: NO (FP)
Caso 11: (Esc=3, Holg=6, Cong=9, Desp=1) -> Predicción: SÍ (Cumple Regla 2) | Real: SÍ (VP)
Caso 12: (Esc=3, Holg=1, Cong=8, Desp=1) -> Predicción: SÍ (Cumple Regla 2 y Regla 3) | Real: SÍ (VP)
Caso 13: (Esc=3, Holg=2, Cong=6, Desp=0) -> Predicción: SÍ (Cumple Regla 1 y Regla 3) | Real: SÍ (VP)
Caso 14: (Esc=2, Holg=5, Cong=5, Desp=0) -> Predicción: SÍ (Cumple Regla 1: Despachado = 0 y Holgura menor o igual a 5) | Real: NO (FP)

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 5 | FN = 2 |
| **Real NO** | FP = 3 | VN = 4 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 3 | FN = 0 |
| **Real NO** | FP = 0 | VN = 3 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 9/14 = 64.3% | 6/6 = 100%  |
| Precisión | 5/8 = 62.5%| 3/3 = 100% |
| Sensibilidad | 5/7 = 71.4% | 3/3 = 100% |
| Especificidad | 4/7 = 57.1% | 3/3 = 100% |

**Entrenamiento contra prueba.**
En entrenamiento el sistema alcanza una exactitud del 64.3% (9 de 14 aciertos) con una sensibilidad del 71.4%. En prueba obtiene un 100.0% de aciertos (6 de 6). Esta diferencia no implica sobreajuste ni una generalización perfecta comprobada; responde simplemente al tamaño reducido de la muestra de prueba (6 casos). Sin embargo, confirma que en datos no vistos el sistema no generó Falsos Negativos.

## 4. El error más caro
En este problema el peor error es el Falso Negativo (FN).
¿Por qué? Si el sistema predice NO y el envío llega tarde, el cliente experimenta la demora sin aviso, resultando en penalizaciones económicas y pérdida de confianza. En cambio, un Falso Positivo (FP) solo provoca notificar una demora preventiva y contratar un transporte urgente adicional, costo menor que la pérdida del contrato.
Qué cambiamos: En el diseño inicial del sistema, las reglas estrictas generaban 3 Falsos Negativos en entrenamiento (FN = 3). Con el ajuste de los umbrales (Regla 1 con holgura menor o igual a 5 cuando Despachado = 0, y Regla 3 con holgura menor o igual a 2), los Falsos Negativos bajaron de 3 a 2 en entrenamiento (Casos 5 y 8) y a 0 en prueba, priorizando la captura de las demoras reales

## 5. ¿Comparado contra qué?

**Línea base trivial.**
En entrenamiento hay 7 casos SÍ y 7 casos NO (distribución 50/50).
Un sistema trivial que responda siempre NO obtiene una Exactitud del 50.0% (7 de 14) y una Sensibilidad del 0.0% (falla en detectar todas las demoras).
Nuestro sistema alcanza una Exactitud del 64.3% en entrenamiento (un +14.3% por encima de la línea base trivial) y un 100.0% en prueba, logrando una Sensibilidad del 71.4% en entrenamiento y 100.0% en prueba.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 registros reales de envíos internacionales (14 entrenamiento, 6 prueba) con variables de escalas, holgura, congestión y despacho a tiempo.|
| ¿Qué métrica? | Sensibilidad como métrica principal (capturar demoras) y Exactitud como métrica de control general. |
| ¿Comparado contra qué? | Contra una línea base trivial de respuesta fija (responder siempre NO), que alcanza 50% de exactitud y 0% de sensibilidad. |
| ¿Quién lo midió? | Medición directa realizada por el equipo de estudiantes y Gemini evaluando las reglas caso por caso sobre la partición del dataset M12. |

## 6. Riesgo: el camino completo
Flujo: Registro de congestión desactualizado --> Regla clasifica envío como NO tardío --> Operaciones no reserva flete urgente de respaldo -->El transbordo se pierde-->Penalización contractual y pérdida del cliente.

Riesgo concreto: Que un puerto declare un índice de congestión bajo (ej. 2), pero ocurre una huelga sorpresiva durante el tránsito del buque. La holgura baja (ej. 3 días) no será suficiente para absorber el imprevisto. El modelo habrá devuelto un NO (Falso Negativo), impidiendo actuar preventivamente y desatando el reclamo de penalidad contractual por parte del cliente final.




## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Analizar el dataset de los datos y realizar las consignas del hackaton | Su primer respuesta inicial | Rechazamos su propuesta inicial de realizar una regresion logistica compleja porque con reglas logicas claras se lograba un 100% de efectividad |
|2|Que calcule las métricas de exactitud, precisión, sensibilidad y especificidad.|Las fórmulas matemáticas de clasificación.|Rechazamos el primer conteo automatizado porque alteró los datos originales de los casos 2, 3 y 8.|
|3|Que formule la comparación contra modelos estadísticos o de regresión. |La fundamentación teórica sobre el impacto del Falso Negativo.|Rechazamos implementar algoritmos de regresión logística o redes neuronales porque la consigna requiere un sistema declarativo explicable basado en reglas.|


**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 7 | Operadores no definidos explícitamente; inconsistencia en matriz de entrenamiento (casos 4 y 14); falta de cuantificación del descenso de FN; comparación basada en el indicador de gerencia y no en la línea base trivial; "Quién lo midió" sin responder y bitácora incompleta.| Explicitamos los operadores (mayor o igual, menor o igual, igual) y el orden estricto de reglas; recalculamos la matriz caso por caso (Caso 4 = VN, Caso 14 = FP); documentamos la reducción de FN de 3 a 1; incorporamos la línea base trivial del 50%; respondimos la verificación de medición |
|2| 7 | Inconsistencia en los valores de los casos 2, 3 y 8 respecto al dataset original; conteo de matriz de entrenamiento incorrecto; cálculo de ventaja contra la línea base distorsionado (+21.4%); número de descenso de FN mal afirmado (3 a 1); justificación vaga en el rechazo a la regresión.|Corregimos las filas de los casos 2, 3 y 8 con los valores reales del dataset; recalculamos la matriz de entrenamiento exacta (VP=5, FN=2, FP=3, VN=4); ajustamos la exactitud a 64.3% (+14.3% sobre la línea base); corregimos el descenso de FN a 3 a 2 en entrenamiento; y reescribimos el rechazo de modelos complejos fundándolo en la exigencia de reglas declarativas.|
|3| 9 |Cuando el envío sale del depósito, ¿de dónde sale el índice de congestión del puerto de transbordo? Si todavía no existe ese dato al decidir, ¿qué le pasa a la regla 2?|Nada|
