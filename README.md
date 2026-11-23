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

- **Conexión automática**: la app guarda la sesión de Drive en el navegador y al abrirla reconecta sola, sin ventanas ni clics. Si el token venció (~1 h), reconecta en silencio con el próximo clic o al abrir el botón **Drive** — Google solo permite su ventana de login a partir de un clic del usuario. La primera vez se conecta con el botón **Drive**; después queda automático.
- **Guardar en Drive**: sube el maestro `.xlsx` + el estado completo.
- **Descargar desde Drive**: en otro dispositivo, iniciá sesión con la misma cuenta y con un clic bajás el último estado guardado (con snapshot de seguridad previo). La app te muestra la fecha del archivo en Drive para decidir.
- **Sincronización automática** (opcional, recomendada): guarda en Drive ~25 s después de cada cambio.
- **Documentación por DUA**: en **Aduana / DUA** cada fila tiene un botón 📁 para adjuntar, listar y descargar los PDFs de ese DUA; también se pueden adjuntar por correo.

## Envío de correos con Gmail

Con la misma sesión de Google, desde:

- **Movimientos** → ✉ en cada fila: **comprobante de ingreso/egreso** (PDF generado por la app + Excel completo + documentación del DUA si existe).
- **Clientes** → ✉ en cada fila: **reporte de stock del cliente** (PDF + Excel).
- **Aduana / DUA** → 📁 → «Enviar por correo»: **stock del DUA** (PDF + Excel + los PDFs adjuntos en Drive).
- **Reportes** → «Enviar por correo»: **reporte del período** (rotación y ocupación en PDF + Excel).

El asunto y el cuerpo vienen prellenados; se puede editar destinatario, asunto, mensaje y elegir adjuntos antes de enviar.

---

## Configuración de Google Drive y Gmail (una sola vez, ~3 minutos)

La app incluye esta misma guía en **Configuración → Sincronización con Google Drive y Gmail**, con el *origen exacto* de tu instalación para copiar y pegar en Google Console.

1. Abrí <https://console.cloud.google.com> e iniciá sesión con la cuenta del depósito: **frimaraldap1914@gmail.com**.
2. En la barra superior, desplegá el selector de proyectos → **Nuevo proyecto** → nombre `WMS DAP1914` → **Crear**.
3. Menú ☰ → **APIs y servicios → Biblioteca** → buscá **Google Drive API** → **Activar**. Repetí con **Gmail API**.
4. **APIs y servicios → Pantalla de consentimiento OAuth** → tipo **Externo** → Crear → nombre de la app `WMS DAP1914` → guardá. En **Usuarios de prueba** agregá **frimaraldap1914@gmail.com** *(obligatorio mientras el proyecto quede en modo Testing)*.
5. **Credenciales → Crear credenciales → ID de cliente de OAuth** → tipo **Aplicación web**.
6. En **Orígenes de JavaScript autorizados** → **Agregar URI** y pegá el origen donde usás la app, por ejemplo:
   - `https://sebasm2kuy.github.io` (GitHub Pages), y/o
   - `http://localhost:8000` (servidor local).
7. Creá el ID de cliente y **copiá el valor** (termina en `.apps.googleusercontent.com`).
8. En la app: **Configuración → Sincronización con Google** → pegá el **Google Client ID** → **Guardar** → **Conectar con Google Drive**. La primera vez Google avisa que la app «no está verificada»: es lo esperado en modo Testing con tu propia cuenta → **Continuar**.

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

- **Sebastián** (`sebastian`, con contraseña) — **acceso total**: los 13 permisos + **panel administrativo «Usuarios»** para crear cuentas y asignar permisos a medida. Su cuenta no se puede eliminar ni desactivar, y siempre conserva acceso total.
- **Invitado** (sin contraseña, un clic) — **solo consulta**: ve todas las vistas, imprime y exporta (comprobantes, listados, reportes, CSV, Excel) y puede **descargar los datos actualizados desde Drive**, pero no modifica nada: los botones de escritura están ocultos y las acciones bloqueadas con aviso.

