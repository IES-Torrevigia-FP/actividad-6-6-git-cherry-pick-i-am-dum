# 🍒 Actividad 6.6 – `git cherry-pick`: seleccionar commits concretos

> En las actividades anteriores has trabajado con ramas, merge y rebase.
> Ahora vas a aprender a usar **`git cherry-pick` para aplicar commits concretos de una rama en otra, sin mezclar todo el historial**.

---

## 🎯 Objetivos de la actividad

Al finalizar esta actividad serás capaz de:

- Explicar qué es `git cherry-pick` y para qué se usa.
- Cherry‑pickear uno o varios commits de una rama a otra.
- Resolver un conflicto simple durante un cherry‑pick.
- Entender en qué situaciones es apropiado usar cherry‑pick y por qué debe usarse con moderación.

---

## 🧠 Conceptos clave: ¿qué es `git cherry-pick`?

- `git cherry-pick <hash>` toma **un commit concreto** de otra rama y aplica sus cambios en la rama actual, creando un **nuevo commit** con un hash distinto.
- Se usa cuando quieres **reutilizar cambios específicos** sin traer toda la rama (por ejemplo, aplicar un hotfix a producción o corregir un commit hecho en la rama equivocada).
- Debe usarse **con cuidado**, porque puede duplicar commits y complicar el historial si se abusa.

---

## 🧩 Parte 1 – Preparar el escenario

### Reto 1.1 – Crear un repositorio con rama de desarrollo y rama de producción

1. Crea una carpeta para esta actividad:

   ```bash
   mkdir actividad-6-6-cherry-pick
   cd actividad-6-6-cherry-pick
   ```

2. Inicializa el repositorio:

   ```bash
   git init
   ```

3. Crea el archivo `app.txt` con:

   ```text
   Versión 1.0 de la aplicación.
   ```

4. Añade y haz commit en `main`:

   ```bash
   git add app.txt
   git commit -m "Crea versión 1.0 de la app"
   ```

5. Crea una rama `develop` para desarrollo:

   ```bash
   git checkout -b develop
   ```

6. En `develop`, añade una línea:

   ```bash
   echo "Funcionalidad A: en desarrollo" >> app.txt
   git add app.txt
   git commit -m "Añade funcionalidad A en develop"
   ```

7. Añade otra funcionalidad:

   ```bash
   echo "Funcionalidad B: en desarrollo" >> app.txt
   git add app.txt
   git commit -m "Añade funcionalidad B en develop"
   ```

8. Añade un hotfix (arreglo crítico) en `develop`:

   ```bash
   echo "HOTFIX: corrige error crítico en login" >> app.txt
   git add app.txt
   git commit -m "HOTFIX: corrige error crítico en login"
   ```

Ahora `develop` tiene varios commits, incluido uno de **hotfix** que te interesa llevar a `main` sin traer todas las funcionalidades A y B.

### Checkpoint 1 – Evidencia

- Crea `cp1-historial-develop.txt` con:
  - La salida de `git log --oneline --all`.
  - Marca (con un comentario en el archivo) cuál es el commit del **HOTFIX** que quieres cherry‑pickear.

---

## 🧩 Parte 2 – Cherry‑pick de un único commit

### Reto 2.1 – Aplicar solo el HOTFIX en `main`

1. Vuelve a la rama `main`:

   ```bash
   git checkout main
   ```

2. Comprueba el contenido actual de `app.txt` (no debería tener las funcionalidades A y B ni el HOTFIX).

3. En `develop`, identifica el **hash** del commit HOTFIX (por ejemplo `abc1234`) usando:

   ```bash
   git log --oneline develop
   ```

4. En `main`, ejecuta:

   ```bash
   git cherry-pick <hash_del_hotfix>
   ```

   Esto aplicará los cambios de ese commit en `main` y creará un nuevo commit con un hash distinto.

5. Comprueba:

   ```bash
   git log --oneline
   cat app.txt
   ```

   Verás que ahora `main` tiene un commit con el HOTFIX, pero **sin** la funcionalidad A ni B.

### Checkpoint 2 – Evidencia

- Crea `cp2-cherry-pick-simple.txt` con:
  - La salida de `git log --oneline` en `main`.
  - El contenido de `app.txt` en `main` tras el cherry‑pick.
  - Una frase explicando por qué esto es útil para aplicar hotfixes.

---

## 🧩 Parte 3 – Cherry‑pick de varios commits

### Reto 3.1 – Traer funcionalidades seleccionadas a otra rama

Supón que tienes una rama `test` donde solo quieres probar la **funcionalidad B** y el **HOTFIX**, pero no la funcionalidad A completa.

1. Desde `main`, crea la rama `test`:

   ```bash
   git checkout -b test
   ```

2. Consulta el log de `develop` para obtener los hashes de:
   - Commit de **funcionalidad B**.
   - Commit de **HOTFIX** (si no lo tienes ya anotado).

   ```bash
   git log --oneline develop
   ```

3. En `test`, cherry‑pickea **ambos commits** (puedes hacerlo en dos comandos seguidos):

   ```bash
   git cherry-pick <hash_funcionalidad_B> <hash_hotfix>
   ```

   o uno a uno:

   ```bash
   git cherry-pick <hash_funcionalidad_B>
   git cherry-pick <hash_hotfix>
   ```

   Cherry‑pick aplica los cambios de esos commits, creando commits nuevos en la rama `test`.

