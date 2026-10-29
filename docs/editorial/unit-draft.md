# Unidad editorial — borrador editable

> **Estado: BLOQUEADO — intake y fuentes autorizadas pendientes.** Este archivo es un
> entregable de trabajo; no es una pieza lista para publicar.

## Ficha de la unidad

| Campo | Estado |
| --- | --- |
| Unidad | Pendiente de identificar |
| Obra o proyecto | Pendiente de identificar |
| Tema y ángulo | Pendientes; no aparecen en el brief recibido |
| Audiencia y propósito | Pendientes |
| Formato, extensión y canal | Pendientes |
| Idioma de trabajo | Español provisional, sujeto a confirmación |
| Fuentes autorizadas | No entregadas en este worktree |
| Estado editorial | No iniciado; bloqueado en intake |

## Continuidad recibida

El entregable de dirección editorial disponible en el worktree hermano
`office/byte-89a8` deja el intake bloqueado. Su registro indica que todavía faltan el tema o
historia, el ángulo, la audiencia, el formato, las fuentes y los responsables de aprobación.
También establece que no deben completarse esos campos por suposición.

No hay una unidad anterior, un canon aprobado ni un dossier de investigación que esta unidad
pueda continuar. No se añaden nombres, fechas, cifras, citas, escenas, atribuciones ni
conclusiones que no estén respaldados por una fuente recibida.

## Borrador editable

### Encabezado de la unidad

`[PENDIENTE: título o función de la unidad]`

### Texto

No se redacta prosa sustantiva mientras falten el brief cerrado y las fuentes autorizadas. Este
marcador evita convertir una ausencia de evidencia en contenido editorial.

```text
[PENDIENTE DE BRIEF Y FUENTES: redactar aquí una unidad cuando estén confirmados
el tema, el ángulo, la función dentro de la obra y el material que puede utilizarse.]
```

### Continuidad que debe completarse antes de redactar

- Unidad anterior y hechos/decisiones que deben conservarse: `[pendiente]`.
- Hilo que esta unidad debe abrir, desarrollar o cerrar: `[pendiente]`.
- Canon de voz, terminología y límites: `[pendiente de aprobación]`.
- Unidad siguiente o consecuencia narrativa/informativa: `[pendiente]`.
- Afirmaciones con fuente y marcadores de verificación: `[pendiente de dossier]`.

## Resumen de acción y evidencias

**Acción completada:** se revisó el brief de coordinación disponible, se comprobó el almacén
editorial local en modo lectura y se persistió este borrador controlado. No se consultaron ni se
incorporaron fuentes externas.

| ID | Evidencia verificable | Resultado |
| --- | --- | --- |
| E-01 | `docs/editorial/brief-canon-plan.md` en el worktree `office/byte-89a8`; SHA-256 `C740A28F32B0B58614A263C82628952EBDAAE7C9B92209101D473578A0E9417F`. | El intake de dirección está bloqueado y exige no inventar tema, fuentes ni responsables. |
| E-02 | Base local `.agent-office/editorial/editorial.sqlite`, consultada en solo lectura el 2026-10-29; tablas `projects`, `revisions`, `sources` y `artifacts`: `0` registros cada una. SHA-256 observado: `4DC98989153237DD19155650F5D2CA82A14BE9E1F849966E3C2A03E599F7E3D9`. | No existe en el almacén compartido un proyecto o fuente persistida que pueda sustentar la unidad. |
| E-03 | Petición de esta tarea. | Define escritura por unidades, continuidad, evidencia verificable y bloqueadores explícitos; no aporta tema ni fuentes. |

## Decisiones y verificaciones requeridas

Antes de sustituir los marcadores por texto, el propietario editorial debe confirmar:

1. Tema, obra y función exacta de la unidad.
2. Audiencia, propósito, voz y extensión.
3. Formato, idioma y canal de entrega.
4. Fuentes autorizadas, permisos y qué afirmaciones pueden sostenerse con cada una.
5. Continuidad disponible: unidad anterior, canon vigente y siguiente hito.
6. Persona propietaria del contenido y persona autorizada para aprobarlo.

## Bloqueadores

- **B-01 — Sin brief cerrado:** no hay objeto, ángulo ni función de la unidad.
- **B-02 — Sin fuentes autorizadas:** no se pueden sostener hechos, citas, cifras o atribuciones.
- **B-03 — Sin continuidad:** no existe una unidad anterior ni canon aprobado que preservar.
- **B-04 — Sin decisión de formato y audiencia:** no se puede fijar estructura, tono ni extensión.
- **B-05 — Sin aprobador identificado:** no hay quien resuelva las decisiones pendientes ni valide
  el paso a una versión publicable.

**Siguiente acción:** entregar el brief y el paquete de fuentes, o confirmar explícitamente los
campos anteriores. Hasta entonces, este archivo permanece como borrador editable bloqueado y no
debe presentarse como contenido factual ni publicarse.