El **panel «Usuarios»** (visible solo para administradores, en Administración → Usuarios) permite **crear, editar, activar/desactivar y eliminar cuentas**, con contraseña o de acceso directo, y asignar **13 permisos granulares**: movimientos (incluye anular y conteos), traslados, carga rápida, pallets vacíos, catálogos (clientes/productos/posiciones), reservas, aduana/DUA, imprimir y exportar, enviar correos, descargar de Drive, respaldar en Drive, configuración e importación, y administración total. Cada inicio de sesión, cierre, creación y cambio de permisos queda en la **Auditoría** con el usuario responsable.

Las contraseñas se guardan como **hash SHA-256** (nunca en texto plano) dentro de los datos de la app; con ellas, el respaldo JSON y los archivos de Drive incluyen las cuentas. Aclaración honesta: al no haber servidor, este control es de **gestión operativa** (qué puede hacer cada persona en la app), no criptografía contra alguien que manipule el navegador a nivel técnico.

---

## Impresión y trazabilidad operativa

- **Comprobante de ingreso / egreso / traslado** — vista de impresión limpia (Ctrl+P → PDF) con encabezado del depósito, **fecha y hora de emisión**, cliente, **DUA y/o contenedor**, detalle por posición con **sector (Rack A / Rack B / Piso)**, N.º stock, SKU/producto, medida, cantidades, total y **espacio de firmas** (Operario · Conforme/Chofer · Control depósito). Se emite: al confirmar un ingreso («Registrar e imprimir comprobante» en el formulario de movimiento), en la **carga rápida por contenedor** (junto a las etiquetas de lote), en el **traslado** (ambos movimientos), desde cada fila de **Movimientos** y desde la **trazabilidad de un DUA** (comprobante de todos sus ingresos). El mismo comprobante sale en PDF al enviarlo por correo.
- **Trazabilidad por DUA** — búsqueda rápida desde el botón **«Trazabilidad DUA»** del Panel y de Aduana, y como primera opción de la **búsqueda global** (Ctrl+K): agrupa **todo** lo vinculado al DUA en una sola pantalla — resumen con saldo vigente, cajas ingresadas/egresadas, días de permanencia y estado documental, tabla de **movimientos** (ingresos, egresos y traslados) y tabla de **posiciones vigentes** con su saldo — con botones para **imprimir** la trazabilidad, imprimir el **comprobante de ingreso**, **exportar CSV**, **enviar por correo** (PDF + Excel + documentación) y abrir la **documentación en Drive**.
- **Listado de pallets y su posición** — desde **Posiciones → «Listado de pallets»**: todos los pallets activos con **Posición/Ubicación, Sector, Cliente, N.º stock, SKU/Producto, Medida, DUA y Contenedor, y Cajas**, con **filtros rápidos** (por cliente, por Rack A / Rack B / Piso y por DUA) y **impresión directa** del listado filtrado o completo — incluye **referencia al saldo de pallets vacíos** — más exportación a CSV y envío por correo (PDF).
- **Reporte del día por DUA** — desde **Movimientos → «Reporte del día»**: todos los movimientos de una fecha **agrupados por DUA**, con subtotal por DUA y total del día (cliente, contenedor, N.º stock, producto, medida, posición, sector, cajas y referencia). **Se abre solo al registrar un egreso**, con la fecha del movimiento, listo para imprimir y firmar (Entregó/Recibió/Conforme); también con fecha, tipo y cliente ajustables, exportación a CSV y envío por correo (PDF). Ideal para documentar las **salidas del día por el DUA**.

---

## Qué incluye

### Los 7 módulos de tu planilla (mismas reglas)

