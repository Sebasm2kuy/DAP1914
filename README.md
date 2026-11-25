# DAP1914 · WMS — Control Operativo de Depósito Fiscal

Aplicación web completa (WMS) que reemplaza la planilla **Control_Stock_Deposito_Fiscal_CLIENTES.xlsx**: mismos datos, mismas reglas de negocio, operación en tiempo real.

**Arquitectura sin servidores**: la app corre 100% en el navegador y usa la cuenta de Google del depósito — **frimaraldap1914@gmail.com** — como base de datos y respaldo (**Google Drive**) y para enviar correos (**Gmail**). Sin Supabase, sin Firebase, sin backend: solo un `index.html` y tu cuenta de Google. El acceso se controla con **usuarios y permisos** (ver abajo).

```
┌─────────────────────────────  Navegador (cualquier PC / celular)  ─────────────────────────────┐
│  index.html (app completa)                                                                      │
│   ├─ Caché local inmediata (LocalStorage) → cargas, filtros y consultas al instante, offline    │
│   ├─ OAuth2 client-side (Google Identity Services, login_hint: frimaraldap1914@gmail.com)       │
│   ├─ Google Drive API → carpeta WMS_DAP1914 (maestro .xlsx, respaldos fechados, PDFs de DUAs)   │
│   └─ Gmail API → comprobantes y reportes por correo con Excel/PDF adjuntos                      │
└─────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Cómo empezar

### Opción A · GitHub Pages (recomendada — habilita Drive y Gmail)

1. Este repositorio ya está publicado: activá **Settings → Pages → Source: Deploy from a branch → `main` / `/ (root)` → Save**.
2. En un minuto queda andando en `https://sebasm2kuy.github.io/DAP1914/`.
3. Abrí ese enlace, cargá tus datos y conectá Google Drive (ver guía abajo).

### Opción B · Servidor local en tu PC (también habilita Drive y Gmail)

```bash
cd carpeta/donde/esta/index.html
python -m http.server 8000
# abrí http://localhost:8000
```

### Opción C · Archivo local (100% offline, sin Google)

Doble clic en `index.html`. **Todo funciona igual** (importar/exportar Excel, respaldos JSON) salvo la sincronización con Drive/Gmail, que Google no permite desde `file://`.

Al abrir la app por primera vez elegís: **Importar mi Excel** (recomendado), **cargar los datos históricos incluidos** (foto de tu planilla al 19/11/2026: 117 movimientos, 2 clientes, 12 SKUs, pallets vacíos) o **empezar de cero**. Antes de cada importación se crea un snapshot automático.

---

## Sincronización con Google Drive (frimaraldap1914@gmail.com)

Tocá el botón **Drive** del encabezado (o la tarjeta de sincronización del Panel). Al conectar con la cuenta del depósito — que ya viene **preseleccionada en la pantalla de login de Google** (`login_hint: frimaraldap1914@gmail.com`) — la app crea (o encuentra, si ya existe) esta estructura en **Mi Unidad**:

| Archivo / carpeta | Qué es |
|---|---|
| `WMS_DAP1914/WMS_DAP1914_DB.xlsx` | **Archivo maestro** con las pestañas del sistema: Panel, STOCK ACTUAL, Movimientos, Posiciones, Resumen SKU, Clientes, PALLETS VACIOS (+ Productos, el catálogo maestro). Se actualiza en cada sincronización. |
| `WMS_DAP1914/WMS_DAP1914_estado.json` | Estado completo de la app (reservas, estados aduaneros, auditoría, configuración) para continuar en otro dispositivo **sin perder nada**. |
| `WMS_DAP1914/Respaldos/WMS_DAP1914_DB_YYYYMMDD_HHMM.xlsx` | **Copias de respaldo fechadas** en cada «Respaldar ahora» (se conservan las últimas 15). |
| `WMS_DAP1914/Documentacion_DUAs/` | **PDFs adjuntos por DUA** (DUA escaneado, remitos, documentación aduanera) vinculados a cada DUA o movimiento. |

