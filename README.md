# mirandaperezhita — portfolio dinámico

Sitio estático minimalista construido con HTML, CSS y JavaScript vainilla. Carga los proyectos desde archivos JSON, cambia de idioma en caliente y muestra un panel “About” superpuesto sin recargar la página.

## Arquitectura

- `index.html`: sólo declara el layout (sidebar, contenedor de proyectos y panel About). Todo el contenido llega vía JS.
- `js/app.js`: arranque `init()`, descarga de JSON, renderizado de menú y secciones, sincronización de scroll y panel About.
- `css/styles.css`: tipografía EB Garamond, layout de dos columnas, sistema de imágenes basado en `--ratio`/`--scale` y transiciones suaves para el About.
- `data/`: `home.json` define colores y qué proyectos son visibles; cada `*.json` describe un proyecto.
- `images/`: recursos usados en los proyectos (con fallback en `images/reference/`).

```
mirandaperezhita/
├── index.html
├── css/styles.css
├── js/app.js
├── data/
│   ├── home.json
│   ├── about.json
│   └── [slug].json
└── images/
```

## Flujo de la aplicación

1. `init()` se ejecuta en `DOMContentLoaded`.
2. Descarga `data/home.json`, prepara la mezcla de colores y pinta el fondo del sidebar según el idioma.
3. Carga en paralelo los JSON de los proyectos visibles (`data/[slug].json`).
4. Renderiza el menú lateral (botón por proyecto) y todas las secciones dentro de `#projects-container`.
5. Activa listeners: cambio de idioma, apertura/cierre del About (`setAboutOpen`), sincronización de scroll (IntersectionObserver) y recalculo de `safe-area`.
6. `updateProjectsContent()` reutiliza el DOM existente cuando se cambia de idioma para mantener todo ligero.

### Datos esperados

`data/home.json`:

```json
{
  "home_colors": {
    "cat": "#E6D8C3",
    "es": "#F8E3CE"
  },
  "nota_de_curt": 50,
  "projectes_visibles": [
    {
      "slug": "new-ywork",
      "color": "#748873",
      "visible": true
    }
  ]
}
```

- `home_colors`: color de fondo por idioma (fallback a `cat`).
- `nota_de_curt`: 0–100. Con 0 se consideran todos los colores “oscuros” y se mezcla hacia negro.
- `projectes_visibles`: orden y color base de cada proyecto.

`data/[slug].json`:

```json
{
  "slug": "new-ywork",
  "primera_imatge": { "src": "images/NY/1.webp", "size": "100" },
  "titol": { "cat": "New Ywork", "es": "New Ywork" },
  "client": { "cat": "Jaume Cloret", "es": "Jaume Cloret" },
  "data": "2024",
  "text": {
    "cat": [
      "Disseny del catàleg…"
    ],
    "es": [
      "Diseño del catálogo…"
    ]
  },
  "imatges": [
    { "src": "images/NY/2.webp", "size": "50", "w": 6355, "h": 7944 }
  ]
}
```

- `size` es un porcentaje 1–100 que se traduce a `--scale` dentro de la `media-frame`.
- `w` / `h` son las dimensiones reales en píxeles de la imagen. Opcionales, pero recomendados: permiten que el navegador reserve el espacio antes de cargar la imagen, evitando saltos de layout.
- Las claves de texto aceptan objeto por idioma: cada idioma puede ser string único o array de párrafos. El formateador (`formateador.html`) genera el array listo para pegar.

## Deep links

Cada proyecto tiene su URL directa usando el hash del slug:

```
https://mirandaperezhita.com/#carnaval
https://mirandaperezhita.com/#ejercicios
https://mirandaperezhita.com/#new-ywork
```

Al navegar por el portfolio el hash se actualiza solo (`history.replaceState`), sin añadir entradas al historial del navegador. Si se accede directamente a una URL con hash, la página hace scroll automático al proyecto correspondiente al cargar.

## Detalles de implementación

- **Colores dinámicos**: `prepareProjectColorData()` convierte hex → RGB, calcula tono y mezcla con negro/blanco para obtener colores de texto con buen contraste. Con `nota_de_curt = 0` se mantiene la mezcla hacia negro, que actualmente funciona bien con la paleta.
- **Imágenes**: `makeMediaFrame()` crea un contenedor con `aspect-ratio`. El ancho máximo es el tamaño natural de la imagen; por eso las imágenes pequeñas (como `images/monicaPlanes/3.webp`) quedan centradas sin escalar para evitar pixelado. Si se quisiera forzar a pantalla completa, se puede romper esa limitación y aceptar la pérdida de nitidez (ver notas en `todo.md`).
- **Panel About**: contenido se genera desde `data/about.json`. El botón del sidebar siempre se ve en negro (override directo en CSS) para mantener referencia estable.
- **Accesibilidad básica**: el botón activo de proyecto usa `aria-current="true"`, el panel About alterna `aria-hidden`/`aria-expanded`, y `Esc` cierra el overlay.

## Desarrollo y pruebas

1. Clonar el repo y abrir la carpeta en tu editor.
2. Servir con cualquier servidor estático, por ejemplo:

```bash
python3 -m http.server 8080
```

3. Abrir `http://localhost:8080`.
4. Editar JSON en `data/` y recargar el navegador para ver cambios.

## Mantenimiento

- El listado de tareas y pendientes se consolida en `todo.md`.
- Para cambiar el color base de la home, ajustar `home_colors` en `data/home.json`.
- Para añadir un proyecto, duplicar un JSON existente en `data/`, actualizar rutas de imagen y referenciarlo en `home.json`.

## Créditos

Diseño y contenidos: miranda perez hita.  
Implementación front-end: HTML/CSS/JS vainilla con enfoque minimalista.
