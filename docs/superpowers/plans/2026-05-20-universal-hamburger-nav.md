# Re3Move Landing — Navegación universal con hamburguesa: Plan de Implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Sustituir la barra de enlaces inline + botón CTA del top-bar por una hamburguesa universal (todos los tamaños, las 4 páginas) que abre un panel plano con 9 ítems iguales en todas las páginas, marcando con `aria-current="page"` el activo.

**Architecture:** Estática (HTML/CSS, sin framework). El cambio toca `assets/styles.css` (reglas base + limpiar dead code) y las 4 páginas HTML (eliminar la barra inline + reescribir el contenido del panel `.mobile-menu`). Sin commits intermedios — un único commit al final (el menú quedaría inconsistente entre páginas si se commitea parcial). Push y redeploy igual que el commit anterior: fast-forward a `origin/main`, Railway redespliega.

**Tech Stack:** HTML estático, CSS, vanilla JS, nginx Docker (sin cambios). Sin tests (no hay framework; verificación = `grep` + serve + curl).

**Spec:** `docs/superpowers/specs/2026-05-20-universal-hamburger-nav-design.md`

---

## Datos compartidos (usar literalmente)

**Markup del panel `.mobile-menu` — mismo en las 4 páginas.** Solo varía: (a) el `id` del `<div>` raíz, que debe conservar el de cada página (`mobileMenu` para index/clinica-deportiva, `mobile-menu` para fisioterapia-avanzada/mujer), (b) qué item lleva `aria-current="page"`. Reemplazar TODO el contenido interno actual del `<div class="mobile-menu" id="...">…</div>` por el siguiente bloque (manteniendo el `<div class="mobile-menu" id="..." role="dialog" aria-label="Menú de navegación">` como wrapper en mujer y fisio — y añadiéndolos si no los tiene en index/clinica-deportiva):

```html
  <a href="/" data-nav="inicio">Inicio</a>
  <a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada">Fisioterapia avanzada</a>
  <a href="/clinica-deportiva" data-nav="clinica-deportiva">Clínica deportiva</a>
  <a href="/mujer" data-nav="mujer">Mujer</a>
  <a href="/#metodologia">Método</a>
  <a href="/#equipo">Equipo</a>
  <a href="/#ubicacion">Ubicación</a>
  <a href="/#contacto">Contacto</a>
  <a href="https://wa.me/34620326447" target="_blank" rel="noopener" class="mm-cta">Pedir cita</a>
```

Por página, añadir `aria-current="page"` al ancla correspondiente:
- `index.html` → en el `<a href="/" data-nav="inicio">` → `aria-current="page"`.
- `clinica-deportiva.html` → en el `<a href="/clinica-deportiva" data-nav="clinica-deportiva">` → `aria-current="page"`.
- `fisioterapia-avanzada.html` → en el `<a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada">` → `aria-current="page"`.
- `mujer.html` → en el `<a href="/mujer" data-nav="mujer">` → `aria-current="page"`.

**Wrappers `role="dialog"`:** En index.html y clinica-deportiva.html el `<div class="mobile-menu" id="mobileMenu">` actualmente NO tiene `role="dialog"`. Añadirlo, igualando el patrón de mujer/fisio: `<div class="mobile-menu" id="mobileMenu" role="dialog" aria-label="Menú de navegación">`.

**Listener de Escape (JS) — añadir en cada página al final del script existente, adaptando los nombres de las variables (algunas páginas usan `mobileMenu`/`hamburger` globales; fisio/mujer usan IIFE con `var menu`/`var btn`).** Patrón canónico para index/clinica-deportiva (variables globales):

```js
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && mobileMenu.classList.contains('open')) {
    mobileMenu.classList.remove('open');
    document.getElementById('hamburger').setAttribute('aria-expanded', 'false');
    document.getElementById('hamburger').focus();
  }
});
```

Para fisio/mujer (dentro de la misma IIFE donde `btn` y `menu` están en scope, justo antes del cierre `})();`):