4. Comprueba:

   ```bash
   git log --oneline
   cat app.txt
   ```

   Deberías ver en `test` la funcionalidad B y el HOTFIX, pero no la funcionalidad A si no la has cherry‑pickeado.

### Checkpoint 3 – Evidencia

- Crea `cp3-multiples-commits.txt` con:
  - La salida de `git log --oneline` en `test`.
  - El contenido de `app.txt` en `test`.
  - Una explicación breve de la diferencia entre:
    - Traer una rama entera con `merge`.
    - Traer commits concretos con `cherry-pick`.

---

## 🧩 Parte 4 – Cherry‑pick con conflicto

### Reto 4.1 – Provocar un conflicto al cherry‑pickear

1. Sitúate en `develop` y modifica `app.txt` en la misma zona donde está el HOTFIX, pero con texto diferente:

   ```bash
   git checkout develop
   echo "HOTFIX: cambio alternativo en login (develop)" >> app.txt
   git add app.txt
   git commit -m "Modifica de nuevo la zona del hotfix en develop"
   ```

2. Vuelve a `main` o `test` (elige una rama que ya tenga el HOTFIX cherry‑pickeado antes):

   ```bash
   git checkout main # o test
   ```

3. Intenta cherry‑pickear el nuevo commit de `develop` que tocaba la misma zona:

   ```bash
   git cherry-pick <hash_del_ultimo_commit_develop>
   ```

Es probable que Git muestre un **conflicto** porque ambas ramas han tocado la misma parte de `app.txt`.

4. Ejecuta:

   ```bash
   git status
   ```

   Verás el cherry‑pick en curso y el archivo en conflicto.

5. Abre `app.txt` y localiza los marcadores de conflicto (`<<<<<<<`, `=======`, `>>>>>>>`).

### Reto 4.2 – Resolver conflicto y continuar

1. Edita `app.txt` para resolver el conflicto dejando un texto coherente (por ejemplo, combinando información de ambas versiones).
2. Elimina los marcadores de conflicto.
3. Marca el archivo como resuelto:

   ```bash
   git add app.txt
   ```

4. Continúa el cherry‑pick:

   ```bash
   git cherry-pick --continue
   ```

   Si cambias de opinión, podrías usar:

   ```bash
   git cherry-pick --abort
   ```

   para cancelar la operación.

5. Comprueba:

   ```bash
   git status
   git log --oneline
   ```

### Checkpoint 4 – Evidencia

- Crea `cp4-conflicto-cherry-pick.txt` con:
  - La salida de `git status` durante el conflicto.
  - Una descripción de cómo has resuelto el conflicto y si has usado `--continue` o `--abort`.

---

## 🧩 Parte 5 – Buenas prácticas y cuándo usarlo

### Reto 5.1 – Reflexión final

Crea `reflexion-6-6.md` y responde:

1. Explica con tus palabras qué hace `git cherry-pick` y en qué se diferencia de `merge` y `rebase`.
2. Indica **tres situaciones reales** donde cherry‑pick es una buena idea (por ejemplo, hotfix en producción, commit en rama equivocada, reutilizar un pequeño cambio en varias ramas).
3. ¿Por qué se recomienda usar `git cherry-pick` **con moderación** y no como herramienta principal para integrar ramas?

---

## 📦 Entrega esperada en el repositorio

Tu repositorio de la Actividad 6.6 debe incluir:

- `app.txt` (versión final en la rama que indique el profesorado).
- `cp1-historial-develop.txt`
- `cp2-cherry-pick-simple.txt`
- `cp3-multiples-commits.txt`
- `cp4-conflicto-cherry-pick.txt`
- `reflexion-6-6.md`

---

## 🏁 Evaluación sugerida

- 4 pts – Uso correcto de `git cherry-pick` con commits únicos y múltiples.
- 3 pts – Gestión adecuada de conflictos durante cherry‑pick (`--continue` / `--abort`).
- 3 pts – Calidad de la reflexión en `reflexion-6-6.md` (comprensión de cuándo y por qué usar cherry‑pick y sus riesgos).

**Puntuación máxima: 10 puntos.**

---

## 🤖 Evaluación automática y redacción de informe

Esta actividad incluye una evaluación automática similar a la de la actividad anterior, adaptada a `cherry-pick` y resolución de conflictos:

- Workflow: `.github/workflows/evaluacion.yml`
- Script de evaluación: `tools/evaluar_repo.py`
- Script de redacción IA: `tools/ai_redactor.py`

### ¿Qué genera?

Al ejecutarse en GitHub Actions, se generan los siguientes archivos en `reportes/`:

- `informe.md` (resumen técnico de evaluación)
- `informe.json` (métricas en formato estructurado)
- `metricas.csv` (puntuaciones por criterio)
- `arbol.txt` (resumen del árbol del repositorio)

Si hay credenciales de IA configuradas en Secrets/Variables, también se generan:

- `retro.md` (retroalimentación redactada por IA)
- `retro.docx` (versión formal en Word)

### Parámetros del workflow (`workflow_dispatch`)

- `min_commits` (por defecto: `6`)
- `required_files` (por defecto: `app.txt,cp1-historial-develop.txt,cp2-cherry-pick-simple.txt,cp3-multiples-commits.txt,cp4-conflicto-cherry-pick.txt,reflexion-6-6.md`)

---
