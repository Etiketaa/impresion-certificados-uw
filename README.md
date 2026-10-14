# Impresión de Certificados Municipales — Ultrawhite

Herramienta web para imprimir los **certificados municipales de control de plagas**
directamente desde el planillero mensual de Excel, sobre el membrete ya impreso.

Pensada como alternativa ligera: un **link** que se abre en el navegador del cliente,
se arrastra el Excel, se elige el mes y se imprime. **Sin instalar nada.**

---

## Qué hace

1. **Arrastrás el Excel** (`MUNICIPALIDAD CERTIFICADOS 2025.xls`) al recuadro.
2. **Elegís la hoja del mes** (se autoselecciona el mes en curso si existe).
3. **Revisás los datos** en la tabla y ajustás el rango de filas a imprimir.
4. **Vista previa** con 2 certificados por hoja A4, posicionados exactamente sobre
   las líneas del membrete.
5. **Imprimís** desde el navegador (Ctrl+P o botón). Elegir *Sin márgenes* /
   *Tamaño real (100%)* para que las posiciones calcen con el papel.

---

## Detalles técnicos

- **100% standalone.** Un solo archivo `index.html`. La librería de lectura de
  Excel (**SheetJS**) va incrustada, así que funciona **sin conexión a internet**
  una vez abierto.
- **Posicionamiento en mm absolutos.** Cada dato se coloca en coordenadas exactas
  medidas sobre el escaneo del membrete (`GEOMETRIA.md` de la plantilla original).
  El segundo certificado de la hoja se deriva sumando **135.7 mm** en Y.
- **Detección de columnas por nombre** (tolerante a acentos, mayúsculas y orden),
  con fallback posicional si la hoja no tiene fila de encabezado.
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
- Gráficos de fondo: **activado** (si se quisiera imprimir el marco; por defecto
  solo salen los datos)

---

## Estructura

```
impresion-certificados-uw/
├── index.html      # La herramienta (standalone, SheetJS incrustado)
├── vercel.json     # Config mínima de deploy
├── README.md
└── AGENTS.md
```