- **Preconfiguración de fábrica**: el **Google Client ID** del proyecto WMS DAP1914 y la cuenta del depósito (**frimaraldap1914@gmail.com**) ya vienen integrados en el código: no hay que pegarlos nunca, y siguen ahí aunque se limpie el navegador, se importen datos o se actualice el sitio. «Sincronización automática» viene **activada por defecto**.
- **Conexión y descarga automáticas al iniciar sesión** (Sebastián e Invitado): la app guarda la sesión de Drive en el navegador y al entrar reconecta sola, sin ventanas ni clics (`prompt=none` + `login_hint`); si el token venció (~1 h), reconecta en silencio con el próximo clic o al abrir el botón **Drive**. Después de conectar baja y aplica **automáticamente la última versión guardada en Drive** (con snapshot de seguridad previo) cuando Drive tiene algo más nuevo que lo local — o cuando el navegador está vacío. La única vez que pide autorización interactiva es la primera en cada navegador. Si la reconexión silenciosa no puede completarse (navegador limpiado o sin sesión de Google activa en ese navegador), la app **avisa con el paso a seguir** (botón Drive → Conectar → «Descargar de Drive») en lugar de fallar en silencio; y al reconectar con un clic **baja la última copia automáticamente**, sin buscarla a mano.
- **Las cuentas de usuario viajan con la copia de Drive**: al restaurar (automática o manualmente), las cuentas se **combinan** con las del navegador — las que solo estaban en Drive vuelven (por ejemplo, Raúl en un navegador limpiado o nuevo) y la cuenta con la que entraste conserva su versión local. Queda registrado en la Auditoría («Cuentas restauradas desde Drive»). Además, cada cambio de cuentas (crear, editar, activar/desactivar, enviar a la papelera o restaurar) **sube a Drive en el momento** — antes solo se guardaba en el navegador, y un respaldo viejo podía restaurar una base sin la cuenta nueva.
- **Protección del respaldo**: si un navegador solo tiene los datos de fábrica (demo) cargados tras un borrado local, **no se suben a Drive** cuando ya existe una base real guardada — la copia buena nunca se reemplaza con datos de demostración; primero se restaura desde Drive y la bandera demo se apaga sola.
- **Guardar en Drive**: sube el maestro `.xlsx` + el estado completo.
- **Descargar desde Drive**: en otro dispositivo, con un clic bajás el último estado guardado (con snapshot de seguridad previo). La app te muestra la fecha del archivo en Drive para decidir.
- **Sincronización automática** (activada por defecto): guarda en Drive ~25 s después de cada cambio.
- **Documentación por DUA**: en **Aduana / DUA** cada fila tiene un botón 📁 para adjuntar, listar y descargar los PDFs de ese DUA; también se pueden adjuntar por correo.

## Envío de correos con Gmail

Con la misma sesión de Google, desde:

- **Movimientos** → ✉ en cada fila: **comprobante de ingreso/egreso** (PDF generado por la app + Excel completo + documentación del DUA si existe).
- **Movimientos** → 🏷 en cada fila (solo **Sebastián y Raúl**): **reimpresión de la etiqueta de pallet** de ese movimiento — ver «Impresión y trazabilidad operativa».
- **Clientes** → ✉ en cada fila: **reporte de stock del cliente** (PDF + Excel).
- **Aduana / DUA** → 📁 → «Enviar por correo»: **stock del DUA** (PDF + Excel + los PDFs adjuntos en Drive).
- **Reportes** → «Enviar por correo»: **reporte del período** (rotación y ocupación en PDF + Excel).

El asunto y el cuerpo vienen prellenados; se puede editar destinatario, asunto, mensaje y elegir adjuntos antes de enviar.

---

## Configuración de Google Drive y Gmail (ya viene hecha)

El **Client ID de fábrica ya está integrado y activo**: no hay que configurar nada para usar Drive y Gmail. Esta guía queda como referencia **solo si algún día hay que recrear las credenciales** en Google Cloud (por ejemplo, si se elimina el proyecto); la app incluye la misma guía en **Configuración → Sincronización con Google Drive y Gmail**, con el *origen exacto* de tu instalación para copiar y pegar.

1. Abrí <https://console.cloud.google.com> e iniciá sesión con la cuenta del depósito: **frimaraldap1914@gmail.com**.
2. En la barra superior, desplegá el selector de proyectos → **Nuevo proyecto** → nombre `WMS DAP1914` → **Crear**.
3. Menú ☰ → **APIs y servicios → Biblioteca** → buscá **Google Drive API** → **Activar**. Repetí con **Gmail API**.
4. **APIs y servicios → Pantalla de consentimiento OAuth** → tipo **Externo** → Crear → nombre de la app `WMS DAP1914` → guardá. En **Usuarios de prueba** agregá **frimaraldap1914@gmail.com** *(obligatorio mientras el proyecto quede en modo Testing)*.
5. **Credenciales → Crear credenciales → ID de cliente de OAuth** → tipo **Aplicación web**.
6. En **Orígenes de JavaScript autorizados** → **Agregar URI** y pegá el origen donde usás la app, por ejemplo:
   - `https://sebasm2kuy.github.io` (GitHub Pages), y/o
   - `http://localhost:8000` (servidor local).
