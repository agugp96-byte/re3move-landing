# Re3Move Landing — Spec de diseño
**Fecha:** 2026-05-04  
**Autor:** Claude (con Agustín García)  
**Estado:** Aprobado

---

## Objetivo

Reescribir y mejorar la landing pública de Re3Move para desplegarla en Railway. Partimos de los archivos más avanzados del Design System (`Re3Move Design System (1)/landing/`), corregimos bugs, añadimos SEO completo, compartimos el CSS entre páginas y lo empaquetamos con un Dockerfile nginx listo para Railway.

---

## Páginas incluidas

| Página | Archivo | Fuente canónica |
|--------|---------|-----------------|
| Home | `index.html` | Design System `landing/index.html` |
| Clínica deportiva | `clinica-deportiva.html` | Design System `landing/clinica-deportiva.html` |
| Fisioterapia avanzada | `fisioterapia-avanzada.html` | Design System `landing/fisioterapia-avanzada.html` |
| Mujer | `mujer.html` | Design System `landing/mujer.html` |

Las versiones antiguas de `landing/` (fisioterapia.html, readaptacion.html, suelo-pelvico.html, embarazo.html) **no se migran** — están supersedidas por las versiones del Design System.

---

## Estructura de archivos

```
re3move-landing/
├── public/
│   ├── index.html
│   ├── clinica-deportiva.html
│   ├── fisioterapia-avanzada.html
│   ├── mujer.html
│   └── assets/
│       ├── styles.css              ← CSS compartido
│       ├── photos/                 ← Copias de las fotos del Design System
│       │   ├── clinic-01-pregnant-monitor.jpeg
│       │   ├── clinic-02-physio-baby.jpeg
│       │   ├── clinic-03-ecografia.jpeg
│       │   ├── clinic-04-pelvis-model.jpeg
│       │   ├── clinic-05-pregnancy-side.jpeg
│       │   ├── clinic-06-prenatal-class.jpeg
│       │   ├── clinic-07-judith-pelvis.jpeg
│       │   ├── clinic-08-postpartum-floor.jpeg
│       │   ├── training-01-stability.png
│       │   ├── training-02-cable-rotation.png
│       │   ├── training-03-step-up.png
│       │   └── training-04-band-balance.png
│       ├── logo.jpg
│       ├── favicon.svg
│       └── icon.svg
├── Dockerfile
├── nginx.conf
└── docs/
    └── superpowers/specs/
        └── 2026-05-04-re3move-landing-design.md
```

---

## CSS compartido (`assets/styles.css`)

Extrae los bloques que se repiten en todas las páginas:

- Variables CSS (tokens de color, tipografía, sombras, radios)
- Reset universal (`box-sizing`, márgenes)
- Import de fuentes Google (Syne, DM Sans, Barlow Condensed)
- Componentes nav: `.nav`, `.nav-inner`, `.nav-logo`, `.nav-logo-mark`, `.nav-logo-name`, `.nav-links`, `.nav-dd`, `.nav-dd-menu`, `.nav-right`, `.nav-cta`, `.hamburger`
- Menú mobile: `.mobile-menu`, `.mm-group`, `.mm-submenu`, `.mm-caret`
- Footer: `.footer`, `.footer-inner`, `.footer-top`, `.footer-brand-name`, `.footer-tagline`, `.footer-desc`, `.footer-col-title`, `.footer-links`, `.footer-divider`, `.footer-bottom`, `.footer-copy`, `.footer-socials`, `.soc`
- Botones compartidos: `.btn-wa`, `.btn-ghost`, `.btn-primary`, `.btn-secondary`, `.nav-cta`
- WhatsApp float: `.wa-float`
- Utilidades: `.container`, `.section`, `.tag`, `.section-title`, `.section-sub`
- Animaciones: `@keyframes fadeUp`, `.fade-up`, `.d1`–`.d4`
- Scroll reveal JS compartido (mismo bloque en todas las páginas)

Cada HTML incluye: `<link rel="stylesheet" href="/assets/styles.css">` y añade solo su CSS específico en un `<style>` inline.

---

## Mejoras por área

### Rutas de assets
- Todas las referencias a imágenes pasan de `../assets/photos/foto.jpeg` a `/assets/photos/foto.jpeg`
- Garantiza que funcionen independientemente del path de la URL

