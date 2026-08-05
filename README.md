# Tablero de Demanda y Fiscalización — Transporte Público Regional

Herramienta de análisis de demanda y evasión para servicios de transporte
público regional, que integra datos de recaudo (validaciones de tarjeta),
contadores automáticos de pasajeros (APC), localización de vehículos (AVL) y
la red programada (GTFS), y los despliega en un tablero web interactivo.

---

## Metodología

### Fuentes de datos y su rol

El análisis combina tres flujos automáticos que se complementan justo donde
cada uno es débil:

- **Recaudo / AFC (validaciones bip):** registra cada **subida pagada**, con
  identificación del medio de pago. No registra bajadas y, por definición, no
  ve al pasajero que no valida.
- **APC (contadores automáticos):** registra **subidas y bajadas** por puerta,
  reflejando la demanda total pague o no. Es la única fuente de bajadas, pero
  tiene sesgo de sensor y suele no cubrir el 100% de la flota.
- **AVL (localización):** aporta la trayectoria del vehículo; se usa para
  georreferenciar las validaciones y los eventos APC (que no traen coordenada
  confiable), y para identificar cada expedición.
- **GTFS (red programada):** aporta la geometría de cada servicio, la secuencia
  y distancia de sus paradas, y la frecuencia programada.

La fusión de AFC + APC + AVL + GTFS es el enfoque estándar en la literatura de
transporte para reconstruir demanda cuando ninguna fuente por sí sola es
completa.

### Zonas de fiscalización

Los eventos se agregan sobre una grilla hexagonal regular. Por cada hexágono se
comparan las **subidas validadas** (recaudo) contra las **bajadas** (APC),
clasificando en tres categorías:

- **Subida** — alta concentración de subidas: fiscalizar en origen.
- **Bajada** — alta concentración de descensos sin subida validada equivalente:
  es donde desciende el usuario de tramo corto; conviene fiscalizar **aguas
  arriba**, antes de la zona de bajada.
- **Mixta** — alta subida y bajada: nodo de recambio de pasajeros, máxima
  prioridad.

Se reporta además el **número de servicio-sentido (NSS)** que transita por cada
zona: una zona con muchos servicios es un nodo de red donde un operativo cubre
varias líneas.

### Perfil de carga a bordo

La carga a bordo se reconstruye como la suma acumulada de subidas menos bajadas
a lo largo de la secuencia de paradas del servicio (ordenada por la distancia
sobre la ruta que provee el GTFS). El cálculo se hace **por expedición** —para
que subidas y bajadas de un mismo viaje se acumulen juntas— y luego se promedia
por servicio, hora de inicio y tipo de día.

Como la cobertura de contadores es parcial, el balance de una expedición no
siempre cierra en cero, lo que produce cargas negativas espurias. Se aplica una
**normalización de cierre** (re-escalar subidas y bajadas al promedio de ambos
totales) que fuerza el cierre del balance preservando la forma de la curva —el
mismo tipo de *balancing algorithm* documentado en la literatura APC. Un
indicador de calidad marca las expediciones cuyo dato original no cerraba, para
trazabilidad.

### Demanda vs oferta

La demanda observada por servicio y hora se contrasta con la frecuencia
programada del GTFS, permitiendo identificar tramos y horarios sub- o
sobre-ofertados para apoyar decisiones de programación.

---

## Vistas del tablero

- **TRX + APC (brecha):** subidas desde recaudo, bajadas desde APC. Resalta las
  zonas de bajada y mixtas para focalizar fiscalización.
- **Solo APC:** subidas y bajadas ambas desde contadores (demanda total).
  Habilita el perfil de carga a bordo.
- **Solo TRX:** demanda validada desde recaudo; caracteriza demanda pagada pero
  no dispone de bajadas ni perfil de carga.

Filtros: servicio, hora de inicio de expedición, tipo de día (laboral / sábado
/ domingo) y categoría de zona. La vista de ruta dibuja el recorrido del
servicio con sus paradas coloreadas por carga a bordo.

---

## Uso

1. Ejecutar el procesador (`Fiscalizacion_TRX_APC_GTFS_v3.py`) como Script Tool
   de ArcGIS Pro, con las capas TRX y APC, el GTFS de la ciudad y la opción de
   detalle por unidad. Genera las capas de análisis en la geodatabase y un
   archivo `agregado_<ciudad>_<mes>.json` para el tablero.
2. Renombrar ese JSON a `agregado.json` y ubicarlo junto a `tablero.html`.
3. Publicar (ver despliegue) o abrir en un navegador mediante un servidor local.

### Despliegue en GitHub Pages

Repositorio público → subir `tablero.html` y `agregado.json` → Settings → Pages
→ Deploy from a branch (`main` / root). La actualización mensual consiste en
reemplazar el `agregado.json`.

---

## Notas de alcance

Los valores son de carácter tendencial: restan por incorporar factores de
ajuste y validación fina de sensores, pero la forma y magnitud relativa son
utilizables para planificación, programación y gestión operacional.

---

## Fuentes y referencias

Metodología general de fusión AFC/APC/AVL/GTFS y buenas prácticas de conteo:

- Furth, P. G., Strathman, J. G., Hemily, B. (2005). *Making Automatic
  Passenger Counts Mainstream.* Transportation Research Record. — Algoritmos de
  balance para cargas negativas y penetración parcial de flota.
  https://journals.sagepub.com/doi/abs/10.1177/0361198105192700124
- TCRP Report 113. *Using Archived AVL-APC Data to Improve Transit Performance
  and Management.* Transportation Research Board.
  https://nap.nationalacademies.org/read/13907/chapter/10
- TCRP Synthesis 77. *Passenger Counting Systems.* Transportation Research
  Board. https://www.trb.org/Publications/Blurbs/160654.aspx
- Trillium (2021). *Automatic Passenger Counting (APC) and Automatic Fare
  Collection (AFC): White Paper.* Oregon DOT. — Uso de GTFS para vincular
  conteos a paradas/rutas y seguimiento de evasión.
  https://www.oregon.gov/odot/RPTD/RPTD%20Document%20Library/APC-AFC-White-Paper-Trillium-2021.pdf
- A Guidebook for Using Automatic Passenger Counter Data for National Transit
  Database Reporting (2010). National Center for Transit Research.

Estimación de matrices origen-destino y carga desde AVL/APC/AFC:

- Optimization Models for Estimating Transit Network Origin-Destination Flows
  with AVL/APC Data. https://arxiv.org/pdf/1911.05777
- Network-wide occupancy inference combining AFC and partial-fleet APC (survey
  de aplicaciones de datos en transporte inteligente).
  https://arxiv.org/pdf/1803.10902

Especificación de datos de red:

- General Transit Feed Specification (GTFS). https://gtfs.org
- TransitWiki — Automated Passenger Counter.
  https://www.transitwiki.org/TransitWiki/index.php/Automated_passenger_counter
