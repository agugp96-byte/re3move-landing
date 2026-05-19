# Re3Move Landing — Mejoras: Plan de Implementación

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Corregir todos los bugs técnicos, hacer accesible y SEO-completa la landing de Re3Move, cablear contenido real (Google Reviews), aplicar mejoras visuales/de orden de bajo riesgo y corregir el copy en español, dejándola lista para redeploy en Railway.

**Architecture:** Sitio estático (4 HTML + CSS compartido + nginx). Los cambios transversales (fuentes, contraste, accesibilidad, animaciones, panel de testimonios) viven en `assets/styles.css` y se hacen una sola vez. Cada HTML recibe ajustes de `<head>` (fuentes/SEO), skip-link, `<main>` y `aria`. Bugs concretos y reorden solo en `index.html`. Nuevos archivos: `robots.txt`, `sitemap.xml`, `.gitignore`.

**Tech Stack:** HTML estático, CSS, nginx (Docker), sin framework de tests. Verificación = `grep` + `npx serve public` + inspección manual.

**Spec:** `docs/superpowers/specs/2026-05-18-re3move-landing-improvements-design.md`

**Nota crítica de orden:** Las 4 páginas dependen del `@import` de Google Fonts en `styles.css`. La Tarea 1 elimina ese `@import`, por lo que las Tareas 2–5 (que añaden el `<link>` de fuentes a cada HEAD) deben completarse antes de verificar fuentes. No hacer commit del repo en un estado donde `styles.css` no tenga `@import` y algún HTML tampoco tenga el `<link>`. Por eso Tarea 1 y la edición de HEAD de cada página se agrupan lógicamente; commitear solo cuando fuentes funcionen en las 4.

---

## Datos compartidos (usar literalmente)

**Bloque de fuentes** — añadir/garantizar en `<head>` de cada página (los `preconnect` ya existen en las 4; añadir SOLO la línea `stylesheet` de Google Fonts justo antes de `<link rel="stylesheet" href="/assets/styles.css">`):

```html
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Syne:wght@400;500;600;700;800&family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;1,9..40,400&family=Barlow+Condensed:wght@500;600;700;800&display=swap">
```

**Bloque SEO** — añadir en `<head>` de cada página, justo después de la línea `<meta property="og:locale" content="es_ES">`. Sustituir `{{URL}}`, `{{OG_TITLE}}`, `{{OG_DESC}}`, `{{OG_IMG}}` por los valores de la tabla:

```html
<meta property="og:url" content="{{URL}}">
<meta property="og:site_name" content="Re3Move">
<meta property="og:image:alt" content="Re3Move — clínica de fisioterapia en Palma de Mallorca">
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="{{OG_TITLE}}">
<meta name="twitter:description" content="{{OG_DESC}}">
<meta name="twitter:image" content="{{OG_IMG}}">
<meta name="theme-color" content="#3a5748">
<meta name="robots" content="index,follow">
```

| Página | {{URL}} | canonical correcto | {{OG_TITLE}} | {{OG_DESC}} | {{OG_IMG}} |
|---|---|---|---|---|---|
| index.html | `https://re3move.es/` | `https://re3move.es/` (ya OK) | `Re3Move — Clínica de Fisioterapia en Mallorca` | `Clínica de Recuperación, Readaptación y Rendimiento. Fisioterapia especializada, entrenamiento y suelo pélvico en Mallorca.` | `/assets/photos/clinic-02-physio-baby.jpeg` |
| clinica-deportiva.html | `https://re3move.es/clinica-deportiva` | cambiar de `...clinica-deportiva.html` a `https://re3move.es/clinica-deportiva` | `Clínica deportiva — Re3Move` | `Readaptación tras lesión, entrenamiento de rendimiento y programación online. Con criterio clínico y supervisión real.` | `/assets/photos/clinic-02-physio-baby.jpeg` |
| fisioterapia-avanzada.html | `https://re3move.es/fisioterapia-avanzada` | `https://re3move.es/fisioterapia-avanzada` (ya OK) | `Fisioterapia avanzada — Re3Move` | `Ecografía, EPI, neuromodulación percutánea, diatermia, superinductiva y VOESS. Tecnología clínica al servicio de tu recuperación en Palma de Mallorca.` | `/assets/photos/clinic-03-ecografia.jpeg` |
| mujer.html | `https://re3move.es/mujer` | `https://re3move.es/mujer` (ya OK) | `Mujer — Suelo pélvico & embarazo — Re3Move` | `Suelo pélvico, embarazo y posparto atendidos por Judith Iglesias — fisioterapeuta especialista con dos másteres en salud pélvica y obstétrica. Palma de Mallorca.` | `/assets/photos/clinic-07-judith-pelvis.jpeg` |

