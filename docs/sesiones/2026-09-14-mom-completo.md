# HANDOFF — Reporte MoM Julio vs Agosto 2026 · sesión 13-14 sep

## QUÉ SE HIZO (resumen)
Se construyó el tab "MoM" completo en el dashboard live (leninead.github.io/naturelits-dashboard)
y una PPT de presentación para el cliente (Roger). Todo pusheado. Ambos repos al día.

## CONTEXTO DEL PEDIDO
- Roger pidió (2ª meet de las 3 transcripts) un reporte mensual de agosto: "evolutivos de dónde
  estábamos a dónde estamos, imputar bien conversiones, verlo a nivel producto, funnels, comparativas,
  más chicha, so-what". El MoM del dashboard existía como sub-tab de Comparativa (jul vs ago) pero
  era solo totales.
- Después Roger encontró el dashboard "demasiado denso / too much" y pidió una PPT con gráficos de
  "evolución de ventas del mes Y ventas de publi", que "se vea bien todo".

## DATOS OFICIALES (verificados contra el Sheet, defendibles)
Fuente Sheet cliente ID 1gsanYhxoCEWS2k8FgiF6DbVFC6Eyne9bXqK5U1xegW4

### MoM_KPIs (totales mensuales — números oficiales):
- Inversión: jul €3.394 / ago €4.090 (+20,5%) — es SP+SB+SD
- Ventas Ads: jul €23.417 / ago €22.749 (-2,9%) — SP+SB+SD, con efecto halo
- Ventas totales: jul €67.907 / ago €82.499 (+21,5%)
- Órdenes: jul 149 / ago 169 (+13,4%)
- ROAS: jul 6,9x / ago 5,6x (-19,4%)
- ACoS: jul 14,5% / ago 18,0% (+3,5pp)
- Conversión de negocio: jul 2,0% / ago 2,4% (items pedidos/sesiones, del by-date). OJO: la
  conversión de ADS (órdenes/clics) bajó 2,45%→2,10% — son métricas distintas, por eso el texto
  de Indicadores dice 'cómo ese tráfico publicitario convierte' sin contradecir el 2,4%.
- BuyBox: jul 97,4% / ago 98,1% (cargado A MANO en J5/J6, el by-date no lo trae)
- Impresiones: jul 1.349.834 / ago 1.769.792 (+31,1%)
- Clics: jul 6.084 / ago 8.037 (+32,1%)

### MoM_Modelos (por modelo, 14 filas, hoja NUEVA cargada a mano):
Columnas: Mes|Modelo|Inv_SP|Ventas_Ads|ROAS_SP|Ventas_Totales|Pct_Ads|Sesiones|Unidades|CVR|BuyBox|Share_Inv|Share_Ventas
JULIO:  Serene 28.387 (ROAS 4,24) · Serene Hybrid 16.529 (4,76) · Moon 11.631 (12,22) · Orbit 6.554 (16,66) · Nature 2.447 (4,88) · Xstar 1.257 (0) · Palace 1.102 (1,70)
AGOSTO: Serene 36.371 (4,03) · Serene Hybrid 15.386 (3,60) · Moon 17.607 (5,92) · Orbit 10.983 (4,98) · Nature 1.949 (0) · Xstar 447 (0) · Palace 0 (0)
- Serene + Hybrid = 63% del negocio. Serene Hybrid = 30,4% Pct_Ads jul / 23,5% ago (el más dependiente de ads).
- Serene Hybrid es el ÚNICO grande que retrocede (-6,9% ventas, ROAS 4,76→3,60).
- Moon +51,4% ventas / Orbit +67,6%, pero ROAS se desplomó (Moon 12,22→5,92, Orbit 16,66→4,98).
- Serene Hybrid = override "modelo: Serene Hybrid" en asin_map child B0CNVYKZL1 (familia queda Serene, no rompe Guardián).

### Delivery_Promise (DP día a día, 25-ago→13-sep, 6 refes):
Patrón: jueves/viernes 11-12 días vs domingo-martes 9-10 días (en las refes de entrega rápida).
Moon 135 y Nature van 13-17 (rango distinto). Táctica: reforzar inversión dom-lun-mar, aflojar jue-vie.

