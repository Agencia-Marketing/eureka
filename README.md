# Eureka Rain — landing animada (GSAP)

Landing de una sola página, estática, con lluvia cartoon de fondo y animaciones GSAP.
Sin build, sin framework: 1 archivo HTML + 4 SVG. GSAP se carga por CDN.

**Live:** https://eureka-rain.programacionagencia.workers.dev

---

## Stack

- **HTML/CSS/JS vanilla** en un solo archivo (`index.html`).
- **GSAP 3.12.5** + plugin **ScrollTrigger** (CDN cdnjs).
- Hospedaje: **Cloudflare Workers** (static assets) vía `wrangler`.

## Archivos

| Archivo | Rol |
|---|---|
| `index.html` | Todo el sitio: markup, estilos (`<style>`), lógica (`<script>`). |
| `eureka-azul.svg` / `eureka-rojo.svg` / `eureka-verde.svg` / `eureka-amarillo.svg` | Personajes "eurekan" (blobs de color con cara). |
| `wrangler.jsonc` | Config de deploy (Worker solo-assets). |
| `.assetsignore` | Excluye del deploy: `wrangler.jsonc`, `.assetsignore`, `.agents/`, `skills-lock.json`. |

> Nota: los SVG se renombraron sin espacios para evitar problemas de rutas en hosting.

---

## Estructura de la página

Fondo global: `<svg id="rain">` fijo (`position:fixed`) cubre todo el viewport, detrás de todo.

1. **`#hero`** — pantalla completa, *pin* con scroll (`ScrollTrigger` scrub).
   - 4 personajes asomando por los bordes laterales (verde/azul arriba, rojo/amarillo abajo),
     verde+rojo girados 90° derecha, azul+amarillo 90° izquierda.
   - Al hacer scroll: cada personaje sale por su lado (`xPercent ±260` + fade) y el
     texto `#copy` ("WERE SPARK MEETS STRATEGY") sube y aparece.
2. **`#rotator-section`** — frase "If you are" + palabra rotatoria.
   - Una palabra a la vez (`entrepreneur`, `content creator`, `bold brand`),
     apareciendo **letra por letra** (stagger), en bucle. Ancho fijado a la palabra
     más larga para evitar saltos de layout.
3. **`#cta`** — "Ready To Get Started?" + formulario.
   - Decoraciones cartoon **SVG line-art** (`.deco`): nubes, garabatos (`+`, círculos,
     squiggles), árboles y arbustos en la base.
   - Burbuja de diálogo con `eureka-amarillo.svg`.
   - Al entrar en viewport (una vez): encabezado y burbuja entran, la **burbuja cae**
     desde arriba con rebote (`bounce.out`) y al aterrizar **aparece el formulario**.
   - Formulario **demo** (sin backend): valida Name/Email/Phone + checkbox Confirm;
     errores marcan borde rojo; éxito colapsa el form y muestra mensaje.

---

## Detalles de animación (GSAP)

- **Lluvia:** cada gota es una `<line>` SVG; `makeDrop()` la crea, `animateDrop()` la
  hace caer en bucle con `x`, largo, grosor (`stroke-width` 0.8–2) y velocidad aleatorios.
  Independiente del scroll. `COUNT = 90` gotas. `resize()` reajusta el viewBox.
- **Hero pin/scrub:** timeline ligada al scroll del `#hero` (`pin:true`, `scrub:true`).
- **Rotador de palabras:** timelines recursivas (`cycle`) tras `document.fonts.ready`.
- **CTA intro:** `ScrollTrigger.create({ once:true, onEnter })`; la distancia de caída
  de la burbuja se calcula con `getBoundingClientRect()` hacia el centro del form.

## Decoraciones cartoon (cómo se dibujan)

Todo es **SVG inline** con `fill:none; stroke:#111` (line-art):
- Nube = un `<path>` de curvas `Q` (cuadráticas) encadenadas.
- Árbol frondoso = 3 `<circle>` (copa) + `<path>` recto (tronco).
- Árbol hoja = `<path>` en gota con curvas `C` + tronco.
- Arbusto = `<path>` de bultos `Q` sobre la base.
- Cada deco es `.d` (`position:absolute`) posicionada con `top/left/right/bottom` + `width` inline.

---

## Desarrollo local

Abrir `index.html` en el navegador (doble clic) o servir estático:

```bash
npx serve .
# o
python -m http.server
```

## Deploy (Cloudflare Workers)

```bash
npm i -g wrangler
wrangler login        # interactivo, abre navegador
wrangler deploy
```

`wrangler.jsonc` usa `assets.directory: "."` → sirve `index.html` en `/`.
Re-deploy: editar y volver a correr `wrangler deploy`.

---

## Pendientes / ideas

- Confinar la lluvia a una sección concreta (hoy es global, `position:fixed`).
  Opciones: mover `#rain` dentro de la sección (`absolute` + `overflow:hidden`),
  o togglear `opacity` por scroll con ScrollTrigger.
- Conectar dominio propio (hoy en `*.workers.dev`).
- Formulario: conectar a backend/endpoint real (hoy es demo sin envío).
- Reemplazar textos placeholder del hero/rotador por copy final.
