````markdown
# 📡 Tech Emerging Radar: Agregador y Dashboard de Noticias

Este proyecto implementa un flujo automatizado (workflow) que **agrega las últimas noticias de tecnología y tendencias de IA** desde múltiples fuentes RSS, las filtra para obtener las más recientes (últimas 24 horas), y luego las presenta a través de dos canales:
1.  Un **Dashboard web** minimalista para visualización.
2.  Un **Webhook POST** para enviarlas a una plataforma de comunicación (como Slack, Telegram, etc.).

## 🚀 Componentes del Proyecto

El proyecto consta de dos partes principales: el **Flujo de Automatización (n8n)** y la **Interfaz Web (Frontend)**.

---

## 1. Flujo de Automatización (n8n)

El flujo llamado **"Tech Emerging Radar"** se encarga de la recopilación, procesamiento y envío de los datos.


### ⚙️ Nodos y Funcionalidad

| Nodo | Nombre | Tipo | Descripción |
| :--- | :--- | :--- | :--- |
| **cron** | Cada 3 horas | `cron` | **Activador**. Ejecuta el flujo automáticamente cada 3 horas. |
| **rss...** | RSS (5 fuentes) | `rssFeedRead` | Lee las últimas entradas de los feeds RSS de **TechCrunch, MIT Tech Review, Wired, VentureBeat AI y The Verge Tech**. |
| **merge** | Merge All RSS | `merge` | **Combina** todas las entradas de los 5 feeds RSS en una sola lista. |
| **filtro24h** | Filtrar últimas 24h | `function` | Filtra la lista para incluir **solo los artículos publicados en las últimas 24 horas**. |
| **sort** | Ordenar por fecha | `function` | Ordena los artículos filtrados por fecha de publicación de forma **descendente** (más recientes primero). |
| **top5** | Top 5 | `function` | Limita la lista a los **5 artículos más recientes** y relevantes. |
| **formateo** | Armar Mensaje | `function` | Formatea el *Top 5* en un **mensaje de texto estructurado** (Markdown) para su envío a través del Webhook. |
| **postWebhook** | Enviar POST | `httpRequest` | Envía el mensaje de texto formateado a una URL Webhook predefinida (`https://tu-endpoint.com/hook`). |
| **Generación de JSON** | *No visible en el flujo* | *Custom* | **(Nota)** Para el dashboard web, debes añadir un paso adicional no especificado aquí para guardar la lista completa de artículos recientes en un archivo llamado `news.json`. |

### 🛠️ Código de la Lógica (Function Nodes)

#### 📝 **`filtro24h` (Filtrar últimas 24h)**
```javascript
const now = new Date();
const limit = now.getTime() - (24 * 60 * 60 * 1000); // 24 horas en milisegundos

return items.filter(item => {
  const d = new Date(item.json.pubDate || item.json.isoDate || 0).getTime();
  return d >= limit;
});
````

#### 📝 **`sort` (Ordenar por fecha)**

```javascript
return items.sort((a, b) => {
  const da = new Date(a.json.pubDate || a.json.isoDate).getTime();
  const db = new Date(b.json.pubDate || b.json.isoDate).getTime();
  return db - da; // Orden descendente (más nuevo primero)
});
```

#### 📝 **`top5` (Top 5)**

```javascript
return items.slice(0, 5); // Toma solo los 5 primeros elementos
```

#### 📝 **`formateo` (Armar Mensaje para Webhook)**

```javascript
let text = '*Nuevas Tecnologías Detectadas (últimas 24h)*\n\n';
items.forEach((item, i) => {
  text += `*${i+1}.* ${item.json.title}\n${item.json.link}\n\n`;
});
return [{ json: { message: text } }];
```

-----

## 2\. Interfaz Web (Dashboard)

El archivo `index.html` proporciona un dashboard limpio y moderno para visualizar todas las noticias recopiladas desde un archivo `news.json` que debe ser generado por un paso adicional en el flujo de n8n.

### ✨ Tecnologías Utilizadas

  * **HTML5**
  * **Tailwind CSS**: Para un diseño responsivo y fácil de mantener.
  * **Feather Icons**: Para iconos sencillos y estéticos.
  * **JavaScript (Vanilla)**: Para la carga dinámica y el renderizado de noticias.

### 🎨 Diseño y Estilo

El dashboard utiliza un tema oscuro con acentos en cian (`var(--accent)` y `var(--highlight)`) para ofrecer una experiencia visual agradable.

  * **Clase `.card`**: Define el estilo de cada noticia, con un borde izquierdo en el color de acento para una mejor separación visual.
  * **`loadNews()`**: La función principal de JavaScript que:
    1.  Carga el archivo **`news.json`**.
    2.  Itera sobre los datos y crea un elemento `div.card` para cada noticia.
    3.  Muestra el **título**, el **enlace**, la **fecha de publicación** formateada localmente, y un **fragmento del contenido**.
    4.  Utiliza la función `truncate` para asegurar que el resumen de la noticia no sea demasiado largo.

### 🔗 Requisito para el Frontend

Para que este dashboard funcione, el flujo de n8n debe generar y actualizar un archivo llamado **`news.json`** en el mismo directorio donde se encuentra el `index.html`. Este JSON debe contener la lista de todos los artículos filtrados de las últimas 24 horas.

**Estructura esperada del `news.json` (ejemplo):**

```json
[
  {
    "title": "Título de la Noticia 1",
    "link": "https://...",
    "isoDate": "2025-12-05T18:00:00.000Z",
    "contentSnippet": "Primeras palabras del contenido...",
    // ... más campos
  },
  // ... más noticias
]
```

## 🛠️ Cómo Empezar

1.  **Configurar n8n**: Importa el flujo JSON a tu instancia de n8n.
2.  **Actualizar Webhook**: En el nodo **"Enviar POST"**, cambia la URL `https://tu-endpoint.com/hook` por el endpoint real de tu plataforma de comunicación.
3.  **Configurar Archivo JSON**: Asegúrate de que tu flujo de n8n incluya un paso (como un nodo `Write File` o `S3 Upload` si estás en la nube) para generar el archivo **`news.json`** con las noticias filtradas.
4.  **Desplegar Dashboard**: Sube el archivo `index.html` y el `news.json` generado al servidor web de tu elección (ej. GitHub Pages, Vercel, o un simple servidor local).

-----

```
```