**Skip-link** — primer elemento dentro de `<body>` (antes de `<nav>`):

```html
<a href="#main" class="skip-link">Saltar al contenido</a>
```

**`<main>` wrapper** — abrir `<main id="main">` justo después del cierre del menú móvil (`</div>` del `mobile-menu`/`mobileMenu`) y cerrar `</main>` justo antes de `<footer ...>`. El `<a class="wa-float">` y `<script>` quedan FUERA de `<main>` (después de `</footer>`).

**URLs reales de reseñas:**
- Leer reseñas: `https://share.google/M8TjRifNDFBQa3QtY`
- Dejar reseña: `https://g.page/r/CYDagfJmTET-EBM/review`

**Mapa (nuevo `src` del iframe en index.html):**
```
https://maps.google.com/maps?q=Carrer%20George%20Orwell%201%2C%2007002%20Palma%2C%20Illes%20Balears&output=embed
```

---

## Task 1: CSS compartido — fuentes, accesibilidad, contraste, imágenes de cards, panel de reseñas

**Files:**
- Modify: `public/assets/styles.css`

- [ ] **Step 1: Quitar el `@import` de Google Fonts**

Eliminar por completo la línea 5 (`@import url('https://fonts.googleapis.com/css2?family=Syne...&display=swap&font-display=swap');`) y la línea en blanco sobrante. Las fuentes pasan a cargarse vía `<link>` en cada HTML (Tareas 2–5).

- [ ] **Step 2: Añadir foco visible global, skip-link y reduced-motion**

Añadir al final de `public/assets/styles.css`:

```css
/* ── Accesibilidad ── */
:focus-visible { outline: 2px solid var(--green); outline-offset: 2px; border-radius: 4px; }
.skip-link {
  position: absolute; left: -9999px; top: 0; z-index: 999;
  background: var(--green); color: #fff; padding: 12px 20px;
  font-family: var(--font-b); font-size: 14px; font-weight: 600; border-radius: 0 0 8px 0;
}
.skip-link:focus { left: 0; }
.nav-dd-label { cursor: pointer; }
.nav-dd:focus-within .nav-dd-menu { opacity: 1; pointer-events: auto; transform: translateX(-50%) translateY(0); }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after { animation-duration: 0.001ms !important; animation-iteration-count: 1 !important; transition-duration: 0.001ms !important; scroll-behavior: auto !important; }
  .fade-up { animation: none !important; opacity: 1 !important; transform: none !important; }
}
```

- [ ] **Step 3: Subir contraste de textos claros (editar reglas existentes en styles.css)**

En las reglas del footer dentro de `styles.css`, cambiar:
- `.footer-tagline { ... color: rgba(255,255,255,0.3); ... }` → `rgba(255,255,255,0.62)`
- `.footer-desc { ... color: rgba(255,255,255,0.4); }` → `rgba(255,255,255,0.62)`
- `.footer-col-title { ... color: rgba(255,255,255,0.35); ... }` → `rgba(255,255,255,0.6)`
- `.footer-links a { ... color: rgba(255,255,255,0.55); ... }` → `rgba(255,255,255,0.72)`
- `.footer-copy { ... color: rgba(255,255,255,0.3); }` → `rgba(255,255,255,0.6)`
- `.soc { ... color: rgba(255,255,255,0.45); ... }` → `rgba(255,255,255,0.7)`

- [ ] **Step 4: CSS para imágenes `<img>` de cards (sustituir background por object-fit)**

