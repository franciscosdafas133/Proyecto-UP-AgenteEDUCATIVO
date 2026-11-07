# Descubrimiento

> Depende de: producto  
> Dependen de este: arquitectura/requisitos, aprendizaje/requisitos

## Áreas (R03)

Para cada área: ¿aplica? · qué se sabe · qué falta · cómo averiguarlo.

| Área | Aplica | Se sabe | Falta | Cómo averiguarlo | Pregunta |
|---|---|---|---|---|---|
| Quién lo usa | sí | Solo alumnos; el equipo cura el corpus al inicio | — | Resuelto con el usuario | ASK-01 |
| Datos | sí | Material académico del curso → pasajes UTF-8 (title/text/idx); reglas de pasaje; metadatos externos; corpus en español con términos en inglés | Nombre de curso(s) y universidad; si entran libros y transcripciones; tamaño real del corpus | Preguntar al usuario (NOTE-03) | ASK-02 |
| Otras apps y claves | ? | — | Proveedor de LLM/embeddings, credenciales, inicio de sesión | Aplazada hasta tener el recuperador | ASK-03 |
| Cosas que tardan | ? | Indexar (OpenIE con LLM sobre todo el corpus) es un proceso largo | Si se reindexa automáticamente | Aplazada hasta tener el recuperador | ASK-04 |
| Choques | ? | — | — | Aplazada hasta tener el recuperador | ASK-05 |
| Camino feliz | ? | — | Recorrido del alumno en la capa de enseñanza | Aplazada hasta tener el recuperador | ASK-06 |
| Cómo sabemos que está bien | sí | Conjunto propio de preguntas de evaluación en español | Métrica y umbral de aceptación | Aplazada hasta tener el recuperador | ASK-07 |
| Dónde se usa | ? | — | Local o web | Aplazada hasta tener el recuperador | ASK-08 |
| Después de publicar | ? | Cambiar LLM o embeddings obliga a reconstruir el grafo | Costos, copias, reindexado | Aplazada hasta tener el recuperador | ASK-09 |

## Preguntas abiertas

- **ASK-01** (Quién lo usa): ¿Quién va a usar esto (dueño, cliente, empleado) y qué puede hacer cada uno?
- **ASK-02** (Datos): ¿Qué información guarda y qué pasa si se pierde o se borra?
- **ASK-03** (Otras apps y claves): ¿Habla con WhatsApp, pagos u otras apps, o necesita contraseñas?
- **ASK-04** (Cosas que tardan): ¿Hay algo que tarde o corra solo (mensajes, reportes)?
- **ASK-05** (Choques): ¿Pueden dos personas pelearse por el mismo turno o el mismo dato?
- **ASK-06** (Camino feliz): ¿Cuál es el camino más corto para que el usuario termine lo que quiere?
- **ASK-07** (Cómo sabemos que está bien): ¿Cómo sabremos que una versión funciona bien?
- **ASK-08** (Dónde se usa): ¿Lo usarán solo en el local o también por internet?
- **ASK-09** (Después de publicar): ¿Qué hay que cuidar después (copias de seguridad, avisos, costos)?

## Estado de las preguntas

- ASK-01, ASK-02: resueltas (NOTE-01, NOTE-02). Queda un pendiente en ASK-02 (NOTE-03).
- ASK-03 a ASK-09: aplazadas por el usuario el 2026-11-07 hasta tener el recuperador.

## Hipótesis

| Id | Texto | Estado | Procedencia |
|---|---|---|---|
| OBJ-01 | La calidad del aprendizaje depende primero de recuperar bien el conocimiento; la capa socrática va encima | hipotesis | idea inicial del usuario |
