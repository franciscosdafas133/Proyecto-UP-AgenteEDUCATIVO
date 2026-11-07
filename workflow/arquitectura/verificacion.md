# Verificación — Arquitectura (recuperador)

> Depende de: arquitectura/requisitos  
> Estado: **propuesto**; falta el OK del usuario a este plan.

| CHECK | Requisito | Qué se comprueba | Tipo |
|---|---|---|---|
| CHECK-01 | REQ-04 | Cada pasaje recuperado incluye su fuente (curso, unidad, archivo, página) y su camino de indexación | auto |
| CHECK-02 | REQ-07 | Las preguntas sin respuesta en el corpus devuelven "sin soporte" y no un resultado forzado | auto |
| CHECK-03 | REQ-08 | Lo recuperado coincide con los pasajes de soporte del conjunto de evaluación. Métrica y umbral: ASK-07; el paper usa *passage recall@5* | auto |
| CHECK-04 | REQ-09 | Todo pasaje del corpus está en el grafo | auto |
| CHECK-05 | REQ-09 | Los triples están respaldados por su pasaje (revisión de una muestra) | manual |
| CHECK-06 | REQ-02, REQ-03 | Los pasajes cumplen el formato y las reglas medibles: UTF-8, `title`/`text`/`idx`, 100–300 palabras, sin duplicados ni relleno | auto |
| CHECK-07 | REQ-03 | Los pasajes son autocontenidos y tratan un solo tema (revisión de una muestra) | manual |
| CHECK-08 | REQ-05 | Las tildes y la ñ se conservan de punta a punta, y el índice rechaza consultas hechas con otros modelos | auto |
