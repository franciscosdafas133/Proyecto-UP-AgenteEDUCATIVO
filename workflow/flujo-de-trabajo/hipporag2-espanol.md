# Qué pasar al español en el flujo del código

Identificación para una posible adaptación. No aprueba AD-2: en `producto.md` sigue **por aprobar**.

El recorrido es el de `hipporag2-flujo.png` (código oficial, commit `d5c8329`). El método del paper no cambia: siguen el grafo, las semillas, PageRank, los pesos y los cortes. Lo que se puede pasar al español es el **texto que lee el modelo** en cinco pasos. Las claves, los marcadores y el molde con los que el código lee la respuesta se quedan igual.

| Marca en el gráfico | Qué significa aquí |
|---|---|
| Caja azul | Hay texto en inglés. Abajo está qué parte de esa caja se reescribe y qué parte no. |
| Caja verde, violeta o blanca | No tiene instrucción en inglés. No entra en esta adaptación. |

## Los cinco pasos

| Paso en el gráfico | Archivo | Se reescribe en español | Se deja igual |
|---|---|---|---|
| 4 NER | `prompts/templates/ner.py` | La instrucción y el ejemplo | La clave `named_entities` y el hueco `${passage}` |
| 5 Triples | `prompts/templates/triple_extraction.py` | La instrucción y los triples del ejemplo | La clave `triples` y los huecos `{passage}` y `{named_entity_json}` |
| 15 Dos embeddings de la pregunta | `prompts/linking.py` | Las dos frases que sí se usan | Las claves y el molde `Instruct:` / `Query:` |
| 18 Filtro | `prompts/filter_default_prompt.py` | La orden y los 10 ejemplos | Los marcadores `[[ ## ... ## ]]` y la clave `fact` |
| 25 Lectura | `prompts/templates/rag_qa_musique.py` y tres líneas de `HippoRAG.qa` | El sistema, el ejemplo y las etiquetas, las cuatro a la vez | El corte `qa_top_k = 5` |

El ejemplo de Radio City está escrito una vez en `ner.py` y los triples lo importan. Si el párrafo cambia, cambian juntos la lista de entidades y la lista de triples. Si no, el modelo aprende con un ejemplo que no coincide.

## Paso 4. NER

`OpenIE.ner` arma el mensaje con `ner.py` y mete el pasaje en `${passage}`.

Se reescribe:

- `Your task is to extract named entities from the given paragraph.`
- `Respond with a JSON list of entities.`
- El párrafo de Radio City y los nombres de esa lista (`Radio City`, `India`, `3 July 2001`, y el resto).

Se deja igual, porque el parser las busca así:

- La clave `named_entities`.
- El hueco `${passage}`.
- El orden del mensaje: sistema, ejemplo de usuario, ejemplo de asistente, pasaje real.
- El tope de 2048 tokens y la regla de quitar duplicados conservando el orden.

La tarea sigue siendo la del paper: sacar las entidades del pasaje. Cambia el idioma, no el paso.

## Paso 5. Triples

`OpenIE.triple_extraction` corre después, con el mismo pasaje y con las entidades que devolvió el paso 4.

Se reescribe:

- La instrucción de construir el grafo RDF.
- Las dos reglas: cada triple lleva al menos una entidad de la lista, de preferencia dos; los pronombres se resuelven al nombre.
- `Convert the paragraph into a JSON dict, it has a named entity list and a triple list.`
- Los predicados y objetos del ejemplo (`located in`, `started on`, `plays songs in`, y el resto). Tienen que salir del párrafo ya traducido en el paso 4.

Se deja igual:

- La clave `triples`.
- Los huecos `{passage}` y `{named_entity_json}`.
- `filter_invalid_triples`: solo entran listas de exactamente tres elementos, sin repetir.
- El tope de 4096 tokens.

## Paso 15. Dos embeddings de la pregunta

Con `nvidia/NV-Embed-v2` cada pregunta se codifica dos veces, antes del bucle. El modelo antepone `Instruct: {frase}\nQuery:`.

Se reescriben solo estas dos, que son las que el camino llama:

| Clave | Frase actual |
|---|---|
| `query_to_fact` | `Given a question, retrieve relevant triplet facts that matches this question.` |
| `query_to_passage` | `Given a question, retrieve relevant documents that best answer the question.` |

Se deja igual:

- Los nombres `query_to_fact` y `query_to_passage`. El código los pide así.
- El molde `Instruct:` y `Query:` de `NVEmbedV2.py`. Es el formato de ese modelo, no una instrucción del paper.
- `ner_to_node`, `query_to_node` y `query_to_sentence`. Están en el mismo archivo y este camino no las usa.

