# 🥦 Mi Despensa

Control de stock de alimentos en casa, diseñado para Argentina. Una sola página HTML, sin instalación, sin cuenta, sin internet obligatorio — todo corre en el navegador y se guarda en el dispositivo.

---

## Tabla de contenidos

- [Demo rápida](#demo-rápida)
- [Características](#características)
- [Cómo usar](#cómo-usar)
  - [Stock](#stock)
  - [Escanear](#escanear)
  - [Factura / Ticket](#factura--ticket)
  - [Lista de compras](#lista-de-compras)
- [Pipeline de búsqueda de productos](#pipeline-de-búsqueda-de-productos)
- [Exportar e importar](#exportar-e-importar)
- [Formato JSON](#formato-json)
- [Almacenamiento y privacidad](#almacenamiento-y-privacidad)
- [Tecnologías usadas](#tecnologías-usadas)
- [Instalación](#instalación)
- [Compatibilidad](#compatibilidad)
- [Estructura del código](#estructura-del-código)
- [Contribuir](#contribuir)

---

## Demo rápida

```
Abrí mi-despensa.html en cualquier navegador → listo.
```

No requiere servidor, build, ni dependencias adicionales. La única librería externa que se carga es `html5-qrcode` desde un CDN, para el escáner de códigos de barras. EAN (European Article Number)

---

## Características

### Gestión de stock

- **Tarjetas de producto** con nombre, marca, categoría, cantidad actual, stock mínimo y fecha de vencimiento
- **Alertas visuales** automáticas: amarillo cuando el stock cae al mínimo, rojo cuando llega a cero
- **Alertas de vencimiento**: naranja para productos que vencen en menos de 14 días, rojo para vencidos
- **Filtros rápidos**: Todos · Stock bajo · Sin stock · Por vencer
- **Buscador** por nombre o marca en tiempo real
- **Botones +/−** para ajustar cantidad en un toque
- **Botón editar ✏️** en cada tarjeta: abre un modal con todos los datos pre-cargados para modificar cualquier campo
- **Botón eliminar 🗑** con confirmación
- **Toggle de imágenes**: activa o desactiva la vista de fotos de productos en las tarjetas

### Estadísticas de cabecera

Tres indicadores siempre visibles en la pantalla de stock:

| Indicador | Descripción |
|-----------|-------------|
| Productos | Total de ítems en la despensa |
| Stock bajo | Cantidad en nivel de alerta (en amarillo) |
| Sin stock  | Cantidad en cero (en rojo) |

### Tema claro / oscuro

Botón 🌙/☀️ en el header. Detecta automáticamente la preferencia del sistema operativo en el primer uso y la guarda en `localStorage`.

---

## Cómo usar

### Stock

La pantalla principal muestra todos los productos de la despensa. Los controles disponibles son:

- **Agregar** (botón verde en el header): abre el modal de alta manual
- **Imágenes**: muestra u oculta las fotos de los productos en las tarjetas
- **Exportar JSON**: descarga el stock completo como archivo `.json`
- **Importar JSON**: carga un archivo `.json` exportado previamente, con opción de fusionar o reemplazar

**Estados de stock:**

```
🟢 Verde  — cantidad > mínimo configurado
🟡 Amarillo — cantidad ≤ mínimo (stock bajo)
🔴 Rojo    — cantidad = 0 (sin stock)
```

**Editar un producto:**

Tocá ✏️ en cualquier tarjeta. El modal se abre pre-cargado con todos los datos actuales. Podés modificar:

- Nombre y marca
- Código de barras
- Cantidad actual y mínimo de alerta
- Unidad de medida (unid · kg · g · l · ml · paq)
- Categoría
- Fecha de vencimiento
- **URL de imagen**: pegá cualquier URL de imagen (go-upc S3, vtexassets de DIA, etc.) para ver la preview en tiempo real dentro del modal
- Precio unitario en ARS

---

### Escanear

Tres modos de captura del código de barras:

1. **Escáner en vivo**: usa la cámara trasera del dispositivo en tiempo real. Al detectar un código lo busca automáticamente. Requiere HTTPS o localhost.
2. **Tomar foto**: abre la cámara nativa del sistema operativo, toma la foto y extrae el código. Funciona siempre, incluso con CORS restrictivo.
3. **Escribir el código**: campo numérico para ingresar manualmente el EAN-13 u otro código.

**Resultado del escaneo:**

- Si el producto **ya está en la despensa**: muestra la foto guardada, el nombre y el stock actual. Un toque suma +1 a la cantidad.
- Si **no está**: ejecuta el [pipeline de búsqueda](#pipeline-de-búsqueda-de-productos) y muestra nombre, imagen, precio y badge de fuente. Un toque abre el modal para confirmar el alta.
- Si **no se encuentra en ninguna fuente**: ofrece agregar manualmente.

Cuando go-upc y %DIA devuelven datos del mismo producto, el resultado muestra ambos badges y fusiona lo mejor de cada fuente (imagen de go-upc S3 + precio real de %DIA).

---

### Factura / Ticket

Importación masiva de productos desde el texto de una factura digital o ticket de supermercado (Carrefour, Coto, Jumbo, %DIA, etc.).

**Paso 1 — Pegar el texto**

Copiá el texto del email de confirmación, del PDF o de la web del supermercado y pegalo en el textarea. El parser acepta el formato estándar de tickets argentinos:

```
CANT    EAN             DESCRIPCIÓN                          PRECIO_UNIT   IMPORTE
2,000   7798024290318   GALLETITAS PALITOS ALMENDRA TRIMAK   ARS 1.009,00  ARS 2.018,00
1,000   7791720038031   PIZZITOS BULNEZ X 120 GRS            ARS 1.590,00  ARS 1.590,00
0,500   2509084000008   QUESO GOUDA TREMBLAY TROZADO X KG    ARS 14.990,00 ARS 7.495,00
```

Las líneas de cupones, descuentos, ajustes, envíos y promociones se ignoran automáticamente.

**Paso 2 — Enriquecimiento automático**

Después de interpretar el texto, el sistema consulta en paralelo (de a 2 productos simultáneos):

- **go-upc.com** → nombre oficial, imagen de alta calidad, marca, categoría
- **%DIA Argentina** → precio real actualizado, imagen vtexassets

Una barra de progreso muestra el avance. Los datos del ticket (precio del día de compra) tienen prioridad sobre los de las APIs.

**Paso 3 — Confirmar e importar**

Tabla editable con cinco columnas por producto:

| Columna | Qué es |
|---------|--------|
| Producto | Nombre + miniatura + marca + últimos 4 dígitos del EAN |
| Cant. | Cantidad y unidad (editables) |
| Cat. | Categoría (editable) |
| $ Unit. | Precio unitario ARS (editable) |
| ✓ | Toggle para incluir/excluir del import |

El resumen muestra cuántos son nuevos, cuántos actualizan un existente y el total del ticket.

Al importar:
- Productos **nuevos** se crean con badge `📄 Factura`
- Productos **ya existentes** (buscados por EAN o nombre) suman su cantidad al stock actual y actualizan el precio

---

### Lista de compras

Se genera automáticamente con todos los productos que tienen `cantidad ≤ mínimo`. Se ordena por urgencia: sin stock primero, stock bajo después.

Cada ítem muestra:
- Miniatura del producto (si tiene imagen guardada)
- Nombre y marca
- Cantidad sugerida a comprar
- Último precio conocido en ARS
- Badge de urgencia (Sin stock / Bajo)

**Funciones:**

- **Tilde ✓**: marcar un ítem como "ya lo tengo en el carrito"
- **Limpiar ✓**: desmarcar todos los tildados
- **Exportar** (botón): descarga la lista en formato `.csv` con precio unitario, cantidad sugerida y total estimado por ítem

El badge numérico rojo en el ícono del carrito muestra cuántos productos necesitan reposición.

---

## Pipeline de búsqueda de productos

Cuando se escanea un código de barras, el sistema sigue esta estrategia:

```
1. go-upc.com          ← fuente principal (mejor cobertura EAN argentinos)
      ↓ si encuentra
2. %DIA Argentina      ← consulta en paralelo, solo para capturar precio real y descuentos
      ↓ si go-upc no encontró nada
3. Open Food Facts     ← fallback internacional
      ↓ si tampoco
4. Precios Claros AR   ← API oficial del gobierno argentino
      ↓ si tampoco
5. %DIA Argentina      ← último intento, como fuente completa
      ↓ si tampoco
→ Alta manual
```

**Datos que aporta cada fuente:**

| Fuente | Nombre | Imagen | Marca | Categoría | Precio |
|--------|--------|--------|-------|-----------|--------|
| go-upc.com | ✅ en español | ✅ S3 alta calidad | ✅ | ✅ en inglés | ❌ |
| %DIA Argentina | ✅ | ✅ vtexassets | ✅ | ✅ en español | ✅ precio actual + descuento |
| Open Food Facts | ✅ | ✅ | ✅ | ✅ | ❌ |
| Precios Claros AR | ✅ | ⚠️ variable | ✅ | ❌ | ✅ precio regulado |

**Fusión de fuentes:** cuando go-upc y %DIA tienen datos del mismo producto, el resultado combina la imagen S3 de go-upc (si no hay vtexassets) con el precio real de %DIA. Se muestran dos badges de fuente simultáneamente.

**Acceso CORS:** go-upc y %DIA se acceden mediante proxy CORS (`allorigins.win` con `corsproxy.io` como respaldo). Ambas funciones tienen timeout de 8 segundos y prueban el segundo proxy si el primero falla.

---

## Exportar e importar

### Exportar stock (JSON)

Botón **Exportar JSON** en la pantalla de Stock. Genera:

```
despensa-stock-YYYY-MM-DD.json
```

Incluye absolutamente todos los datos de cada producto, incluyendo la URL de imagen.

### Exportar lista de compras (CSV)

Botón **Exportar** en la pantalla de Compras. Genera:

```
despensa-compras-YYYY-MM-DD.csv
```

Compatible con Excel, Google Sheets y LibreOffice Calc. Incluye precio unitario y total estimado.

### Importar stock (JSON)

Botón **Importar JSON** en la pantalla de Stock. Al seleccionar un archivo `.json` exportado previamente, el sistema pregunta el modo:

- **Fusionar (OK)**: compara por `id`, `barcode` o nombre. Actualiza los existentes y agrega los nuevos. Útil para sincronizar entre dispositivos.
- **Reemplazar (Cancelar)**: borra todo el stock actual y carga el del archivo.

El importador acepta tanto el formato propio de Mi Despensa como variantes con campos en español (`nombre`, `cantidad`, `unidad`, `categoria`, `precio`, `ean`).

---

## Formato JSON

El archivo exportado tiene esta estructura:

```json
{
  "version": 1,
  "exported": "2026-03-21T14:30:00.000Z",
  "products": [
    {
      "id": 1711027800000,
      "name": "Mayonesa Liviana Hellmann's",
      "brand": "Hellmann's",
      "barcode": "7794000007109",
      "qty": 2,
      "min": 1,
      "unit": "unid",
      "cat": "Almacén",
      "exp": "2026-12-01",
      "src": "goupc",
      "image": "https://go-upc.s3.amazonaws.com/images/213729603.png",
      "price": 2800,
      "checked": false
    }
  ]
}
```

**Campos de cada producto:**

| Campo | Tipo | Descripción |
|-------|------|-------------|
| `id` | número | Timestamp de creación (identificador único) |
| `name` | string | Nombre del producto |
| `brand` | string | Marca |
| `barcode` | string | Código EAN-13 u otro código de barras |
| `qty` | número | Cantidad actual en stock |
| `min` | número | Mínimo para disparar alerta |
| `unit` | string | `unid` · `kg` · `g` · `l` · `ml` · `paq` |
| `cat` | string | Categoría (ver lista abajo) |
| `exp` | string | Fecha de vencimiento en formato `YYYY-MM-DD` (vacío si no aplica) |
| `src` | string | Fuente de origen (ver lista abajo) |
| `image` | string | URL completa de la imagen del producto |
| `price` | número | Precio unitario en ARS (0 si no se conoce) |
| `checked` | boolean | Si está marcado en la lista de compras |

**Categorías válidas:**

`Almacén` · `Lácteos` · `Carnes` · `Verduras` · `Frutas` · `Bebidas` · `Limpieza` · `Higiene` · `Otro`

**Fuentes válidas (`src`):**

| Valor | Descripción |
|-------|-------------|
| `manual` | Cargado a mano |
| `goupc` | go-upc.com |
| `dia` | %DIA Argentina |
| `openfoodfacts` | Open Food Facts |
| `preciosclaros` | Precios Claros AR (gobierno) |
| `factura` | Importado desde ticket o factura |

---

## Almacenamiento y privacidad

- **Todo se guarda en `localStorage`** del navegador bajo la clave `despensa_v7`. No hay servidor, no hay base de datos remota, no hay cuenta de usuario.
- **No se envía ningún dato personal** a ningún servidor. Las consultas a las APIs externas (go-upc, %DIA, Open Food Facts, Precios Claros) solo incluyen el código de barras del producto.
- **Los proxies CORS** (`allorigins.win`, `corsproxy.io`) reciben la URL del producto a consultar pero no datos del usuario.
- **El tema y la preferencia de imágenes** también se guardan en `localStorage`.
- Los datos persisten hasta que el usuario borra el historial del navegador o usa la función de importar en modo "Reemplazar".

Para **respaldar los datos** se recomienda usar el botón "Exportar JSON" periódicamente y guardar el archivo en un lugar seguro (nube, email, etc.).

---

## Tecnologías usadas

| Tecnología | Uso |
|------------|-----|
| HTML5 + CSS3 + JavaScript vanilla | Toda la aplicación |
| `html5-qrcode` v2.3.8 | Escáner de códigos de barras (cámara + foto) |
| `localStorage` | Persistencia de datos |
| `DOMParser` | Parseo de HTML de go-upc y %DIA |
| `allorigins.win` / `corsproxy.io` | Proxy CORS para acceder a go-upc y %DIA |
| Open Food Facts API v2 | Datos de productos (internacional) |
| Precios Claros API | Datos de productos (Argentina, gobierno) |
| go-upc.com | Identificación de EAN + imágenes |
| %DIA Argentina (diaonline.supermercadosdia.com.ar) | Precios reales + imágenes vtexassets |
| CSS custom properties | Sistema de temas claro/oscuro |
| `safe-area-inset` | Soporte de notch en iPhone |

**Sin frameworks, sin bundlers, sin Node.js.** Un solo archivo `.html` de ~1.750 líneas.

---

## Instalación

### Opción A — Uso directo (recomendada)

```bash
# Descargar el archivo
curl -O https://...../mi-despensa.html

# Abrir en el navegador
open mi-despensa.html          # macOS
xdg-open mi-despensa.html     # Linux
start mi-despensa.html        # Windows
```

O simplemente hacer doble clic en el archivo descargado.

> **Nota:** El escáner en vivo con cámara requiere HTTPS o `localhost`. Para uso en dispositivo móvil con cámara en vivo, servir el archivo desde un servidor local o HTTPS.

### Opción B — Servidor local

```bash
# Python 3
python3 -m http.server 8080

# Node.js (npx)
npx serve .

# PHP
php -S localhost:8080
```

Luego abrir `http://localhost:8080/mi-despensa.html`.

### Opción C — PWA / Pantalla de inicio (móvil)

En iOS Safari o Android Chrome: abrir el archivo, tocar "Compartir" → "Agregar a pantalla de inicio". La app queda instalada como ícono y se comporta como una app nativa con pantalla completa.

---

## Compatibilidad

| Plataforma | Soporte |
|------------|---------|
| Chrome / Edge (desktop) | ✅ Completo |
| Safari (macOS) | ✅ Completo |
| Firefox (desktop) | ✅ Completo |
| Chrome (Android) | ✅ Completo, incluye cámara |
| Safari (iOS 15+) | ✅ Completo, incluye cámara |
| Safari (iOS < 15) | ⚠️ Sin escáner en vivo, foto funciona |
| Opera / Brave | ✅ Completo |

**Requisitos mínimos:**
- Navegador con soporte de `localStorage`, `fetch`, `DOMParser` y CSS custom properties
- Para escáner en vivo: HTTPS o localhost + permiso de cámara
- Para consultas a APIs externas: conexión a internet (el stock funciona offline)

---

## Estructura del código

El archivo `mi-despensa.html` está organizado en secciones claramente delimitadas:

```
mi-despensa.html
├── <head>
│   └── CDN: html5-qrcode
├── <style>
│   ├── Variables CSS (tema claro/oscuro)
│   ├── Reset y base
│   ├── Layout: header, páginas, nav
│   ├── Componentes: stats, chips, search, cards
│   ├── Pills y badges de fuente
│   ├── Estilos del escáner (html5-qrcode overrides)
│   ├── Botones y toolbar
│   ├── API result boxes
│   ├── Lista de compras
│   ├── Modal (agregar/editar)
│   ├── Página de factura
│   └── Toast, responsive
├── <body>
│   ├── Header (título + tema + agregar)
│   ├── .pages
│   │   ├── #page-stock     (stock + filtros + toolbar)
│   │   ├── #page-scan      (escáner + búsqueda manual)
│   │   ├── #page-factura   (import ticket/factura)
│   │   └── #page-compras   (lista de compras)
│   ├── Nav inferior (4 pestañas)
│   ├── Modal overlay (agregar/editar producto)
│   └── Toast notification
└── <script>
    ├── Datos y localStorage
    ├── Tema claro/oscuro
    ├── Helpers (statusOf, expStatusOf, fmtARS)
    ├── Render: renderStats, renderStock, renderShop
    ├── Escáner (html5-qrcode, scanFromPhoto)
    ├── APIs
    │   ├── fetchDIA          (proxy + regex imagen/precio)
    │   ├── fetchGoUPC        (proxy dual + 3 estrategias imagen)
    │   ├── fetchOpenFoodFacts
    │   └── fetchPreciosClaros
    ├── doLookup (pipeline de búsqueda con fusión)
    ├── Modal (openAdd, openEdit, openFromAPI, saveOrUpdateProduct)
    ├── Navegación (goPage)
    ├── Parser de tickets (parseTicketText)
    ├── Enriquecimiento async (enrichItemsFromGoUPC)
    ├── Render tabla de ítems (renderRecognizedItems)
    ├── importItems
    ├── Exportar/importar (exportStockJSON, importStockJSON, exportShopCSV)
    └── Init
```

---

## Contribuir

El proyecto es un único archivo HTML autocontenido. Para proponer mejoras:

1. Hacé un fork o descargá el archivo
2. Editá directamente el `.html` con cualquier editor de texto
3. Probá abriendo el archivo en el navegador
4. Para cambios en el escáner, servilo desde localhost (requiere HTTPS para cámara en vivo)

**Áreas de mejora posibles:**

- Soporte para múltiples despensas / ubicaciones
- Sincronización entre dispositivos vía archivo JSON en la nube
- Notificaciones push para vencimientos próximos
- Integración con más supermercados argentinos
- Historial de precios por producto
- Modo lista de compras con categorías agrupadas
- Exportación a Google Sheets

---

## Licencia

MIT — Libre para usar, modificar y distribuir.

---

*Hecho con 🥦 para cocinas argentinas*
