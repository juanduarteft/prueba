# Guía: agregar repositorio remoto y trabajar con rama demo

Esta guía es el paso a paso para instanciar una rama remota nueva (ejemplo con `demo`),
agregarle cambios y subirla al `main` de `demo`.

Contexto actual de `devprueba`:
- `Branches > main` -> sigue a `dev/main` (desarrollo diario, repo `devprueba.git`)
- `Branches > pro` -> sigue a `pro/pro` (copia local de prod, repo `prueba.git`)
- `Branches > demo` -> creada desde `main`, aún sin `fetch` de `demo`
- `Remotes`:
  - `dev` = `https://github.com/juanduarteft/devprueba.git`
  - `pro` = `https://github.com/juanduarteft/prueba.git`
  - `demo` = `https://github.com/juanduarteft/demoprueba.git`

> Idea clave: `remoto` = apodo de un repo en GitHub. `rama local` = donde editas.
> `demo/main` (remote-tracking) = foto solo-lectura de cómo está `main` en GitHub.
> Nunca commiteas directo en `demo/main`, siempre en tu local `demo`.

---

## 1. Agregar el remoto (una sola vez por remoto)

Ya lo tienes hecho para `demo`, pero este es el comando para repetirlo:

Fork:
1. `Remotes >右击 / clic derecho > Add Remote`
2. Name: `demo` (simple, sin `/`, sin `main`)
3. URL: `https://github.com/juanduarteft/demoprueba.git`
4. `Fetch` marcado.

Consola:
```powershell
git remote add demo https://github.com/juanduarteft/demoprueba.git
git remote -v
# debe listar dev, pro, demo
```

Mal: llamar al remoto `demo/main`. Eso crea carpeta `demo/main` y confunde con rama.

## 2. Traer la foto del remoto (fetch)

Fork:
1. Click derecho en `Remotes > demo` > `Fetch`
2. Debe aparecer `Remotes > demo > main`

Consola:
```powershell
git fetch demo
# ver qué trajo:
# en Fork verás demo/main, en consola no hay output si ya estaba al día
```

Si `refs/remotes/demo` no existe es que aún no hiciste fetch. Hazlo ahora.

## 3. Crear la rama local que representa a demo

Regla: 1 rama local por entorno que manipulas.

Tienes `demo` local creada desde `main` (`ed0b41`), no desde `demo/main`.
Para dejarla bien enlazada tienes 2 opciones:

Opción A - recomendada si `demo` local aún no tiene trabajo propio:
```powershell
git checkout main
git branch -D demo
git checkout -b demo demo/main
git branch -vv
# debe decir: demo ... [demo/main]
```

Opción B - si ya tienes commits en `demo` local que no quieres perder:
```powershell
git checkout demo
git fetch demo
git branch -u demo/main
git merge demo/main --allow-unrelated-histories
# solo la primera vez pide --allow-unrelated-histories porque
# devprueba y demoprueba nacieron como repos separados
```

Fork equivalente opción B:
1. Parado en `demo`, click derecho en `Remotes > demo > main` > `Set Upstream / Track`
2. Luego click derecho > `Merge demo/main into demo`

## 4. Agregar cambios a la branch demo

Siempre parado en `demo` (✓ en `Branches > demo`).

1. Ver estado:
```powershell
git branch --show-current
# debe decir demo
git status
```

2. Edita / crea archivos. Verás en Fork `Local Changes`.

3. Si es trabajo a medias y necesitas cambiar de rama sin commitear (stash):
```powershell
git stash push -m "wip demo"
git stash list
git stash pop
# pop = recupera y borra de la lista
# apply = recupera y lo mantiene
```

4. Si ya está listo (commit + push):
```powershell
git add .
git commit -m "add: cambio para agregar demo"
git log --oneline -5
git status
```

Fork: `Changes > Stage > Commit`. Mensaje corto en imperativo.

## 5. Subir demo a su main (push correcto)

Este fue el error que tuvimos con `pro`: pusheamos a `To: pro` y se creó `pro/pro`
en vez de actualizar `pro/main`.

Fork, estando en `demo`:
1. `Push`
2. Revisa los 3 campos:
   - `Branch: demo` (origen local)
   - `Remote: demo` (repo demoprueba)
   - `To: main` (¡solo `main`! NO `demo`, NO `demo/main`)
3. `Create tracking reference` marcado la primera vez.
4. `Force push` DESMARCADO.
5. `Push`.

Consola:
```powershell
git checkout demo
git push -u demo demo:main
# siguientes veces solo:
git push
```

Verificación:
- En `All Commits`, `demo` y `demo/main` quedan en el mismo commit.
- Ya no ves dos líneas separadas.
- Borra la rama sobrante si creaste `demo/demo` por error:
```powershell
git push demo --delete demo
git fetch demo --prune
```

## 6. Pasar cambios de dev (main) a demo

Flujo normal: trabajas en `main` (dev), cuando está listo lo fusionas en `demo`.

```powershell
git checkout demo
git merge main
# primera vez entre repos distintos:
git merge main --allow-unrelated-histories
git push demo demo:main
```

Fork:
1. Parado en `demo`
2. Click derecho en `Branches > main` > `Merge main into demo`
3. Resuelve conflictos si pide, completa merge commit
4. `Push` con `To: main` como en paso 5.

No hagas `force push` para esto. Si te pide force es señal de que falta el merge del paso 6, no de que debas forzar. Forzar borra `cm1 prod` / historial de demo en GitHub.

---

## Resumen rápido

| Quiero... | Comando |
|---|---|
| ver dónde estoy | `git branch --show-current` + `git status` |
| agregar remoto | `git remote add demo <url>` |
| traer foto | `git fetch demo` |
| crear local desde remoto | `git checkout -b demo demo/main` |
| guardar temporal | `git stash push -m "wip"` / `git stash pop` |
| guardar definitivo local | `git add .` + `git commit -m "..."` |
| subir a main de demo | `git push demo demo:main` |
| unir dev en demo | `git checkout demo` + `git merge main` |

## Checklist demo

- [ ] `git remote -v` muestra `demo`
- [ ] `Fetch demo` crea `demo/main`
- [ ] local `demo` trackea `demo/main` (`git branch -vv`)
- [ ] `Local Changes` vacío antes de merge
- [ ] merge `main` o `demo/main` en `demo` con merge commit
- [ ] `Push` con `To: main`, sin force
- [ ] `demo` y `demo/main` en mismo commit
