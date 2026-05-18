# Re3Move Landing — Mejoras (bugs, a11y, SEO, contenido, visual)

**Fecha:** 2026-05-18
**Autor:** Claude (con Augusto García)
**Estado:** Aprobado
**Base:** Sitio estático existente en `public/` (4 páginas + `styles.css`), desplegado con Docker/nginx en Railway.

---

## Objetivo

Pasar una mejora integral sobre la landing pública de Re3Move: corregir el 100% de los bugs técnicos, hacerla accesible (WCAG AA en lo razonable), completar SEO, cablear el contenido real (Google Reviews) y aplicar mejoras visuales/de orden de bajo riesgo. Se mantiene el lenguaje visual actual y todo el copy en español, equipo, precios del plan online y los enlaces de WhatsApp/email.

---

## Alcance

Aplica a las 4 páginas (`index.html`, `clinica-deportiva.html`, `fisioterapia-avanzada.html`, `mujer.html`), `assets/styles.css`, `nginx.conf` y la raíz del repo. Los bugs concretos están concentrados en `index.html`; los problemas compartidos viven en `styles.css` y en el `<head>` de cada página.

**Fuera de alcance:** cambiar copy en español, equipo (Xavier Calvo, Augusto García, Judith Iglesias), precios del plan online (79€/129€/199€), enlaces WhatsApp (`wa.me/34620326447`), email (`re3movecenter@gmail.com`). No se añaden secciones nuevas (precios/FAQ) ni rediseño de hero.

---

## 1. Bugs técnicos

| # | Bug | Ubicación | Solución |
|---|-----|-----------|----------|
| 1 | Mapa con `pb=` inventado y place ID falso (`0x1`) → ubicación incorrecta | `index.html:491` | Reemplazar el `src` del iframe por embed sin API key: `https://maps.google.com/maps?q=Carrer%20George%20Orwell%201%2C%2007002%20Palma%2C%20Illes%20Balears&output=embed`. Mantener `loading="lazy"` (válido en iframe) y el `title`. |
| 2 | `loading="lazy"` en `<div class="service-bg">` (no funciona en divs) | `index.html:338` | Ver §3 (conversión a `<img>`). Eliminar el atributo muerto. |
| 3 | Imágenes de cards como `background-image` CSS → nunca lazy, sin alt, con CLS | `index.html` services/mujer | Ver §3. |
| 4 | Fuentes vía `@import` en CSS (render-blocking) + parámetro basura `&font-display=swap` | `styles.css:5` | Quitar el `@import`. Añadir en cada `<head>`: `<link rel="preconnect" href="https://fonts.googleapis.com">`, `<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>` y `<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Syne:wght@400;500;600;700;800&family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;1,9..40,400&family=Barlow+Condensed:wght@500;600;700;800&display=swap">` (sin el `&font-display=swap`). `index.html` ya tiene los preconnect; el resto hay que añadirlos. |
| 5 | Link de reseñas placeholder `https://g.page/r/review` | `index.html:553` | Ver §2. |

---

## 2. Contenido real (cableado)

- "Ver todas las opiniones" / leer reseñas → `https://share.google/M8TjRifNDFBQa3QtY`
- CTA "Dejar reseña" → `https://g.page/r/CYDagfJmTET-EBM/review`
- Stats del hero (`+2.000 personas tratadas`, `8+ años`, `3 pilares`) → **se conservan tal cual, son reales.**
- Schema.org: añadir `sameAs: ["https://www.instagram.com/re3move/"]` y `hasMap` apuntando al embed/maps.

---

## 3. Imágenes de cards: `background-image` → `<img>`

En `index.html`, las cards de Services (`.service-card`/`.service-bg`) y Mujer (`.mujer-card`/`.mujer-bg`) usan `<div style="background-image:url(...)">`. Se reestructuran a:

```html
<a href="..." class="service-card">
  <img class="service-bg" src="/assets/photos/..." alt="[descripción concreta de la foto]" loading="lazy" width="W" height="H" decoding="async">
  <div class="service-overlay"></div>
  <div class="service-body">...</div>
</a>
```