Las reglas `.service-bg` y `.mujer-bg` viven en el `<style>` inline de `index.html` (no en styles.css). NO tocar styles.css para esto; se hace en Task 6. (Este step es un no-op recordatorio para no duplicar.)

- [ ] **Step 5: Añadir estilos del nuevo panel de reseñas (Google) a styles.css**

Añadir al final de `public/assets/styles.css`:

```css
/* ── Panel de reseñas Google ── */
.reviews-panel { max-width: 720px; margin: 0 auto; background: var(--white); border: 1px solid var(--border); border-radius: 20px; padding: 48px 40px; text-align: center; box-shadow: 0 8px 28px rgba(0,0,0,0.05); }
.reviews-logo { display: inline-flex; align-items: center; gap: 10px; margin-bottom: 18px; }
.reviews-logo span { font-family: var(--font-h); font-weight: 800; font-size: 18px; color: var(--ink); letter-spacing: -0.02em; }
.reviews-panel h3 { font-family: var(--font-h); font-weight: 800; font-size: clamp(22px,3vw,30px); color: var(--ink); letter-spacing: -0.02em; margin-bottom: 12px; }
.reviews-panel p { font-size: 15px; color: var(--muted); line-height: 1.65; max-width: 460px; margin: 0 auto 28px; }
.reviews-actions { display: flex; gap: 12px; justify-content: center; flex-wrap: wrap; }
.reviews-actions .btn-primary, .reviews-actions .btn-secondary { text-decoration: none; }
@media (max-width: 560px) { .reviews-panel { padding: 36px 22px; } .reviews-actions { flex-direction: column; } .reviews-actions a { justify-content: center; } }
```

- [ ] **Step 6: Verificar CSS**

Run: `grep -n '@import' public/assets/styles.css`
Expected: sin resultados (vacío).
Run: `grep -n 'skip-link\|prefers-reduced-motion\|reviews-panel\|focus-visible' public/assets/styles.css`
Expected: todas presentes.

(Commit se hace junto con Tareas 2–6 — ver nota crítica de orden.)

---

## Task 2: index.html — fuentes, SEO, bugs, imágenes, reorden, testimonios, a11y

**Files:**
- Modify: `public/index.html`

- [ ] **Step 1: Añadir `<link>` de fuentes**

Justo antes de la línea `<link rel="stylesheet" href="/assets/styles.css">` (línea 18), insertar el **Bloque de fuentes** de "Datos compartidos".

- [ ] **Step 2: Añadir Bloque SEO**

Tras la línea `<meta property="og:locale" content="es_ES">` (línea 15), insertar el **Bloque SEO** con los valores de la fila `index.html` de la tabla.

- [ ] **Step 3: Enriquecer Schema.org**

En el bloque `<script type="application/ld+json">`, añadir dentro del objeto, tras `"openingHours": "Mo-Fr 08:00-21:00"`:

```json
,
  "image": "https://re3move.es/assets/photos/clinic-02-physio-baby.jpeg",
  "areaServed": "Palma de Mallorca",
  "sameAs": ["https://www.instagram.com/re3move/"],
  "hasMap": "https://maps.google.com/maps?q=Carrer%20George%20Orwell%201%2C%2007002%20Palma%2C%20Illes%20Balears"
```

(No añadir `priceRange` — el sitio dice "Tarifas a consultar".)

- [ ] **Step 4: Arreglar el mapa**

En el `<iframe>` de `.location-map` (línea ~491), reemplazar el atributo `src="https://www.google.com/maps/embed?pb=!1m18...4v1"` por:
`src="https://maps.google.com/maps?q=Carrer%20George%20Orwell%201%2C%2007002%20Palma%2C%20Illes%20Balears&output=embed"`
Mantener `allowfullscreen`, `loading="lazy"`, `referrerpolicy`, `title`.

- [ ] **Step 5: Convertir `.service-bg` y `.mujer-bg` a `<img>`**

En el `<style>` inline de index.html, cambiar:
- `.service-bg { position: absolute; inset: 0; background-size: cover; background-position: center; transition: transform 0.4s ease; }` → `.service-bg { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; object-position: center; transition: transform 0.4s ease; display: block; }`
- `.mujer-bg { position: absolute; inset: 0; background-size: cover; background-position: center; transition: transform 0.4s ease; }` → `.mujer-bg { position: absolute; inset: 0; width: 100%; height: 100%; object-fit: cover; object-position: center; transition: transform 0.4s ease; display: block; }`