| Módulo | Qué hace |
|---|---|
| **Panel** | 4 KPIs (Stock actual · cajas, Líneas de inventario con saldo, SKU registrados, Movimientos cargados), stock por producto con %, historial de contenedores/DUA con saldo remanente, **mapa visual interactivo de racks** (clic → detalle, salida, traslado, etiqueta), **tarjeta de sincronización con Google Drive** y acceso directo a la **trazabilidad por DUA**. |
| **Stock Actual** | Foto real por posición (A000–A990 y B000–B990 de 10 en 10, más Piso). Stock = Σ impacto de movimientos. Estados automáticos LIBRE / OCUPADO / REVISAR SALDO y **PALET COMPARTIDO (N ítems)** con desglose expandible. |
| **Movimientos** | Fuente única de verdad con las 13 columnas de tu planilla. Tipos: Ingreso, Egreso, Ajuste +, Ajuste −. **Traslado** que genera el egreso de origen y el ingreso en destino. Cada fila: **comprobante imprimible** y **envío del comprobante por correo**. |
| **Posiciones** | Inventario por la clave (N.º stock · Cliente · Producto · Medida normalizada · Coordenada) con estado OK / SIN STOCK / REVISAR SALDO + **listado imprimible de pallets con filtros**. |
| **Resumen SKU** | Consolidado por (Producto + Medida) — agrupa mayúsculas/minúsculas y ×/x — con cajas actuales, ubicaciones activas y % del stock. |
| **Clientes** | Maestro con código, estado, contacto y observaciones + **envío de stock por correo**. |
| **Pallets Vacíos** | Registro cronológico con saldo corrido automático y KPI de saldo disponible. |

### Extensiones WMS incorporadas

- **Aduana / DUA** — saldos, **trazabilidad completa por DUA** (movimientos, posiciones y saldo, imprimible/exportable), **días de permanencia** (umbral configurable y alertas en el Panel), **estado documental** editable y **documentación PDF en Google Drive por DUA**.
- **Catálogo de Productos / SKUs** — maestro que se completa al vuelo y autocompleta formularios.
- **Anulación controlada** con ajuste de reversa + motivo; ediciones con motivo de corrección.
- **Reservas / bloqueos**, **conteo cíclico**, **etiquetas con QR**, **remitos**, **reportes por período** (CSV/PDF/correo).
- **Auditoría completa** de operaciones **con usuario responsable**, y **usuarios con permisos granulares** (invitado de solo consulta, cuentas con contraseña, panel administrativo).
- **Respaldos automáticos** (snapshots) + respaldo JSON manual + **respaldos fechados en Drive**.
- **Importación/exportación Excel** (SheetJS) y **modo offline completo**: la app funciona sin conexión; al reconectar, sincroniza.
- **Búsqueda global instantánea** (Ctrl+K), filtros en todas las tablas, modo claro/oscuro, diseño responsive (móvil, tablet, escritorio).

## Reglas de negocio replicadas

- Coordenadas válidas: **Rack A y Rack B de `A000` a `B990` de 10 en 10** (100 posiciones c/u) + zona especial **Piso**.
- `Impacto stock`: +cantidad para Ingreso y Ajuste +; −cantidad para Egreso y Ajuste − (editable para preservar correcciones históricas).
- KPI **Líneas de inventario con saldo** = posiciones físicas ocupadas en racks A y B, excluyendo Piso y sin duplicar compartidas.
- No se eliminan clientes ni SKUs con movimientos históricos (se marcan Inactivo).
- La exportación reproduce la estructura exacta de las pestañas originales.

## Verificación

- KPIs de la app 1:1 con la planilla original: **6.247 cajas · 98 posiciones ocupadas · 9 SKU con saldo · 99 posiciones con movimientos · 107 líneas de Posiciones · saldo de pallets −30**.
- Suite de pruebas automática incluida: 104 checks de núcleo (SHA-256 con vectores NIST, permisos y sesión, bootstrap de usuarios, generador PDF con tabla xref verificada byte a byte y **límites de página validados texto por texto**, constructor MIME de Gmail con roundtrip de adjuntos, codificación base64/WinAnsi, constructores de comprobante/trazabilidad/listado/reporte del día y filtros de pallets) + 123 checks de navegador (Playwright: pantalla de acceso con rechazo de clave incorrecta, contraseña sin autocompletado, invitado sin contraseña con botones de escritura ocultos y acciones bloqueadas, sesión por ventana que se cierra al abrir una pestaña nueva, conexión automática de Drive con token guardado y reconexión verificadas, las 13 vistas, modales Drive/correo/configuración/trazabilidad/listado/reporte del día, flujo completo egreso → reporte del día, panel administrativo con alta/login/permisos/desactivación/baja de usuarios, generación de PDF en página validada con poppler, exportación Excel, persistencia, cero errores de consola) + chequeo responsive en 375/768/1360 px **incluida la pantalla de acceso**.

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
