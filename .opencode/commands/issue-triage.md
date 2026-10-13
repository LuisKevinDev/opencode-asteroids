---
description: Formatea y etiqueta un issue preservando el texto original y añadiendo contexto para revisarlo
---

Número de issue recibido: $ARGUMENTS

Repo objetivo: `LuisKevinDev/opencode-asteroids` (si dudas del owner/repo, sácalo de `git remote get-url origin`). Todos los comandos `gh` llevan `--repo <owner>/<repo>`.

## Pasos

1. Si `$ARGUMENTS` viene vacío o no es un número, pide el número del issue y no ejecutes nada más.
2. **Credenciales:** ejecuta `gh auth status`. Usa únicamente las credenciales que ya trae la sesión (token de la app de opencode intercambiado por OIDC). Si no hay token válido, detente y avisa: no crees ni busques tokens, no dependas de `secrets.GITHUB_TOKEN` del workflow y no inventes rutas alternativas.

3. Lee el issue:

   ```
   gh issue view <N> --repo <owner>/<repo> --json number,title,body,author,createdAt,authorAssociation,url
   ```

   `body` puede llegar vacío o `null` → trátalo como cadena vacía.

4. **Idempotencia:** si el `body` ya contiene el marcador `<!-- triage:v1 -->`, responde «El issue #N ya está formateado, no se toca nada» y termina aquí sin editar.

5. Elige **una** etiqueta de tipo, en este orden de prioridad. Se buscan prefijos en el título y palabras clave en `título + cuerpo`, sin distinguir mayúsculas ni acentos:

   | Prioridad | Prefijo en título | Palabras clave | Etiqueta |
   |---|---|---|---|
   | 1 | `[BUG]`, `[ERROR]` | error, bug, falla, fallo, no funciona, roto, crashea, se cierra, excepción, pantalla en blanco, parpadea | `bug` |
   | 2 | `[DOCS]` | readme, documentación, documentacion, documentar, comentarios, agents.md | `documentation` |
   | 3 | `[PREGUNTA]`, `[DUDA]`, `[Q]` | cómo, como hago, duda, pregunta, por qué, por que | `question` |
   | 4 | `[FEAT]`, `[MEJORA]` | añadir, anadir, agregar, nueva, nuevo, mejora, idea, sugerencia, feature | `enhancement` |
   | — | sin coincidencia | | `needs-triage` |

   Si toca `needs-triage`, créala antes (idempotente con `--force`):

   ```
   gh label create needs-triage --repo <owner>/<repo> --color BFDADC --description "Pendiente de clasificar" --force
   ```

6. Obtén la referencia de código: rama por defecto con `gh repo view <owner>/<repo> --json defaultBranchRef` y su SHA (acepto `git rev-parse --short HEAD` como respaldo). Con eso arma las URLs `commit/<sha>` y `blob/<sha>/<archivo>`.

7. Construye el cuerpo nuevo siguiendo **exactamente** la plantilla de abajo. El bloque de código va así: el texto original se copia **crudo y literal** dentro de `<details>` — sin resumir, sin traducir, sin escapar su markdown. Si el resultado supera 65000 caracteres, recorta el original y añade una línea `> _Texto original truncado por longitud; ver el historial de ediciones del issue._`.

8. Escribe el cuerpo resultante en un archivo temporal **fuera del repo**: usa `$RUNNER_TEMP/triage-<N>.md` y, si esa variable no existe, `../triage-<N>.md`. Luego aplícalo:

   ```
   gh issue edit <N> --repo <owner>/<repo> --body-file "<ruta temporal>"
   ```

9. Aplica la etiqueta elegida:

   ```
   gh issue edit <N> --repo <owner>/<repo> --add-label "<etiqueta>"
   ```

10. Borra el archivo temporal y responde con: etiqueta aplicada, enlace al issue y el recordatorio de que queda pendiente rellenar el checklist y el entorno.

## Plantilla del cuerpo

```
<!-- triage:v1 -->

### Ficha

- **Autor:** @`<login>` (`<authorAssociation>`)
- **Creado:** `<createdAt>`
- **Referencia de código:** `<rama>`@`<sha>` ([commit](https://github.com/<owner>/<repo>/commit/<sha>))
- **Archivos implicados:** [`index.html`](https://github.com/<owner>/<repo>/blob/<sha>/index.html) · [`game.js`](https://github.com/<owner>/<repo>/blob/<sha>/game.js)

### Pasos para reproducir

- [ ] _Por completar_

### Comportamiento esperado

- [ ] _Por completar_

### Comportamiento actual

- [ ] _Por completar_

### Entorno

- **SO:** _por completar_
- **Navegador + versión:** _por completar_
- **Resolución / zoom:** _por completar_

<details>
<summary><strong>Texto original (sin editar)</strong></summary>

<CUERPO ORIGINAL CRUDO, TAL CUAL>

</details>
```

Las URLs absolutas se construyen como `https://github.com/<owner>/<repo>/commit/<sha>` y `https://github.com/<owner>/<repo>/blob/<sha>/game.js`.

## Reglas

- No crees ramas, no hagas commits, no abras PRs. No modifiques `game.js`, `index.html`, `AGENTS.md` ni ningún otro archivo del repo.
- Como máximo **una** edición de cuerpo y **una** de etiquetas por ejecución.
- Nada de `/oc` ni `/opencode` en tu respuesta: re-dispararía el workflow de comentarios.
- Todo el texto visible al usuario en **español**, igual que la UI de `game.js` y `index.html` (`lang="es"`).
- Si un comando `gh` falla, muestra el error tal cual y detente; no intentes rutas alternativas ni reescribas el cuerpo a mano sin pasar por `--body-file`.
- `gh` no está instalado en Windows local: si falla con «command not found», dilo y termina (el comando está pensado para ejecutarse en el runner de GitHub).
