# Requisitos — Arquitectura (recuperador)

> Depende de: producto, descubrimiento  
> Dependen de este: arquitectura/sistema, arquitectura/verificacion

## Requisitos centrales

Pedidos por el usuario el 2026-11-07.

## REQ-04

**Trazabilidad**

Todo conocimiento recuperado debe poder rastrearse hasta dos cosas:

- su **fuente**: curso, unidad, archivo y página;
- **cómo fue indexado**: qué pasaje lo contiene, qué triples se extrajeron de ese pasaje y qué nodos del grafo llevaron a recuperarlo.

Los metadatos de fuente se guardan fuera de HippoRAG 2, enlazados a cada pasaje.

## REQ-07

**No alucinar**

- El recuperador solo devuelve pasajes que existen en el corpus.
- Si no encuentra soporte suficiente para una pregunta, lo dice y no fuerza un resultado.
- Cuando se genere una respuesta, se apoya solo en los pasajes recuperados y los cita.

## REQ-08

**Concordancia entre lo que se pregunta y lo que se recupera**

Los pasajes recuperados deben corresponder a lo que se preguntó. Se mide con el conjunto de evaluación (REQ-06). La métrica y el umbral se definen en ASK-07.

## REQ-09

**Indexado correcto**

- **Completo:** todo pasaje del corpus entra al grafo.
- **Fiel:** cada triple extraído está respaldado por el texto de su pasaje; el LLM no agrega hechos que el pasaje no contiene.
- **Inspeccionable:** se puede ver qué se extrajo de cada pasaje y cómo quedó enlazado en el grafo.

## Requisitos del corpus y del idioma

Tomados del texto del usuario sobre la preparación del corpus (ASK-02 / NOTE-02).

## REQ-02

**Preparación del corpus como etapa previa**

Es una etapa externa a HippoRAG 2. Convierte fuentes heterogéneas (PDF digital o escaneado, diapositivas, libros, transcripciones) en pasajes de texto plano UTF-8 con los campos `title`, `text` e `idx`. Tiene cuatro pasos:

1. extracción del texto, según el tipo de archivo;
2. limpieza;
3. segmentación en pasajes;
4. registro de la procedencia.

## REQ-03

**Reglas de calidad de pasaje**

Cada pasaje debe cumplir estas reglas:

- **Autocontenido:** empieza indicando su tema y no depende de pronombres ni de referencias a otras partes del documento.
- **Extensión:** entre 100 y 300 palabras, sobre un solo tema y sin cortar ideas.
- **Prosa completa:**
  - las viñetas se convierten en oraciones;
  - las tablas, en texto o markdown;
  - las fórmulas, en LaTeX con una explicación verbal;
  - las imágenes, en una descripción.
- **Terminología consistente:** las siglas se desarrollan en su primera mención.
- **Texto limpio:** sin encabezados, pies de página, numeración ni errores de OCR.

Además, en el corpus:

- se excluye el relleno: índices, bibliografía y diapositivas sin contenido académico;
- se evita el solapamiento excesivo entre pasajes;
- no hay pasajes duplicados.

## REQ-05

**HippoRAG 2 adaptado al español**

El corpus y las consultas están en español, con terminología técnica en inglés. La adaptación tiene cinco partes:

1. embeddings multilingües en lugar de NV-Embed-v2;
2. prompts de OpenIE, del filtro de triples y de respuesta, en español;
3. umbral de sinonimia recalibrado;
4. normalización de texto que preserva las tildes y la ñ;
5. un LLM con buen desempeño en español.

El LLM y el modelo de embeddings se fijan antes de indexar, porque cambiarlos obliga a reconstruir el grafo.

## REQ-06

**Conjunto de evaluación en español**

Es un conjunto propio de preguntas en español, cada una con sus pasajes de soporte, porque no existen benchmarks equivalentes a los del artículo original. Con él se comprueban REQ-08 y REQ-09.

## Escenarios

| Id | Escenario | Estado | Cubre |
|---|---|---|---|
| ESC-01 | Se pregunta por un tema del curso → se recuperan los pasajes correctos, con su fuente y su camino de indexación | propuesto | REQ-04, REQ-08 |
| ESC-02 | La pregunta exige unir información de varios pasajes o documentos → el grafo los conecta y los recupera todos | propuesto | REQ-08 |
| ESC-03 | La pregunta usa un término en inglés o una sigla → se recupera el mismo concepto que aparece en español | propuesto | REQ-03, REQ-05 |
| ESC-04 | La pregunta no tiene respuesta en el corpus → el recuperador indica que no hay soporte | propuesto | REQ-07 |
