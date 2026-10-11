---
description: Crea un worktree nuevo en .worktrees/<nombre>
---

Nombre recibido: $ARGUMENTS

Conviértelo a kebab-case en minúsculas: los espacios y guiones bajos se vuelven guiones, sin espacios sobrantes. Ejemplo: "triple shots" → "triple-shots".

Con ese nombre ejecuta únicamente este comando, desde la raíz del proyecto y sin hacer `cd`:

git worktree add -b <nombre> .worktrees/<nombre>

Reglas:

- No ejecutes ningún otro comando: ni `git status`, ni `git fetch`, ni `git checkout`, ni `rm`.
- No creas ni modifiques archivos.
- Si `$ARGUMENTS` viene vacío, pide el nombre y no ejecutes nada.
- Limítate a mostrar la salida del comando.