Los embeddings del paso 3 y del paso 8 no llevan estas frases. Traducirlas no cambia cómo se indexa el pasaje.

Esto depende de qué modelo quede aprobado en AD-1, que también está por aprobar. Con NV-Embed-v2 las frases sí entran. Un modelo cuyo nombre empieza por `Transformers/` las ignora, y los embeddings de OpenAI también. En ese caso traducirlas no hace nada.

## Paso 18. Filtro

`DSPyFilter.rerank` usa `filter_default_prompt.py` porque `rerank_dspy_file_path` es null. El JSON `filter_llama3.3-70B-Instruct.json` no se carga.

Se reescribe:

- La orden del sistema: elegir como máximo 4 hechos de la lista, no inventar hechos, devolver una lista vacía si ninguno sirve.
- Los 10 ejemplos. Preguntas y hechos están en inglés (Imperial River, The Ancestor, Teafuone, S.O.B., Gloria, The Old Guard, Aulad, Angelical Tears, Virgin Mountain, The Virtuous Model). Cada ejemplo se traduce entero: la pregunta, los hechos de entrada y los hechos que el ejemplo deja.

Se deja igual, porque `rerank.py` parte la respuesta con la expresión `[[ ## (palabra) ## ]]` y se queda con el campo `fact_after_filter`:

- `[[ ## question ## ]]`
- `[[ ## fact_before_filter ## ]]`
- `[[ ## fact_after_filter ## ]]`
- `[[ ## completed ## ]]`
- La clave JSON `fact` y la forma de cada hecho: tres strings, sujeto, predicado, objeto.
- El tope de 512 tokens.
- El paso 19. `difflib` no es texto: obliga a devolver un hecho que ya estaba entre los cinco candidatos.

El límite de 4 y la prohibición de inventar son la regla del filtro del paper. En español se dicen en español; no se aflojan.

`antecedentes.md` ya indica cómo meterlo sin tocar el parser: un JSON en español apuntado por `rerank_dspy_file_path`.

## Paso 25. Lectura

`HippoRAG.qa` corta a 5 pasajes y arma el mensaje en `HippoRAG.py` antes de aplicar la plantilla. Si no hay `rag_qa_<dataset>`, usa `rag_qa_musique.py`.

Se reescribe, en el mismo cambio:

- El sistema: analizar los pasajes, razonar y cerrar con una respuesta corta.
- El ejemplo de Neville A. Stanton, los cinco pasajes de Wikipedia y la respuesta `1862`.
- Cuatro etiquetas que hoy están en inglés. Tres las escribe `HippoRAG.py` (`Wikipedia Title:`, `Question:`, `Thought:`) y la cuarta la busca el paso 26 con `split('Answer:')`.

Esas cuatro etiquetas se pueden decir en español. El código tiene que escribir y buscar la misma palabra. Si la plantilla cierra con `Respuesta:` y el `split` sigue buscando `Answer:`, la respuesta no se parte y el producto devuelve el texto crudo. Por eso no se traduce solo el archivo de la plantilla.

Se deja igual:

- `qa_top_k = 5`. La lectura sigue viendo cinco pasajes, no los 200.
- El hueco `${prompt_user}`.
- La regla de elegir `rag_qa_<dataset>` y, si no existe, caer en musique. La plantilla en español entra como `rag_qa_<nuestro_dataset>`, que es lo que ya dice `antecedentes.md`.

## Lo que no se traduce

Estos pasos del mismo gráfico no llevan una instrucción en inglés. Pasarlos “al español” sería cambiar el método.

| Pasos | Por qué se quedan |
|---|---|
| 1, 2, 13, 14, 16, 17, 24 | Reciben texto, cortan o puntúan. No hay prompt. |
| 3, 6, 9, 10, 12, 23 | Guardan la traza. El id, la metadata y las aristas no tienen idioma. |
| 7 | `text_processing` ya conserva ñ y tildes. |
| 8, 11, 20, 21, 22 | Embeddings de entidades y hechos, KNN, pesos y PageRank. Comparan vectores. |
| 19 y 26 | `difflib` y `split` no son prompts. El `split` solo cambia si cambia la etiqueta del paso 25, y cambia a la par. |
| Desvío 20b | Misma búsqueda por embeddings, sin PageRank. |
| Nota de IRCoT | `retrieve_ircot` no corre con `rag_qa`. El defecto de `max_qa_steps` es 1. |
| `ner_query.py` | Nadie lo importa en este commit. |

El corpus y las preguntas del curso entran por los pasos 1 y 13. Que estén en español es el material del producto, no una plantilla del código.