```js
document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && menu.classList.contains('open')) {
    menu.classList.remove('open');
    btn.setAttribute('aria-expanded', 'false');
    btn.focus();
  }
});
```

---

## Task 1: CSS — hamburguesa universal + estilo de item activo + limpiar dead code

**Files:**
- Modify: `public/assets/styles.css`

- [ ] **Step 1: Mostrar hamburguesa siempre y eliminar la barra inline**

Borrar las 3 reglas de `.nav-links` en `public/assets/styles.css`:
- L60: `.nav-links { display: flex; align-items: center; gap: 32px; }`
- L61: `.nav-links > a, .nav-links > .nav-dd > .nav-dd-label { ... }`
- L62: `.nav-links > a:hover, .nav-links > .nav-dd:hover > .nav-dd-label { ... }`

Cambiar la regla `.hamburger` (L79) de `display: none` a `display: flex; align-items: center; justify-content: center;`:

Antes:
```css
.hamburger { display: none; background: none; border: none; color: var(--ink); padding: 4px; cursor: pointer; }
```
Después:
```css
.hamburger { display: flex; align-items: center; justify-content: center; background: none; border: none; color: var(--ink); padding: 4px; cursor: pointer; }
```

En el media query `@media (max-width: 900px)` (alrededor de L161–164), borrar las dos líneas:
```
  .nav-links { display: none; }
  .hamburger { display: flex; align-items: center; justify-content: center; }
```
Si el bloque del media query queda vacío, borrar el `@media` completo. Si tiene otras reglas (p.ej. `.footer-top { grid-template-columns: 1fr 1fr; }`), conservarlas.

- [ ] **Step 2: Borrar reglas de acordeón obsoletas**

Borrar de `public/assets/styles.css`:
- `.mm-caret { ... }` (L120)
- `.mm-group.open > .mm-group-label .mm-caret { ... }` (L121)
- `.mm-submenu { ... }` (L122)
- `.mm-submenu > div { ... }` (L123)
- `.mm-group.open > .mm-submenu { ... }` (L124)
- `.mm-submenu a { ... }` (L125, regla multi-línea — borrar el bloque completo hasta su `}`)
- `.mm-submenu a small { ... }` (L133)

En la regla compuesta `.mobile-menu a, .mm-group > .mm-group-label { ... }` (L105) y `.mobile-menu a:hover, .mm-group > .mm-group-label:hover { ... }` (L119), eliminar la parte `, .mm-group > .mm-group-label` y `, .mm-group > .mm-group-label:hover` respectivamente (conservar la regla, solo dropear el selector co-seleccionado obsoleto).

- [ ] **Step 3: Añadir estilo de item activo**

Añadir al final de `public/assets/styles.css`:

```css
/* ── Item activo del menú ── */
.mobile-menu a[aria-current="page"] { font-weight: 700; color: var(--ink); background: rgba(45,122,79,0.06); }
```

- [ ] **Step 4: Verificar**

```bash
grep -n 'nav-links\|mm-group\|mm-submenu\|mm-caret' public/assets/styles.css
```
Esperado: sin resultados.

```bash
grep -n '^\.hamburger ' public/assets/styles.css
```
Esperado: una línea cuya declaración empieza con `display: flex`.

```bash
grep -n 'aria-current="page"' public/assets/styles.css
```
Esperado: una línea con la regla `.mobile-menu a[aria-current="page"]`.

(NO commit todavía. El commit único es Task 7.)

---

## Task 2: index.html — quitar nav inline + reescribir panel + JS Esc + active

**Files:**
- Modify: `public/index.html`

- [ ] **Step 1: Eliminar `<div class="nav-links">…</div>`**

Localizar `<div class="nav-links">` (alrededor de L192) y borrar todo el bloque hasta su `</div>` de cierre. Conservar el `</div>` que cierra `.nav-inner` y el resto del nav. Verificar con un grep que la estructura sigue siendo válida.

- [ ] **Step 2: Eliminar el botón `<a class="nav-cta">…</a>`**