En el HTML, reemplazar cada `<div class="service-bg" style="background-image:url('RUTA');" ...></div>` por `<img class="service-bg" src="RUTA" alt="ALT" ...>` con estos valores (la 1ª card sin `loading="lazy"`; resto con `loading="lazy"`; `decoding="async"`; añadir `width`/`height` con la relación de aspecto intrínseca real de cada archivo — leerla con `file`/`sips -g pixelWidth -g pixelHeight public/assets/photos/<archivo>` y usar esos valores):

- Card 01 (`/assets/photos/clinic-02-physio-baby.jpeg`): `alt="Fisioterapeuta tratando a una paciente en consulta"` — SIN loading lazy (above the fold).
- Card 02 (`/assets/photos/training-03-step-up.png`): `alt="Ejercicio de readaptación con step"` — `loading="lazy"`. (Quita el `loading="lazy"` muerto que estaba en el div.)
- Card 03 (`/assets/photos/training-01-stability.png`): `alt="Entrenamiento de estabilidad y fuerza"` — `loading="lazy"`.
- Mujer card 1 (`/assets/photos/clinic-03-ecografia.jpeg`): `alt="Valoración con ecografía funcional de suelo pélvico"` — `loading="lazy"`.
- Mujer card 2 (`/assets/photos/clinic-05-pregnancy-side.jpeg`): `alt="Acompañamiento en embarazo y posparto"` — `loading="lazy"`.

Mantener el orden DOM: `<img class="service-bg">` antes de `<div class="service-overlay">`.

- [ ] **Step 6: Reordenar — mover Testimonios antes de Ubicación**

Cortar el bloque completo `<!-- TESTIMONIALS -->` … `</section>` (sección `id="testimonios"`, líneas ~545–594) y pegarlo justo antes de `<!-- LOCATION -->` (antes de `<section class="section location" id="ubicacion">`, línea ~467). Orden resultante: Method → Team → Testimonials → Location → Contact.

- [ ] **Step 7: Rediseñar bloque Testimonios → panel de reseñas Google**

Reemplazar TODO el contenido interno de `<section class="section testimonials" id="testimonios">` (desde `<div class="container">` hasta su `</div>` de cierre, incluido el header inline y `.test-grid`) por:

```html
  <div class="container">
    <div class="test-header" style="text-align:center;max-width:560px;margin:0 auto 44px;">
      <div class="tag">Opiniones</div>
      <h2 class="section-title">Lo que dicen nuestros pacientes</h2>
      <p class="section-sub" style="margin:14px auto 0;">Nuestras opiniones son reales y verificadas en Google. Léelas tú mismo o cuéntanos tu experiencia.</p>
    </div>
    <div class="reviews-panel">
      <div class="reviews-logo">
        <svg width="22" height="22" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg" aria-hidden="true" focusable="false"><path d="M22.56 12.25c0-.78-.07-1.53-.2-2.25H12v4.26h5.92c-.26 1.37-1.04 2.53-2.21 3.31v2.77h3.57c2.08-1.92 3.28-4.74 3.28-8.09z" fill="#4285F4"/><path d="M12 23c2.97 0 5.46-.98 7.28-2.66l-3.57-2.77c-.98.66-2.23 1.06-3.71 1.06-2.86 0-5.29-1.93-6.16-4.53H2.18v2.84C3.99 20.53 7.7 23 12 23z" fill="#34A853"/><path d="M5.84 14.09c-.22-.66-.35-1.36-.35-2.09s.13-1.43.35-2.09V7.07H2.18C1.43 8.55 1 10.22 1 12s.43 3.45 1.18 4.93l3.66-2.84z" fill="#FBBC05"/><path d="M12 5.38c1.62 0 3.06.56 4.21 1.64l3.15-3.15C17.45 2.09 14.97 1 12 1 7.7 1 3.99 3.47 2.18 7.07l3.66 2.84c.87-2.6 3.3-4.53 6.16-4.53z" fill="#EA4335"/></svg>
        <span>Reseñas verificadas en Google</span>
      </div>
      <h3>Pacientes reales, opiniones reales.</h3>
      <p>Preferimos que escuches a quienes ya han pasado por la clínica. Lee las reseñas de Re3Move en Google — y si te hemos ayudado, deja la tuya.</p>
      <div class="reviews-actions">
        <a href="https://share.google/M8TjRifNDFBQa3QtY" target="_blank" rel="noopener" class="btn-primary">Leer reseñas en Google</a>
        <a href="https://g.page/r/CYDagfJmTET-EBM/review" target="_blank" rel="noopener" class="btn-secondary">Dejar una reseña</a>
      </div>
    </div>
  </div>
```