### Serie semanal (WoW_KPIs, para gráficos de evolución):
Serie limpia (descartada semana duplicada "3-9 jul"): 26jun-2jul→2-8jul→9-15jul→16-22jul→23-29jul→
30jul-5ago→6-12ago→13-19ago→20-26ago→27ago-2sep.
Ventas totales: 15161·12974·15044·14823·20414·22748·25971·16638·16513·15301 (pico €26k a mediados de agosto)
Ventas publi: 4449·3373·5339·3385·7994·6414·6574·4333·4180·3820
Inversión: 694·659·724·734·888·986·848·920·969·890
OJO: suma semanal NO cuadra con MoM_KPIs (+6,7%) por atribución retroactiva (serie preliminar). NO mostrar totales al lado.

## DISCREPANCIA VENTAS ADS (resuelta, importante)
3 números distintos, los 3 correctos, miden scopes distintos:
- MoM_Modelos €12.914 (jul) = SP solo SKU anunciado (para atribución por modelo)
- MoM_KPIs €23.417 = SP+SB+SD completo con efecto halo (el "oficial" de cuenta)
- Consola de ads €20.616 = vista Default que FILTRA campañas (le faltan €283 de inversión)
La consola muestra menos porque oculta campañas; el dashboard tiene el número más completo. NO es error.
DEUDA: los SB/SD de MoM_KPIs salen de 4 reports NO congelados (no reproducibles) → ver docs/deuda-reporting.md.

## DECISIONES CLAVE TOMADAS
- Xstar y Palace: NO "candidatos a revisar" — YA PAUSADOS por decisión de Roger (recorte 24-ago, pausa 3-sep). El gasto de agosto es previo.
- Nature: inversión sostenida sin retorno = apuesta de posicionamiento en curso, NO fuga.
- Encuadre SIN el "11-ago" ni "-37%" (no había dato que lo respaldara, se sacó).
- ROAS por modelo = solo SP; el general incluye todos los formatos (aclarado en nota al pie).

## TAB MoM EN EL DASHBOARD (código)
- 4 bloques: encuadre (banner) + Indicadores generales (MoM_KPIs) + Por modelo (MoM_Modelos, tabla
  horizontal, columna Modelo sticky, Serene Hybrid hero, Xstar/Palace atenuados) + Plazo de entrega
  (Delivery_Promise, jue/vie en ámbar).
- Cada bloque tiene intro de análisis Modo B (client-safe, verificado contra tablas).
- Todo hardcodeado (meses, ventana DP, título, intros) → cambiar de mes = tocar todo eso (ver CLAUDE.md).
- Commits: ed59bef (tab) · 0d58f06 (encuadre) · 61ab268 (sticky) · bc75cd3 (análisis) · 9ae7904 (redacción)
  · d25bbb2 (presentación: footer/€/ROAS) · 399220a (análisis senior) · dd8401e (CLAUDE.md). TODOS pusheados.

## PPT PARA ROGER (entregable cliente)
- Generada con pptxgenjs + gráficos como IMÁGENES matplotlib (Google Slides no renderiza charts nativos).
- Paleta AZUL MARINO (#1E3A5F) — Roger la aceptó antes, dijo "se ve menos AI". NO el verde del dashboard.
- 7 slides: portada · el mes en 3 números · evolución de ventas · evolución de publicidad · por modelo · plazo de entrega · foco del mes.
- Estilo: kicker gris + línea bajo título (como PPT que Roger aprobó), sin tics de IA.
- Archivo entregado, script en /home/claude/ppt/ (efímero, no versionado — regenerar desde este handoff si hace falta).

## PENDIENTES PRÓXIMA SESIÓN (no urgentes)
1. Banner del tab Entrega dice que patrón jue/vie "no se puede afirmar" — el tab MoM ya lo afirma. Alinear.
2. Congelar SB/SD mensuales antes del próximo MoM (docs/deuda-reporting.md).
3. asin_map incompleto: 17 children jul / 15 ago sin mapear (docs/deuda-asin-map.md).
4. B0CNVYKZL1 hardcodeada en DATA.acciones línea ~716 del dashboard (array muerto, sacar).
5. Al armar MoM del mes que viene: MoM_Modelos y BuyBox de MoM_KPIs se cargan A MANO.

## MAIL/WSP ENVIADOS A ROGER
Se le mandó la PPT por WhatsApp con nota corta. Antes se le había mandado el análisis por mail con
link al dashboard. Roger pidió la PPT porque el dashboard le resultó denso.