- CSS: `.service-bg`/`.mujer-bg` pasan de `background-size/position` a `position:absolute; inset:0; width:100%; height:100%; object-fit:cover; object-position:center;`. El hover `transform:scale(1.04)` se mantiene (ahora sobre el `<img>`).
- `width`/`height` se fijan con la relación de aspecto intrínseca real de cada foto (leída en implementación), no con valores asumidos, para dar hint de aspect-ratio sin distorsionar el `object-fit:cover`.
- `alt` descriptivo por imagen. La primera card (above the fold) va **sin** `loading="lazy"` para no penalizar LCP; las demás `lazy`.
- Beneficio: lazy real + accesibilidad (alt) + menos layout shift.
- Mismo patrón si las subpáginas usan el mismo recurso (verificar en implementación; sólo donde aplique).

---

## 4. Reorden de secciones (`index.html`)

Orden actual: Hero → Services → Mujer → Method → Team → **Location → Contact → Testimonials** → Footer.

Orden nuevo: Hero → Services → Mujer → Method → Team → **Testimonials → Location → Contact** → Footer.

Mover el bloque `<section ... id="testimonios">` para que quede antes de `id="ubicacion"`. La prueba social genera confianza antes del CTA de contacto. Solo cambia el orden del markup; no se tocan estilos de sección.

---

## 5. Rediseño bloque Testimonios

Se mantiene la sección (`#testimonios`) pero se eliminan las 3 tarjetas con citas/nombres inventados (`María G.`, `Ana F.`, `Pedro S.`). Reemplazo:

- Header con `tag` "Opiniones" + título existente.
- Panel central de confianza: logo de Google (SVG ya presente), texto "Reseñas verificadas en Google", y dos CTAs:
  - **Leer reseñas** → `https://share.google/M8TjRifNDFBQa3QtY` (botón secundario/ghost).
  - **Dejar una reseña** → `https://g.page/r/CYDagfJmTET-EBM/review` (botón primario).
- Sin estrellas/quotes/avatares ficticios. Estilos nuevos van a `styles.css` (no inline).
- Se eliminan reglas CSS muertas de `.test-card/.test-stars/.test-quote/.test-author/.test-avatar/.test-name/.test-service` que dejen de usarse, sustituidas por las del nuevo panel.

---

## 6. Accesibilidad (todas las páginas)

- **Skip link:** `<a href="#main" class="skip-link">Saltar al contenido</a>` como primer elemento del `<body>` (después de nada, antes del nav). El contenido principal de cada página se envuelve en `<main id="main">` (abre tras el menú móvil, cierra antes del `<footer>`). Estilo: oculto fuera de pantalla salvo `:focus`/`:focus-visible`.
- **Nav por teclado:** `.nav-dd-label` focusable (`tabindex="0"` o `<button>`); abrir menú con `.nav-dd:focus-within .nav-dd-menu` además del `:hover`. Foco visible global: `:focus-visible { outline: 2px solid var(--green); outline-offset: 2px; }`.
- **Hamburguesa:** `aria-expanded` y `aria-controls="mobileMenu"`, toggle en JS al abrir/cerrar; `mobileMenu` con `aria-hidden` sincronizado o role apropiado.
- **`mm-group-label`:** focusable y operable por teclado (Enter/Espacio) para abrir submenús.
- **SVGs decorativos:** `aria-hidden="true"` + `focusable="false"`. Iconos que son único contenido de un enlace mantienen `aria-label` en el `<a>`.
- **Imágenes:** `alt` significativo (cards) o `alt=""` si decorativas.
- **Contraste (subir a AA para texto):** `.hero-sub` `rgba(255,255,255,0.5)`→`0.75`; `.hero-stat-label` `0.38`→`0.62`; `.hero-kicker-text` `0.45`→`0.62`; `.footer-desc`/`.footer-links a`/`.footer-copy` subir opacidades a ≥`0.6`. Mantener jerarquía visual; solo los mínimos para legibilidad.
- **Movimiento:** `@media (prefers-reduced-motion: reduce)` → desactivar `fadeUp`, transiciones de transform/scale y el scroll-reveal (mostrar contenido sin animación).

---

## 7. SEO (todas las páginas)

