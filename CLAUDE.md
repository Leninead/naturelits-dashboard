# CLAUDE.md — Naturelits Dashboard

Panel cliente de Naturelits. **Un solo archivo: `index.html`** — HTML + CSS + un único
`<script>` inline, sin build, sin dependencias, sin framework. Se sirve por GitHub Pages.

- Repo: `Leninead/naturelits-dashboard` · branch `main` · local `C:\proyectos\naturelits-dashboard`
- Datos en vivo: Google Sheet `1gsanYhxoCEWS2k8FgiF6DbVFC6Eyne9bXqK5U1xegW4`, vía gviz CSV
- El repo hermano con la pipeline que genera esos datos es `naturelits-ppc` (worktrees
  `naturelits-ppc-reporting`, `-weekly`); su `CLAUDE.md` cubre el cálculo, este cubre el panel

---

## 🔬 Revisión local — SIEMPRE por http, nunca por `file://`

```powershell
py -3.12 -m http.server 8000
# → http://localhost:8000/index.html
```

**Abrir el archivo con doble clic (`file://`) no sirve para revisar datos.** Los 15 fetches
al Sheet fallan por CORS desde un origen `file://`, el panel cae al snapshot local y el badge
de estado queda en *"Datos locales · snapshot"*. Lo que se ve entonces no es lo que verá
Roger.

Recargar **con DevTools abierto y "Disable cache" tildado**: el HTML es un único archivo
grande y el navegador lo cachea con ganas.

---

## 🔻 Embudo (`renderEmbudo`) — invariantes

Dos recorridos en el mismo tab: **publicitario** arriba y **negocio** abajo, con el mismo
juego de componentes.

| Componente | Qué dibuja |
|---|---|
| `barra(n)` | nivel de pirámide: valor grande, "antes X", delta. Ancho por clase CSS |
| `barraPend(lab,fase,ancho,ico,dato)` | **fila horizontal**: ícono · label · badge |
| `paso(a,b,alerta)` | tasa de paso "X de cada 1.000" entre dos niveles |

### Vistas de ficha — fila horizontal fija, nunca barra

Sólo en el embudo de **negocio**, entre *Visitas totales* y *Añaden al carrito*. Va **siempre**
como `barraPend` al **100 %**, con dato y sin dato.

- Fuente: la fila **`vistas de ficha`** de la hoja `Funnel` — el match es contra la **columna
  A exacta** tras `trim()` + minúsculas.
- Con dato: badge **`N vistas · X,X por visita`** (vistas / sesiones, un decimal, es-ES).
  Sin sesiones o con sesiones en 0, sólo `N vistas`.
- Sin la fila o con la celda vacía: badge **`Pendiente`**.

**Por qué nunca barra:** su valor **supera** al de Visitas totales —una visita ve varias
fichas—, así que dibujarlo como escalón más angosto con un número mayor narraba un
estrechamiento que no ocurre. Es también la razón de que el ratio sea "por visita" y no
"de cada 1.000".

### Loyalty — `barraPend` al 100 % en los dos embudos

A 20 % el label y el badge no entraban en una línea y envolvían, y el bloque se leía como una
card en vez de una fila. Al 100 % entra en una línea.

### Otras reglas del embudo

- El **cuello** (`Visitas totales → Pedidos negocio`, y su equivalente en el publicitario) se
  calcula **directo**, saltando las subfases. Es el salto que marca dónde se pierde la venta.
- Las subfases sin fuente **no se dibujan con 0**: llevan badge. No tener el dato y tenerlo en
  cero son cosas distintas.
- Las filas de fidelización conservan el trazo punteado **a propósito** aunque traigan cifra:
  su dato es de otra ventana (jun-ago) y otra fuente que el resto del recorrido.

---

## 📅 MoM (`renderMoM`) — reporte mensual julio vs agosto

Tab **"MoM"** del grupo PPC, justo después de *Comparativa*. Cuatro bloques verticales, cada
uno con **su propia guarda**: si una hoja falla, ese bloque cae a empty-state y los otros siguen.

| Bloque | Fuente | Qué dibuja |
|---|---|---|
| Encuadre | texto fijo en el HTML | `.famv-banner` con la lectura del mes |
| Indicadores generales | `MoM_KPIs` (`val(5)`) | `compTableHTML` con las 2 últimas filas por `periodMonthIdx` — la misma tabla que *Comparativa → Mes* |
| Por modelo | `MoM_Modelos` (`val(14)`, guarda `momModValido`) | tabla horizontal 7 modelos × 2 meses + Δ ROAS y Δ Ventas tot |
| Plazo de entrega día × referencia | `Delivery_Promise` (`val(9)`, ya validada por `dpValido`) | 20 días × 6 referencias, celda `DP_min-DP_max` |

