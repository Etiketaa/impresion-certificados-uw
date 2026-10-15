# AGENTS.md — impresion-certificados-uw

Herramienta web standalone para imprimir certificados municipales de control de
plagas (Ultrawhite) desde el planillero mensual de Excel, sobre membrete impreso.

## Qué es

Un **único `index.html`** con toda la lógica. No hay build, no hay dependencias de
npm, no hay backend. La librería **SheetJS** va **incrustada** en el archivo para
que funcione 100% offline una vez abierto.

Deploy estático en **Vercel** (detecta `index.html` en la raíz). Se vende como
opción ligera ("link") junto al sistema de gestión completa.

## Comandos

```bash
# No hay build. Abrir directo:
# doble click en index.html  —  o servirlo:
python3 -m http.server 8080   # http://localhost:8080

# Deploy (requiere vercel CLI autenticado)
vercel          # preview
vercel --proda  # producción
```

## Convenciones clave

- **Cero estilos inline en el HTML markup.** Todo el CSS vive en el `<style>` del
  mismo archivo (regla de Franco). Los posicionamientos de campos de certificado
  sí son inline porque son datos dinámicos generados por JS, no estilos.
- **Posiciones en mm** tomadas de la geometría del membrete escaneado
  (`GEOMETRIA.md` en `/home/franco/Ultrawhite/plantilla-certificados/`).
  - Caja inferior = caja superior + **135.7 mm** en Y.
  - `BASE_ADJ = 0.7 mm`: el dato se apoya sobre la línea impresa.
- **Detección de columnas** por nombre, normalizando acentos/mayúsculas/espacios.
  La columna `N°` (altura del domicilio) se matchea **solo por igualdad exacta**
  (`"N°"` → `"n"`) para no confundirse con cualquier otra columna que contenga
  la letra "n". *No volver a usar substring para esa columna.*
- **El rango de impresión es por N° de certificado (columna C), no por fila.**
  Los N° son correlativos. La selección filtra por el valor de cada fila
  (`certNumOf`), así que respeta huecos y N° fuera de orden, y no hay que contar
  filas. `loadSheet` deja los campos con el mín/máx de certificados de la hoja.
  Si la hoja no tiene N° numéricos (ej. `Esporadicos`), cae a selección por fila.

## Fuente de verdad de los datos

El Excel `MUNICIPALIDAD CERTIFICADOS 2025.xls` tiene una hoja por mes. Columnas
estándar (9): `Fecha del servicio`, `Fecha Municipio`, `N° Certificado`, `Rubro`,
`Cliente`, `Domicilio`, `N°`, `Tipo de Tratamiento`, `Principio Activo`.
Algunas hojas nuevas pueden tener menos columnas o no tener encabezado.

## Trampas conocidas

- El navegador de verificación (browser tool) no puede hacer drag & drop; para
  probar se inyecta el workbook por `browser.evaluate` (fetch al archivo servido
  + `XLSX.read`).
- El archivo pesa ~900 KB por SheetJS incrustado. Es esperado, no es un error.
- Reemplazar SheetJS: descargar `xlsx.full.min.js` e inlinearlo; **no** dejar un
  `<script src=cdn>` porque la herramienta debe andar offline.

## Pendiente

- Verificar impresión real sobre membrete físico (calibración final en papel).
- Confirmar si conviene gating por cantidad de hojas/meses según el plan de venta.