7. Creá el ID de cliente y **copiá el valor** (termina en `.apps.googleusercontent.com`).
8. En la app: **Configuración → Sincronización con Google** ya viene el campo **precargado de fábrica**; solo si lo cambiás, **Guardá**. Apretá **Conectar con Google Drive** (una única vez por navegador). La primera vez Google avisa que la app «no está verificada»: es lo esperado en modo Testing con tu propia cuenta → **Continuar**.

**Permisos mínimos que pide la app** (verificalos en la pantalla de consentimiento):

| Permiso | Alcance real |
|---|---|
| `drive.file` | Ver y gestionar **solo los archivos que esta app crea** (carpeta `WMS_DAP1914`). Nunca ve el resto de tu Drive. |
| `gmail.send` | **Solo enviar** correos en tu nombre. No lee tu bandeja. |
| `userinfo.email` | Saber con qué cuenta iniciaste sesión. |

El token de acceso se guarda solo en tu navegador (localStorage) para que la app reconecte sola con Drive; nunca se sube a ningún lado, no se guarda en el repo, se borra al desconectar y la app solo lo usa para las operaciones anteriores.

---

## Usuarios y permisos

Al abrir la app aparece la **pantalla de acceso**: se elige el usuario y (si tiene) se escribe la contraseña — el navegador **no puede autocompletarla** (se teclea cada vez). La sesión queda activa **solo mientras esa ventana esté abierta**: al cerrar la página se cierra sola y la próxima visita vuelve a pedir usuario (recargar la pestaña sí mantiene la sesión); también se puede salir antes con «Cerrar sesión» en el menú lateral.

- **Sebastián** (`sebastian`, con contraseña) — **acceso total**: los 13 permisos + **panel administrativo «Usuarios»** para crear cuentas y asignar permisos a medida. Su cuenta no se puede eliminar ni desactivar, y siempre conserva acceso total. El **nombre y el usuario de ingreso** de las cuentas del sistema se pueden corregir desde «Editar» (por ejemplo, ajustar el usuario de Sebastián Cabrera si quedó cargado distinto); la cuenta sigue siendo no eliminable ni desactivable y mantiene su acceso total.
- **Invitado** (sin contraseña, un clic) — **solo consulta**: ve todas las vistas, imprime y exporta (comprobantes, listados, reportes, CSV, Excel) y al entrar la app intenta **conectar y descargar sola** los datos actualizados de Drive (silenciosamente, con `prompt=none`); si ese navegador no tiene sesión de Google del depósito, ve **la última copia local sincronizada** como respaldo de lectura, pero no modifica nada: los botones de escritura están ocultos y las acciones bloqueadas con aviso. En la tarjeta de Google Drive del Panel solo ve el **estado de conexión y sincronización** (sin «Guardar», «Respaldar» ni «Sincronización», que quedan reservados al administrador); en el modal de Drive conserva la **descarga de la última versión**.

El **panel «Usuarios»** (visible solo para administradores, en Administración → Usuarios) permite **crear, editar, activar/desactivar y eliminar cuentas**, con contraseña o de acceso directo, y asignar **13 permisos granulares**: movimientos (incluye anular y conteos), traslados, carga rápida, pallets vacíos, catálogos (clientes/productos/posiciones), reservas, aduana/DUA, imprimir y exportar, enviar correos, descargar de Drive, respaldar en Drive, configuración e importación, y administración total. Cada inicio de sesión, cierre, creación y cambio de permisos queda en la **Auditoría** con el usuario responsable. **Eliminar una cuenta no la borra**: va a la **Papelera de cuentas** al pie de la vista (con fecha y responsable) y se restaura con su contraseña y permisos intactos; su nombre de usuario queda reservado para evitar duplicados. Todo cambio de cuentas sube a Drive en el momento, así ningún respaldo viejo puede traer de vuelta una base sin las cuentas nuevas.

