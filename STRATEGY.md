---
name: Life OS
last_updated: 2026-05-07
---

# Life OS Strategy

## Target problem

La información sobre lo que hago, con quién me reúno, lo que capturo y lo que aprendo vive en fuentes distintas (calendar, Granola, Jira, Slack, Telegram, notas) y nunca la junto a tiempo para saber si lo que hago va en línea con mis goals.

## Our approach

El sistema se mantiene solo gracias a la IA. En vez de enlazar, organizar y reconciliar manualmente entre fuentes, los agentes (1) juntan la información y detectan contradicciones, (2) confrontan al usuario cuando los actos no cuadran con las intenciones, y (3) actúan como thinking partner para cuestionar ideas, no solo registrarlas.

## Who it's for

**Primary:** Profesional que opera en múltiples frentes a la vez — varios proyectos cliente vía Jira, producto interno, iniciativas, salud, aprendizaje, contenido, vida personal — y genera información dispersa entre muchas fuentes.

Lo contrata para extraer valor de la información que ya tiene (relaciones, ideas, soluciones, drafts de contenido, menús, entrenamientos) sin morir buscándola, para recibir confrontación honesta cuando sus actos divergen de sus goals, y para tener una contraparte que cuestione sus ideas en vez de un journaling pasivo.

## Key metrics

- **Días/semana con ritual matutino completo** — leading: si dejas de empezar el día con Life OS, algo falla. Medido en logs de skills.
- **% de focos diarios alineados con un goal mensual** — mide el pilar de confrontación: ¿lo que haces realmente apunta a lo que dijiste querer hacer? Calculado cruzando daily notes y `goals.yaml`.
- **Outputs/mes generados desde el sistema** — artículos publicados, menús, entrenamientos, drafts, briefings que no existirían sin Life OS. Mide extracción de valor.
- **Tiempo manual de organización/semana** — debería bajar mes a mes. Mide el pilar "agente-mantenido". Auto-medido (timestamps de skills) o estimación semanal.
- **Conversaciones thinking-partner/semana** — mide si el sistema funciona como sparring real, no solo como capturador.

## Tracks

### Sync (raw)

Conectores que ingestan datos externos al vault sin intervención: calendar, Granola, Jira, Slack, Telegram, training. Materializa la capa `raw/` del modelo de tres capas.

_Why it serves the approach:_ sin sync no hay "agente-mantenido" — el primer paso es que la información llegue sola.

### Wiki (curado)

Capa de conocimiento mantenida por la IA: capturas que se convierten en entradas, relaciones automáticas entre ideas, maturity de temas, content-drafts alimentándose del grafo. Materializa la capa `wiki/`.

_Why it serves the approach:_ es donde se materializa "extraer valor de info que ya tengo" — el grafo curado es el sustrato de toda extracción posterior.

### Rituales

Momentos donde el agente sintetiza y confronta: morning, today, close, week, refine. Las skills atómicas (process-inbox, clarify, review-backlog, dump) son sub-rutinas dentro de rituales, no comandos top-level.

_Why it serves the approach:_ encarna el pilar de confrontación honesta — la IA no solo guarda, también devuelve preguntas incómodas en momentos definidos.

### Extracción

Outputs que el sistema produce desde el wiki: content (artículos/posts), prep (briefings de reuniones), status (dashboard). Cierra el loop "info dispersa → valor concreto".

_Why it serves the approach:_ es la prueba de que el sistema produce más de lo que cuesta mantenerlo.

## Not working on

- Skills de learning (`learn`, `end-learn`, `init-learning`, `new-learning`, `learning-status`, `learning-report`, `learning-insights`) — son territorio de un compañero, fuera del scope de Life OS.
