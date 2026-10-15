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
  - **La base (no la caja) va sobre la regla**: `top = (y_regla − 0.7) − ascendente`.
    `fieldAscentMM()` mide el ascendente real en el DOM (fallback 4.35 mm, el mismo
    del `build.py` de la plantilla Word). No volver a poner `top = y − 0.7` a secas:
    el texto queda ~4.4 mm bajo la línea.
- **La impresión arranca en (0,0) de la hoja.** En `@media print` se pone
  `.app-main` en `display:block; padding:0; margin:0; max-width:none` y se limpian
  `.preview-section` / `.preview-body`. Si no, el `padding`+`gap` del contenedor
  meten la hoja ~13 mm abajo (y genera una página de más). `@page { size:A4; margin:0 }`.
- **Ajuste fino (control de calibración).** `offX`/`offY` (±15 mm, paso 0.1)
  desplazan todos los campos; se guardan en `localStorage` (`uw_print_offset_mm`).
  Sirve para corregir el corrimiento propio de la impresora sin tocar el código.
- **Detección de columnas** por nombre, normalizando acentos/mayúsculas/espacios.
  La columna `N°` (altura del domicilio) se matchea **solo por igualdad exacta**
  (`"N°"` → `"n"`) para no confundirse con cualquier otra columna que contenga
  la letra "n". *No volver a usar substring para esa columna.*
- **El rango de impresión es por N° de certificado (columna C), no por fila.**
  *Los N° NO son correlativos* en varias hojas: hay huecos (Junio 25 falta 148,
  163, 164; AGOSTO 26 falta 1193, 1197, 1212, 1265, 1267, 1272) y filas fuera de
  orden (Julio 25: 195, 198, 223, 235; Octubre 25: 418). La selección filtra por
  el valor de cada fila (`certNumOf`) y **ordena ascendente** por N° antes de
  armar las hojas (`generatePreview`), así la impresión sale 1,2,3… sin importar
  el orden de la planilla. **El N° se imprime literal de la columna: nunca se
  renumera ni se rellenan huecos** (si falta el 1193, salen 1192 y 1194). Si el
  rango tiene huecos o repetidos, avisa en la
  vista previa (`faltantesDeRango` / `numerosRepetidos` vía `showStatus`), y ese
  aviso **no se imprime** (`.status-msg` se oculta en `@media print`). Nota:
  `faltantesDeRango` solo corre con `ns.length > 1` para no avisar en rangos de
  un solo N°. `loadSheet` deja los campos con el mín/máx de certificados de la
  hoja. Si la hoja no tiene N° numéricos (ej. `Esporadicos`), cae a selección por
  fila.

## Fuente de verdad de los datos

El Excel `MUNICIPALIDAD CERTIFICADOS 2025.xls` tiene una hoja por mes. Columnas
estándar (9): `Fecha del servicio`, `Fecha Municipio`, `N° Certificado`, `Rubro`,
`Cliente`, `Domicilio`, `N°`, `Tipo de Tratamiento`, `Principio Activo`.
Algunas hojas nuevas pueden tener menos columnas o no tener encabezado.

## Trampas conocidas

- El navegador de verificación (browser tool) no puede hacer drag & drop; para
  probar se inyecta el workbook por `browser.evaluate` (fetch al archivo servido
  + `XLSX.read`).
- **Trampa al inyectar un `<script>` de prueba en `index.html`:** la **primera**
  ocurrencia de `</body>` en el archivo **no** es el cierre del documento, sino el
  string `var Wm="</body></html>"` **dentro del bundle de SheetJS** (línea ~26). Un
  `replace('</body>', …)` a secas mete el `<script>` en medio del bundle y lo
  rompe (síntoma: `SyntaxError: Unexpected identifier 'load'`, y `XLSX` queda
  `undefined`). Anclar la inyección a algo único (`<head>`, o el `</body>` final)
  y verificar con `node --check` que el bundle sigue siendo JS válido.
- El archivo pesa ~900 KB por SheetJS incrustado. Es esperado, no es un error.
- Reemplazar SheetJS: descargar `xlsx.full.min.js` e inlinearlo; **no** dejar un
  `<script src=cdn>` porque la herramienta debe andar offline.

## Pendiente

- Verificar impresión real sobre membrete físico (calibración final en papel).
- Confirmar si conviene gating por cantidad de hojas/meses según el plan de venta.
