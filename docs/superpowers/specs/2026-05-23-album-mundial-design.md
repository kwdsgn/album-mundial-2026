# Diseño: Álbum Mundial 2026 Tracker

**Fecha:** 2026-05-23  
**Proyecto:** awc (Album World Cup)  
**Tipo:** Single-page web app — mobile-first

---

## Resumen

App web estática para hacer seguimiento del álbum Panini del Mundial 2026. El usuario puede ver qué láminas tiene, cuáles le faltan, y marcar láminas nuevas a medida que las consigue. Sin backend, sin login, sin instalación en el teléfono.

---

## Decisiones de Diseño

### Tecnología
- **Un solo archivo `index.html`** — HTML, CSS y JS inline. Cero dependencias externas.
- **Hosting:** GitHub Pages (URL pública, abre en cualquier navegador del teléfono)
- **Persistencia:** `localStorage` del navegador. Se borra limpiamente con "Limpiar datos del sitio".

### Data
- **Fuente original:** Google Sheets público del usuario (48 secciones × 20 láminas = 960 láminas totales)
- **Estrategia:** El estado inicial del Sheet se hardcodea como objeto JS en el HTML (el URL del Sheet tiene redirects CORS que bloquean fetch desde el navegador). No hay fetch en runtime.
- **Persistencia:** Al tocar cualquier lámina, el estado completo se serializa y guarda en `localStorage` bajo la clave `"awc_state"`. Al cargar la app, se lee de localStorage; si no existe, se usa el estado inicial hardcodeado.
- **Schema localStorage:**
  ```json
  {
    "stickers": {
      "Argentina": [true, false, true, ...],
      "Brasil": [false, true, false, ...]
    },
    "duplicates": {
      "Argentina": [false, false, true, ...],
      "Brasil": [false, false, false, ...]
    }
  }
  ```

### Estructura de secciones (regiones)

| Tab | Países |
|-----|--------|
| FWC | FWC |
| Américas | Argentina, Brasil, Canadá, Colombia, Ecuador, Haití, México, Panamá, Paraguay, Uruguay, USA |
| Europa | Austria, Bélgica, Bosnia, Croacia, Czechia, Inglaterra, Alemania, Francia, Países Bajos, Noruega, Portugal, Escocia, España, Suecia, Suiza |
| África | Argelia, Cabo Verde, Congo DR, Côte d'Ivoire, Egipto, Ghana, Marruecos, Túnez, Senegal, Sudáfrica |
| Asia | Japón, Qatar, Corea, Arabia Saudita, Irán, Irak, Jordania, Uzbekistán, Turquía |
| Oceanía | Australia, Nueva Zelanda |

---

## Layout & UI

### Header
- Logo oficial FIFA World Cup 2026 (imagen embebida en base64 o URL pública)
- Texto "Álbum Mundial 2026"
- Barra de progreso global (láminas conseguidas / total)
- Contador: "362 / 960 láminas · 38% completado" (calculado dinámicamente)

### Navegación — Tabs de región
- Barra horizontal scrolleable con tabs: **FWC | Américas | Europa | África | Asia | Oceanía**
- Tab activo marcado con borde inferior negro
- Sticky debajo del header

### Vista principal — Álbum
- Lista scrolleable de tarjetas por país
- Cada tarjeta tiene:
  - Nombre del país con emoji de bandera
  - Contador "X/20" con color dinámico:
    - 🟢 Verde: ≥75% completado
    - 🟡 Amarillo: 25–74%
    - 🔴 Rojo: <25%
  - Grid de **círculos** numerados (1–20) por lámina:
    - ✅ **Verde** (`#dcfce7` fill, `#86efac` border) = tiene la lámina
    - ❌ **Rojo** (`#fee2e2` fill, `#fca5a5` border) = le falta
  - Tocar un círculo **alterna** el estado (tiene ↔ falta) y guarda en localStorage

### Vista — Progreso 🏆
- Resumen visual: total de láminas, completadas, faltantes
- Lista de países ordenada por % de completitud (más completos primero)
- Indicador de "selecciones completas" (100%)

### Vista — Repetidas 🎴
- Misma estructura de tarjetas por país que la vista Álbum, pero solo muestra láminas que el usuario **ya tiene** (`stickers[país][i] === true`)
- Tocar un círculo en esta vista lo marca/desmarca como repetida (azul)
- Útil para saber qué láminas ofrecer para intercambiar

### Bottom Nav
- **⚽ Álbum** — vista principal
- **🏆 Progreso** — estadísticas
- **🎴 Repetidas** — láminas duplicadas

---

## Interacción

| Acción | Resultado |
|--------|-----------|
| Tocar círculo (vista Álbum) | Alterna tiene ↔ falta, guarda en localStorage |
| Tocar círculo (vista Repetidas) | Marca/desmarca esa lámina como repetida (azul) — solo aplica a láminas que ya se tienen |
| Cambiar tab de región | Filtra países por continente (solo en vista Álbum) |
| Cambiar tab de nav | Cambia entre Álbum / Progreso / Repetidas |

---

## Visual

- **Fondo:** Blanco (`#ffffff`), cards con sombra sutil
- **Tipografía:** system-ui / -apple-system (sin fuentes externas)
- **Colores de estado:**
  - Tiene: `#dcfce7` (verde claro) · border `#86efac`
  - Falta: `#fee2e2` (rojo claro) · border `#fca5a5`
  - Repetida: `#dbeafe` (azul claro) · border `#93c5fd`
- **Chips:** Círculos de 32px × 32px
- **Mobile-first:** Diseñado para pantallas de 375px+

---

## Fuente de datos

- **Sheet original:** `https://docs.google.com/spreadsheets/d/1qoLraBjhWGQYtc_BnkocT3xb8ANkzgA-zXh7iEcnWxc/edit`
- **Columnas:** Nombres de países en fila 1 (48 columnas + columna vacía inicial)
- **Filas:** Posiciones de lámina 1–20 (fila 2 = posición 1, fila 21 = posición 20)
- **Valor `x`:** Lámina poseída · Celda vacía: falta
- **Carga:** Estado inicial hardcodeado como constante JS `INITIAL_STATE` en el HTML. No hay fetch en runtime.

---

## Fuera de scope

- Login / autenticación
- Sincronización automática con Google Sheets
- Notificaciones push
- Modo offline (PWA / Service Worker)
- Múltiples álbumes o usuarios