Las contraseñas se guardan como **hash SHA-256** (nunca en texto plano) dentro de los datos de la app; con ellas, el respaldo JSON y los archivos de Drive incluyen las cuentas. Aclaración honesta: al no haber servidor, este control es de **gestión operativa** (qué puede hacer cada persona en la app), no criptografía contra alguien que manipule el navegador a nivel técnico.

---

## Impresión y trazabilidad operativa

- **Comprobante de ingreso / egreso / traslado** — vista de impresión limpia (Ctrl+P → PDF) con encabezado del depósito, **fecha y hora de emisión**, cliente, **DUA y/o contenedor**, detalle por posición con **sector (Rack A / Rack B / Piso)**, N.º stock, SKU/producto, medida, cantidades, total y **espacio de firmas** (Operario · Conforme/Chofer · Control depósito). Se emite: al confirmar un ingreso («Registrar e imprimir comprobante» en el formulario de movimiento), en la **carga rápida por contenedor** (junto a las etiquetas de lote), en el **traslado** (ambos movimientos), desde cada fila de **Movimientos** y desde la **trazabilidad de un DUA** (comprobante de todos sus ingresos). El mismo comprobante sale en PDF al enviarlo por correo.
- **Etiqueta de pallet (reimpresión desde Movimientos)** — ícono de etiqueta en cada fila del **Registro de movimientos**, **exclusivo para Sebastián y Raúl** (por identidad de usuario, exacta y sin acentos-intercambio; el Invitado · consulta y cualquier otro usuario, aunque tenga permiso de administración, no la ve ni puede ejecutarla por consola). Imprime la etiqueta del pallet de esa línea con sus **datos reales**: **Coordenada en grande (+ Nivel físico si es mayor a 1)**, Producto, Medida, Cantidad (cajas), N.º stock, Cliente, DUA, Contenedor, Fecha y Observaciones, sobre `#printArea` / `@media print` (sin menús ni la tabla, lista para pegar en el pallet). La puerta de la acción es la identidad (no requiere el permiso genérico de impresión), los movimientos **anulados no se etiquetan** y cada reimpresión queda en la **Auditoría** con el usuario.
- **Trazabilidad por DUA** — búsqueda rápida desde el botón **«Trazabilidad DUA»** del Panel y de Aduana, y como primera opción de la **búsqueda global** (Ctrl+K): agrupa **todo** lo vinculado al DUA en una sola pantalla — resumen con saldo vigente, cajas ingresadas/egresadas, días de permanencia y estado documental, tabla de **movimientos** (ingresos, egresos y traslados) y tabla de **posiciones vigentes** con su saldo — con botones para **imprimir** la trazabilidad, imprimir el **comprobante de ingreso**, **exportar CSV**, **enviar por correo** (PDF + Excel + documentación) y abrir la **documentación en Drive**.
- **Listado de pallets y su posición** — desde **Posiciones → «Listado de pallets»**: todos los pallets activos con **Posición/Ubicación, Sector, Cliente, N.º stock, SKU/Producto, Medida, DUA y Contenedor, y Cajas**, con **filtros rápidos** (por cliente, por Rack A / Rack B / Piso y por DUA) y **impresión directa** del listado filtrado o completo — incluye **referencia al saldo de pallets vacíos** — más exportación a CSV y envío por correo (PDF).
- **Reporte del día por DUA** — desde **Movimientos → «Reporte del día»**: todos los movimientos de una fecha **agrupados por DUA**, con subtotal por DUA y total del día (cliente, contenedor, N.º stock, producto, medida, posición, sector, cajas y referencia). **Se abre solo al registrar un egreso**, con la fecha del movimiento, listo para imprimir y firmar (Entregó/Recibió/Conforme); también con fecha, tipo y cliente ajustables, exportación a CSV y envío por correo (PDF). Ideal para documentar las **salidas del día por el DUA**.

---

## Qué incluye

### Los 7 módulos de tu planilla (mismas reglas)