Cada bloque lleva un `<p class="mom-intro">` de análisis entre el título y la tabla.

### Por modelo

- Filas ordenadas por **Ventas_Totales de agosto**, desc. Cruza la fila de cada modelo para
  `MOM_MES_PREV` / `MOM_MES_CUR` (`"2026-07"` / `"2026-08"`).
- **Columna Modelo sticky** (`.wt.mom-mod td:first-child`) con fondo sólido `var(--blanco)`.
  La celda de las filas atenuadas **no usa `opacity`**: en una celda sticky eso vuelve
  translúcido el fondo y se ve lo que scrollea debajo. Se atenúa por `color`.
- **Serene Hybrid** = hero (`MOM_HERO`, fondo `--verde-bg`). **Xstar y Palace** = cola
  (`MOM_COLA`, atenuados). Serene Hybrid va con su nombre acá, a diferencia del tab Entrega,
  donde `DP_NOMBRE` lo muestra como "Serene".
- **Vacío ≠ 0**: `Pct_Ads` vacío (Palace ago, ventas 0) → `"—"`, chequeado **antes** de
  formatear. ROAS 0 con inversión es un 0 real → `0,00x`.
- ROAS con **dos decimales y coma** (`momX`), no `fmtVal("x")`, que usa `toFixed(1)` con punto.
- El ROAS por modelo es **solo publicidad de producto**; el del encuadre y el de Indicadores
  generales incluye todos los formatos. La nota al pie lo aclara — no quitarla.

### Plazo de entrega

- Lee **`DP_min` / `DP_max` numéricos, nunca `Entrega_texto`**: cambió de formato el 06-sep
  (`"2-3 septiembre"` → `"22-23 de septiembre"`).
- Columnas en el orden de `MOM_DP_REFS`; una referencia nueva en la hoja se agrega al final.
- **Jueves y viernes en ámbar** (`.mom-jv`), día calculado con `dpDiaSem` desde la fecha ISO.
- Ventana **congelada** `MOM_DP_DESDE` → `MOM_DP_HASTA` (25-ago → 13-sep): la hoja sigue
  sumando días, el informe no.

### Pasar al mes siguiente — nada se deriva solo

`MOM_MES_PREV` / `MOM_MES_CUR`, la ventana `MOM_DP_*`, el título *"Julio vs Agosto 2026"*,
los cabezales `jul` / `ago` de la tabla y **los 4 textos de análisis** están escritos a mano.
Cambiar de mes es tocar todo eso.

### Hoja `MoM_Modelos` — carga manual

- **No hay script de carga al Sheet.** `scripts/mom_modelos.py` (repo reporting) genera
  `datos/procesado/mom_modelos_<meses>.csv` con punto decimal y filas TOTAL / Sin clasificar;
  el TSV que se pega sale de ese CSV **sin TOTAL ni Sin clasificar** y con **coma decimal**.
- 13 columnas: `Mes | Modelo | Inv_SP | Ventas_Ads | ROAS_SP | Ventas_Totales | Pct_Ads |
  Sesiones | Unidades | CVR | BuyBox | Share_Inv | Share_Ventas`.
- **`Mes` como TEXTO** (`"2026-07"`): Sheets lo convierte en fecha al pegar. Formatear la
  columna A como *Texto sin formato* **y reescribir** los valores — aplicar el formato no
  convierte fechas ya existentes. Verificar por gviz JSON que venga `type: string`.
- **Ratios como fracción** (`0,0148`), igual que `MoM_KPIs`.
- **Serene Hybrid** sale de un override por child en `guardian/data/asin_map.yaml` (repo
  reporting): `B0CNVYKZL1` lleva `modelo: Serene Hybrid` con `familia: Serene` intacta, para
  no tocar break-even ni umbrales del Guardián.

### Textos — Modo B

Client-safe: sin siglas internas, sin nombres de herramientas, sin nombres de campaña, sin
causas que el dato no sostenga. Antes de commitear un texto nuevo, buscar las vetadas en el
panel servido.

### Pendientes (13-sep)

1. ✅ *(24-sep: el banner ahora se calcula desde el cuadro, ver abajo)* **Banner del tab Entrega desalineado**: sigue diciendo que el patrón jueves/viernes no se
   puede afirmar, y el tab MoM ya lo afirma (3 semanas: jue 12,3 · vie 12,7 días contra
   10,3–11,7 el resto).
2. **BuyBox de cuenta en `MoM_KPIs` J5/J6 cargado a mano** (97,4 % / 98,1 %): el by-date no
   trae la columna. Al cargar un mes nuevo hay que completarlo a mano o queda en 0 y el panel
   muestra "0%".
