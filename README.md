# Tablero de Demanda y Fiscalización — Transporte Público Regional

Herramienta de análisis de demanda y evasión para servicios de transporte
público regional, a partir de datos de recaudo (TRX), contadores de pasajeros
(APC) y la red programada (GTFS). Genera zonas de fiscalización, perfiles de
carga por servicio y análisis de demanda vs oferta, y los despliega en un
tablero web interactivo.

## Qué contiene

- **`Fiscalizacion_TRX_APC_GTFS_v3.py`** — procesador (Script Tool de ArcGIS
  Pro). Lee las capas TRX y APC de una GDB más el GTFS de la ciudad, y produce
  las capas de análisis en la misma GDB y un archivo `agregado_<ciudad>_<mes>.json`
  compacto para el tablero.
- **`tablero.html`** — tablero web autocontenido. Lee un archivo `agregado.json`
  y despliega mapa de zonas, perfil de carga, demanda/oferta y tabla filtrable.
  No requiere servidor: se abre en cualquier navegador.

## Uso mensual

1. Correr el procesador en ArcGIS Pro con los 4 parámetros (TRX, APC, GTFS zip,
   detalle por unidad). Genera las capas y el JSON en la carpeta de la GDB.
2. Renombrar el `agregado_<ciudad>_<mes>.json` a `agregado.json` y ponerlo
   junto al `tablero.html`.
3. Abrir `tablero.html` (doble clic) o publicarlo (ver abajo).

---

## Publicar en GitHub Pages (URL pública)

Como los datos son públicos, se puede desplegar en una URL compartible gratis.

### Opción rápida (sin instalar nada, desde el navegador)

1. Crear una cuenta en https://github.com si no tienes.
2. Botón **New repository**. Nombre por ejemplo `fiscalizacion-temuco`.
   Marcar **Public**. Crear.
3. En el repo, botón **Add file → Upload files**. Arrastrar `tablero.html`
   y `agregado.json`. Escribir un mensaje ("primera versión") y **Commit changes**.
4. Ir a **Settings → Pages** (menú lateral). En **Source** elegir
   **Deploy from a branch**, rama `main`, carpeta `/ (root)`. **Save**.
5. Esperar ~1 minuto. Aparece la URL:
   `https://<tu-usuario>.github.io/fiscalizacion-temuco/tablero.html`

Para actualizar cada mes: subir el nuevo `agregado.json` (mismo nombre) con
**Upload files** y listo — la URL se refresca sola.

### Opción con Git (si trabajas desde tu PC)

```bash
git init
git add tablero.html agregado.json
git commit -m "Tablero fiscalizacion Temuco"
git branch -M main
git remote add origin https://github.com/<tu-usuario>/fiscalizacion-temuco.git
git push -u origin main
```

Luego activar Pages en Settings → Pages igual que arriba.

---

## Estructura sugerida del repositorio

```
fiscalizacion-temuco/
├── tablero.html              ← el tablero (no cambia entre meses)
├── agregado.json             ← datos del mes actual (se reemplaza)
├── historico/                ← opcional: guardar meses anteriores
│   ├── agregado_202605.json
│   └── agregado_202606.json
├── procesador/
│   └── Fiscalizacion_TRX_APC_GTFS_v3.py
└── README.md
```

Si quieres un selector de mes en el tablero, se puede extender para que lea
varios JSON de la carpeta `historico/`.

---

## Modos de vista del tablero

- **TRX + APC (brecha):** compara subidas validadas contra bajadas del contador.
  Resalta zonas de bajada alta (fiscalizar aguas arriba) y mixtas.
- **Solo APC:** subidas y bajadas totales, habilita el perfil de carga a bordo.
- **Solo TRX:** demanda validada, sin bajadas ni perfil de carga.

Filtros: servicio, hora de inicio de expedición, tipo de día (L/S/D) y
categoría de zona.

## Notas de datos

- El perfil de carga se calcula por expedición y se normaliza para cerrar el
  balance (corrige cobertura parcial de flota); el campo `CIERRE_OK` marca los
  grupos cuyo dato original no cerraba, para trazabilidad.
- Los valores son tendenciales: faltan factores de ajuste finos, pero la forma
  es utilizable para planificación, programación y gestión operacional.
