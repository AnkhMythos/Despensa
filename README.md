# 🥦 Mi Despensa

App web para controlar el stock de alimentos en casa. Funciona desde el celular sin instalar nada.

## ✨ Funciones

- 📦 **Control de stock** — cantidad actual, mínimo de alerta, fecha de vencimiento
- 📷 **Escáner de código de barras** — cámara en vivo o foto
- 🔍 **Búsqueda automática** en Open Food Facts y Precios Claros AR
- 🛒 **Lista de compras** generada automáticamente cuando el stock baja
- 🌙 **Tema oscuro / claro** con toggle manual
- 📱 **100% responsive** — pensada para celular, tipografía grande

---

## 🚀 Publicar en GitHub Pages (gratuito, 5 minutos)

La cámara en vivo **requiere HTTPS**. GitHub Pages lo provee gratis.

### Paso 1 — Crear repositorio

1. Entrá a [github.com](https://github.com) y creá una cuenta si no tenés
2. Clic en **"New repository"** (botón verde)
3. Nombre: `despensa` (o el que quieras)
4. Dejalo en **Public**
5. Clic en **"Create repository"**

### Paso 2 — Subir el archivo

En la página del repositorio vacío:

1. Clic en **"uploading an existing file"**
2. Arrastrá el archivo `index.html` a la zona de subida
3. Abajo escribí: `Versión inicial`
4. Clic en **"Commit changes"**

### Paso 3 — Activar GitHub Pages

1. Ir a **Settings** (pestaña del repositorio)
2. En el menú izquierdo: **Pages**
3. En "Source" elegir **Deploy from a branch**
4. En "Branch" elegir **main** → carpeta **/ (root)**
5. Clic en **Save**

### Paso 4 — Acceder desde el celular

Después de 1-2 minutos tu app estará en:

```
https://TU_USUARIO.github.io/despensa/
```

Reemplazá `TU_USUARIO` con tu usuario de GitHub.

---

## 📱 Instalar como app en el celular

### Android (Chrome)
1. Abrí la URL en Chrome
2. Tocá los **⋮ tres puntos** → **"Agregar a pantalla de inicio"**
3. La app aparece como ícono en el home

### iPhone (Safari)
1. Abrí la URL en Safari
2. Tocá el ícono de **compartir** (cuadrado con flecha)
3. → **"Agregar a pantalla de inicio"**

---

## 🔍 APIs utilizadas

| API | Descripción | Costo |
|-----|-------------|-------|
| [Open Food Facts](https://world.openfoodfacts.org) | Base de datos mundial de productos con foto, marca y categoría | Gratis, sin API key |
| [Precios Claros AR](https://preciosclaros.gob.ar) | Productos de supermercados argentinos (Gobierno Nacional) | Gratis, sin API key |

Si un producto no aparece en ninguna API, podés cargarlo manualmente. También podés contribuir productos a Open Food Facts desde su app.

---

## 💾 Almacenamiento de datos

Los datos se guardan en el **localStorage** del navegador del celular. No se envían a ningún servidor. Si limpiás el navegador se borran los datos.

---

## 🛠️ Tecnologías

- HTML5 + CSS3 + JavaScript vanilla (sin frameworks)
- [html5-qrcode](https://github.com/mebjas/html5-qrcode) para escaneo de códigos de barras
- Google Fonts (Nunito) — se carga online

---

## 📋 Notas técnicas

- **Cámara en vivo**: requiere HTTPS (GitHub Pages lo provee) o localhost
- **Foto**: funciona siempre, incluso desde archivo local (`file://`)
- **BarcodeDetector nativo**: Chrome Android lo soporta; la librería html5-qrcode es el fallback universal
- Compatibilidad: Chrome Android, Safari iOS 15+, Firefox Android (sin cámara en vivo, usa foto)
