# M12 · ¿Este envió internacional llega tarde?

**Integrantes:** Valentin Galeano · Emanuel Xie
**IA declarada:** Gemini
**Pista:** <Neurona / Reglas / Agente>
**Publicación en LinkedIn:** <link>

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

**Cómo decide.** Reglas: Definimos una regla basada en la interacción de las variables de congestión y escalas frente a los días de holgura:
REGLA 1: Si Despachado a tiempo del depósito = 0 Y Días de holgura 7, entonces SÍ.
REGLA 2: Si Escalas 3 Y Congestión portuaria 5 Y Días de holgura 7, entonces SÍ.
REGLA 3: Si Congestión portuaria 8 Y Días de holgura 4, entonces SÍ.
REGLA 4: En cualquier otro caso, la Salida es NO.


**Variables que usa y por qué.** 
Escalas (transbordos): Cada escala añade un punto de falla donde pequeños retrasos se acumulan (efecto mariposa).
Días de holgura: El "colchón" del envío para absorber imprevistos.
Congestión portuaria (0 a 10): Multiplicador de demoras en la ruta.
Despachado a tiempo del depósito (0/1): Indica si el envío ya arranca con retraso desde el origen.


**Cómo se corre en vivo.** 
1. Para un nuevo envío se revisan los 4 atributos.
2. Se evalúan las Reglas 1, 2 y 3 en orden.
3. Si el envío cumple cualquiera de los criterios de riesgo combinados con baja holgura, el sistema devuelve SÍ (llegará tarde). De lo contrario, emite NO.


**Dos casos de ejemplo.**
| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| Caso A | Escalas: 4, Holgura: 3, Congestión: 6, Despachado a tiempo: 0 | SÍ |
| Caso B | Escalas: 1, Holgura: 9, Congestión: 4, Despachado a tiempo: 1 | NO |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 7 | FN = 0 |
| **Real NO** | FP = 0 | VN = 7 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 3 | FN = 0 |
| **Real NO** | FP = 0 | VN = 3 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 14/14 = 100% | 6/6 = 100%  |
| Precisión | 7/7 = 100% | 3/3 = 100% |
| Sensibilidad | 7/7 = 100% | 3/3 = 100% |
| Especificidad | 7/7 = 100% | 3/3 = 100% |

**Entrenamiento contra prueba.**
En ambas fases el modelo alcanzó una métrica perfecta (100%). No hay evidencia de sobreajuste (overfitting) porque la regla formulada responde directamente a la lógica física del problema (la relación entre amplificadores de retraso y los días de holgura disponible). La capacidad de generalización en los casos 15–20 se mantuvo intacta.

## 4. El error más caro
En esta operación el Falso Negativo (FN) es el error más caro.
¿Por qué? Si el sistema da un Falso Negativo (dice NO y era SÍ), el área de operaciones confía en que el envío llegará a tiempo y no toma ninguna medida. El envío sufre la demora encadenada, el cliente recibe el paquete tarde, interrumpe su producción y cobra la penalidad contractual (además del daño reputacional). En cambio, un Falso Positivo (FP) solo provoca contratar un flete urgente preventivo de más, cuyo costo es acotado y asumible frente al valor del cliente.

Qué cambiamos: Se diseñaron las reglas de decisión priorizando los días de holgura como filtro restrictivo (evaluando 7 días como zona de riesgo si existen escalas o congestión), asegurando que ningún caso con riesgo de efecto mariposa quede clasificado como NO.


## 5. ¿Comparado contra qué?

**Línea base trivial.**
Si utilizáramos una línea base basada en el dato que mira la gerencia ("Despachado a tiempo del depósito"):
Predecir que todo lo que sale a tiempo NO llegará tarde:
En el dataset de entrenamiento (14 casos), 9 salieron a tiempo del depósito. De esos 9, 4 llegaron tarde igual (casos 6, 8, 9 y 12) por efecto mariposa en las escalas y puertos.
La exactitud de esa regla simple sería de solo 10 / 14 = 71.4% y dejaría pasar 4 Falsos Negativos graves.
Nuestro sistema (100% de exactitud y 0% FN) demuestra que mirar solo el inicio de la cadena no alcanza para prever demoras internacionales.


| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 20 envíos internacionales del dataset M12 con 4 características por envío.|
| ¿Qué métrica? | Sensibilidad (para capturar demoras) y Exactitud global. |
| ¿Comparado contra qué? | Contra el indicador gerencial simple ("Despachado a tiempo del depósito") |
| ¿Quién lo midió? | El área de Operaciones utilizó la división entre 14 casos de entrenamiento y 6 de prueba. |

## 6. Riesgo: el camino completo

<Datos → modelo → decisión → consecuencia aplicado a su caso. Al menos un riesgo concreto.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | <> | <> | <> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <> | <> | <> |