- Añadir en cada `<head>`: `<meta property="og:url" content="[URL canónica de la página]">`, `<meta property="og:site_name" content="Re3Move">`, `<meta property="og:image:alt" content="...">`, `<meta name="twitter:card" content="summary_large_image">`, `<meta name="twitter:title">`, `<meta name="twitter:description">`, `<meta name="twitter:image">`, `<meta name="theme-color" content="#3a5748">`, `<meta name="robots" content="index,follow">`.
- **Canonicals consistentes:** unificar sin `.html` (nginx ya sirve rutas limpias). `clinica-deportiva.html` pasa de `https://re3move.es/clinica-deportiva.html` → `https://re3move.es/clinica-deportiva`. `index` → `https://re3move.es/`.
- **Schema.org (`index.html`):** añadir `image` (foto real del sitio), `sameAs: ["https://www.instagram.com/re3move/"]`, `hasMap` (URL del mapa) y `areaServed: "Palma de Mallorca"`. **No** añadir `priceRange` (el sitio dice "Tarifas a consultar"; inventarlo sería engañoso). No inventar coordenadas exactas; usar la dirección postal ya presente.
- **`public/robots.txt`:** permitir todo + `Sitemap: https://re3move.es/sitemap.xml`.
- **`public/sitemap.xml`:** 4 URLs (`/`, `/clinica-deportiva`, `/fisioterapia-avanzada`, `/mujer`) con `lastmod` 2026-05-18.

---

## 8. Higiene de código e infra

- **`.gitignore`** en la raíz: `.DS_Store`, `*.log`, `.idea/`, `.vscode/`. Quitar del índice los `.DS_Store` ya trackeados (`git rm --cached`), sin borrarlos del disco si el usuario los tiene localmente (se borran del repo).
- **Inline styles:** mover a `styles.css` (o `<style>` de página) los bloques inline grandes de `index.html` (header de testimonios, botón Google, `margin` repetidos de `section-sub`). Crear utilidades en vez de repetir inline. No es refactor masivo: solo lo que toca esta mejora.
- **`nginx.conf` — cabeceras de seguridad** (bloque server, aplican a todo):
  - `add_header X-Content-Type-Options "nosniff" always;`
  - `add_header X-Frame-Options "SAMEORIGIN" always;`
  - `add_header Referrer-Policy "strict-origin-when-cross-origin" always;`
  - Mantener el resto del `nginx.conf` igual (puerto 8080, gzip, cache assets 1y, rutas limpias).

---

## Verificación

1. Servir `public/` localmente (`npx serve public` o build Docker) y abrir las 4 páginas.
2. Mapa carga la ubicación real (Carrer George Orwell 1, Palma).
3. Imágenes de cards visibles e idénticas; en DevTools, las below-the-fold cargan `lazy`.
4. Tab-navegación: skip link aparece al primer Tab; dropdowns del nav se abren con teclado; foco visible.
5. Reseñas: ambos botones abren las URLs correctas.
6. `view-source`: og/twitter/canonical correctos por página; `robots.txt` y `sitemap.xml` accesibles.
7. Lighthouse/axe rápido: sin errores críticos de contraste/a11y nuevos; `prefers-reduced-motion` desactiva animaciones.
8. `git status`: `.DS_Store` ya no trackeado.

---

## Secuencia de implementación (alto nivel)

1. `styles.css`: quitar `@import`, ajustar `.service-bg/.mujer-bg` a `<img>`, contrastes, focus-visible, reduced-motion, estilos del nuevo panel de testimonios, utilidades para ex-inline.
2. `index.html`: fuentes en head, mapa, conversión imgs, reorden secciones, rediseño testimonios, a11y (skip-link/aria/main), SEO head, Schema.org, limpiar inline.
3. `clinica-deportiva.html`, `fisioterapia-avanzada.html`, `mujer.html`: fuentes en head, a11y (skip-link/aria/main/foco), SEO head, canonicals.
4. `public/robots.txt`, `public/sitemap.xml`.
5. `.gitignore` + `git rm --cached` de `.DS_Store`.
6. `nginx.conf`: cabeceras de seguridad.
7. Verificación local de las 4 páginas.
