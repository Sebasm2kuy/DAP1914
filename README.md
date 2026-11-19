# DAP1914 · WMS — Control Operativo de Depósito Fiscal

Aplicación web completa (WMS) que reemplaza la planilla **Control_Stock_Deposito_Fiscal_CLIENTES.xlsx**: mismos datos, mismas reglas de negocio, operación en tiempo real.

**Un solo archivo, sin instalación ni dependencias**: abrí `index.html` en cualquier navegador (Chrome, Edge, Firefox) y listo. También funciona publicada en GitHub Pages.

---

## Qué incluye

### Los 7 módulos de tu planilla (mismas reglas)

| Módulo | Qué hace |
|---|---|
| **Panel** | 4 KPIs (Stock actual · cajas, Líneas de inventario con saldo, SKU registrados, Movimientos cargados), stock por producto con %, historial de contenedores/DUA con saldo remanente y **mapa visual interactivo de racks** (clic en una posición → detalle, salida, traslado, etiqueta). |
| **Stock Actual** | Foto real por posición (A000–A990 y B000–B990 de 10 en 10, más Piso). Stock = Σ impacto de movimientos. Estados automáticos LIBRE / OCUPADO / REVISAR SALDO y **PALET COMPARTIDO (N ítems)** con desglose expandible. |
| **Movimientos** | Fuente única de verdad con las 13 columnas de tu planilla (Fecha, Tipo, N.º stock, Cliente, Contenedor, DUA, Producto, Medida, Coordenada, Cantidad, Ref., Observaciones, Impacto). Tipos: Ingreso, Egreso, Ajuste +, Ajuste −. **Traslado** que genera automáticamente el egreso de origen y el ingreso en destino. |
| **Posiciones** | Inventario por la clave (N.º stock · Cliente · Producto · Medida normalizada · Coordenada) con estado OK / SIN STOCK / REVISAR SALDO. |
| **Resumen SKU** | Consolidado por (Producto + Medida) — agrupa mayúsculas/minúsculas y ×/x — con cajas actuales, ubicaciones activas y % del stock. |
| **Clientes** | Maestro con código, estado Activo/Inactivo, contacto y observaciones. No se puede eliminar un cliente con movimientos históricos: se marca Inactivo. |
| **Pallets Vacíos** | Registro cronológico con saldo corrido automático (stock anterior + ingreso − egreso) y KPI de saldo disponible. |

### Extensiones WMS incorporadas

- **Aduana / DUA** — saldos, trazabilidad, **días de permanencia** (con umbral configurable y alertas en el Panel) y **estado documental** editable (OK, pendiente de liberación, despacho, pesaje, pendiente de documentación).
- **Catálogo de Productos / SKUs** — maestro (Producto, Medida, cliente asociado, unidad/observaciones) que se completa automáticamente al usar productos nuevos (alta al vuelo) y autocompleta medida/cliente en los formularios.
- **Anulación controlada de movimientos** — no se borra: se registra un **ajuste de reversa** con motivo, el original queda marcado ANULADO y todo queda en la auditoría. Las ediciones aceptan **motivo de corrección**.
- **Reservas / bloqueos** de mercadería por línea: el saldo disponible descuenta lo reservado; visible en el mapa (⚑) y en el detalle de posición.
- **Conteo cíclico / inventario físico** — teórico vs. contado por zona, con generación automática de Ajustes ± por diferencia.
- **Etiquetas de pallet/ubicación con código QR** imprimibles (una o por lote tras una carga por contenedor) y **remitos imprimibles** (Ctrl+P → guardar como PDF).
- **Reportes por período** — rotación, ingresos/egresos/neto por producto, cliente y posición + foto de ocupación; exportables a CSV o PDF.
- **Auditoría completa** — cada creación, edición, anulación, importación, reserva y cambio de configuración queda registrado con fecha, hora y rol.
- **Roles** operario / administrador con **PIN** opcional: el operario carga movimientos y consulta; el administrador gestiona todo lo demás.
- **Respaldos automáticos (snapshots)** antes de cada importación o restauración (se conservan los últimos 3, restaurables desde Configuración), más respaldo/restauración manual en JSON.
- **Importación/exportación Excel** con SheetJS: lee tu archivo actual (con fórmulas o valores cacheados) y exporta un `.xlsx` limpio con las **8 pestañas** (las 7 originales + Productos).
- **Búsqueda global instantánea** (Ctrl+K) por coordenada, N.º stock, DUA, contenedor, cliente o producto; filtros en todas las tablas; modo claro/oscuro; diseño responsive (móvil, tablet, escritorio).

## Cómo empezar

1. Abrí `index.html` en el navegador.
2. En la pantalla de bienvenida elegí:
   - **Importar mi Excel** → seleccioná tu `Control_Stock_Deposito_Fiscal_CLIENTES.xlsx` (recomendado), o
   - **Cargar datos históricos incluidos** → foto de tu planilla al 19/11/2026 (117 movimientos, 2 clientes, 12 SKUs, pallets vacíos), o
   - **Empezar de cero**.
3. Operá. Antes de importar se crea un snapshot automático por si querés volver atrás.

Los datos se guardan en el **almacenamiento local del navegador**. Para pasarlos a otra máquina: `Exportar → Respaldo JSON` y después `Restaurar respaldo`, o exportá el Excel. Cada navegador/PC mantiene sus propios datos.

## Reglas de negocio replicadas

- Coordenadas válidas: **Rack A y Rack B de `A000` a `B990` de 10 en 10** (100 posiciones cada uno) + zona especial **Piso**.
- `Impacto stock`: +cantidad para Ingreso y Ajuste +; −cantidad para Egreso y Ajuste − (editable manualmente para preservar correcciones históricas, ej. A020).
- KPI **Líneas de inventario con saldo** = posiciones físicas ocupadas en racks A y B, **excluyendo Piso y sin duplicar coordenadas compartidas**.
- No se eliminan clientes ni SKUs con movimientos históricos (se marcan Inactivo / se retiran del catálogo solo los sin uso).
- La exportación reproduce la estructura exacta de las pestañas originales (encabezados, KPIs en Panel y Stock Actual, orden cronológico con el orden original del archivo como desempate).

## Verificación

Los KPIs calculados por la app coinciden 1:1 con la planilla original: **6.247 cajas · 98 posiciones ocupadas · 102 libres · 9 SKU con saldo · 99 posiciones con movimientos · 107 líneas de Posiciones · saldo de pallets −30**.

## Estructura del repositorio

```
index.html   → la aplicación completa (autocontenida: CSS, JS, SheetJS y generador QR inline)
README.md    → este archivo
```

Librerías incrustadas (con sus licencias): [SheetJS xlsx 0.20.3](https://sheetjs.com) (Apache-2.0) y [qrcode-generator 1.4.4](https://github.com/kazuhikoarase/qrcode-generator) (MIT).

> Nota: el archivo `index.html` incluye la carga histórica embebida (tu foto al 19/11/2026) para poder arrancar con un clic. Si el repositorio es público y preferís no incluir esos datos operativos, borrrá el bloque `const WMS_SEED = ...` del archivo: la app seguirá funcionando y podrás importar tu Excel igual.
