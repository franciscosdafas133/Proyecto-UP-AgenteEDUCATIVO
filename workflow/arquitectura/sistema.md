# Sistema — Arquitectura (recuperador)

> Depende de: arquitectura/producto, arquitectura/antecedentes, arquitectura/requisitos  
> Dependen de este: arquitectura/verificacion, plan

Método: HippoRAG 2 (Gutiérrez et al., ICML 2025), §3 y Apéndice G.1. Archivo: `../../01_HippoRAG2_ICML2025.pdf`.

Este documento define **qué piezas hay y cómo se conectan**. La tecnología concreta (modelos, librerías, almacenamiento) se decide más adelante, en decisiones (DEC) dentro de cada tarea.

## Cómo funciona HippoRAG 2, en simple

Funciona en dos fases.

**1. Indexar (offline, una vez):**

- Un LLM lee cada pasaje y extrae hechos en forma de **triples** (sujeto, relación, objeto). Por ejemplo, de un pasaje sobre fotosíntesis puede salir (fotosíntesis, ocurre en, cloroplasto).
- Con esos triples se arma un **grafo de conocimiento**:
  - los sujetos y objetos son nodos de **frase**;
  - cada pasaje es un nodo de **pasaje**, conectado a las frases que salieron de él;
  - las frases que significan lo mismo se unen con aristas de **sinonimia**, que se detectan por similitud de embeddings.

**2. Recuperar (online, en cada pregunta):**

- La pregunta se compara con los triples del grafo y se toman los más parecidos.
- Un LLM **filtra** esos triples y descarta los irrelevantes. El paper llama a este paso "memoria de reconocimiento".
- Las frases que sobreviven sirven de **punto de partida** para un recorrido por el grafo, el algoritmo Personalized PageRank (PPR). El recorrido se expande hacia los conceptos y pasajes conectados.
- Los pasajes que reciben más puntaje en ese recorrido son el resultado.

**Por qué sirve para este producto:**

- **Trazabilidad:** cada resultado tiene un camino visible: pregunta → triples → frases → pasaje → fuente.
- **Conexión de ideas:** el grafo une conceptos de distintos documentos, así que puede recuperar todo lo que una pregunta necesita aunque esté repartido en varias fuentes.

## Componentes

| # | Componente | Responsabilidad | Cubre |
|---|---|---|---|
| C1 | Preparación del corpus | Extraer, limpiar y segmentar las fuentes en pasajes; registrar su procedencia. Es externo a HippoRAG 2 | REQ-02, REQ-03 |
| C2 | Registro de trazabilidad | Guardar la fuente de cada pasaje (curso, unidad, archivo, página) y el camino de indexación (pasaje → triples → nodos) | REQ-04, REQ-09 |
| C3 | Indexador | Hacer OpenIE con el LLM; construir el grafo con nodos de frase y de pasaje, y aristas de relación, de sinonimia y de contexto | REQ-05, REQ-09 |
| C4 | Recuperador | Comparar la pregunta con los triples, aplicar el filtro, fijar las semillas, ejecutar PPR y ordenar los pasajes; devolver cada resultado con su trazabilidad | REQ-04, REQ-07, REQ-08 |
| C5 | Evaluación | Medir el recuperador con el conjunto de preguntas en español | REQ-06, REQ-08 |

La adaptación al español (REQ-05) no es un componente aparte. Afecta a C3 y C4: prompts, embeddings, umbral de sinonimia y normalización.

## Trazabilidad

```text
Fuente (curso, unidad, archivo, página)
   └─ Pasaje (idx)
        └─ Triples extraídos (sujeto, relación, objeto)
             └─ Nodos de frase del grafo
                  └─ Pregunta → triples filtrados → semillas → PPR → pasaje recuperado
```

Cada resultado del recuperador debe incluir este camino (REQ-04).

## Dónde puede aparecer una alucinación y cómo se controla

| Punto | Riesgo | Control |
|---|---|---|
| OpenIE (C3) | El LLM extrae un triple que el pasaje no dice | Cada triple queda enlazado a su pasaje y se comprueba que el pasaje lo respalda (REQ-09) |
| Filtro de triples (C4) | El LLM descarta triples buenos o deja pasar malos | Se mide con el conjunto de evaluación (REQ-08) |
| Resultado (C4) | Forzar un resultado cuando no hay soporte | Si no queda ningún triple ni pasaje con similitud suficiente, se indica que no hay soporte (REQ-07) |
| Respuesta | Inventar contenido fuera de lo recuperado | La respuesta solo usa pasajes recuperados y los cita (REQ-07). Se genera en la capa de aprendizaje |

## Flujo de indexación

1. Fuentes → C1 → pasajes (`title`, `text`, `idx`), con su procedencia guardada en C2.
2. Pasaje → LLM (OpenIE) → triples. El sujeto y el objeto son nodos de frase; la relación es una arista de relación.
3. Pasaje → nodo de pasaje, unido por una arista de contexto ("contiene") a sus frases.
4. Se calculan los embeddings de frases, triples y pasajes. Los pares de frases con similitud mayor que el umbral reciben una arista de sinonimia.
5. Se guardan el grafo, los embeddings y el registro de qué LLM y qué modelo de embeddings se usaron.

## Flujo de recuperación

Valores entre paréntesis: los que usa el paper.

1. Pregunta → embedding → los k triples más parecidos (k = 5).
2. El LLM filtra los triples irrelevantes.
3. Si no queda ningún triple, se devuelven los mejores pasajes por similitud de embedding, sin recorrer el grafo.
4. Si quedan triples, se fijan las semillas:
   - **frases:** hasta 5, cada una con el puntaje promedio de los triples filtrados en que aparece;
   - **pasajes:** todos, cada uno con su similitud de embedding multiplicada por un factor de peso (0,05).
5. Se ejecuta PPR y se ordenan los pasajes por su puntaje.
6. Se devuelven los mejores pasajes, cada uno con su trazabilidad (C2).

## Diagrama

```text
 Fuentes ─▶ [C1 Preparación] ─────procedencia─────▶ [C2 Trazabilidad]
                 │ pasajes                               ▲   │
                 ▼                                       │   │
           [C3 Indexador] ─ OpenIE ─▶ LLM                │   │
                 │        ─ embeddings ─▶ Modelo emb.    │   │
                 ▼                                       │   │
     Grafo: frase ─relación─ frase                       │   │
            frase ─sinonimia─ frase                      │   │
            pasaje ─contiene─▶ frase ── camino ──────────┘   │
                 │                                           │
  Pregunta ─▶ [C4 Recuperador]                               │
     triples → filtro (LLM) → semillas → PPR                 │
                 │ pasajes + trazabilidad ◀──────────────────┘
                 ▼
          [C5 Evaluación]   ·   (luego) Capa de aprendizaje
```

## Decisiones pendientes

Se toman dentro de su tarea, no ahora.

- Modelo de embeddings multilingüe.
- LLM y proveedor.
- Base de implementación: repo oficial, fork o implementación propia.
- Almacenamiento del grafo, los embeddings y la trazabilidad.