Eliminar del `<style>` inline de index.html las reglas ya no usadas: `.test-grid`, `.test-card`, `.test-stars`, `.test-quote`, `.test-author`, `.test-avatar`, `.test-name`, `.test-service`. Conservar `.testimonials` y `.test-header` (este último ya no necesita las reglas flex; si la regla `.test-header` solo tenía `margin-bottom`, puede quedarse).

- [ ] **Step 8: Skip-link y `<main>`**

Insertar el **skip-link** como primer elemento tras `<body>`. Abrir `<main id="main">` tras el cierre del `<div class="mobile-menu" id="mobileMenu">…</div>` y cerrar `</main>` antes de `<!-- FOOTER -->` (`<footer class="footer">`). El `wa-float` y `<script>` quedan fuera.

- [ ] **Step 9: a11y del nav y SVGs**

- En `<button class="hamburger" id="hamburger" aria-label="Menú">` añadir `aria-expanded="false"` y `aria-controls="mobileMenu"`.
- En el `<script>`, en el listener del hamburger, tras `mobileMenu.classList.toggle('open')`, añadir: `const isOpen = mobileMenu.classList.contains('open'); document.getElementById('hamburger').setAttribute('aria-expanded', isOpen);` y al cerrar por click fuera / por click en enlace, poner `aria-expanded` a `false`.
- A cada `<svg>` puramente decorativo (carets del nav `.nav-dd-caret`/`.mm-caret`, iconos dentro de enlaces que ya tienen texto, icono del mapa/horario/teléfono, estrellas) añadir `aria-hidden="true" focusable="false"`. Los `<a class="soc">`/`wa-float`/`nav-cta` que son solo icono ya tienen `aria-label`: a su `<svg>` interno añadir `aria-hidden="true" focusable="false"`.
- Hacer los `.nav-dd-label` operables por teclado: añadir `tabindex="0"` y `role="button"` a cada `<span class="nav-dd-label">`. (El CSS de Task 1 step 2 ya abre el menú con `:focus-within`.)

- [ ] **Step 10: Verificar index.html**

Run: `grep -n 'maps.google.com/maps?q=Carrer\|class="service-bg" src\|reviews-panel\|skip-link\|<main id="main"\|aria-expanded' public/index.html`
Expected: todas presentes.
Run: `grep -n 'g.page/r/review\|maps/embed?pb=\|background-image:url.*service-bg\|<div class="service-bg"' public/index.html`
Expected: sin resultados.
Run: `grep -n 'id="testimonios"' public/index.html` y confirmar que aparece ANTES de `id="ubicacion"` (comparar números de línea).

---

## Task 3: clinica-deportiva.html — fuentes, SEO, canonical, skip-link, main, a11y

**Files:**
- Modify: `public/clinica-deportiva.html`

- [ ] **Step 1: Fuentes** — insertar el Bloque de fuentes justo antes de `<link rel="stylesheet" href="/assets/styles.css">` (línea ~17).

- [ ] **Step 2: Corregir canonical** — cambiar `<link rel="canonical" href="https://re3move.es/clinica-deportiva.html">` por `<link rel="canonical" href="https://re3move.es/clinica-deportiva">`.

- [ ] **Step 3: Bloque SEO** — insertar tras `<meta property="og:locale" content="es_ES">` el Bloque SEO con la fila `clinica-deportiva.html`.

