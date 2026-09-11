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

**Abrir el archivo con doble clic (`file://`) no sirve para revisar datos.** Los 14 fetches
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

## 📄 Lectura del Sheet — trampas que ya costaron un rato

- **`get(etapa)` hace match exacto** tras `trim` + minúsculas, y **`find` devuelve la primera
  coincidencia**. Dos filas con la misma etapa → gana la primera, en silencio.
- **Si una pestaña no existe, gviz NO devuelve 404**: devuelve la primera hoja del documento.
  Por eso cada módulo tiene su **guarda de esquema** (`dpValido`, `natValido`, `refValido`,
  `venValido`, `dppValido`) en vez de comprobar `.length`.
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
