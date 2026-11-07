# Producto

> Depende de: —  
> Dependen de este: descubrimiento, arquitectura/, aprendizaje/, entregas/

## Qué es

Un agente educativo que ayuda al alumno a aprender mejor los temas de su curso. Responde preguntas, pero no da la respuesta de forma fácil: con la capa de aprendizaje le enseña cómo se conectan las ideas.

Antes de construir esa capa hay que tener bien resuelta la **capa de recuperación de conocimiento**, porque es la base de todo lo demás. Elegimos HippoRAG 2 para esa capa por dos razones:

- **Trazabilidad:** se puede ver de qué fuente viene cada conocimiento y cómo fue indexado.
- **Evitar alucinaciones:** el agente se apoya en lo que recupera de las fuentes, no en lo que inventa.

Fuente: usuario, 2026-11-07.

## Capas

| Capa | Carpeta | Qué es | Cuándo |
|---|---|---|---|
| **Arquitectura** (recuperador) | [arquitectura/](arquitectura/) | Recuperador de conocimiento basado en HippoRAG 2: indexa las fuentes y recupera lo que se pregunta, con trazabilidad | **Ahora**: es el foco de las próximas semanas |
| **Aprendizaje** | [aprendizaje/](aprendizaje/) | Métodos de enseñanza (socrático, conexión de ideas) sobre lo recuperado | Después de la arquitectura |

Las tareas se separan por capa.

```text
 Alumno ─pregunta─▶ [Aprendizaje] ─consulta─▶ [Arquitectura: HippoRAG 2] ─▶ Corpus indexado
        ◀─guía──── [Aprendizaje] ◀─pasajes + trazabilidad─
```

## Usuarios (REQ-01)

- **Alumnos:** son los únicos usuarios del producto (ASK-01).
- **Equipo del proyecto:** construye y cura el corpus en la etapa inicial. Es un rol interno, no un usuario del producto (ASK-01).

## Objetivo inmediato

Comprobar que el recuperador **indexa bien** el conocimiento de las fuentes y **recupera bien** lo que se le pide.

## Fuera de alcance

- Interfaz para que los docentes carguen material; por ahora lo hace el equipo (ASK-01).
- (pendiente: confirmar con el usuario qué más no es el producto)

## Restricciones

- (pendiente: el usuario aún no ha definido las limitaciones)

## Afirmaciones

| Id | Texto | Estado | Procedencia |
|---|---|---|---|
| OBJ-01 | Agente educativo cuya base es un recuperador trazable (HippoRAG 2), sobre el que una capa de aprendizaje enseña a conectar ideas | hipotesis | idea inicial y aclaración del usuario (2026-11-07) |