- [ ] **Step 4: Skip-link + `<main>`** — skip-link como primer elemento tras `<body>` (línea 240). Abrir `<main id="main">` tras el cierre de `<div class="mobile-menu" id="mobileMenu">…</div>` (~línea 305+) y cerrar `</main>` antes de `<footer class="simple-footer">` (línea 659).

- [ ] **Step 5: a11y hamburger** — en `<button class="hamburger" ...>` añadir `aria-expanded="false"` y `aria-controls="mobileMenu"` si no los tiene; en su JS de toggle, sincronizar `aria-expanded` (mismo patrón que Task 2 Step 9). SVGs decorativos: `aria-hidden="true" focusable="false"`. `.nav-dd-label` → `tabindex="0" role="button"`.

- [ ] **Step 6: Verificar** — Run: `grep -n 'fonts.googleapis.com/css2\|canonical.*clinica-deportiva"\|og:url\|<main id="main"\|skip-link' public/clinica-deportiva.html` → todas presentes. Run: `grep -n 'clinica-deportiva.html"' public/clinica-deportiva.html` → el canonical ya NO debe aparecer con `.html`.

---

## Task 4: fisioterapia-avanzada.html — fuentes, SEO, skip-link, main, a11y, copy

**Files:**
- Modify: `public/fisioterapia-avanzada.html`

- [ ] **Step 1: Fuentes** — insertar Bloque de fuentes antes de `<link rel="stylesheet" href="/assets/styles.css">` (~línea 25).

- [ ] **Step 2: Bloque SEO** — insertar tras `<meta property="og:locale" content="es_ES">` el Bloque SEO con la fila `fisioterapia-avanzada.html`. (Canonical ya OK, no tocar.)

- [ ] **Step 3: Skip-link + `<main>`** — skip-link tras `<body>` (línea 144). Abrir `<main id="main">` tras el cierre de `<div class="mobile-menu" id="mobile-menu" ...>…</div>` (~línea 194+) y cerrar `</main>` antes de `<footer class="tfoot">` (línea 786).

- [ ] **Step 4: Fix copy "postparto" → "posparto"** — en líneas 172 y 203, cambiar `Suelo pélvico, embarazo, postparto` por `Suelo pélvico, embarazo, posparto`.

- [ ] **Step 5: a11y hamburger/SVG** — `<button class="hamburger">` con `aria-expanded` sincronizado en JS + `aria-controls="mobile-menu"`. SVGs decorativos `aria-hidden="true" focusable="false"`. `.nav-dd-label` → `tabindex="0" role="button"`.

- [ ] **Step 6: Verificar** — Run: `grep -n 'fonts.googleapis.com/css2\|og:url\|twitter:card\|<main id="main"\|skip-link' public/fisioterapia-avanzada.html` → todas presentes. Run: `grep -n 'postparto' public/fisioterapia-avanzada.html` → sin resultados.

---

## Task 5: mujer.html — fuentes, SEO, skip-link, main, a11y, copy

**Files:**
- Modify: `public/mujer.html`

- [ ] **Step 1: Fuentes** — insertar Bloque de fuentes antes de `<link rel="stylesheet" href="/assets/styles.css">` (línea 25).

- [ ] **Step 2: Bloque SEO** — insertar tras `<meta property="og:locale" content="es_ES">` (línea 14) el Bloque SEO con la fila `mujer.html`. (Canonical ya OK.)

- [ ] **Step 3: Skip-link + `<main>`** — skip-link tras `<body>` (línea 150). Abrir `<main id="main">` tras el cierre de `<div class="mobile-menu" id="mobile-menu" ...>…</div>` (cierra ~línea 216) y cerrar `</main>` antes de `<footer class="footer">` (línea 556).

- [ ] **Step 4: Fix copy "postparto" → "posparto"** — líneas 178 y 209: `Suelo pélvico, embarazo, postparto` → `Suelo pélvico, embarazo, posparto`.

- [ ] **Step 5: a11y** — el hamburger YA tiene `aria-expanded="false"` y el JS ya lo sincroniza; añadir `aria-controls="mobile-menu"` al botón. SVGs decorativos `aria-hidden="true" focusable="false"`. `.nav-dd-label` → `tabindex="0" role="button"`.