3. **SB/SD del `Ad_Sales` de `MoM_KPIs` no reproducibles**: salen de campaign reports mensuales
   no congelados. Ver `docs/deuda-reporting.md` en el repo reporting.

---

## 🔧 Fixes 24-sep — derivar de los datos, no fijar texto

Commits `305a77e` → `eac0a7f` en `main`. Regla común: **todo texto con cifra o fecha se
calcula de la hoja**; el texto fijo del HTML queda solo como respaldo.

- **Nature:** el subtítulo sale de `Grupo`/`Start_date`/`Dias_launch` (inicio por grupo y
  fin = inicio + días − 1). Con `Ventas_EUR = 0`, ACoS y ROAS se muestran "—", no "0,0%" / "0".
  Nota de atribución: 7 días en anuncios de producto, 14 en marca y display.
- **Referencias:** lee `Fecha_push` y `DP_inicio` (I:J, carga manual) por nombre, fuera de
  `REF_COLS`. Nota al pie: base 3-9 sep · acumulado por semanas cerradas desde el 9-sep ·
  Δ% por venta media diaria · sin Δ en refes que entraron al impulso después del 9-sep.
  Fuera la nota interna "⚠ Las 3 referencias rojas…".
- **Entrega:** el banner se calcula del cuadro: días medidos, fecha de inicio y plazos del
  último día (rango más repetido + referencias fuera de él con la diferencia en días).
- **Negocio:** el contenedor `#fam` ("Próximamente…") se oculta; sin dato no se muestra.
- **Modelos:** el peso de Serene sale del cuadro (ventas Serene / TOTAL, semana actual).
  Nota al pie: si el TOTAL por producto difiere más de 1 € de `Ventas_Totales` de
  Resultados, se declara por semana.
- **Resultados y Comparativa:** ROAS con coma ("2,3x") en la tarjeta; % con 1 decimal fijo
  en `fmtVal("pct")` (`pctES(v, minDec)`); un Δ que redondea a 0,0 sale "0,0pp" / "0,0%",
  sin signo y en gris.
- **Evolutivo:** filas sin dato en las 3 columnas se ocultan; sin ningún dato del año
  anterior se ocultan "Año ant." y "Δ YoY" (y el subtítulo lo dice). Vuelven solas al
  cargar `YoY_Negocio`.
- **Comparativa abre en Semana (WoW)** (`draw("wow")` + clase `active` estática en ese botón).

---

## 📄 Lectura del Sheet — trampas que ya costaron un rato

- **`get(etapa)` hace match exacto** tras `trim` + minúsculas, y **`find` devuelve la primera
  coincidencia**. Dos filas con la misma etapa → gana la primera, en silencio.
- **Si una pestaña no existe, gviz NO devuelve 404**: devuelve la primera hoja del documento.
  Por eso cada módulo tiene su **guarda de esquema** (`dpValido`, `natValido`, `refValido`,
  `venValido`, `dppValido`, `momModValido`) en vez de comprobar `.length`.
- **Cuidado con los nombres de las guardas.** `DP_COLS` es de `Delivery_Promise`; la de
  `DP_Push` es `DPP_COLS`. Redeclarar un `const` en este ámbito **no rompe un módulo, rompe el
  `<script>` entero** — es error de parseo y la página queda en blanco.
- El array **`names` de `boot()` es posicional** (`val(i)`): las hojas nuevas van **al final**.
- Las columnas de ventana se **autodetectan por prefijo** (`^Valor_`, `^Ventas_`, `^Inv_SP_`):
  la primera es la previa y la segunda la actual, en el orden en que están en la hoja.
- `REF_COLS` y compañía son **guardas**, no la lista de lo que se dibuja. Agregar ahí una
  columna que el Sheet no tiene manda el módulo entero al empty-state.
- `cleanRows()` descarta toda fila cuya **primera celda** esté vacía o empiece con `ⓘ`.

---

## ✅ Verificación sin navegador

El `<script>` se puede ejecutar con el motor de Node sobre un DOM simulado: se extrae el
bloque inline, se corre en un `vm` con un `document.getElementById` que captura `innerHTML`, y
se invoca el render con filas de prueba. Sirve para comprobar el HTML que produce cada
función sin abrir el panel, y es lo que conviene correr antes de cada commit.

Ojo con dos cosas al escribir esas comprobaciones:
- Los `const` del script **no se enganchan** al contexto del `vm`; hay que reexportarlos.
- **es-ES no agrupa los miles en números de 4 dígitos**: `4507` se imprime `4507`, no `4.507`.
