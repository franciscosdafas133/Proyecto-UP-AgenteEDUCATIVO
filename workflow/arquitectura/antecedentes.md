# Antecedentes — Arquitectura (recuperador)

> Depende de: arquitectura/producto  
> Dependen de este: arquitectura/sistema, tareas de arquitectura

Revisado el 2026-11-07. El código se leyó en una copia temporal; no se copió nada al proyecto.

## 1. El código oficial de HippoRAG 2

| | |
|---|---|
| Repositorio | https://github.com/OSU-NLP-Group/HippoRAG (enlazado en el propio paper) |
| Autores | Los mismos del paper (OSU NLP Group) |
| Licencia | MIT: se puede usar y modificar citando el original |
| Versión | Paquete `hipporag` 2.0.0a5 (alfa). La rama `main` es HippoRAG 2; la rama `legacy` es HippoRAG 1 |
| Commit revisado | `d5c8329` (2026-10-06) |
| Python | 3.10, con dependencias fijadas (torch 2.5.1, transformers 4.45.2, python-igraph 0.11.8) |
| Valores por defecto | Coinciden con el paper: `linking_top_k`=5, `passage_node_weight`=0,05, `damping`=0,5, `synonymy_edge_sim_threshold`=0,8, `qa_top_k`=5 |

Partes del repositorio que corresponden al método:

| Paso del paper | Archivo (`src/hipporag/`) |
|---|---|
| OpenIE: NER y luego extracción de triples | `information_extraction/openie_openai.py` y `prompts/templates/ner.py`, `triple_extraction.py` |
| Grafo: aristas de relación, de contexto y de sinonimia | `HippoRAG.py`: `add_fact_edges`, `add_passage_edges`, `add_synonymy_edges` |
| Query-to-triple | `HippoRAG.py`: `get_fact_scores` |
| Filtro de reconocimiento | `rerank.py` (`DSPyFilter`) y `prompts/filter_default_prompt.py` |
| Semillas y PPR | `HippoRAG.py`: `graph_search_with_fact_entities`, `run_ppr` |
| Respuesta (QA) | `HippoRAG.py`: `qa` y `prompts/templates/rag_qa_musique.py` |
| Evaluación | `evaluation/retrieval_eval.py` (recall@k) y `qa_eval.py` |

El repositorio trae además funciones que el paper no describe: servidor MCP, bases vectoriales externas, borrado de documentos y pasarelas de LLM. **No las usamos**, porque estaríamos fuera del método.

## 2. Qué código hay que tocar para adaptarlo al español

| # | Punto | Dónde está | Qué hay hoy | Qué cambia | Cómo se cambia | Adaptación |
|---|---|---|---|---|---|---|
| 1 | Prompt de NER | `prompts/templates/ner.py` | Instrucción en inglés y un ejemplo (párrafo sobre "Radio City") | Instrucción y ejemplo en español, con la misma estructura (sistema + 1 ejemplo) | Editar la plantilla | AD-2 |
| 2 | Prompt de triples | `prompts/templates/triple_extraction.py` | Instrucción en inglés y el mismo ejemplo | Traducir, con la misma estructura y las mismas reglas (resolver pronombres, usar las entidades del NER) | Editar la plantilla | AD-2 |
| 3 | Prompt del filtro | `prompts/filter_default_prompt.py` | Prompt optimizado con DSPy para Llama-3.3-70B, con ejemplos en inglés | Versión en español. Se mantienen los campos `[[ ## fact_after_filter ## ]]`, que el código usa para leer la respuesta | Archivo JSON indicado en `rerank_dspy_file_path`, **sin tocar el código** | AD-2 |
| 4 | Prompt de respuesta | `prompts/templates/rag_qa_musique.py` | En inglés | Nueva plantilla `rag_qa_<nuestro_dataset>` en español | Agregar una plantilla | AD-2 |
| 5 | Modelo de embeddings | Configuración: `embedding_model_name` | `nvidia/NV-Embed-v2` | `Transformers/<modelo multilingüe>`, que corre en CPU | Solo configuración | AD-1 |
| 6 | Instrucciones de consulta para embeddings | `prompts/linking.py` | Instrucciones en inglés | Si el modelo es de tipo sentence-transformers (por ejemplo, BGE-M3), el código las ignora y no hay que cambiarlas. Si el modelo usa instrucciones o prefijos (Qwen3-Embedding, E5), sí hay que adaptarlas | Depende de AD-1 | AD-1 |
| 7 | Umbral de sinonimia | Configuración: `synonymy_edge_sim_threshold` | 0,8 | Valor recalibrado para el nuevo modelo | Solo configuración | AD-3 |
| 8 | LLM | Configuración: `llm_name` y `llm_base_url` | `gpt-4o-mini` | El LLM que se elija (API) | Solo configuración | — |
| 9 | Normalización de texto | `utils/misc_utils.py`: `text_processing` | Pasa a minúsculas y conserva letras y números Unicode | **Nada.** Probado: conserva tildes y ñ | — | — |

