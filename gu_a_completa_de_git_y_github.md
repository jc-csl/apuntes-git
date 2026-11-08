# 📘 Guía Completa de Git y GitHub: De Principiante a Experto

Esta guía consolida todos los conceptos, comandos, flujos de trabajo y resolución de problemas para dominar Git y GitHub.

## 📑 Índice
1. [Introducción y Configuración Inicial](#1-introducción-y-configuración-inicial)
2. [Autenticación y Conexión (SSH vs HTTPS)](#2-autenticación-y-conexión-ssh-vs-https)
3. [Creación y Clonación de Repositorios](#3-creación-y-clonación-de-repositorios)
4. [Flujo de Trabajo Básico (Local)](#4-flujo-de-trabajo-básico-local)
5. [Trabajando con Ramas (Branches)](#5-trabajando-con-ramas-branches)
6. [Sincronización Local y Remoto](#6-sincronización-local-y-remoto)
7. [Deshacer Cambios y Viajar en el Tiempo](#7-deshacer-cambios-y-viajar-en-el-tiempo)
8. [Herramientas Avanzadas y Casos Especiales](#8-herramientas-avanzadas-y-casos-especiales)
9. [Flujos de Trabajo en Equipo y Colaboración](#9-flujos-de-trabajo-en-equipo-y-colaboración)
10. [Hoja de Referencia Rápida (Cheat Sheet)](#10-hoja-de-referencia-rápida-cheat-sheet)

---

## 1. Introducción y Configuración Inicial

Git opera con una jerarquía de archivos de configuración (Local, Global, Sistema). La configuración local tiene prioridad sobre la global, y la global sobre la del sistema.

### Niveles de Configuración
*   **LOCAL:** Se aplica a un repositorio particular. Ubicación: `.git/config` dentro de tu proyecto.
*   **GLOBAL:** Se aplica a todos los repositorios de un usuario. Ubicación: `~/.gitconfig` (Linux/Mac) o `C:\Users\<Usuario>\.gitconfig` (Windows).
*   **SISTEMA:** Se aplica a todos los usuarios de la máquina. Requiere permisos de administrador.

### Configuración Básica (Global)
Al instalar Git por primera vez, debes identificarte:

```bash
# Tu identidad (Obligatorio)
git config --global user.name "Tu Nombre"
git config --global user.email "tuemail@dominio.com"

# Editor por defecto (ej. nano o vscode)
git config --global core.editor "nano"

# Evitar problemas de saltos de línea (Windows CRLF vs Linux LF)
git config --global core.autocrlf false

# Nombre de la rama principal por defecto (main en lugar de master)
git config --global init.defaultBranch main
```

### Comandos útiles de configuración
```bash
git config --list --show-origin  # Ver todas las configuraciones y de dónde provienen
git config --global alias.last 'log -1 HEAD' # Crear un atajo. Ahora "git last" mostrará el último commit.
```

---

## 2. Autenticación y Conexión (SSH vs HTTPS)

Para comunicarte con GitHub de forma segura sin introducir tu contraseña constantemente, lo ideal es usar SSH.

### Generar y Configurar Claves SSH
1. **Generar la clave:**
   ```bash
   ssh-keygen -t ed25519 -C "tuemail@dominio.com"
   # Pulsa Enter a todo para guardar en la ruta por defecto (~/.ssh/id_ed25519)
   ```
2. **Iniciar el agente SSH y añadir la clave:**
   ```bash
   eval "$(ssh-agent -s)"
   ssh-add ~/.ssh/id_ed25519
   ```
3. **Copiar la clave pública a GitHub:**
   ```bash
   cat ~/.ssh/id_ed25519.pub
   # Copia el texto que sale y pégalo en GitHub -> Settings -> SSH and GPG keys -> New SSH key
   ```
4. **Probar la conexión:**
   ```bash
   ssh -T git@github.com
   # Debe responder: "Hi usuario! You've successfully authenticated..."
   ```

### 🚨 Diagnóstico: Problemas con HTTPS vs SSH
Si configuraste SSH pero Git sigue pidiéndote usuario y contraseña, probablemente clonaste el repo usando la URL `https://` en lugar de `git@`.
Para solucionarlo, cambia la URL del remoto:

```bash
git remote -v # Verás https://github.com/...
git remote set-url origin git@github.com:tu-usuario/tu-repositorio.git
git remote -v # Ahora verás git@github.com:...
```

### 🛡️ Mantener tu configuración en un entorno volátil (Deepfreeze)
Si trabajas en ordenadores donde la unidad `C:` se borra al reiniciar, guarda tu configuración en `D:` usando estos scripts:

**Script 1: Guardar Configuración (Ejecutar antes de apagar)** - `guardar-config.sh`
```bash
#!/bin/bash
BACKUP_DIR="D:/git_backup"
mkdir -p "$BACKUP_DIR"
cp ~/.gitconfig "$BACKUP_DIR/"
cp -rf ~/.ssh "$BACKUP_DIR/"
echo "Copia de seguridad finalizada en $BACKUP_DIR"
```

**Script 2: Restaurar Configuración (Ejecutar al iniciar sesión)** - `restaurar-config.sh`
```bash
#!/bin/bash
BACKUP_DIR="D:/git_backup"
cp "$BACKUP_DIR/.gitconfig" ~/
cp -rf "$BACKUP_DIR/.ssh" ~/
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
echo "Restauración finalizada"
```

---

## 3. Creación y Clonación de Repositorios

### Iniciar un Repositorio desde Cero
Siempre debes crear primero el repositorio vacío en GitHub.
```bash
mkdir mi-proyecto
cd mi-proyecto
git init
echo "# Mi Proyecto" > README.md
git add .
git commit -m "Commit inicial"
git branch -M main
git remote add origin git@github.com:tu-usuario/mi-proyecto.git
git push -u origin main
```

### Clonar un Repositorio Existente
Al usar `git clone`, te ahorras hacer `git init` y `git remote add origin`, ya que Git lo hace por ti.
```bash
git clone git@github.com:usuario/nombre-repo.git
cd nombre-repo
```

---

## 4. Flujo de Trabajo Básico (Local)

El flujo de trabajo en Git pasa por 3 estados: **Workspace** (Directorio de trabajo) ➔ `add` ➔ **Index/Staging** (Área de preparación) ➔ `commit` ➔ **Repository** (Historial).

### Comandos de Uso Diario
```bash
git status               # Verifica qué archivos están modificados
git add archivo.txt      # Añade un archivo específico al staging
git add .                # Añade TODOS los cambios al staging
git commit -m "Mensaje"  # Guarda los cambios en el historial local
```

### Analizar Diferencias (`git diff`)
*   `git diff`: Compara el directorio de trabajo vs Staging.
*   `git diff --staged`: Compara Staging vs Último Commit.
*   `git diff HEAD`: Muestra todos los cambios locales desde el último commit.

### Ignorar Archivos (`.gitignore`)
Crea un archivo llamado `.gitignore` para listar lo que Git NO debe rastrear (ej. contraseñas, dependencias, compilados).
```bash
# Ejemplo de .gitignore
/logs/*
!logs/.gitkeep  # Ignora toda la carpeta logs, excepto .gitkeep
*.swp
/node_modules
```
*Si ya añadiste un archivo por error y luego lo pones en el `.gitignore`, debes sacarlo de la caché para que Git deje de seguirlo:*
`git rm -r --cached nombre_carpeta/` -> `git commit -m "Eliminar carpeta del rastreo"`

---

## 5. Trabajando con Ramas (Branches)

Las ramas son universos paralelos para tu código. Permiten trabajar en características sin romper la rama `main`.

### Ciclo de vida de una rama
1. **Crear y cambiar a una rama nueva:**
   ```bash
   git switch -c feature/pagina-acerca-de
   # (Atajo para: git branch feature/... + git switch feature/...)
   ```
2. **Trabajar y hacer commits:**
   ```bash
   echo "Contenido" > acerca-de.html
   git add .
   git commit -m "Añade página Acerca de"
   ```
3. **Fusionar (Merge) de vuelta a main:**
   ```bash
   git switch main
   git merge feature/pagina-acerca-de
   ```
4. **Limpieza (Borrar la rama fusionada):**
   ```bash
   git branch -d feature/pagina-acerca-de
   ```

---

## 6. Sincronización Local y Remoto

Cuando trabajas con un remoto (`origin`), tu rama local y la remota pueden divergir.

### Inspeccionar antes de actuar (Recomendado)
```bash
git fetch origin           # Descarga info remota pero NO modifica tu código local
git diff main origin/main  # Compara tu main local con el remoto
```

### Estrategias de Sincronización
*   **Descargar cambios (Pull):** `git pull origin main` (Equivale a `fetch` + `merge`).
*   **Subir cambios (Push):** `git push origin main`. (Usa `-u` la primera vez para vincularlas).

### 💥 Resolviendo Conflictos
Si Git rechaza tu push (`non-fast-forward`), es porque hay cambios en remoto que no tienes en local.

**Opción A: Integrar cambios sin conflictos complejos (`git pull --rebase`)**
Reescribe tu historial poniendo tus commits *después* de los del servidor, creando una línea limpia.
```bash
git pull --rebase origin main
git push origin main
```

**Opción B: Fusión estándar con conflictos (`git pull`)**
Si editaste la misma línea que tu compañero:
1. Haces `git pull origin main`. Git dirá `CONFLICT (content)`.
2. Abres el archivo con conflicto en tu editor. Verás algo así:
   ```text
   <<<<<<< HEAD
   <h1>Título Local</h1>
   =======
   <h1>Título Remoto</h1>
   >>>>>>> origin/main
   ```
3. Borras los marcadores (`<<<`, `===`, `>>>`) y dejas la línea como deba quedar final (ej. `<h1>Título Combinado</h1>`).
4. Lo marcas como resuelto y creas el commit de fusión:
   ```bash
   git add archivo.html
   git commit -m "Resuelve conflicto de fusión en título"
   git push origin main
   ```

**Opción C: Opción Nuclear (Descartar todo lo tuyo y quedarte con el remoto)**
*¡CUIDADO! Perderás tus cambios locales.*
```bash
git fetch origin
git reset --hard origin/main
```

---

## 7. Deshacer Cambios y Viajar en el Tiempo

Git es como un "Ctrl+Z" con superpoderes.

### 1. `git restore` (Descartar cambios locales no guardados)
*   **Si no hiciste `git add`:** `git restore archivo.txt` (Vuelve al estado del último commit).
*   **Si hiciste `git add`:** `git restore --staged archivo.txt` (Saca el archivo del staging, pero mantiene tus modificaciones en el código).

### 2. `git commit --amend` (Corregir el último commit local)
Reemplaza el commit anterior. Ideal para cambiar el mensaje o añadir un archivo olvidado. **No lo uses si ya hiciste push.**
```bash
git add archivo_olvidado.txt
git commit --amend -m "Mensaje corregido"
```

### 3. `git reset` (Viajar al pasado localmente)
Mueve el puntero HEAD hacia atrás. **No lo uses si ya hiciste push.**
*   `git reset --soft HEAD~1`: Borra el commit pero deja los cambios listos en el Staging.
*   `git reset HEAD~1` (o `--mixed`): Borra el commit y deja los cambios en tu directorio de trabajo.
*   `git reset --hard HEAD~1`: 🚨 Borra el commit y **ELIMINA** todos los cambios.

### 4. `git revert` (Deshacer de forma segura en equipo)
Crea un **nuevo commit** que aplica los cambios inversos. Es la forma profesional de arreglar errores en commits que ya están subidos al servidor remoto.
```bash
git log --oneline # Copias el HASH del commit malo (ej. d4e5f6g)
git revert d4e5f6g
git push origin main
```

### 5. `git reflog` (El salvavidas definitivo)
Guarda un historial de absolutamente todos los movimientos (incluso los borrados por `reset --hard`).
```bash
git reflog
# Buscas el punto al que quieres volver (ej. HEAD@{2})
git reset --hard HEAD@{2}
```

---

## 8. Herramientas Avanzadas y Casos Especiales

### Guardar Trabajo Temporalmente (`git stash`)
Si estás a medias de algo y necesitas cambiar de rama urgentemente sin hacer un commit a medias:
```bash
git stash          # Guarda tus cambios en un "cajón" temporal y limpia tu directorio
# ... cambias de rama, haces lo que necesites, y vuelves ...
git stash pop      # Saca los cambios del cajón y los vuelve a aplicar en tu código
git stash list     # Ver todos los stashes guardados
git stash drop     # Eliminar un stash específico
```

### Combinar Commits (`git rebase -i` / Squash)
Limpia tu historial antes de subirlo, combinando 5 commits pequeños (ej. "wip", "typo") en uno solo.
```bash
git rebase -i HEAD~5
# Cambia 'pick' por 'squash' (o 's') en los commits que quieras fusionar hacia arriba.
```

### Archivos Pesados (Git LFS)
Si vas a subir ISOs, vídeos o datasets gigantes que romperían Git (>100MB):
```bash
git lfs install
git lfs track "*.iso"
git add .gitattributes
git add archivo_grande.iso
git commit -m "Añadir ISO grande mediante LFS"
```

### Problemas de Fin de Línea (CRLF vs LF)
Si al ejecutar scripts `.sh` en Git Bash te da error `$'\r': command not found`, es porque Windows añade retornos de carro. Soluciónalo con:
```bash
dos2unix scripts/*.sh
# O usando sed: sed -i 's/\r$//' scripts/*.sh
```

---

## 9. Flujos de Trabajo en Equipo y Colaboración

### A. Colaborar en Proyectos Open Source (Fork & Pull Request)
1. **Fork:** Ve a GitHub y haz clic en "Fork" en el repo original.
2. **Clonar tu Fork:** `git clone https://github.com/TU_USUARIO/REPO.git`
3. **Rama nueva:** `git switch -c mi-mejora`
4. **Trabajar y Subir:** Haces commits y `git push origin mi-mejora`
5. **Pull Request:** Ve a GitHub, entra en tu fork y haz clic en "Compare & pull request" para enviar tu propuesta al dueño original.

### B. Flujo de Equipo Pequeño (Con Supervisores)
1. **Preparación (Supervisor):** Crea el repo y añade miembros en *Settings > Collaborators*.
2. **Sincronizar (Tú):** SIEMPRE antes de empezar el día: `git switch main` y `git pull origin main`.
3. **Tu trabajo:** `git switch -c tu-nombre/nueva-tarea`, haces commits.
4. **Subir tu rama:** `git push origin tu-nombre/nueva-tarea`.
5. **Integración:** En GitHub abres un Pull Request. El supervisor lo revisa, resuelve conflictos si los hay, y aprueba (Merge).

---

## 10. Hoja de Referencia Rápida (Cheat Sheet)

### Básicos
| Acción | Comando |
| :--- | :--- |
| Inicializar | `git init` |
| Ver estado | `git status` |
| Preparar todo | `git add .` |
| Confirmar | `git commit -m "Mensaje"` |
| Historial gráfico | `git log --oneline --graph --all` |

### Ramas
| Acción | Comando |
| :--- | :--- |
| Listar ramas | `git branch` |
| Crear y cambiar | `git switch -c <rama>` |
| Fusionar | `git merge <rama>` |
| Eliminar local | `git branch -d <rama>` |

### Remotos y Sincronización
| Acción | Comando |
| :--- | :--- |
| Ver remotos | `git remote -v` |
| Añadir remoto | `git remote add origin <url>` |
| Descargar sin fusionar | `git fetch origin` |
| Descargar y fusionar | `git pull origin main` |
| Subir nueva rama | `git push -u origin <rama>` |
| Forzar subida | `git push --force-with-lease` |

### Deshacer y Rescate
| Acción | Comando |
| :--- | :--- |
| Descartar edición local | `git restore <archivo>` |
| Sacar del Staging | `git restore --staged <archivo>` |
| Cambiar último commit | `git commit --amend` |
| Borrar último commit (cód. intacto) | `git reset HEAD~1` |
| Deshacer subido (seguro) | `git revert <hash>` |
| Rescate total (Historial) | `git reflog` |