- [ ] **Step 6: Verificar** — Run: `grep -n 'fonts.googleapis.com/css2\|og:url\|twitter:card\|<main id="main"\|skip-link\|aria-controls' public/mujer.html` → todas presentes. Run: `grep -n 'postparto' public/mujer.html` → sin resultados.

---

## Task 6: Archivos nuevos — robots.txt, sitemap.xml, .gitignore + destrackear .DS_Store

**Files:**
- Create: `public/robots.txt`
- Create: `public/sitemap.xml`
- Create: `.gitignore`

- [ ] **Step 1: `public/robots.txt`**

```
User-agent: *
Allow: /

Sitemap: https://re3move.es/sitemap.xml
```

- [ ] **Step 2: `public/sitemap.xml`**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url><loc>https://re3move.es/</loc><lastmod>2026-05-18</lastmod><priority>1.0</priority></url>
  <url><loc>https://re3move.es/clinica-deportiva</loc><lastmod>2026-05-18</lastmod><priority>0.8</priority></url>
  <url><loc>https://re3move.es/fisioterapia-avanzada</loc><lastmod>2026-05-18</lastmod><priority>0.8</priority></url>
  <url><loc>https://re3move.es/mujer</loc><lastmod>2026-05-18</lastmod><priority>0.8</priority></url>
</urlset>
```

- [ ] **Step 3: `.gitignore` (raíz del repo)**

```
.DS_Store
**/.DS_Store
*.log
.idea/
.vscode/
```

- [ ] **Step 4: Destrackear `.DS_Store`**

Run: `git rm --cached .DS_Store 2>/dev/null; git rm --cached --ignore-unmatch '**/.DS_Store' 2>/dev/null; true`
(Borra del índice, no del disco.)

- [ ] **Step 5: Verificar** — Run: `ls public/robots.txt public/sitemap.xml .gitignore` → existen. Run: `git status --porcelain | grep -i ds_store || echo "ds_store no trackeado OK"`.

---

## Task 7: nginx.conf — cabeceras de seguridad

**Files:**
- Modify: `nginx.conf`

- [ ] **Step 1: Añadir headers de seguridad**

Dentro del bloque `server { ... }`, tras la línea `charset utf-8;`, añadir:

```nginx
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

No tocar `listen 8080`, gzip, location blocks.

- [ ] **Step 2: Verificar** — Run: `grep -n 'X-Content-Type-Options\|X-Frame-Options\|Referrer-Policy\|listen 8080' nginx.conf` → las 4 presentes.

---

## Task 8: Revisión de copy en español (proofread global)

**Files:**
- Modify: cualquiera de los 4 HTML donde se detecte error.

- [ ] **Step 1: Proofread**

Leer el texto visible de las 4 páginas y corregir SOLO errores objetivos: ortografía, tildes ausentes (á/é/í/ó/ú/ñ), espacios dobles, comillas inconsistentes, y consistencia de términos. Ya identificado y cubierto en Tasks 4–5: `postparto`→`posparto`. NO reescribir estilo ni mensajes de marketing; no tocar nombres propios, WhatsApp, email ni precios. Si no hay más errores, dejar constancia ("sin más correcciones").

- [ ] **Step 2: Verificar** — Run: `grep -rn '  \|postparto' public/*.html | grep -v '<' | head` (heurística de dobles espacios en texto / término); revisar manualmente resultados.

---

## Task 9: Verificación integral local