**Conclusión:**

- **No se toca el algoritmo:** grafo, semillas y PPR quedan tal cual.
- **Cambios concretos:**
  - 3 plantillas de prompt (puntos 1, 2 y 4);
  - 1 archivo JSON para el filtro (punto 3);
  - configuración (puntos 5, 7 y 8).
- **Restricción técnica:** la carpeta de plantillas está fija dentro del paquete; no se puede configurar. Por eso los prompts en español obligan a mantener una **copia propia del repositorio, fijada al commit `d5c8329`**, donde solo cambian las plantillas. Cualquier otro cambio en esa copia se considera una desviación.
- `ner_query.py` no hace falta traducirlo: HippoRAG 2 no lo usa, porque enlaza la consulta con triples y no con entidades.

## 3. Cómo ingresa el material

### Lo que el código espera

- **Entrada:** `index(docs)` recibe una lista de textos o de objetos `Chunk(content, source_id, metadata)`. El `metadata` se guarda y `retrieve()` lo devuelve con cada pasaje. Esto sirve para la trazabilidad (REQ-04).
- **El código no divide el texto:** su preprocesador por defecto convierte cada documento en un solo pasaje. Lo que le entregamos **es** el pasaje que se indexa, así que la segmentación es enteramente responsabilidad nuestra (REQ-02, REQ-03).
- **El título forma parte del pasaje:** el script del paper arma cada pasaje como `title + "\n" + text`. El título lo lee el LLM al extraer triples y entra en el embedding.
- **Identidad por contenido:** el id de cada pasaje es un hash de su texto. Dos pasajes idénticos se funden en uno, y cambiar una sola letra crea un pasaje nuevo.
- **Costo:** al indexar se hacen 2 llamadas al LLM por pasaje (NER y triples); en cada consulta, 1 llamada (el filtro).

### Preguntas abiertas sobre el material

Hay que resolverlas antes de preparar el corpus.

1. **¿Quién convierte viñetas, tablas e imágenes en prosa?** Si lo hace un LLM, puede introducir contenido que no está en la fuente **antes** de indexar. Eso rompe la trazabilidad y la regla de no alucinar, y el paper no lo cubre. Hay tres opciones: lo hace una persona, lo hace un LLM con revisión humana, o solo se transforma el formato sin reescribir.
2. **¿Qué va en el `title`?** Forma parte de lo que el LLM convierte en triples. Puede ser el nombre del tema (como en los datos del paper, que son títulos de Wikipedia) o una ruta del tipo "Curso › Unidad › Tema". Esto afecta qué nodos aparecen en el grafo.
3. **Versiones del corpus:** como el id es el hash del texto, corregir un pasaje lo convierte en otro. Hay que definir cómo se corrige el material sin desordenar el índice ni el conjunto de evaluación.
4. **El conjunto de evaluación depende del corpus final:** las preguntas apuntan a pasajes concretos, así que se arma después de fijar los pasajes, no antes.
5. **Fórmulas en LaTeX:** no está claro qué triples saca el LLM de una fórmula. Conviene probarlo con una muestra antes de procesar todo.
6. **Terminología:** para usar los términos de forma consistente (REQ-03) hace falta un glosario por curso, con la forma en español, la forma en inglés y la sigla. ¿Quién lo mantiene?
7. **Contenido repetido en varias fuentes:** el mismo concepto aparece en diapositivas, separatas y libro con distinta redacción. El código solo funde los duplicados exactos. ¿Se indexa todo o se elige una versión?
8. **Derechos:** ¿qué libros se pueden incluir? (pendiente desde ASK-02)
9. **Muestra piloto:** ¿con qué curso y cuántos documentos probamos primero, antes de procesar el corpus completo?
