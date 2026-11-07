# Producto — Arquitectura (recuperador)

> Depende de: ../producto.md  
> Dependen de este: requisitos, sistema, verificacion

## Qué vamos a construir

Un **recuperador de conocimiento** que implementa el método de HippoRAG 2 (Gutiérrez et al., ICML 2025). Hace dos cosas:

1. **Indexa** el corpus del curso como un grafo de conocimiento.
2. **Recupera** los pasajes que responden a una pregunta, mostrando de qué fuente vienen y cómo fueron indexados.

Es la base sobre la que después se construye la capa de aprendizaje.

**Meta de esta etapa:** comprobar que **indexa bien** y **recupera bien** lo que le pedimos.

## Limitante principal: fidelidad al paper

- El recuperador sigue **estrictamente** el método del paper: indexación (OpenIE → grafo con nodos de frase y de pasaje, aristas de relación, de sinonimia y de contexto) y recuperación (query-to-triple → filtro de reconocimiento → semillas → PPR).
- **Cualquier desviación del método se considera un defecto del producto.** No se agregan, quitan ni reemplazan pasos.
- Los hiperparámetros son los del paper, salvo que una adaptación aprobada diga otra cosa.
- **Solo se aceptan adaptaciones** que el usuario haya aprobado explícitamente. Cada una queda registrada en la tabla de abajo, con su justificación.

Fuente: usuario, 2026-11-07.

### Hiperparámetros del paper

Comprobados en el paper (§3, Apéndice G.1) y en el código oficial (`config_utils.py`):

| Parámetro | Valor |
|---|---|
| Triples recuperados por consulta (`linking_top_k`) | 5 |
| Frases semilla máximas | 5 |
| Factor de peso de nodos de pasaje (`passage_node_weight`) | 0,05 |
| Factor de amortiguación de PPR (`damping`) | 0,5 |
| Umbral de similitud para sinonimia (`synonymy_edge_sim_threshold`) | 0,8 |
| Pasajes que recibe la respuesta (`qa_top_k`) | 5 |
| Grafo | no dirigido |

### Adaptaciones al método

Todas requieren aprobación del usuario.

| # | Adaptación | Por qué | ¿Cambia el método? | Estado |
|---|---|---|---|---|
| AD-1 | Embeddings multilingües en lugar de NV-Embed-v2 | Corpus y consultas en español (REQ-05) | No. El método no depende del modelo de embeddings, y el paper (Tabla 7) muestra que la mejora se mantiene con distintos recuperadores | por aprobar |
| AD-2 | Prompts de NER, OpenIE, filtro y respuesta traducidos al español, con la misma estructura y los mismos ejemplos | Grafo consistente en un solo idioma (REQ-05) | No. Solo cambia el idioma | por aprobar |
| AD-3 | Recalibrar el umbral de sinonimia (0,8) | El valor depende del modelo de embeddings | Sí, cambia un hiperparámetro del paper | por aprobar |

## Cómo lo vamos a construir

### Existe código oficial

Los autores publicaron su implementación (MIT): https://github.com/OSU-NLP-Group/HippoRAG. La ficha completa, el código que hay que adaptar al español y cómo entra el material están en [antecedentes.md](antecedentes.md).

### Recomendación

Usar el código oficial como base, fijado a una versión exacta, y aplicar encima solo las adaptaciones aprobadas (AD-1 a AD-3). Las razones:

- Es la forma más segura de cumplir el limitante de fidelidad.
- Escribirlo desde cero agregaría riesgo de desviarnos sin aportar nada.

Lo que sí construimos nosotros es lo que el paper deja fuera:

- la preparación del corpus (REQ-02, REQ-03);
- el registro de procedencia (REQ-04);
- los prompts en español (AD-2);
- el conjunto de evaluación en español (REQ-06).

**Estado:** propuesta. La decisión (DEC) la toma el usuario.

## Fuera de alcance de la arquitectura

- Capa de aprendizaje (método socrático, conexión de ideas).
- Interfaz para el alumno.
- Funciones del código oficial que no están en el paper.

## Pendientes

1. Aprobar o rechazar la base de código (código oficial frente a escribir desde cero).
2. Aprobar o rechazar las adaptaciones AD-1, AD-2 y AD-3.
3. **REQ-07 frente a la fidelidad:** el paper no tiene una salida "sin soporte". Hay dos opciones:
   - a) REQ-07 se limita a lo que el método garantiza: devuelve solo pasajes reales del corpus, con su trazabilidad;
   - b) se agrega la salida "sin soporte" como adaptación AD-4, fuera del método.