- [ ] **Step 1: Servir** — Run (background): `cd public && npx --yes serve -l 5050 . &` ; esperar arranque.
- [ ] **Step 2: Smoke HTTP** — Run: `for p in "" clinica-deportiva fisioterapia-avanzada mujer robots.txt sitemap.xml; do echo -n "$p -> "; curl -s -o /dev/null -w "%{http_code}\n" "http://localhost:5050/$p"; done` → todas 200.
- [ ] **Step 3: Fuentes presentes en las 4** — Run: `for f in index clinica-deportiva fisioterapia-avanzada mujer; do echo -n "$f: "; grep -c 'fonts.googleapis.com/css2' public/$f.html; done` → cada una 1.
- [ ] **Step 4: Sin restos de bugs** — Run: `grep -rn 'g.page/r/review\b\|maps/embed?pb=\|@import' public/ ` → sin resultados.
- [ ] **Step 5: Inspección manual** — abrir las 4 páginas (5050): fuentes Syne/DM Sans cargan; mapa muestra Carrer George Orwell 1; cards con imágenes visibles e idénticas; Tab → skip-link aparece y dropdowns abren con teclado; sección Testimonios = panel Google con 2 botones que abren las URLs reales; Testimonios va antes de Ubicación; footer legible (contraste). Reportar resultado.
- [ ] **Step 6: Parar server** — matar el proceso `serve`.

---

## Task 10: Commit (NO push todavía)

- [ ] **Step 1: Stage + commit**

```bash
git add public/ nginx.conf .gitignore docs/superpowers/plans/2026-05-18-re3move-landing-improvements.md
git rm --cached .DS_Store 2>/dev/null; true
git commit -m "$(cat <<'EOF'
fix: bugs, accesibilidad, SEO, copy y mejoras visuales de la landing

- Mapa Google con embed real (dirección correcta)
- Imágenes de cards a <img> (lazy real + alt + menos CLS)
- Fuentes vía <link> en cada head (quitado @import render-blocking)
- Reseñas: panel Google real (URLs share/g.page) reemplaza testimonios ficticios
- Reorden: testimonios antes de contacto
- a11y: skip-link, <main>, foco visible, nav por teclado, aria, reduced-motion, contraste AA
- SEO: og:url/twitter/theme-color, canonicals unificados, schema, robots.txt, sitemap.xml
- copy: postparto -> posparto
- nginx: cabeceras de seguridad; .gitignore + .DS_Store destrackeado

Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>
EOF
)"
```

- [ ] **Step 2:** `git status` limpio y `git log --oneline -3` muestra el commit.

---

## Task 11: Push + redeploy (REQUIERE confirmación del usuario sobre estrategia de rama)

> Railway despliega desde `main` (auto-build por Dockerfile; no hay CLI). El push a `main` es lo que dispara el redeploy de un sitio de producción real. **No ejecutar sin que el usuario confirme: (a) merge de `claude/blissful-heyrovsky-0b78b2` a `main` y push de `main`, o (b) push de la rama + PR.**

- [ ] **Step 1:** Confirmar estrategia con el usuario (a o b).
- [ ] **Step 2 (si a):** `git checkout main && git merge --no-ff claude/blissful-heyrovsky-0b78b2 && git push origin main`. Railway detecta el push y redespliega automáticamente.
- [ ] **Step 2 (si b):** `git push -u origin claude/blissful-heyrovsky-0b78b2` y `gh pr create` contra `main`; el deploy ocurre al mergear el PR.
- [ ] **Step 3:** Confirmar al usuario que el push se hizo y que Railway está redeployando (verificable en el dashboard de Railway).

---

## Self-Review (cobertura del spec)

- §1 Bugs: mapa (T2.S4), lazy/img (T2.S5), @import fuentes (T1.S1 + T2–5.S1), g.page (T2.S7) ✓
- §2 Contenido real: URLs reseñas (T2.S7), schema sameAs/hasMap (T2.S3), stats intactas ✓
- §3 img conversion: T2.S5 + CSS en index `<style>` ✓
- §4 Reorden: T2.S6 ✓
- §5 Rediseño testimonios: T2.S7 + CSS panel T1.S5 ✓
- §6 a11y: skip-link/main/focus/reduced-motion (T1.S2, T2.S8, T3–5), aria/teclado (T2.S9, T3–5.S5), contraste (T1.S3) ✓
- §7 SEO: bloque SEO (T2–5.S2), canonicals (T3.S2), schema (T2.S3), robots/sitemap (T6) ✓
- §8 Higiene: .gitignore/DS_Store (T6), inline reducido (T2.S7 mueve testimonios a clase), nginx headers (T7) ✓
- Copy review (instrucción del usuario): T4.S4, T5.S4, T8 ✓
- Push/redeploy (instrucción del usuario): T11 con gate de confirmación ✓