Localizar `<a href="https://wa.me/34620326447" target="_blank" class="nav-cta">…</a>` (alrededor de L238–241, dentro de `<div class="nav-right">`) y borrarlo completo (incluido el `<svg>` interno y el texto "Pedir cita"). Conservar el `<div class="nav-right">` con el `<button class="hamburger">` dentro.

- [ ] **Step 3: Añadir `role="dialog"` y `aria-label` al wrapper del panel**

En `<div class="mobile-menu" id="mobileMenu">` (alrededor de L251) cambiar a:
```html
<div class="mobile-menu" id="mobileMenu" role="dialog" aria-label="Menú de navegación">
```

- [ ] **Step 4: Reescribir el contenido del panel**

Reemplazar TODO el contenido interno entre `<div class="mobile-menu" id="mobileMenu" role="dialog" aria-label="Menú de navegación">` y su `</div>` de cierre por exactamente:

```html
  <a href="/" data-nav="inicio" aria-current="page">Inicio</a>
  <a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada">Fisioterapia avanzada</a>
  <a href="/clinica-deportiva" data-nav="clinica-deportiva">Clínica deportiva</a>
  <a href="/mujer" data-nav="mujer">Mujer</a>
  <a href="/#metodologia">Método</a>
  <a href="/#equipo">Equipo</a>
  <a href="/#ubicacion">Ubicación</a>
  <a href="/#contacto">Contacto</a>
  <a href="https://wa.me/34620326447" target="_blank" rel="noopener" class="mm-cta">Pedir cita</a>
```

- [ ] **Step 5: Limpiar el JS del menú móvil**

En el `<script>` al final del body, borrar el bloque `document.querySelectorAll('.mm-group-label').forEach(el => { ... });` (toggling de acordeones — ya no aplica). El resto del script (scroll-reveal, toggle hamburguesa, click-outside, cierre al hacer click en enlaces) se conserva intacto.

- [ ] **Step 6: Añadir listener de Escape**

Inmediatamente después del bloque que cierra el menú al hacer click en `.mobile-menu a` (donde está `document.querySelectorAll('.mobile-menu a').forEach(...)`), añadir:

```js
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && mobileMenu.classList.contains('open')) {
    mobileMenu.classList.remove('open');
    document.getElementById('hamburger').setAttribute('aria-expanded', 'false');
    document.getElementById('hamburger').focus();
  }
});
```

- [ ] **Step 7: Verificar**

```bash
grep -n 'class="nav-links"\|class="nav-cta"' public/index.html
```
Esperado: sin resultados.

```bash
grep -n '>Inicio<\|>Fisioterapia avanzada<\|>Clínica deportiva<\|>Mujer<\|>Método<\|>Equipo<\|>Ubicación<\|>Contacto<\|class="mm-cta"' public/index.html
```
Esperado: cada uno presente al menos una vez (los `>Inicio<` etc. dentro del panel).

```bash
grep -n 'aria-current="page"' public/index.html
```
Esperado: 1 (el ítem Inicio).

```bash
grep -n 'mm-group-label\|mm-submenu\|mm-caret' public/index.html
```
Esperado: sin resultados.

```bash
grep -c '<main id="main"' public/index.html
```
Esperado: 1 (no se debe haber roto el `<main>` ni el resto de la estructura).

---

## Task 3: clinica-deportiva.html — quitar nav inline + reescribir panel + JS Esc + active

**Files:**
- Modify: `public/clinica-deportiva.html`

- [ ] **Step 1: Eliminar `<div class="nav-links">…</div>`** alrededor de L260 (bloque completo hasta su `</div>`).

- [ ] **Step 2: Eliminar el botón `<a class="nav-cta">Pedir cita</a>`** alrededor de L304–307 (incluido el `<svg>` interno).

- [ ] **Step 3: Añadir `role="dialog"` y `aria-label` al panel**

En `<div class="mobile-menu" id="mobileMenu">` (alrededor de L316) cambiar a:
```html
<div class="mobile-menu" id="mobileMenu" role="dialog" aria-label="Menú de navegación">
```

