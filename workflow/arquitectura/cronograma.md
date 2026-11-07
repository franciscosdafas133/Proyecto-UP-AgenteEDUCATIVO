# Cronograma — Arquitectura (7 días)

> Depende de: producto, antecedentes, requisitos, sistema, verificacion  
> Estado: aprobado por el usuario (2026-11-07). Fecha de inicio: (por definir)  
> Versión compartible: [documento del cronograma](https://claude.ai/artifact/1HUavyedVxkH4fWefTTrwW). Si se cambia algo, actualizar ambos.

**Objetivo de la semana:** tener el recuperador HippoRAG 2 funcionando en español, fiel al paper, y una medición de si **indexa bien** y **recupera bien**.

**Reglas del cronograma:**

- Cada día termina con una **puerta de salida**. Si no se cumple, el día siguiente no empieza; se repite lo que falta.
- Toda desviación del método se registra y la aprueba el usuario (ver `producto.md`).
- Las duraciones son orientativas, para una jornada de unas 6 h.

## Mapa del paper por dificultad

Lectura completa del paper hecha el 2026-11-07. Con este mapa se reparten los días 1 y 2.

| Bloque | Secciones | Contenido | Dificultad | Para qué nos sirve |
|---|---|---|---|---|
| A | Abstract, §1, §2 | Por qué existe HippoRAG 2. Las tres capacidades de memoria: factual, *sense-making* (interpretar contextos amplios) y asociatividad (multi-salto). Por qué otros RAG con grafo (GraphRAG, LightRAG, RAPTOR) empeoran en preguntas simples | Baja | Entender el objetivo del método |
| B | §3.1 y Figura 2 | Analogía con la memoria humana (LLM = neocorteza, grafo + PPR = hipocampo, encoder = región parahipocampal); fases offline y online | Media | La vista de punta a punta |
| C | §3.2–§3.5, Apéndice G.1, Apéndice B (Figura 5) | Nodos de pasaje y aristas de contexto; query-to-triple; filtro de reconocimiento; selección de semillas, probabilidades de reinicio, factor 0,05 y PPR | **Alta** | El núcleo: lo que no se puede tocar |
| D | §4, §5 | Datasets, métricas (passage recall@5, F1), implementación (Llama-3.3-70B, NV-Embed-v2, filtro optimizado con DSPy, top-5 triples), resultados (Tablas 2 y 3) | Media | Cómo medir; contra qué compararnos |
| E | §6, Apéndice E | Ablaciones (Tabla 4), factor de reinicio (Tabla 5), corpus que crece, cambio de recuperador (Tabla 7), ejemplos y análisis de errores | **Alta** | Qué componente pesa más y dónde falla el método |
| F | Apéndices A, C, D, F, G | Prompt del filtro (Figura 4), tablas extra, estadísticas del grafo, costos, hiperparámetros (Tabla 13) | Baja (consulta) | Valores exactos y costos |

**Lo que hay que saber antes de empezar:**

- PageRank y Personalized PageRank: qué son las probabilidades de reinicio y el factor de amortiguación.
- Embeddings y similitud coseno.
- Qué es un triple (sujeto, relación, objeto) y qué es OpenIE.

**Datos del paper que guían las pruebas:**

- **Ablaciones (Tabla 4, recall@5 promedio):**

  | Variante | Recall@5 | Diferencia frente a HippoRAG 2 |
  |---|---|---|
  | HippoRAG 2 completo | 87,1 | — |
  | Sin filtro | 86,4 | −0,7 |
  | Sin nodos de pasaje | 81,0 | −6,1 |
  | NER-to-node en lugar de query-to-triple | 74,6 | −12,5 |

  Query-to-triple y los nodos de pasaje son lo crítico.
- **Errores del método (Apéndice E):** la mayoría vienen del filtro y del recorrido del grafo.
  - En el 18 % de los casos el filtro deja 0 triples y el método recurre a la búsqueda solo por embeddings.
  - En el 26 % de los casos, ninguna frase de los pasajes de soporte sobrevive al filtro.
- **Costo de indexar (Apéndice F):** 11 656 pasajes consumieron 9,2 M tokens de entrada y 3,0 M de salida, es decir, unos 790 de entrada y 260 de salida por pasaje.

## Día 1 — Entender el método de punta a punta

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Repasar los conceptos previos (PPR, embeddings, triples) | 1 h |
| 2 | Leer el bloque A y luego el bloque B (§3.1 y Figura 2) | 1,5 h |
| 3 | Leer el bloque C: §3.2–§3.5 junto con el Apéndice G.1 | 2 h |
| 4 | Recorrer a mano el ejemplo de la Figura 5 (Erik Hort): pregunta → 5 triples → 2 triples filtrados → frases semilla con su puntaje → pasajes semilla × 0,05 → pasajes devueltos | 1,5 h |

**Entregable:** contrastar [sistema.md](sistema.md) con el paper y corregir lo que no coincida.

**Puerta de salida:** el equipo explica sin mirar el paper:

- los 3 pasos de la indexación y los 5 de la recuperación;
- por qué existen los nodos de pasaje;
- qué hace el filtro;
- cómo se asignan las probabilidades de reinicio;
- qué pasa si el filtro deja 0 triples.

## Día 2 — Entender por qué funciona y cómo se mide

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Leer el bloque D (§4–§5): métricas, configuración, resultados | 1,5 h |
| 2 | Leer el bloque E (§6 y Apéndice E): ablaciones, factor 0,05, cambio de recuperador, errores | 2 h |
| 3 | Leer el bloque F: prompt del filtro (Figura 4), Tabla 13, costos | 1 h |
| 4 | Definir cómo vamos a medir: métrica, umbral y línea base (búsqueda solo por embeddings, con el mismo modelo, como en el paper) | 1,5 h |

**Entregable:** cerrar ASK-07 (métrica y umbral) y actualizar [verificacion.md](verificacion.md).

**Puerta de salida:**

- La métrica y el umbral de aceptación están aprobados por el usuario.
- El equipo sabe qué componentes son críticos según las ablaciones.

**Decisiones que deben estar tomadas al cerrar el día** (las toma el usuario):

- DEC: base de código (oficial o desde cero);
- DEC: LLM y proveedor, con acceso a la API listo;
- DEC: modelo de embeddings (AD-1).

## Día 3 — El código: entorno y ejecución del original

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Leer el código con el mapa paso del paper → archivo ([antecedentes.md](antecedentes.md) §1). Comprobar en `run_ppr`, `graph_search_with_fact_entities` y `rerank_facts` que hace lo que dice el paper | 2 h |
| 2 | Crear el entorno de Python 3.10 e instalar el código oficial fijado al commit `d5c8329` | 1 h |
| 3 | Ejecutar el ejemplo oficial **en inglés y sin cambios** (`reproduce/dataset/sample`) de punta a punta | 2 h |
| 4 | Revisar lo que produjo: triples por pasaje, aristas de sinonimia, semillas, pasajes devueltos | 1 h |

**En paralelo (equipo):** preparar un **corpus piloto** en español de un solo tema del curso (unos 30–50 pasajes que cumplan REQ-03) y entre 15 y 20 preguntas de evaluación, cada una con sus pasajes de soporte. Las preguntas se escriben **después** de fijar los pasajes.

**Entregable:** el original corriendo, con una línea base en inglés que sirve de referencia.

**Puerta de salida:**

- El ejemplo oficial corre sin errores.
- El equipo localiza en el código cada paso del método.
- El corpus piloto y las preguntas están listos.

## Día 4 — Adaptaciones al español

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Crear la copia propia fijada a `d5c8329` | 0,5 h |
| 2 | Traducir los prompts de NER y de triples (`ner.py`, `triple_extraction.py`) con la misma estructura y un ejemplo en español del dominio (AD-2) | 2 h |
| 3 | Escribir el filtro en español como JSON (`rerank_dspy_file_path`), manteniendo los campos `[[ ## ... ## ]]`; y la plantilla de respuesta `rag_qa_<nuestro>` (AD-2) | 1,5 h |
| 4 | Configurar el LLM y los embeddings elegidos (AD-1); dejar el umbral en 0,8 por ahora | 0,5 h |
| 5 | Indexar el corpus piloto, con los metadatos de fuente (`Chunk.metadata`) | 1,5 h |

**Entregable:** índice del corpus piloto construido en español.

**Puerta de salida:**

- Las diferencias con el original afectan **solo** a las plantillas de prompt.
- Las tildes y la ñ se conservan (CHECK-08).
- Todos los pasajes están en el grafo (CHECK-04).

## Día 5 — Pruebas de indexación

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Revisar una muestra de triples: ¿el pasaje respalda cada uno? (CHECK-05) | 2 h |
| 2 | Revisar las aristas de sinonimia: listar pares de frases cerca del umbral y juzgar si son sinónimos reales | 1,5 h |
| 3 | Si hace falta, recalibrar el umbral (AD-3, con el OK del usuario) y reindexar | 1,5 h |
| 4 | Ajustar los prompts de NER y triples si fallan en algo concreto, sin cambiar su estructura | 1 h |

**Entregable:** informe de indexación con el porcentaje de triples respaldados, el umbral elegido y sus razones.

**Puerta de salida:**

- Los triples cumplen el criterio de fidelidad acordado.
- El umbral está justificado con ejemplos.

## Día 6 — Pruebas de recuperación

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Ejecutar las preguntas de evaluación y medir recall@2 y recall@5 (CHECK-03) | 1,5 h |
| 2 | Ejecutar la línea base de solo embeddings con las mismas preguntas y comparar | 1 h |
| 3 | Analizar los errores con las categorías del Apéndice E: filtro que deja 0 triples, frase de soporte descartada, semillas correctas pero recorrido equivocado | 2 h |
| 4 | Corregir dentro de lo permitido (prompts del filtro, AD-2) y volver a medir | 1,5 h |

**Entregable:** tabla de resultados HippoRAG 2 frente a la línea base, con el análisis de errores.

**Puerta de salida:**

- Hay resultados medidos y cada error está clasificado.
- No se cambió nada fuera de las adaptaciones aprobadas.

## Día 7 — Cierre y evidencia

| Bloque | Actividad | Duración |
|---|---|---|
| 1 | Medición final con la versión corregida | 1 h |
| 2 | Trazabilidad: cada pasaje recuperado se puede rastrear hasta su fuente y su camino en el grafo (CHECK-01) | 1,5 h |
| 3 | Registrar la evidencia de cada CHECK en el workflow | 1,5 h |
| 4 | Actualizar `producto.md`, `sistema.md` y `antecedentes.md` con lo aprendido; retomar ASK-03 a ASK-09 | 1 h |
| 5 | Decisión del usuario: ¿el recuperador está listo para pasar al corpus completo? | 1 h |

**Entregable:** recuperador en español medido, con su evidencia, y la decisión de seguir o no.

**Puerta de salida:** el usuario aprueba o pide otra iteración.

## Riesgos del plan

| Riesgo | Efecto | Qué hacer |
|---|---|---|
| Sin acceso a un LLM por API al cerrar el día 2 | El día 3 no puede empezar | Tomar la decisión del LLM durante el día 2 |
| Corpus piloto o preguntas sin terminar en el día 3 | No hay con qué probar en los días 4–6 | Asignar ese trabajo al equipo desde el día 1 |
| El código pide Python 3.10 y la máquina tiene 3.13 | Fallos de instalación | Usar un entorno aislado (uv o conda) |
| El filtro funciona peor en español | Más casos con 0 triples, que caen en la búsqueda solo por embeddings | Medirlo el día 6; ajustar solo dentro de AD-2 |
| Todo el trabajo depende de CPU (sin GPU) | Embeddings lentos | El corpus piloto es pequeño a propósito |
