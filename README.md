# Impresión de Certificados Municipales — Ultrawhite

Herramienta web para imprimir los **certificados municipales de control de plagas**
directamente desde el planillero mensual de Excel, sobre el membrete ya impreso.

Pensada como alternativa ligera: un **link** que se abre en el navegador del cliente,
se arrastra el Excel, se elige el mes y se imprime. **Sin instalar nada.**

---

## Qué hace

1. **Arrastrás el Excel** (`MUNICIPALIDAD CERTIFICADOS 2025.xls`) al recuadro.
2. **Elegís la hoja del mes** (se autoselecciona el mes en curso si existe).
3. **Revisás los datos** en la tabla y elegís el rango **por N° de certificado**
   (`Desde N° cert.` / `Hasta N° cert.`). Los N° son correlativos: no hace falta
   contar filas. Al cargar la hoja los campos se llenan solos con el mínimo y el
   máximo de la columna C.
4. **Vista previa** con 2 certificados por hoja A4, posicionados exactamente sobre
   las líneas del membrete.
5. **Imprimís** desde el navegador (Ctrl+P o botón). Elegir *Sin márgenes* /
   *Tamaño real (100%)* para que las posiciones calcen con el papel.
6. Si la impresora tiene un corrimiento propio (hardware), se corrige con el
   **Ajuste fino X / Y (mm)** del panel, sin tocar el código. Es acotado a ±15 mm
   y queda guardado en el navegador.

---

## Detalles técnicos

- **100% standalone.** Un solo archivo `index.html`. La librería de lectura de
  Excel (**SheetJS**) va incrustada, así que funciona **sin conexión a internet**
  una vez abierto.
- **Posicionamiento en mm absolutos.** Cada dato se coloca en coordenadas exactas
  medidas sobre el escaneo del membrete (`GEOMETRIA.md` de la plantilla original).
  El segundo certificado de la hoja se deriva sumando **135.7 mm** en Y.
- **Alineación por línea base, no por caja.** El CSS posiciona el borde superior
  de cada dato; sobre la regla impresa lo que debe caer es la **línea base**
  (`y_regla − 0.7 mm`, igual que el pipeline Word validado). Por eso se resta el
  ascendente real de la fuente, medido en el DOM en tiempo de ejecución
  (`fieldAscentMM()`; fallback 4.35 mm). Sin esto, el texto cae ~4.4 mm por debajo
  de la regla.
- **Impresión desde el origen.** En `@media print` se neutralizan el padding, el
  flex y el `gap` de `.app-main` y el marco de la vista previa, de modo que la
  hoja A4 arranca en (0,0) y no se desplaza hacia abajo (antes ~13 mm).
- **Detección de columnas por nombre** (tolerante a acentos, mayúsculas y orden),
  con fallback posicional si la hoja no tiene fila de encabezado.
- **Selección por N° de certificado** (columna C). Filtra por el valor real de cada
  fila, respetando huecos y números fuera de orden de la planilla. Si una hoja no
  tiene N° numéricos, cae a selección por fila para no romperse.
- **Formato de salida** idéntico al Word original: `Domicilio + N°`, fechas
  `DD/MM/YYYY`, Tahoma 12pt.

### Campos del certificado

| Campo | Origen en el Excel |
|---|---|
| N° certificado | `N° Certificado` |
| Fecha municipal | `Fecha Municipio` |
| Cliente | `Cliente` |
| Domicilio | `Domicilio` + `N°` (concatenados) |
| Tipo de tratamiento | `Tipo de Tratamiento` |
| Principio activo | `Principio Activo` |
| Fecha del servicio | `Fecha del servicio` |

---

## Uso local

Abrir `index.html` con doble click en cualquier navegador. No requiere servidor.

Para servirlo localmente (opcional):

```bash
python3 -m http.server 8080
# abrir http://localhost:8080
```

---

## Deploy en Vercel

El repo está listo para deploy estático: Vercel detecta `index.html` en la raíz
y no necesita build.

```bash
npm i -g vercel
vercel          # preview
vercel --prod   # producción
```

O desde el dashboard de Vercel: *Import Git Repository* → deploy automático en
cada push a `main`.

---

## Nota de impresión

Imprimir sobre el **membrete preimpreso** (2 certificados por hoja A4). En el
diálogo de impresión del navegador:

- Tamaño papel: **A4**
- Escala: **100% / Tamaño real** (no "ajustar a página")
- Márgenes: **Ninguno**
- **Encabezados y pies de página: desactivados** (para que el navegador no agregue
  fecha, título ni URL)
- Gráficos de fondo: **activado** (si se quisiera imprimir el marco; por defecto
  solo salen los datos)

La hoja se genera a **tamaño exacto A4** (`@page { size: A4; margin: 0 }`), así que
con esa configuración los datos caen 1:1 sobre las líneas del membrete, sin escalar.

> **Calibración física pendiente.** La posición se validó contra el pipeline Word
> ya usado con el cliente (desvío < 0.2 mm) y contra líneas de referencia en mm.
> Falta la prueba final impresa: imprimir **una** hoja de prueba sobre el membrete
> real y confirmar en papel que cada dato apoya sobre su línea. Si hay un
> corrimiento propio de la impresora, corregirlo con el *Ajuste fino X / Y*.

---

## Estructura

```
impresion-certificados-uw/
├── index.html      # La herramienta (standalone, SheetJS incrustado)
├── vercel.json     # Config mínima de deploy
├── README.md
└── AGENTS.md
```