- [ ] **Step 4: Reescribir el contenido del panel**

Reemplazar TODO el contenido interno actual del panel `mobileMenu` por exactamente (nota: `aria-current="page"` en Clínica deportiva):

```html
  <a href="/" data-nav="inicio">Inicio</a>
  <a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada">Fisioterapia avanzada</a>
  <a href="/clinica-deportiva" data-nav="clinica-deportiva" aria-current="page">Clínica deportiva</a>
  <a href="/mujer" data-nav="mujer">Mujer</a>
  <a href="/#metodologia">Método</a>
  <a href="/#equipo">Equipo</a>
  <a href="/#ubicacion">Ubicación</a>
  <a href="/#contacto">Contacto</a>
  <a href="https://wa.me/34620326447" target="_blank" rel="noopener" class="mm-cta">Pedir cita</a>
```

- [ ] **Step 5: Limpiar JS** — Borrar el bloque `document.querySelectorAll('.mm-group-label').forEach(...)` si existe en el `<script>`. El resto del script (toggle/click-outside/aria-expanded sync) se conserva.

- [ ] **Step 6: Añadir listener de Escape**

Adaptar al patrón del script de esta página. Si el script usa variables globales `mobileMenu`/`hamburger` (mismo patrón que index.html), añadir:

```js
document.addEventListener('keydown', (e) => {
  if (e.key === 'Escape' && mobileMenu.classList.contains('open')) {
    mobileMenu.classList.remove('open');
    document.getElementById('hamburger').setAttribute('aria-expanded', 'false');
    document.getElementById('hamburger').focus();
  }
});
```

Si el script de esta página usa IIFE con `var menu`/`var btn` en scope (patrón de fisio/mujer), añadir dentro de la misma IIFE antes del cierre:

```js
document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && menu.classList.contains('open')) {
    menu.classList.remove('open');
    btn.setAttribute('aria-expanded', 'false');
    btn.focus();
  }
});
```

- [ ] **Step 7: Verificar**

```bash
grep -n 'class="nav-links"\|class="nav-cta"' public/clinica-deportiva.html
```
Esperado: sin resultados.

```bash
grep -n '>Inicio<\|class="mm-cta"' public/clinica-deportiva.html
```
Esperado: cada uno presente al menos una vez.

```bash
grep -n 'aria-current="page"' public/clinica-deportiva.html
```
Esperado: 1 (el ítem Clínica deportiva).

```bash
grep -n 'mm-group-label\|mm-submenu\|mm-caret' public/clinica-deportiva.html
```
Esperado: sin resultados.

```bash
grep -c '<main id="main"' public/clinica-deportiva.html
```
Esperado: 1.

---

## Task 4: fisioterapia-avanzada.html — quitar nav inline + reescribir panel + JS Esc + active

**Files:**
- Modify: `public/fisioterapia-avanzada.html`

- [ ] **Step 1: Eliminar `<div class="nav-links">…</div>`** alrededor de L166.

- [ ] **Step 2: Eliminar el botón `<a class="nav-cta">Pedir cita</a>`** alrededor de L194–197.

- [ ] **Step 3: Panel — ya tiene `role="dialog"` y `aria-label`.** Verificar y NO añadir si ya está (`<div class="mobile-menu" id="mobile-menu" role="dialog" aria-label="Menú de navegación">` en L206).

- [ ] **Step 4: Reescribir el contenido del panel**

Reemplazar TODO el contenido interno del panel `mobile-menu` por exactamente (nota: `aria-current="page"` en Fisioterapia avanzada):

```html
  <a href="/" data-nav="inicio">Inicio</a>
  <a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada" aria-current="page">Fisioterapia avanzada</a>
  <a href="/clinica-deportiva" data-nav="clinica-deportiva">Clínica deportiva</a>
  <a href="/mujer" data-nav="mujer">Mujer</a>
  <a href="/#metodologia">Método</a>
  <a href="/#equipo">Equipo</a>
  <a href="/#ubicacion">Ubicación</a>
  <a href="/#contacto">Contacto</a>
  <a href="https://wa.me/34620326447" target="_blank" rel="noopener" class="mm-cta">Pedir cita</a>
```

