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

**Cómo decide.** <Neurona: pesos y sesgo. Reglas: la lista de reglas si‑entonces. Agente: la regla con la que elige la acción.>

**Variables que usa y por qué.** <Cuáles usa, cuáles descarta y por qué.>

**Cómo se corre en vivo.** <Pasos concretos para resolver un caso nuevo: link al Colab, a la planilla o la cuenta a mano.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <…> | <SÍ/NO> |
| B | <…> | <SÍ/NO> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <> | <> |
| Precisión | <> | <> |
| Sensibilidad | <> | <> |
| Especificidad | <> | <> |

**Entrenamiento contra prueba.** <¿Hay diferencia? ¿Por qué? ¿Sobreajuste o generalización?>

## 4. El error más caro

<¿Es peor el falso positivo o el falso negativo en su problema? ¿Por qué, según la ficha? ¿Qué cambiaron en el sistema para cometer menos ese error?>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Qué resultado saca un sistema que responde siempre lo mismo, y cómo se compara el suyo.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <> |
| ¿Qué métrica? | <> |
| ¿Comparado contra qué? | <> |
| ¿Quién lo midió? | <> |

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
