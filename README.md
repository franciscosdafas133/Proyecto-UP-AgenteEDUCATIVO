# Agente Educativo

Un agente educativo que ayuda al alumno a aprender mejor los temas de su curso. Responde preguntas, pero no da la respuesta de forma fácil: le enseña cómo se conectan las ideas, para que logre un aprendizaje significativo.

## Estado actual

> **Actualizado:** 2026-11-07  
> **Fase:** documentación de la **arquitectura** terminada; arrancamos con el cronograma de 7 días.  
> **Código:** todavía no hay código del proyecto. El repositorio solo tiene documentación.

| Capa | Estado |
|---|---|
| **1. Arquitectura** (recuperador) | **En curso.** Documentada; empieza la implementación |
| **2. Aprendizaje** (métodos de enseñanza) | En espera; empieza cuando la arquitectura esté lista |

**Antes de arrancar el día 3 hay que resolver:**

- [ ] Fecha de inicio del cronograma
- [ ] Base de código: usar el código oficial de HippoRAG 2 (recomendado) o escribirlo desde cero
- [ ] Qué LLM usar (por API; esta máquina no tiene GPU) y qué modelo de embeddings multilingüe
- [ ] Aprobar las adaptaciones al español AD-1, AD-2 y AD-3
- [ ] REQ-07 ("no alucinar"): el paper nunca responde "sin soporte"; decidir si se acepta como una adaptación más
- [ ] Datos del corpus: curso o cursos, universidad, tamaño, si entran libros y transcripciones

## Por qué empezamos por la arquitectura

El agente tiene dos capas:

```text
 Alumno ─pregunta─▶ [2. Aprendizaje] ─consulta─▶ [1. Arquitectura: HippoRAG 2] ─▶ Material del curso
        ◀─guía──── [2. Aprendizaje] ◀─pasajes + su fuente─
```

1. **Arquitectura (recuperador de conocimiento).** Busca en el material del curso los pasajes que responden a una pregunta. Está basada en **HippoRAG 2** (Gutiérrez et al., ICML 2025; ver Referencias). La elegimos por dos razones:
   - **Trazabilidad:** cada resultado muestra de qué fuente viene y cómo fue indexado.
   - **No alucinar:** el agente se apoya en lo que recupera, no en lo que inventa.
2. **Aprendizaje.** Sobre lo recuperado, el agente enseña con método socrático y conexión de ideas.

La capa de aprendizaje depende por completo de lo que recupera la arquitectura. Si la recuperación falla, la enseñanza falla. Por eso primero comprobamos que el recuperador **indexa bien** y **recupera bien**, y recién después construimos la enseñanza.

**Regla principal de la arquitectura:** seguimos el método del paper **al pie de la letra**. Cualquier desviación cuenta como un defecto, salvo las adaptaciones al español que se aprueben explícitamente.

## Qué sigue: cronograma de 7 días

| Días | Fase | Qué se hace |
|---|---|---|
| 1–2 | Comprender | Leer el paper a fondo; definir cómo medimos |
| 3 | Código | Instalar el código oficial y correrlo en inglés sin cambios |
| 4 | Adaptar | Traducir los prompts al español, configurar los modelos e indexar un corpus piloto |
| 5–6 | Probar | ¿Los triples son fieles a su pasaje? ¿Recupera mejor que la búsqueda solo por embeddings? |
| 7 | Cerrar | Medición final, evidencia y decisión |

El detalle de cada día, con sus puertas de salida, está en el documento [Cronograma — Arquitectura HippoRAG 2](https://claude.ai/artifact/1HUavyedVxkH4fWefTTrwW). También está en [workflow/arquitectura/cronograma.md](workflow/arquitectura/cronograma.md).

**El equipo, en paralelo desde el día 1:** preparar un corpus piloto en español (30 a 50 pasajes de un solo tema) y entre 15 y 20 preguntas de evaluación con sus pasajes de soporte.

## Cómo está organizado

```text
Proyecto_UP_Agente_Educativo/
├── README.md                   ← este archivo
├── 01_HippoRAG2_ICML2025.pdf   ← el paper (no está en el repo; ver Referencias)
└── workflow/                   ← toda la documentación del producto
    ├── producto.md             ← qué es el agente, sus capas y sus usuarios
    ├── descubrimiento.md       ← preguntas abiertas y respondidas
    ├── arquitectura/           ← CAPA 1 (en curso)
    │   ├── AGENTS.md           ← empieza aquí: mapa de los documentos de esta carpeta
    │   ├── producto.md         ← qué construimos, regla de fidelidad, adaptaciones
    │   ├── antecedentes.md     ← código oficial, qué adaptar al español, cómo entra el material
    │   ├── requisitos.md       ← trazabilidad, no alucinar, concordancia, indexado correcto…
    │   ├── sistema.md          ← cómo funciona HippoRAG 2, componentes y flujos
    │   ├── verificacion.md     ← cómo comprobamos cada requisito
    │   └── cronograma.md       ← los 7 días
    ├── aprendizaje/            ← CAPA 2 (más adelante)
    └── *.json                  ← estado interno del método de trabajo (no editar a mano)
```

**Para ponerte al día, lee en este orden:**

1. este README;
2. [workflow/producto.md](workflow/producto.md);
3. [workflow/arquitectura/AGENTS.md](workflow/arquitectura/AGENTS.md), que te lleva al resto.

## Referencias

- Paper: *From RAG to Memory: Non-Parametric Continual Learning for Large Language Models* (HippoRAG 2), ICML 2025. https://arxiv.org/abs/2502.14802. Su licencia de arXiv no permite redistribuirlo, así que no está en el repo: descárgalo y guárdalo en la raíz como `01_HippoRAG2_ICML2025.pdf`, que es la ruta que usan los documentos.
- Código oficial (MIT): https://github.com/OSU-NLP-Group/HippoRAG, revisado en el commit `d5c8329`.