- [ ] **Step 5: Limpiar JS** — Borrar el bloque de `.mm-group-label` toggling si existe. Mantener el resto del script.

- [ ] **Step 6: Añadir listener de Escape (IIFE pattern)**

Dentro de la IIFE del mobile-menu en el `<script>` (donde `var menu = document.getElementById('mobile-menu')` y `var btn = document.getElementById('hamburger')` están en scope), justo antes del cierre `})();`, añadir:

```js
document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && menu.classList.contains('open')) {
    menu.classList.remove('open');
    btn.setAttribute('aria-expanded', 'false');
    btn.focus();
  }
});
```

- [ ] **Step 7: Verificar**

```bash
grep -n 'class="nav-links"\|class="nav-cta"' public/fisioterapia-avanzada.html
```
Esperado: sin resultados.

```bash
grep -n '>Inicio<\|class="mm-cta"' public/fisioterapia-avanzada.html
```
Esperado: cada uno presente.

```bash
grep -n 'aria-current="page"' public/fisioterapia-avanzada.html
```
Esperado: 1 (Fisioterapia avanzada).

```bash
grep -n 'mm-group-label\|mm-submenu\|mm-caret' public/fisioterapia-avanzada.html
```
Esperado: sin resultados.

```bash
grep -c '<main id="main"' public/fisioterapia-avanzada.html
```
Esperado: 1.

---

## Task 5: mujer.html — quitar nav inline + reescribir panel + JS Esc + active

**Files:**
- Modify: `public/mujer.html`

- [ ] **Step 1: Eliminar `<div class="nav-links">…</div>`** alrededor de L171.

- [ ] **Step 2: Eliminar el botón `<a class="nav-cta">Pedir cita</a>`** alrededor de L199–202.

- [ ] **Step 3: Panel ya tiene `role="dialog"` y `aria-label`** (`<div class="mobile-menu" id="mobile-menu" role="dialog" aria-label="Menú de navegación">` en L211). Verificar; no añadir si existe.

- [ ] **Step 4: Reescribir el contenido del panel**

Reemplazar TODO el contenido interno del panel `mobile-menu` por exactamente (nota: `aria-current="page"` en Mujer):

```html
  <a href="/" data-nav="inicio">Inicio</a>
  <a href="/fisioterapia-avanzada" data-nav="fisioterapia-avanzada">Fisioterapia avanzada</a>
  <a href="/clinica-deportiva" data-nav="clinica-deportiva">Clínica deportiva</a>
  <a href="/mujer" data-nav="mujer" aria-current="page">Mujer</a>
  <a href="/#metodologia">Método</a>
  <a href="/#equipo">Equipo</a>
  <a href="/#ubicacion">Ubicación</a>
  <a href="/#contacto">Contacto</a>
  <a href="https://wa.me/34620326447" target="_blank" rel="noopener" class="mm-cta">Pedir cita</a>
```

- [ ] **Step 5: Limpiar JS** — Borrar el bloque de `.mm-group-label` toggling si existe en el `<script>`. Mantener el resto.

- [ ] **Step 6: Añadir listener de Escape (IIFE pattern, mismo que fisio)**

```js
document.addEventListener('keydown', function(e) {
  if (e.key === 'Escape' && menu.classList.contains('open')) {
    menu.classList.remove('open');
    btn.setAttribute('aria-expanded', 'false');
    btn.focus();
  }
});
```

- [ ] **Step 7: Verificar**

```bash
grep -n 'class="nav-links"\|class="nav-cta"' public/mujer.html
```
Esperado: sin resultados.

```bash
grep -n '>Inicio<\|class="mm-cta"' public/mujer.html
```
Esperado: cada uno presente.

```bash
grep -n 'aria-current="page"' public/mujer.html
```
Esperado: 1 (Mujer).