### Bugs corregidos
1. **Scroll reveal** — El selector `.ch-btn` no existe; la clase real es `.ch-btn-big`. Corregido en el JS compartido.
2. **Hamburger en desktop** — `@media (min-width: 901px) { .hamburger { display: flex } }` en el Design System index hace que aparezca en escritorio. Corregido: hamburger solo visible `@media (max-width: 900px)`.
3. **Mapa placeholder** — La sección Ubicación tiene un div estático simulando un mapa. Se reemplaza con un `<iframe>` de Google Maps embed apuntando a "Carrer George Orwell 1, Palma de Mallorca".

### SEO
Cada página incluye en `<head>`:
```html
<!-- Open Graph -->
<meta property="og:type" content="website">
<meta property="og:title" content="[Título de página] — Re3Move">
<meta property="og:description" content="[Descripción específica]">
<meta property="og:image" content="/assets/photos/clinic-02-physio-baby.jpeg">
<meta property="og:locale" content="es_ES">

<!-- Canonical -->
<link rel="canonical" href="https://re3move.es/[ruta]">

<!-- Favicon -->
<link rel="icon" href="/assets/favicon.svg" type="image/svg+xml">
<link rel="apple-touch-icon" href="/assets/icon.svg">
```

`index.html` añade Schema.org LocalBusiness:
```json
{
  "@type": "PhysicalTherapist",
  "name": "Re3Move",
  "address": { "streetAddress": "Carrer George Orwell 1", "addressLocality": "Palma", "addressRegion": "Mallorca" },
  "telephone": "+34620326447",
  "email": "re3movecenter@gmail.com",
  "url": "https://re3move.es",
  "openingHours": "Mo-Fr 08:00-21:00"
}
```

### Rendimiento
- `loading="lazy"` en imágenes de las secciones services, mujer, team (below the fold)
- Fuentes con `font-display: swap` añadido al Google Fonts URL
- Las imágenes del hero (above the fold) mantienen carga normal (sin lazy)

### Navegación
- El nav con dropdowns del Design System `index.html` se usa en **todas** las páginas (actualmente las subpáginas tienen navs simplificados distintos)
- Los links del footer apuntan a las páginas reales: `/clinica-deportiva.html`, `/fisioterapia-avanzada.html`, `/mujer.html`
- El año del copyright se actualiza a 2026

---

## Despliegue Railway

### Dockerfile
```dockerfile
FROM nginx:alpine
COPY public/ /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```

### nginx.conf
```nginx
server {
    listen 80;
    root /usr/share/nginx/html;
    index index.html;
    charset utf-8;

    # Gzip
    gzip on;
    gzip_types text/css text/javascript application/javascript image/svg+xml;

    # Cache: assets estáticos un año
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Cache: HTML sin caché
    location ~* \.html$ {
        add_header Cache-Control "no-cache";
    }

    # Rutas limpias
    location / {
        try_files $uri $uri/ $uri.html =404;
    }
}
```

Railway detecta el Dockerfile en la raíz del repositorio y lanza el build automáticamente. No requiere variables de entorno ni secrets.

---

## Contenido — qué no cambia

- Toda la copia en español se mantiene exactamente como está en el Design System
- Los 3 testimonios se conservan
- El equipo (Xavier Calvo, Augusto García, Judith Iglesias) se conserva
- Los precios del plan online (79€, 129€, 199€) se conservan
- Los links de WhatsApp (`https://wa.me/34620326447`) se conservan
- El email (`re3movecenter@gmail.com`) se conserva

---

## Secuencia de implementación

1. Crear estructura de carpetas del proyecto
2. Copiar assets (fotos, logo, favicon) desde el Design System
3. Crear `assets/styles.css` con todos los estilos compartidos
4. Reescribir `index.html` — aplicar mejoras sobre la versión Design System
5. Reescribir `clinica-deportiva.html` — aplicar mejoras
6. Reescribir `fisioterapia-avanzada.html` — aplicar mejoras
7. Reescribir `mujer.html` — aplicar mejoras
8. Crear `Dockerfile` y `nginx.conf`
9. Verificar localmente con Docker o `npx serve public/`
10. Inicializar git repo y hacer primer commit listo para Railway
