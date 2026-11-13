# Cambios de adaptación

Qué pasa si el recuperador se adapta al español. No aprueba AD-1, AD-2 ni AD-3: en `producto.md` siguen **por aprobar**.

El detalle de cada frase, y de lo que el código tiene que seguir buscando igual, está en `../flujo-de-trabajo/hipporag2-espanol.md`. El recorrido es el de `../flujo-de-trabajo/hipporag2-flujo.png`.

## Qué se obtiene

Con el material del curso en español y las cinco partes de abajo reescritas en español, el recuperador indexa ese material y devuelve la respuesta en español, con la fuente del pasaje.

El grafo, las semillas y PageRank siguen igual. Cambia el idioma del texto que lee el modelo: las entidades y los triples salen en español, el filtro elige hechos en español y la lectura cierra en español.

Esto es la base de esta etapa, el recuperador. La capa de aprendizaje del agente y la interfaz del alumno vienen después.

## Las cinco partes que se reescriben

Son las cajas azules del flujo. La tarea de cada una se mantiene; se dice en español.

| Paso | Qué se reescribe | Qué se queda |
|---|---|---|
| 4 NER | La instrucción y el ejemplo de Radio City | La clave `named_entities` y el hueco `${passage}` |
| 5 Triples | La instrucción RDF y los triples de ese mismo ejemplo | La clave `triples` y los huecos `{passage}` y `{named_entity_json}` |
| 15 Embeddings de la pregunta | Las frases `query_to_fact` y `query_to_passage` | Las claves y el molde `Instruct:` / `Query:` |
| 18 Filtro | La orden (como máximo 4, sin inventar) y los 10 ejemplos | Los marcadores `[[ ## ... ## ]]` y la clave `fact` |
| 25 Lectura | El sistema, el ejemplo y las etiquetas `Wikipedia Title:`, `Question:`, `Thought:` y `Answer:` | El corte de 5 pasajes |

El ejemplo de Radio City está en `ner.py` y los triples lo importan. El párrafo, las entidades y los triples se cambian juntos.

En el paso 25, tres etiquetas las escribe `HippoRAG.py` y el paso 26 parte la respuesta con `split('Answer:')`. Se pueden decir en español si la plantilla y el `split` usan la misma palabra. Si solo se traduce la plantilla, la respuesta no se parte.

## Qué más pide REQ-05

Traducir esas cinco partes no basta para el requisito. Faltan tres cosas, todavía sin aprobar:

- **AD-2.** Traducir NER, triples, filtro y lectura. Es la tabla de arriba. Sigue por aprobar.
- **AD-1.** Embeddings multilingües en lugar de NV-Embed-v2. Con el modelo actual, el español se codifica peor. El paper (Tabla 7) muestra que el método se mantiene con otros recuperadores. Sigue por aprobar.
- Un **LLM que rinda bien en español.** Si el modelo de extracción es flojo en español, los triples salen mal y el grafo también. El LLM y el modelo de embeddings se fijan antes de indexar, porque cambiarlos obliga a reconstruir el grafo.

La normalización (`text_processing`, paso 7) ya conserva la ñ y las tildes. No hay que tocarla.

El umbral de sinonimia (**AD-3**, hoy 0,8) solo se recalibra si cambia el modelo de embeddings. Sí cambia un hiperparámetro del paper, y también sigue por aprobar.

Los términos técnicos y las siglas pueden seguir en inglés dentro del material en español. Así está escrito REQ-05.