| Módulo | Qué hace |
|---|---|
| **Panel** | 4 KPIs (Stock actual · cajas, Líneas de inventario con saldo, SKU registrados, Movimientos cargados), stock por producto con %, historial de contenedores/DUA con saldo remanente, **mapa visual interactivo de racks** (clic → detalle, salida, traslado, etiqueta), **tarjeta de sincronización con Google Drive** y acceso directo a la **trazabilidad por DUA**. |
| **Stock Actual** | Foto real por posición (módulos A–F del plano real: 406 posiciones × 4 niveles, más Piso). Stock = Σ impacto de movimientos. Estados automáticos LIBRE / OCUPADO / REVISAR SALDO y **PALET COMPARTIDO (N ítems)** con desglose expandible. Botón **«Imprimir ocupados»** (admin e invitado): imprime **solo las posiciones ocupadas** en una vista de impresión limpia con título, fecha y hora, resumen (posiciones ocupadas, total de cajas, palets compartidos) y el detalle por posición, respetando los filtros de búsqueda y rack activos. |
| **Movimientos** | Fuente única de verdad con las 13 columnas de tu planilla. **Tabla densa: las 15 columnas entran visibles sin scroll horizontal a 100% de zoom en escritorio estándar (1280–1600px)** — encabezados cortos con tooltip (Tipo, Stock, Cont., Coord., Cant., Ref., Obs., Impacto, Vigencia), fuente 12px con `nowrap` controlado en fechas/números/badges y `truncate` con el texto completo al pasar el mouse en Producto, Contenedor, Medida, Ref. y Observaciones; la Vigencia muestra «Vigente · N» con el detalle completo en el tooltip. En pantallas de 981–1366px el menú lateral pasa a **rail de íconos con tooltips** para liberar ancho. El `overflow` del contenedor queda solo como respaldo real para pantallas angostas (≤980px) o paneles abiertos. Tipos: Ingreso, Egreso, Ajuste +, Ajuste −. **Traslado** que genera el egreso de origen y el ingreso en destino. Cada fila: **comprobante imprimible** y **envío del comprobante por correo**. La columna **«Vigencia / saldo»** cruza cada ingreso con el stock actual por partida (N.º stock · cliente · producto · medida · coordenada, palets compartidos incluidos): **Vigente** (verde, conserva todo su saldo), **Parcial** (ámbar, «Saldo: 18 / 72 cajas» tras egresos parciales) o **Agotada** (gris, «Saldo: 0 cajas» — queda como registro histórico/trazabilidad sin parecer stock activo); los egresos muestran su impacto sin confundirse con disponibilidad. **Filtro rápido por estado de saldo** (Todos / Con saldo / Parcial / Agotados). El campo **Ref. / documento** se normaliza a formato oración en todo el sistema («INGRESO» y «Descarga inicial» → «Ingreso», «SALIDA» → «Salida»), tanto en pantalla como al importar, guardar en Drive y exportar a Excel. |
| **Posiciones** | Inventario por la clave (N.º stock · Cliente · Producto · Medida normalizada · Coordenada) con estado OK / SIN STOCK / REVISAR SALDO + **listado imprimible de pallets con filtros**. |
| **Resumen SKU** | Consolidado por (Producto + Medida) — agrupa mayúsculas/minúsculas y ×/x — con cajas actuales, ubicaciones activas y % del stock. |
| **Clientes** | Maestro con código, estado, contacto y observaciones + **envío de stock por correo**. |
| **Pallets Vacíos** | Registro cronológico con saldo corrido automático y KPI de saldo disponible. |

### Extensiones WMS incorporadas

