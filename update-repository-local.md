# Guía: actualizar pro y demo desde dev main (flujo repetible)

Este es el paso a paso que siempre repites cuando ya hiciste un `commit` en dev
y quieres llevarlo a producción (`pro`) o a demo (`demo`).

Tu frase resumida es correcta:
> hago commit en dev main y push a dev main, luego me paso a pro,
> traigo lo de dev main (merge), y pusheo a pro main. Lo mismo para demo.

Matiz: lo de “pull de dev main” es en realidad un `merge`,
porque `main` es una rama **local**. `pull` solo se usa contra **remotos**.

Contexto:
- `Branches > main` = dev local, sigue a `dev/main`
- `Branches > pro` = copia local de prod, sigue a `pro/main`
- `Branches > demo` = copia local de demo, sigue a `demo/main`
- Remotos: `dev` = devprueba.git, `pro` = prueba.git, `demo` = demoprueba.git

---

## Paso 0. Punto de partida: commit en dev

Estando en `main`:

Fork:
1. Verifica `Branches > main` con ✓.
2. `Local Changes` > revisa archivos > `Stage` > `Commit` con mensaje claro.
3. `Push` con `Branch: main / Remote: dev / To: main`.

Consola:
```powershell
git checkout main
git status
git add .
git commit -m "add: mi cambio dev"
git push dev main:main
# si ya tiene upstream a dev/main, basta: git push
```

Hasta aquí `pro/main` y `demo/main` no se enteran. Es normal.

Si tienes trabajo a medias y necesitas cambiar de rama sin commitear:
```powershell
git stash push -m "wip dev"
git checkout pro
# ... luego al volver:
git checkout main
git stash pop
```

## Paso 1. Pasar dev -> pro

Siempre parado en `pro`:

1. Limpieza: `Local Changes` debe estar vacío en `pro`. Si no, `stash` o `commit`.
2. Opcional pero recomendado: `Fetch` en `Remotes > pro` para actualizar la foto `pro/main`.
   Si `pro/main` avanzó en GitHub, primero `Merge pro/main into pro`.
3. Traer dev: click derecho en `Branches > main` > `Merge main into pro`.
4. Resolver merge commit / conflictos si pide.
5. Probar.
6. `Push` con `Branch: pro / Remote: pro / To: main` (solo `main`, NO `pro`, NO `pro/main`).
   `Force push` desmarcado.

Consola:
```powershell
git checkout pro
git status
git fetch pro
git merge pro/main
git merge main
git push pro pro:main
```

Verificación: en `All Commits`, `pro` y `pro/main` quedan en el mismo commit.

## Paso 2. Lo mismo para demo

Idéntico, solo cambia el entorno:

Fork:
1. `Checkout` a `demo`.
2. `Local Changes` vacío.
3. `Fetch` en `Remotes > demo`.
4. Click derecho en `Branches > main` > `Merge main into demo`.
5. `Push` con `Branch: demo / Remote: demo / To: main`.

Consola:
```powershell
git checkout demo
git status
git fetch demo
git merge demo/main
git merge main
git push demo demo:main
```

Verificación: `demo` y `demo/main` en el mismo commit. Si por error creaste
`demo/demo` (como pasó con `pro/pro`), bórrala:
```powershell
git push demo --delete demo
git fetch demo --prune
```

## Resumen para memorizar

```
# dev
checkout main -> add -> commit -> push dev main:main

# prod (repetir siempre igual)
checkout pro -> merge main -> push pro pro:main

# demo (repetir siempre igual)
checkout demo -> merge main -> push demo demo:main
```

Reglas:
- Nunca commiteas en `pro/main` o `demo/main` (son fotos solo-lectura).
- Nunca `To: pro` o `To: demo`, siempre `To: main`.
- Nunca `force push` para esto. Si Git lo pide es señal de que falta un `merge`, no de que debas forzar.
