# konachan-scraper

Lightweight ESM library to fetch random anime image URLs from **konachan.com** / **konachan.net** by tag.

No economy. No gacha. Solo imágenes.

---

## Install

```bash
# dentro de tu proyecto
npm install
# o copia la carpeta src/ directamente
```

> Requiere Node ≥ 18 (fetch nativo + AbortSignal.timeout).

---

## Usage

```js
import { KonachanScraper } from "konachan-scraper";

// URL aleatoria para un tag
const url = await KonachanScraper.getRandomUrl("hatsune_miku");
console.log(url);
// → "https://konachan.com/image/.../sample.jpg"  o null si no hay resultados

// Post completo (con id, tags, rating, source, etc.)
const post = await KonachanScraper.getRandomPost("rem_(re:zero)");
if (post) {
    console.log(post.id, post.tags, post.sample_url);
}

// Limpiar caché manualmente (opcional)
KonachanScraper.clearCache();
```

### En un plugin de Eris-MD

```js
import { KonachanScraper } from "../lib/konachan-scraper/index.js";

handler.command = ["waifu", "anime"];

export async function handler(m, { conn }) {
    const tag = m.text.trim() || "hatsune_miku";
    const url = await KonachanScraper.getRandomUrl(tag);

    if (!url) return m.reply("❌ No encontré imágenes para ese tag.");

    await conn.sendMessage(m.chat, { image: { url }, caption: `🖼️ ${tag}` }, { quoted: m });
}
```

---

## API

### `KonachanScraper.getRandomUrl(tag)` → `Promise<string|null>`

Devuelve la URL de muestra (`sample_url`) de un post aleatorio para el tag dado.

| Arg | Tipo | Descripción |
|-----|------|-------------|
| `tag` | `string` | Nombre del personaje/tag. Espacios se convierten en `_`. |

Orden de prioridad de URL: `sample_url` → `jpeg_url` → `file_url`.

---

### `KonachanScraper.getRandomPost(tag)` → `Promise<object|null>`

Igual que `getRandomUrl` pero devuelve el objeto post completo con todos los campos de la API de Konachan:

```js
{
  id, tags, author, source,
  rating,        // "s" | "q" | "e"
  score,
  file_url,      // imagen original (alta resolución)
  sample_url,    // muestra reducida (recomendada para bots)
  jpeg_url,      // JPEG intermedio
  width, height,
  file_size,
  created_at,
  // ...
}
```

---

### `KonachanScraper.clearCache()` → `void`

Vacía la caché interna de posts. Útil en bots de larga duración si quieres forzar resultados frescos.

---

## Comportamiento interno

| Feature | Detalle |
|---------|---------|
| **Caché** | TTL de 5 min por query. Evita flood a la API. |
| **Dedup de requests** | Si la misma query ya está en vuelo, reutiliza la Promise. |
| **Dual-source** | Consulta `konachan.net` (SFW server-side) + `konachan.com?rating:s` en paralelo y fusiona por ID. |
| **Fallback de tag** | Si `rem_(re:zero)` da < 3 resultados, mezcla con `rem`. |
| **Filtro de contenido** | Bloquea posts con rating `e` (explícito) y tags `loli/shota/child/toddler/infant`. |
| **Timeout** | 6 segundos por request con `AbortSignal.timeout`. |

---

## Ratings

| Rating | Significado | ¿Permitido? |
|--------|-------------|-------------|
| `s`    | Safe        | ✅ |
| `q`    | Questionable | ✅ |
| `e`    | Explicit    | ❌ filtrado |