- **Aduana / DUA** — saldos, **trazabilidad completa por DUA** (movimientos, posiciones y saldo, imprimible/exportable), **días de permanencia** (umbral configurable y alertas en el Panel), **estado documental** editable y **documentación PDF en Google Drive por DUA**.
- **Catálogo de Productos / SKUs** — maestro que se completa al vuelo y autocompleta formularios.
- **Palet compartido multi-producto** — en «Nuevo movimiento» el botón **«+ Agregar producto al pallet»** agrega líneas de Producto/Medida/Cantidad que comparten coordenada, nivel, N.º stock, DUA, contenedor y fecha: cada línea impacta a su propio producto (p. ej. +4 Ajo en Polvo y +4 Ajo Granulado en el mismo pallet), la posición se reconoce como **palet compartido** en Stock Actual, Mapa 3D, mini-mapa del Panel y búsqueda global, y el **Resumen SKU** y «Stock por producto» del Panel suman cada producto por separado (sin ítems combinados artificiales). El egreso también acepta varias líneas con validación de saldo sobre el total; editar y anular siguen operando por línea individual y todo se conserva en el respaldo de Drive y en el Excel (WMS_DAP1914_DB.xlsx).
- **Agregar SKU / caja a una posición existente** — acción directa **«Agregar SKU / caja a esta posición»** en cada fila de Movimientos (ícone «+»), en el detalle de Posición y en las filas compartidas de Stock Actual: abre el formulario con la **cabecera precargada del pallet** (coordenada, nivel, cliente, N.º stock, contenedor, DUA y fecha de esa posición) y muestra el **contenido actual**, para que solo haya que escribir el nuevo Producto, Medida y Cantidad. La cabecera de **Nuevo movimiento** y de la **Carga rápida** además **hereda en la sesión** los últimos valores usados de Cliente, N.º stock, Contenedor, DUA y Ref./documento (fecha actual e Ingreso por defecto), y «+ Agregar producto al pallet» sigue disponible para cargar varios SKUs juntos.
- **Posición ocupada: aviso, nunca bloqueo** — al registrar sobre una posición con stock, el formulario y la Carga rápida muestran un aviso informativo: **mismo SKU** → «⚠ ya ocupada por este mismo SKU: las cajas se suman al producto existente»; **SKU distinto** → «＋ posición compartida: el nuevo SKU se agrega al mismo pallet sin pisar lo guardado». En ambos casos se puede guardar sin trabas; editar un movimiento y cambiarle **Coordenada o Nivel** recalcula el stock en la posición anterior y en la nueva (por altura incluida).
- **Anulación controlada** con ajuste de reversa + motivo; ediciones con motivo de corrección.
- **Reservas / bloqueos**, **conteo cíclico**, **etiquetas con QR**, **remitos**, **reportes por período** (CSV/PDF/correo).
- **Auditoría completa** de operaciones **con usuario responsable**, y **usuarios con permisos granulares** (invitado de solo consulta, cuentas con contraseña, panel administrativo).
- **Respaldos automáticos** (snapshots) + respaldo JSON manual + **respaldos fechados en Drive**.
- **Importación/exportación Excel** (SheetJS) y **modo offline completo**: la app funciona sin conexión; al reconectar, sincroniza.
- **Búsqueda global instantánea** (Ctrl+K), filtros en todas las tablas, modo claro/oscuro, diseño responsive (móvil, tablet, escritorio).

## Reglas de negocio replicadas

- Coordenadas válidas: el **plano real a escala del depósito** (MAPA_DEPOSITO_1914_ESCALA_REAL.xlsx) — **6 módulos A–F con 406 posiciones codificadas × 4 niveles = 1.624 ubicaciones** + zona especial **Piso**. Cada módulo es una franja doble (frente + pasillo de 2,00 m + dorso, 4,40 m de fondo); la numeración es **serpenteante** (frente ascendente `000→450` y el dorso continúa `460→890`, B/C/D con frente `000–430` y dorso `440–870`, E y F en la mitad este) y los **cruces de 2 m** quedan sin posición (A dorso 680/690, B/C frente 200/210 y dorso 660/670, D frente 200/210). El histórico cargado (98 posiciones en A y B) cae 1:1 dentro del plano: el rack A queda exactamente con sus 88 posiciones ocupadas.
- `Impacto stock`: +cantidad para Ingreso y Ajuste +; −cantidad para Egreso y Ajuste − (editable para preservar correcciones históricas).
- Una **partida** es la combinación **N.º stock + cliente + producto + medida + posición** (con su nivel físico): varios productos en el mismo pallet son varias partidas y la posición se marca **palet compartido**; el saldo vigente se calcula por partida.
- KPI **Líneas de inventario con saldo** = posiciones físicas ocupadas en racks A y B, excluyendo Piso y sin duplicar compartidas.
- No se eliminan clientes ni SKUs con movimientos históricos (se marcan Inactivo).
- La exportación reproduce la estructura exacta de las pestañas originales.

## Verificación

