# AGENTS.md — Mapa de la arquitectura (recuperador)

Punto de entrada para cualquier agente o persona que trabaje en la capa de arquitectura. Indica qué documento leer según la pregunta y qué reglas no se pueden romper.

## Reglas (antes de tocar nada)

1. **Fidelidad al paper.** El recuperador sigue estrictamente HippoRAG 2. Cualquier desviación es un defecto, salvo que sea una adaptación aprobada (AD-n en `producto.md`).
2. **Origen de cada contenido.** Todo lo que se escribe aquí sale del usuario, del paper o del código oficial. Si no tiene esa fuente, se pregunta antes de escribirlo.
3. **Decisiones.** Las decisiones (DEC) y las adaptaciones solo las aprueba el usuario. El agente propone; no decide.
4. **Cronograma.** No se avanza de día si no se cumple la puerta de salida del día actual (`cronograma.md`).

## Fuentes de verdad

| Fuente | Dónde |
|---|---|
| Método | `../../01_HippoRAG2_ICML2025.pdf` (Gutiérrez et al., ICML 2025) |
| Código de referencia | https://github.com/OSU-NLP-Group/HippoRAG, commit `d5c8329` |
| Estado del workflow | `../estado.json` y `../relaciones.json` (no se editan a mano) |

## Documentos y orden de lectura

```text
../producto.md               (producto general: capas, usuarios)
      │
      ▼
producto.md     ── qué construimos, limitante de fidelidad, adaptaciones AD-n
      │
      ▼
antecedentes.md ── código oficial, qué tocar para el español, cómo entra el material
      │
      ▼
requisitos.md   ── REQ-02..09 y escenarios
      │
      ▼
sistema.md      ── componentes, trazabilidad, flujos de indexación y recuperación
      │
      ▼
verificacion.md ── CHECK-n: cómo se comprueba cada requisito
      │
cronograma.md   ── plan de 7 días: qué se hace cada día y su puerta de salida
```

## Dónde buscar según la pregunta

| Pregunta | Ir a |
|---|---|
| ¿Qué estamos construyendo y qué no? | `producto.md` |
| ¿Se puede cambiar X del método? | `producto.md` → Limitante y Adaptaciones |
| ¿Qué valor tiene un hiperparámetro? | `producto.md` → Hiperparámetros del paper |
| ¿Qué archivo del repo implementa el paso Y? | `antecedentes.md` §1 |
| ¿Qué hay que modificar para el español? | `antecedentes.md` §2 |
| ¿Qué entrega la adaptación al español y qué falta aprobar? | `cambios-de-adaptacion.md` |
| ¿Cómo se prepara y entra el material? | `antecedentes.md` §3 y `requisitos.md` (REQ-02, REQ-03) |
| ¿Qué debe cumplir el recuperador? | `requisitos.md` |
| ¿Cómo funciona el método, paso a paso? | `sistema.md` |
| ¿Cuál es el flujo del paper, en orden? | `../flujo-de-trabajo/hipporag2-proceso.json`, `.mmd` y `.png` |
| ¿Cómo corre el código, parte por parte, y qué está en inglés? | `../flujo-de-trabajo/hipporag2-flujo.png` y `.mmd` |
| ¿Qué texto de ese flujo se puede pasar al español sin cambiar el método? | `../flujo-de-trabajo/hipporag2-espanol.md` |
| ¿Cuál es la frase exacta de una herramienta en inglés? | `../flujo-de-trabajo/hipporag2-herramientas.json` |
| ¿Qué función implementa cada paso? | `../flujo-de-trabajo/hipporag2-codigo.json` |
| ¿Cómo se rastrea un resultado hasta su fuente? | `sistema.md` → Trazabilidad |
| ¿Dónde puede aparecer una alucinación? | `sistema.md` → Dónde puede aparecer una alucinación |
| ¿Cómo sé que un requisito se cumple? | `verificacion.md` |
| ¿Qué toca hoy y qué falta para avanzar? | `cronograma.md` |
| ¿Qué preguntas siguen abiertas? | `../descubrimiento.md` y la sección Pendientes de cada documento |

## Al cambiar un documento

- Si cambia un requisito, revisar `sistema.md` y `verificacion.md`.
- Si se aprueba una adaptación, actualizar `producto.md` (tabla AD), `antecedentes.md` §2 y el cronograma si cambia el plan.
- Correr el validador: `python3 ~/.claude/skills/product-flow/scripts/validate_workflow.py workflow`