```bash
grep -n 'mm-group-label\|mm-submenu\|mm-caret' public/mujer.html
```
Esperado: sin resultados.

```bash
grep -c '<main id="main"' public/mujer.html
```
Esperado: 1.

---

## Task 6: Verificación integral local

- [ ] **Step 1: Servir local en 5050**

```bash
(cd /Users/agugp96/Desktop/re3move-landing/.claude/worktrees/blissful-heyrovsky-0b78b2/public && npx --yes serve -l 5050 . > /tmp/serve_re3move.log 2>&1 &) && sleep 4
```

- [ ] **Step 2: Smoke HTTP**

```bash
for p in "" clinica-deportiva fisioterapia-avanzada mujer; do
  echo -n "/$p -> "
  curl -s -o /dev/null -w "%{http_code}\n" "http://localhost:5050/$p"
done
```
Esperado: todas 200.

- [ ] **Step 3: Greps cross-file**

```bash
cd /Users/agugp96/Desktop/re3move-landing/.claude/worktrees/blissful-heyrovsky-0b78b2
# barra inline y nav-cta deben estar muertas en todas las páginas:
grep -rn 'class="nav-links"\|class="nav-cta"' public/*.html
# dead code de acordeones:
grep -rn 'mm-group\|mm-submenu\|mm-caret\|nav-dd\|nav-dd-label' public/*.html public/assets/styles.css
# Inicio presente 1 vez por página (en el panel):
for f in index clinica-deportiva fisioterapia-avanzada mujer; do echo -n "$f Inicio: "; grep -c '>Inicio<' public/$f.html; done
# aria-current presente exactamente 1 vez por página:
for f in index clinica-deportiva fisioterapia-avanzada mujer; do echo -n "$f current: "; grep -c 'aria-current="page"' public/$f.html; done
```
Esperado:
- Primer grep: sin resultados.
- Segundo grep: sin resultados (nav-dd y nav-dd-label ya no deben existir en el HTML; en CSS las reglas `.nav-dd*` también deben revisarse — si quedan reglas huérfanas referenciando selectores ausentes, son dead code pero no rompen nada; opcional limpiarlas).
- Tercer y cuarto: cada uno = 1.

- [ ] **Step 4: Sanity served HTML**

```bash
curl -s http://localhost:5050/ | grep -o '>Inicio<\|class="mm-cta"\|aria-current="page"' | sort -u
```
Esperado: tres líneas distintas, una por cada token.

- [ ] **Step 5: Parar el server**

```bash
pkill -f "serve -l 5050" 2>/dev/null; echo "server stopped"
```

- [ ] **Step 6 (opcional pero recomendado): Smoke visual con gstack browse**

Solo si el binario está construido (`~/.claude/skills/gstack/browse/dist/browse` o `.claude/skills/gstack/browse/dist/browse` ejecutables). Servir, luego:

```bash
B="$HOME/.claude/skills/gstack/browse/dist/browse"
[ -x "$B" ] || B="$(git rev-parse --show-toplevel)/.claude/skills/gstack/browse/dist/browse"
$B goto http://localhost:5050/
$B viewport 1280x800
$B screenshot /tmp/nav-desktop-index.png
$B click "#hamburger"
$B screenshot /tmp/nav-desktop-index-open.png
$B viewport 390x844
$B screenshot /tmp/nav-mobile-index.png
$B goto http://localhost:5050/mujer
$B screenshot /tmp/nav-mobile-mujer.png
$B console
```
Esperado: en desktop solo se ve logo + hamburguesa (sin barra de enlaces ni botón "Pedir cita" aparte); al hacer click, panel con los 9 ítems, "Inicio" marcado en mujer.html no aplica (en mujer el activo es "Mujer"). Console sin errores. Si gstack no está construido, omitir este step.

---

## Task 7: Commit único + push + redeploy

> Este flujo es **destructivo en producción** (push a `main` que Railway redespliega). El usuario ya aprobó la estrategia "merge a main y push" en el ciclo anterior, así que se reutiliza.

- [ ] **Step 1: Verificar estado del worktree**