- KPIs de la app 1:1 con la planilla original: **6.247 cajas · 98 posiciones ocupadas · 9 SKU con saldo · 99 posiciones con movimientos · 107 líneas de Posiciones · saldo de pallets −30**.
- Suite de pruebas automática incluida (90 checks de navegador con Playwright + chequeo de sintaxis de los 12 bloques JS con node --check; cero errores de consola en todas): **Mapa real de 6 módulos (24 checks)**: el Mapa 3D y el mapa cenital del Panel reproducen el plano a escala (módulos A 88 · B 84 · C 84 · D 64 · E 44 · F 42 = 406 posiciones × 4 niveles = 1.624 ubicaciones, numeración serpenteante frente/dorso, cruces de 2 m sin posición, D-dorso/E/F en la mitad este), el histórico cae 1:1 en el plano (rack A con sus 88 posiciones), filtros por módulo y presets de cámara A–F, selector de coordenadas con grupos Rack A–F y sectores/contadores recalculados; **Papelera de cuentas — ninguna cuenta se borra definitivamente (21 checks)**: crear y «eliminar» una cuenta la envía a la **Papelera de cuentas** (deja de entrar y de contar, pero conserva contraseña, permisos e historia y se restaura con un clic desde Usuarios), el nombre de usuario queda reservado mientras está en la papelera, una sesión activa o la pantalla de inicio ignoran la cuenta en papelera, el contador del menú no la cuenta, la cuenta del sistema nunca va a la papelera, y las fusiones desde Drive resucitan la cuenta si la copia de Drive la tiene activa (gana la copia de Drive) o la conservan en papelera sin perderla; **Posiciones compartidas y edición de posiciones (29 checks)** + **tabla densa sin scroll y rail de íconos (incluidos en la regresión, 16 checks)**: las 15 columnas del Registro de movimientos visibles sin scroll a 1280/1440/1600px con acciones y Vigencia completas, menú lateral en rail de 56px a ≤1366px, y como antes: herencia de cabecera en la sesión (Cliente, N.º stock, Contenedor, DUA y Ref. se recuerdan entre formularios; fecha actual e Ingreso por defecto), aviso informativo «⚠ ya ocupada por este mismo SKU — se suman» y «＋ Posición compartida» sin bloquear el registro, **«Agregar SKU / caja a esta posición» desde Movimientos, Stock Actual y el detalle de Posición con la cabecera del pallet precargada y el contenido actual visible**, palet compartido reconocido (Mapa 3D y badge «PALET COMPARTIDO» con desglose por SKU en Stock Actual), **recálculo al editar la Coordenada (la posición anterior pierde y la nueva gana) y el Nivel (stock por altura con niveles mapeados)**, Carga rápida que hereda cabecera, muestra banderas informativas por fila y registra sobre posiciones ocupadas sin bloquear, e invitado sin la acción de agregar; **Regresión compacta (24 checks)**: pantalla de acceso con cuentas de fábrica, datos demo marcados como demostración, palet multi-producto («+» con líneas compartidas, preview, partidas, Resumen SKU y catálogo), egreso multi-línea + anulación con reversa, usuarios (cuenta del sistema editable, sin duplicados, no desactivable), importación replace que apaga la bandera demo conservando cuentas, invitado de solo lectura estricta; **Drive automático (16 checks)**: restauración automática desde Drive con fusión de cuentas (las que solo estaban en Drive vuelven — p. ej. Raúl — y la sesión conserva su versión local), snapshot previo, lastSyncAt, bandera demo apagada al restaurar, protección del respaldo (los datos demo con copia existente no se suben), descarga manual con cuentas y avisos accionables si la reconexión silenciosa falla. Las suites históricas (Mapa 3D 54–75, etiqueta 35, saldo/impresión 17, config 12, invitado/Drive 12) se corrieron en sus commits originales; la cobertura viva actual es la descrita arriba.

## Estructura del repositorio

```
index.html   → la aplicación completa (autocontenida: CSS, JS, SheetJS y generador QR inline)
README.md    → este archivo
```

Librerías incrustadas (con sus licencias): [SheetJS xlsx 0.20.3](https://sheetjs.com) (Apache-2.0) y [qrcode-generator 1.4.4](https://github.com/kazuhikoarase/qrcode-generator) (MIT). El flujo OAuth carga [Google Identity Services](https://developers.google.com/identity/gsi/web) desde los servidores de Google solo cuando conectás Drive.

## Seguridad y privacidad

- Los datos operativos viven en el **almacenamiento local del navegador** y, al sincronizar, en **tu propia cuenta de Google** (`frimaraldap1914@gmail.com`): no hay servidores de terceros.
- La app pide los permisos mínimos (`drive.file`, `gmail.send`, `userinfo.email`) y el token de sesión nunca sale del navegador.
- Este repositorio es **público** y `index.html` incluye la carga histórica embebida (foto al 19/11/2026) para arrancar con un clic. Si preferís no exponer esos datos operativos: hacé el repo privado (Settings → General → Danger Zone → Change visibility) o borrá el bloque `const WMS_SEED = ...` del archivo; la app seguirá funcionando igual.
