# Dossier: hooks de Codex en Windows y actividad obsoleta

Fecha de corte: 2026-10-29
Código inspeccionado: `852e2ab28c4f3f31bffdf4b20353bd119c9d3270` (`origin/main`)
Tema: [issue #382](https://github.com/AgentSystemLabs/agent-office/issues/382)

## Resumen ejecutivo

La causa técnica más probable está localizada: la versión inspeccionada de Agent Office construye cada comando de hook de Codex con comillas simples de estilo POSIX. En Windows, ese texto se entrega a un shell; la ruta de `node.exe` queda como un token que `cmd.exe` no puede interpretar y PowerShell no invoca sin el operador de llamada `&`. Una reproducción local del nivel shell devuelve código 1 en ambos casos antes de ejecutar el helper.

La relación con la actividad obsoleta es coherente con el código: el estado de actividad y trabajo se actualiza cuando llega un evento de lifecycle al puente; si el hook no arranca, no llega `SessionStart`, `UserPromptSubmit`, `PreToolUse` ni `Stop`. El proceso de Codex puede seguir vivo y escribir en el terminal mientras la tarjeta conserva el aviso de arranque. Esto es una explicación de alta confianza para el mecanismo, pero no demuestra por sí sola que sea la causa exacta de la sesión histórica de Pixel.

El [PR #376](https://github.com/AgentSystemLabs/agent-office/pull/376) propone un wrapper PowerShell codificado en Windows y aporta tests específicos. Sigue abierto, no está fusionado y no lo trato como arreglo verificado.

## Pregunta y alcance

Se investigó si los errores `Hook failed` / `hook exited with code 1` observados en Windows pueden impedir que Agent Office reciba los eventos de Codex y, por tanto, dejar obsoletos `activity`, `status` y el estado de la cola.

Fuera de alcance: reparar el código, validar una versión concreta de Codex en producción o afirmar la causa de la sesión histórica sin sus artefactos locales.

## Hechos verificados

1. El issue #382 reporta un worker Codex de Windows que continúa ejecutando comandos, muestra repetidos errores de hook, conserva la actividad `Open the terminal: complete login and review Office hooks in /hooks` y mantiene la tarea de cola en ejecución. El issue identifica explícitamente la relación entre fallos de hook y estado obsoleto como sospechada, no confirmada. Fuente: [issue #382](https://github.com/AgentSystemLabs/agent-office/issues/382).

2. En el código base, `shellQuote()` envuelve cada argumento en comillas simples POSIX y `codexHookArgs()` las aplica a `process.execPath`, la ruta del helper y el nombre del evento. La configuración resultante solo escribe `type="command"`, sin un comando alternativo Windows. Fuente: [`src/server/codex.ts`, líneas 85–100](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b20353bd119c9d3270/src/server/codex.ts#L85-L100).

3. El adaptador Codex añade esos overrides a cada lanzamiento, tanto nuevo como reanudado. Fuente: [`src/server/providers/codex.ts`, líneas 89–104](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b20353bd119c9d3270/src/server/providers/codex.ts#L89-L104).

4. El helper solo informa al puente después de leer JSON, validar el evento y completar un `POST` a `/hooks/codex`; ante error devuelve fallo sin actualizar el estado del worker. Fuente: [`src/server/codex.ts`, líneas 120–186](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b203bd119c9d3270/src/server/codex.ts#L120-L186).

5. El servidor acepta el evento únicamente por `POST /hooks/codex`, y `handleProviderHook()` exige worker activo, provider correcto y token válido antes de llamar al adaptador. Fuente: [`src/server/hooks/server.ts`, líneas 24–42](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b203bd119c9d3270/src/server/hooks/server.ts#L24-L42) y [`src/server/workers/manager.ts`, líneas 585–592](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b203bd119c9d3270/src/server/workers/manager.ts#L585-L592).

6. El adaptador normaliza el payload y lo pasa a `reduceLifecycle()`. Un `UserPromptSubmit` marca el worker como `working`, un `PreToolUse` actualiza `activity` con la herramienta y un `Stop` lo marca como `done`. Un evento inválido o ausente no ejecuta ninguna de esas transiciones. Fuente: [`src/server/providers/codex.ts`, líneas 70–86](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b20353bd119c9d3270/src/server/providers/codex.ts#L70-L86) y [`src/server/workers/lifecycle.ts`, líneas 84–156](https://github.com/AgentSystemLabs/agent-office/blob/852e2ab28c4f3f31bffdf4b203bd119c9d3270/src/server/workers/lifecycle.ts#L84-L156).

7. Codex marca que `commandWindows` es el override específico para Windows, y su esquema admite `command` y `commandWindows` en un handler de tipo comando. Fuentes oficiales: [documentación de Hooks de Codex](https://learn.chatgpt.com/docs/hooks#configure-a-command-hook) y [esquema oficial del protocolo](https://github.com/openai/codex/blob/main/codex-rs/app-server-protocol/src/protocol/v2/config.rs#L3457-L3471).

8. El código fuente oficial de Codex selecciona `commandWindows` cuando corre en Windows; en el runner, el shell por defecto usa `cmd.exe /C` y un shell explícito recibe el texto del comando como argumento. Fuentes: [discovery.rs](https://github.com/openai/codex/blob/main/codex-rs/hooks/src/engine/discovery.rs#L3341-L3368) y [command_runner.rs](https://github.com/openai/codex/blob/main/codex-rs/hooks/src/engine/command_runner.rs#L2012-L2058).

9. Existe evidencia independiente en el repositorio oficial de Codex de que un ejecutable entre comillas como primer token falla en Windows, mientras que un ejecutable sin comillas y argumentos citados sí funciona. Es evidencia de comportamiento de Codex, no una prueba de la sesión de Agent Office. Fuente: [openai/codex issue #46454](https://github.com/openai/codex/issues/46454).

## Acciones y evidencias

| ID | Acción | Resultado verificable |
| --- | --- | --- |
| A1 | Consulté el issue del repositorio con `gh issue view 382 --json ...`. | Confirmó síntomas, worker, tarea y rutas de evidencia declaradas; no incluye comentarios ni adjuntos reproducibles. |
| A2 | Inspeccioné `codex.ts`, el adaptador Codex, el servidor de hooks, el manager y el reducer de lifecycle en `852e2ab`. | La cadena `comando → helper → POST → reducer → estado` está implementada y permite localizar dónde se pierde el evento. |
| A3 | Generé un helper temporal que solo lee stdin y escribe `{}`; ejecuté el comando actual con `cmd.exe /d /s /c` y `powershell.exe -NoProfile -NonInteractive -Command`. | Con `C:\Program Files\nodejs\node.exe` como `process.execPath`, el comando actual fue: `'C:\Program Files\nodejs\node.exe' '<helper>' 'UserPromptSubmit'`. `cmd.exe`: código 1, `The filename, directory name, or volume label syntax is incorrect.` PowerShell: código 1, `Unexpected token`. El helper no se ejecutó. |
| A4 | Consulté el diff y el cuerpo del [PR #376](https://github.com/AgentSystemLabs/agent-office/pull/376). | Propone `powershell.exe -EncodedCommand`, operador `&`, entrada UTF-8 y tests con espacios, apóstrofes, metacaracteres y Unicode. El PR declara validación live, pero también 62 fallos del suite completo en Windows y sigue abierto. |
| A5 | Contrasté con documentación y código fuente oficiales de Codex. | Codex documenta `commandWindows`; su runner efectivamente elige ese campo en Windows. |
| A6 | Busqué la evidencia local mencionada en el issue. | `.agent-office/scrollback/b874de4bd6aa.ansi`, `.agent-office/workers.json` y el worktree `fix/codex-windows-hooks` no existen en este worktree. |

### Reproducción mínima del nivel shell

La reproducción A3 no ejecuta una sesión de Codex ni hace llamadas de red. Solo verifica el contrato que recibe Codex: un string de comando y stdin JSON. El entorno fue Windows, Node `v24.18.1`, `codex-cli 0.165.0-alpha.1.2`, `process.execPath = C:\\Program Files\\nodejs\\node.exe`. Por eso prueba el fallo de quoting, pero no sustituye una reproducción end-to-end del issue #382.

## Cadena causal evaluada

```text
codexHookArgs()
  └─ comando POSIX con comillas simples
     └─ shell Windows no arranca node.exe/helper
        └─ no hay POST /hooks/codex
           └─ no entra reduceLifecycle()
              └─ activity/status pueden conservar el aviso anterior
                 mientras el PTY de Codex sigue produciendo salida
```

La cadena desde “no entra el evento” hasta “no cambia la tarjeta” está demostrada por el código. La última observación del issue —que el PTY continúa ejecutando comandos— proviene del reporte del usuario y no pudo verificarse aquí.

## Separación de hechos, inferencias y límites

### Hechos

- El generador actual usa quoting POSIX para los hooks de Codex.
- En la reproducción local, ese comando falla con `cmd.exe` y PowerShell antes de ejecutar el helper.
- El helper y el reducer solo pueden actualizar el estado después de recibir/aceptar un evento.
- El PR #376 está abierto y no fusionado.
- La documentación oficial de Codex contempla una variante `commandWindows`.

### Inferencias

- **Alta confianza:** la incompatibilidad de quoting es una causa suficiente para perder los hooks de la versión inspeccionada cuando Codex usa cualquiera de los dos modos de shell reproducidos.
- **Alta confianza:** si todos los hooks de lifecycle fallan de esa forma, Agent Office puede mostrar una actividad/estado obsoletos aunque el proceso de Codex siga vivo.
- **Media-alta:** ese mecanismo explica los síntomas descritos en #382 y es más fuerte que una hipótesis genérica de “la cola no refresca”.
- **Baja-media:** no está probado que haya sido la única causa de la sesión de Pixel; también podrían intervenir trust de hooks, selección de shell, versión de Codex, reinicio del bridge o un token/worker inválido.

### No comprobable con la evidencia disponible

- No puedo confirmar el contenido de `scrollback` ni `workers.json` de la sesión histórica porque no están en este worktree.
- No puedo confirmar la versión exacta de Codex, el shell elegido por esa instalación ni si el hook llegó a `/hooks/codex` el 2026-10-21.
- No puedo afirmar que el PR #376 resuelva el caso real: no está fusionado y su validación live es la declaración del PR, no una prueba independiente de este dossier.
- No puedo afirmar que la variante `commandWindows` sea directamente intercambiable con el wrapper del PR sin probar la sintaxis exacta que genera el CLI de Agent Office.

## Estado de un posible siguiente paso

El PR #376 es el artefacto más cercano a una mitigación: cambia solo la generación de los comandos y añade pruebas Windows, manteniendo POSIX sin cambios. Antes de considerarlo solución, faltan una prueba en una instalación limpia con la versión de Codex del incidente, captura del `POST /hooks/codex` y una comprobación de que `SessionStart`, `UserPromptSubmit`, `PreToolUse` y `Stop` actualizan la tarjeta y terminan la tarea de cola.

## Bloqueadores de verificación

1. Falta la evidencia local histórica referida por #382.
2. No hay una reproducción controlada del Codex exacto del incidente; el Codex disponible localmente es `0.165.0-alpha.1.2`.
3. La instalación de dependencias del worktree quedó incompleta por extracción lenta de npm en Windows: `npm ci` sufrió además `ECONNRESET` en audit y una segunda instalación sin audit no creó los binarios locales antes de ser interrumpida. Las comprobaciones posteriores fallaron por ese estado: `npm run typecheck` terminó con `node_modules/marked/lib/marked.d.ts(299,11): error TS1005: '}' expected`; `npm test` falló en los 82 archivos porque no encontró `node_modules\\tsx\\index.js`; `npm run build` no pudo resolver `vite`. Por ello no declaro ninguna de las tres como pasada ni atribuyo esos errores al cambio documental.
4. No hay cambio visual; no aplica captura headless.

## Fuentes registradas

- [Agent Office issue #382](https://github.com/AgentSystemLabs/agent-office/issues/382) — reporte primario, síntomas y evidencia declarada.
- [Agent Office PR #376](https://github.com/AgentSystemLabs/agent-office/pull/376) — candidato de corrección y resultados declarados.
- [Commit `491355c`](https://github.com/AgentSystemLabs/agent-office/commit/491355cfe5feb4c0bb13831edb51932f98c69616) y [commit `e84bffd`](https://github.com/AgentSystemLabs/agent-office/commit/e84bffd773fb3851b4739d2c97707f57fe69245f) — commits del candidato.
- [Codex Hooks, documentación oficial](https://learn.chatgpt.com/docs/hooks) — eventos, `commandWindows`, trust y contrato de comandos.
- [Codex `discovery.rs`](https://github.com/openai/codex/blob/main/codex-rs/hooks/src/engine/discovery.rs#L3341-L3368) y [`command_runner.rs`](https://github.com/openai/codex/blob/main/codex-rs/hooks/src/engine/command_runner.rs#L2012-L2058) — selección y ejecución del comando en Windows.
- [openai/codex issue #46454](https://github.com/openai/codex/issues/46454) — corroboración externa del fallo de ejecutable entre comillas en Windows.
- [openai/codex issue #54072](https://github.com/openai/codex/issues/54072) — evidencia relacionada sobre diferencias de código de salida bajo PowerShell; no se usa como prueba del caso Agent Office.
