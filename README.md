# Sistema de Facturación — Grupo

Aplicación web para emitir facturas con NCF, preparar cotizaciones para los clientes,
imprimirlas y llevar métricas de ventas de las 7 empresas del grupo.

## Cómo arrancar

1. Doble clic en **`iniciar.bat`**
2. Se abre el navegador en `http://localhost:3000`

O desde la consola:

```
node server.js
```

## Usuario inicial

| Correo | Contraseña |
|---|---|
| `admin@urtec.com` | `Admin2026!` |

**Cámbiala en cuanto entres.** El sistema no pide cambiarla por obligación, pero es
una contraseña pública (está escrita aquí) y da acceso a las 7 empresas.

Para crear los demás socios: pestaña **Usuarios** → *Nuevo usuario*.

## Qué incluye

- **Facturar**: selector de empresa y de **modo de facturación** —tradicional (NCF
  tipo B) o electrónica (e-CF tipo E)—, NCF escrito a mano con validación, autocompletado
  de clientes y productos, cálculo de ITBIS, retenciones opcionales, vista previa en vivo
  e impresión en A4 con espacio para el sello.
- **Historial**: búsqueda por cliente/NCF/proceso/orden, filtros por empresa, fecha y
  estado; ver, imprimir, registrar pagos y anular.
- **Cotizaciones**: crear cotizaciones por empresa y cliente, con líneas, ITBIS,
  validez, condición de pago y notas; listarlas con filtros, imprimirlas en A4 y
  convertirlas en factura. Cada empresa lleva su propia numeración (`COT-000001`).
- **Clientes**: base de datos central con nombre, RNC/Cédula, teléfono, correo,
  contacto, dirección, ciudad y notas. Al escribir o seleccionar un cliente en
  Facturar o en una cotización se cargan sus datos automáticamente. Se pueden
  crear, editar, buscar y eliminar.
- **Métricas**: ventas del período, ticket promedio, cobrado, por cobrar y vencido;
  ventas por mes, por empresa, top clientes, productos más vendidos y cartera detallada.
- **Empresas**: alta, edición, eliminación y desactivación de empresas, con todos sus
  datos (razón social, RNC, series de NCF y e-NCF, modo permitido, contacto, banco y
  cuenta). Cada empresa guarda la fecha en que se agregó por primera vez al sistema.
  Solo el administrador puede crear, editar, desactivar y eliminar empresas.
- **Usuarios**: alta de socios, activar/desactivar, cambio de contraseña.
- **Auditoría**: cada factura, cotización, anulación, pago y cambio queda registrado.

## Dónde están los datos

Todo vive en un solo archivo: **`datos.db`** (SQLite), en esta misma carpeta.

**Respaldo:** copia `datos.db` a otro sitio (OneDrive, disco externo) una vez por semana.
Con el servidor apagado. Eso es todo el respaldo — no hay base de datos externa.

> No borres `datos.db` ni los archivos `datos.db-wal` / `datos.db-shm`.

## Estructura

```
facturacion/
  server.js            servidor web y API
  db.js                base de datos, tablas y datos iniciales
  lib/factura.js       genera el HTML de la factura para imprimir
  lib/cotizacion.js    genera el HTML de la cotización para imprimir
  public/
    login.html         pantalla de acceso
    app.html           aplicación
    app.js             lógica de la interfaz
    estilos.css        estilos de la aplicación
    factura.css        estilos de la factura impresa
  datos.db             base de datos (se crea sola)
```

## Requisitos

Solo Node.js 22 o superior. No hay dependencias que instalar.

## Cómo hacer una cotización

1. Pestaña **Cotizaciones** → **+ Nueva cotización**.
2. Elige la empresa emisora, la fecha y hasta cuándo es válida.
3. Escribe el cliente (con autocompletado de los clientes ya registrados), la
   condición de pago y las líneas, igual que en una factura.
4. Agrega notas o condiciones si hace falta y pulsa **Guardar e imprimir**.

La cotización sale en A4 con la leyenda **“Documento sin valor fiscal”**. No lleva
NCF ni e-NCF, porque no es un comprobante fiscal.

Cada empresa tiene su propia secuencia: `COT-000001`, `COT-000002`, etc.

### Convertir una cotización en factura

