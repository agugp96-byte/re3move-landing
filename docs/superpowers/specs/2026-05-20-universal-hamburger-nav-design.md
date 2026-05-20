# Re3Move Landing — Navegación universal con hamburguesa

**Fecha:** 2026-05-20
**Autor:** Claude (con Augusto García)
**Estado:** Aprobado
**Base:** Estado post commit `b7188a7` (mejoras de a11y/SEO/visual ya en `main`).

---

## Objetivo

Resolver el problema reportado por el usuario: al entrar en las subpáginas (Mujer, Fisioterapia avanzada, Clínica deportiva) "se pierde la posibilidad de ver el inicio". Unificar la navegación con un único patrón visible y consistente en las 4 páginas y en todos los tamaños: **logo a la izquierda + hamburguesa a la derecha**, con el mismo menú detrás en todas las páginas.

---

## Alcance

Aplica a las 4 páginas (`index.html`, `clinica-deportiva.html`, `fisioterapia-avanzada.html`, `mujer.html`) y a `assets/styles.css`.

**Fuera de alcance:** no se tocan copy, equipo, precios, links de WhatsApp/email, el footer, ni el contenido de las secciones. No se cambia el orden de secciones.

---

## Diseño

### Top-bar (idéntico en las 4 páginas, todos los viewports)

```
[ R3 re³MOVE                                                ☰ ]
```

- Izquierda: el `.nav-logo` actual (`R3` + `re³MOVE`) sigue enlazando a `/`.
- Derecha: solo el botón hamburguesa `.hamburger`. Nada más.
- Se **elimina del DOM** la barra `<div class="nav-links">…</div>` (escritorio) y el botón `<a class="nav-cta">Pedir cita</a>` del top-bar — su contenido pasa al menú hamburguesa.
- CSS: la regla actual `@media (max-width:900px) { .nav-links { display:none } .hamburger { display:flex } }` se sustituye por reglas base: `.hamburger { display:flex }` siempre. La regla de `.nav-links` se elimina (la barra ya no existe en el DOM).
- Estética del top-bar (altura 68px, glass, sombra al hacer scroll) se conserva.

### Hamburguesa (panel `.mobile-menu`, idéntico en las 4 páginas)

Estructura plana, sin acordeones. Los 8 ítems de navegación + 1 CTA, en este orden:

| # | Texto | Href | Notas |
|---|---|---|---|
| 1 | Inicio | `/` | NUEVO |
| 2 | Fisioterapia avanzada | `/fisioterapia-avanzada` | |
| 3 | Clínica deportiva | `/clinica-deportiva` | |
| 4 | Mujer | `/mujer` | |
| 5 | Método | `/#metodologia` | |
| 6 | Equipo | `/#equipo` | |
| 7 | Ubicación | `/#ubicacion` | |
| 8 | Contacto | `/#contacto` | |
| — | **Pedir cita** | `https://wa.me/34620326447` | `target="_blank"`, clase `.mm-cta` (botón verde destacado, al final). Solo este ítem mantiene el estilo CTA. |

- Se **eliminan** los grupos `.mm-group / .mm-submenu / .mm-group-label / .mm-caret` que contenían los acordeones. El JS de toggling de submenus (`document.querySelectorAll('.mm-group-label').forEach…`) se elimina; el resto del JS del menú móvil (toggle hamburguesa, click-outside, foco) se conserva.
- El item activo (la página actual) recibe `aria-current="page"` y un estilo sutil (peso 700, color `--ink`) para orientar al usuario.

### Comportamiento

- Click en hamburguesa → abre/cierra el panel. `aria-expanded` sincronizado (ya implementado).
- Click fuera del panel → cierra. Click en cualquier item → cierra (el navegador navega).
- `Esc` → cierra y devuelve foco al botón. **NUEVO** (mejora de a11y, JS de ~3 líneas).
- Foco visible global (ya implementado vía `:focus-visible` en `styles.css`).

### CSS