```bash
git -C /Users/agugp96/Desktop/re3move-landing/.claude/worktrees/blissful-heyrovsky-0b78b2 status -s
```
Esperado: archivos modificados son `public/assets/styles.css` y los 4 HTML. (Si aparecen otros archivos no relacionados, parar y revisar — no añadir más al commit que lo planeado.)

- [ ] **Step 2: Stage + commit en la rama de trabajo**

```bash
W=/Users/agugp96/Desktop/re3move-landing/.claude/worktrees/blissful-heyrovsky-0b78b2
git -C "$W" add public/assets/styles.css public/index.html public/clinica-deportiva.html public/fisioterapia-avanzada.html public/mujer.html docs/superpowers/plans/2026-05-20-universal-hamburger-nav.md
git -C "$W" commit -m "$(cat <<'EOF'
feat(nav): navegación universal con hamburguesa en las 4 páginas

- Top-bar uniforme: solo logo (izq) + hamburguesa (der), todos los viewports
- Panel del menú plano con 9 items: Inicio + 3 servicios + Método/Equipo/Ubicación/Contacto + Pedir cita
- aria-current="page" en el item de la página activa
- Esc cierra el menú y devuelve foco al botón
- Limpieza de dead code: acordeones .mm-group/.mm-submenu/.mm-caret y barra .nav-links

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
git -C "$W" log --oneline -3
```
Esperado: nuevo commit creado encima de `d10d3af`.

- [ ] **Step 3: Merge a main (fast-forward) y push**

```bash
PW=/Users/agugp96/Desktop/re3move-landing
W=/Users/agugp96/Desktop/re3move-landing/.claude/worktrees/blissful-heyrovsky-0b78b2
# Verificar primary worktree limpio (salvo .DS_Store junk gestionado en ciclo anterior)
git -C "$PW" status -s
# Fast-forward main al tip de la rama de trabajo
git -C "$PW" merge --ff-only claude/blissful-heyrovsky-0b78b2 2>/tmp/fferr || { echo "FF blocked:"; cat /tmp/fferr; echo "discard junk and retry:"; git -C "$PW" checkout -- .DS_Store 2>/dev/null; git -C "$PW" merge --ff-only claude/blissful-heyrovsky-0b78b2; }
# Push a origin/main (dispara redeploy en Railway)
git -C "$PW" push origin main
git -C "$PW" log --oneline -3
```
Esperado: push exitoso. Railway detecta el commit en `origin/main` y comienza el redeploy automáticamente (verificable en el dashboard de Railway).

- [ ] **Step 4: Confirmar al usuario que el push se hizo y Railway está redesplegando.**

---

## Self-Review (cobertura del spec)

- Top-bar uniforme (logo izq + hamburguesa der, sin barra inline ni nav-cta separado): Task 1 (CSS) + Tasks 2–5 (eliminación HTML).
- Hamburguesa siempre visible (todos los viewports): Task 1 Step 1 (`.hamburger { display: flex }` base, sin media query).
- Panel con los 9 ítems unificado en las 4 páginas: Tasks 2–5 Step 4 (markup idéntico salvo el item con `aria-current`).
- `aria-current="page"` por página: Tasks 2–5 Step 4 (item específico marcado).
- `role="dialog"` + `aria-label` en el panel de las 4 páginas: index/clinica-deportiva añaden (Task 2 Step 3 / Task 3 Step 3); fisio/mujer ya lo tienen (Task 4 Step 3 / Task 5 Step 3 confirman).
- Cierre con `Esc` + devolución de foco: Tasks 2–5 Step 6.
- Limpieza de dead code (acordeones, nav-links): Task 1 Steps 1–2 (CSS) + Tasks 2–5 Steps 1, 5 (HTML, JS).
- Pedir cita como item destacado al final del menú: incluido en el bloque HTML de Tasks 2–5 Step 4 con clase `.mm-cta` (estilo ya existente).
- Verificación local con grep + curl + (opcional) smoke visual: Task 6.
- Commit único + push + redeploy: Task 7.