En la lista de cotizaciones, pulsa **Facturar**. El sistema abre la pestaña
**Facturar** con la empresa, el cliente, las líneas y los totales ya cargados, y
muestra un aviso con la cotización de origen. Solo falta elegir el modo y escribir
el NCF o e-NCF a mano; al guardar, la cotización queda marcada como **convertida** y
enlazada a la factura.

- Una cotización convertida ya no se puede volver a facturar.
- Si se anula la factura, la cotización vuelve a quedar **vigente**.
- Las cotizaciones anuladas no se borran: quedan registradas y auditadas.

## Editar, desactivar o eliminar una empresa

En la pestaña **Empresas** (solo administrador):

- **Editar**: elige la empresa, cambia sus datos y pulsa **Guardar cambios**. La fecha
  de alta no cambia.
- **Desactivar**: deja de aparecer en los selectores de Facturar y Cotizaciones, pero
  sus facturas y cotizaciones se conservan y siguen visibles en el historial. Se puede
  **reactivar** en cualquier momento.
- **Eliminar**: borra la empresa definitivamente. Solo se permite si no tiene facturas
  ni cotizaciones. Si ya emitió documentos, el sistema ofrece desactivarla en su lugar.

Siempre debe quedar al menos una empresa activa.

## Clientes

La pestaña **Clientes** es la base de datos central. Todos los socios pueden consultarla,
agregar clientes y editarlos.

- **Nuevo cliente**: nombre o razón social (obligatorio), RNC/Cédula, teléfono, correo,
  persona de contacto, dirección, ciudad y notas internas.
- **Editar**: cambia cualquier dato. La fecha de alta se conserva.
- **Buscar**: filtra por nombre, RNC, teléfono, correo o ciudad.
- **Eliminar**: si el cliente no tiene documentos, se borra de inmediato. Si ya tiene
  facturas o cotizaciones, el sistema pide una confirmación adicional y solo lo quita
  del catálogo; los documentos conservan los datos que tenían al emitirse.

Al escribir el nombre completo de un cliente en **Facturar** o en **Cotizaciones**, o al
elegirlo desde el autocompletado, se llenan automáticamente su RNC, teléfono y dirección
(incluyendo la ciudad). Si el cliente no existe, se puede escribir manualmente y quedará
registrado al guardar el documento.

No se permiten dos clientes con el mismo nombre ni con el mismo RNC/Cédula.

## Los dos modos de facturación

En la parte de arriba de la pestaña **Facturar** se elige el modo antes de emitir:

| Modo | Comprobante | Numeración |
|---|---|---|
| **Tradicional** | NCF tipo B (`B1500000032`) | Independiente por empresa |
| **Electrónica** | e-NCF tipo E (`E440000000001`) | Independiente por empresa y por modo |

- Cada empresa puede permitir ambos modos, solo tradicional o solo electrónica
  (pestaña **Empresas**). Si un modo está deshabilitado, el botón aparece apagado.
- El comprobante se sigue escribiendo a mano. El sistema valida formato, tipo, serie,
  duplicados y saltos de numeración.
- La numeración de factura interna (`000001`, `000002`…) es independiente en cada modo.
- En electrónica se elige el tipo de e-CF (44 gubernamental, 31 crédito fiscal,
  32 consumo). El e-NCF debe empezar con `E` + ese tipo.
- La factura electrónica imprime la leyenda de representación gráfica y queda marcada
  como **pendiente de transmisión**.

### Importante sobre la factura electrónica

Hoy el sistema **emite la representación gráfica** del e-CF y lleva su control, pero
**todavía no lo firma ni lo envía a la DGII**: falta el certificado digital y el
proveedor de transmisión (PAC). Mientras eso se conecta, cada e-CF queda registrado
como *pendiente* en la base de datos para enviarlo después.

La factura tradicional tipo B funciona completa desde ya.

## Notas

- Los montos se guardan en centavos (enteros), sin errores de redondeo.
- El NCF se escribe a mano, pero el sistema bloquea duplicados, valida el formato,
  comprueba la serie de la empresa y avisa si hay saltos o retrocesos.
- Las facturas anuladas no se borran: quedan marcadas y auditadas.
- Las cotizaciones no suman en métricas ni en cuentas por cobrar: solo cuentan
  cuando se convierten en factura.
- El sello y la firma se estampan a mano; el sistema deja el espacio en blanco.

## Para usarlo desde distintos lugares

Ahora mismo solo responde en `localhost` (tu computadora). Para que los socios entren
desde cualquier lado hay que publicarlo en un servidor con HTTPS. Es el siguiente paso.