- `.hamburger { display: flex; ... }` (base, no en media query).
- Borrar el bloque `.nav-links` y todas sus declaraciones internas + las reglas del media query 900px que la ocultaban.
- El panel `.mobile-menu` mantiene su posicionamiento actual sin cambios (`top: 60px; right: 16px; width: 260px;`) — funciona igual de bien en escritorio que en móvil.
- Borrar reglas obsoletas: `.mm-group`, `.mm-group-label`, `.mm-submenu`, `.mm-caret`, `.mm-group.open` (todas se vuelven dead code al quitar los acordeones).
- Estilo opcional pequeño para el item activo:
  ```css
  .mobile-menu a[aria-current="page"] { font-weight: 700; color: var(--ink); }
  ```

### HTML (resumen del cambio en cada página)

- **Quitar:** `<div class="nav-links">…</div>` (toda la barra desktop) y el botón `<a … class="nav-cta">Pedir cita</a>` que estaba dentro de `.nav-right`.
- **Mantener:** `.nav-logo`, `.hamburger` (sin cambios funcionales).
- **Reescribir el contenido de** `<div class="mobile-menu" id="…">` con los 9 ítems planos descritos arriba. Mantener el `id` específico de cada página (`mobileMenu` en index/clinica-deportiva, `mobile-menu` en fisioterapia-avanzada/mujer) y el `aria-controls` del botón hamburguesa coherente con ese id.
- **Marcar como `aria-current="page"`** el item correspondiente a cada página (en `index.html` es "Inicio"; en `mujer.html` es "Mujer"; etc.).

### JS

- Borrar el bloque `document.querySelectorAll('.mm-group-label').forEach(…)` (acordeones).
- Añadir un único listener para cerrar con `Esc`:
  ```js
  document.addEventListener('keydown', (e) => {
    if (e.key === 'Escape' && menu.classList.contains('open')) {
      menu.classList.remove('open');
      btn.setAttribute('aria-expanded', 'false');
      btn.focus();
    }
  });
  ```
  El nombre exacto de la variable del menú/botón varía por página (algunas usan `mobileMenu`/`hamburger`, otras usan IIFEs con `var menu`/`var btn`). Adaptarse a la convención existente de cada archivo.

---

## Riesgos y mitigaciones

- **Cambio de hábito en escritorio:** quitar la barra inline es un cambio visible. Mitigado porque (a) los visitantes recurrentes son pocos (sitio recién lanzado), (b) la hamburguesa es un patrón universal, (c) "Pedir cita" sigue accesible en un click (abre menú → primer botón destacado).
- **Discoverability de "Pedir cita":** al esconder el CTA detrás del menú se pierde la prominencia. Se compensa con: (i) el botón flotante de WhatsApp `.wa-float` (ya existe abajo a la derecha), (ii) los CTAs "Pedir cita por WhatsApp" dentro del contenido de cada página (hero, contacto, CTA finales), (iii) el item "Pedir cita" estilizado como botón verde destacado al final del menú.
- **Regresión de a11y:** cambio mínimo, se conservan `aria-expanded`, `aria-controls`, foco visible, skip-link, `<main>`. Añade cierre con `Esc` (mejora).

---

## Verificación

- Servir local + smoke visual:
  - Top-bar: solo logo a la izq y hamburguesa a la der en las 4 páginas, a 1280px y 390px.
  - Click hamburguesa → panel con los 9 ítems en orden, "Pedir cita" como botón verde al final.
  - "Inicio" presente y enlaza a `/` en las 4 páginas; `aria-current="page"` en el ítem correspondiente.
  - Esc cierra el panel.
- Grep:
  - `grep -rn 'class="nav-links"' public/*.html` → sin resultados.
  - `grep -rn 'class="nav-cta"' public/*.html` → sin resultados.
  - `grep -rn 'mm-group\|mm-submenu\|mm-caret\|mm-group-label' public/*.html public/assets/styles.css` → sin resultados (HTML y CSS limpios).
  - `grep -rn '>Inicio<' public/*.html` → 4 resultados (uno por página).

---

## Despliegue

Mismo mecanismo que el commit anterior: commit en `claude/blissful-heyrovsky-0b78b2`, fast-forward push a `origin/main`, Railway redespliega automáticamente.
