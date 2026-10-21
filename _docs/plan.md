# Plan de Desarrollo - Plataforma de Impresión 3D

## 1. Visión General del Proyecto
Desarrollar una plataforma web profesional para ofrecer servicios de impresión 3D bajo demanda en San Juan del Río, Querétaro, con alcance de envíos a todo México. El sistema debe permitir cotizaciones automáticas basadas en el análisis de archivos STL y cotizaciones manuales para proyectos complejos. Se integrará un flujo de pagos para cobrar el 50% de anticipo y un sistema seguro de gestión de órdenes.

## 2. Stack elegido

Opción 1. La primera versión corre en la máquina. GCP entra cuando el camino cotizar, pagar y rastrear ya se puede recorrer de punta a punta.

La primera versión local nombra tres cosas:

*   **Frontend:** React, Vite, TypeScript, Tailwind CSS y shadcn/ui, en `frontend/`. El visor 3D usa `react-three-fiber`.
*   **Backend:** Python con FastAPI, en `backend/`. Volumen, dimensiones y malla cerrada se calculan con `trimesh`.
*   **Datos:** PostgreSQL en Docker. Los STL, las imágenes de referencia y los PDF de constancias viven en una carpeta del disco. La conexión a la base va en una variable de entorno. La llave de cifrado de datos personales también va en una variable de entorno.

El panel de administración es una segunda app Vite, en `admin/`. Mismo frontend (React, Vite, TypeScript, Tailwind, shadcn/ui). Se construye cuando las seis etapas de abajo ya tienen un resultado que se puede ejecutar. Hasta entonces, el sitio público es la única UI.

Pagos: PayPal, 50% de anticipo. En local se usa el sandbox de PayPal. Sin cuentas de cliente: un Quote ID pasa a Order ID, y el rastreo pide Order ID más correo.

Correo transaccional: desde el backend (Resend o SMTP) cuando el estado de la orden cambia. No depende de GCP.

Cuando cotizar, pagar y rastrear ya funcionen en local, GCP reemplaza estas tres piezas y nada más del producto:

*   Cloud Storage, en lugar de la carpeta de archivos.
*   Cloud SQL, si la base sale de la máquina.
*   Secret Manager o Cloud KMS, en lugar de la llave de cifrado en el entorno.

## 3. Arquitectura y Seguridad
*   **Gestión de Usuarios (Guest Mode):** No hay registro ni login para clientes. Se usa un `Quote ID` para la etapa de cotización que transiciona a un `Order ID` cuando se confirma el pago. Los clientes rastrean su pedido usando su `Order ID` y su correo electrónico.
*   **Privacidad (PII):** Los datos sensibles (nombre, dirección de envío, RFC) se guardan encriptados a nivel de aplicación (FastAPI) en PostgreSQL. En local la llave vive en una variable de entorno.
*   **Archivos:** STL, imágenes y constancias fiscales se guardan en disco. El backend es el único que los lee y los escribe.
*   **Panel de Administración:** Aplicación frontend separada (`admin/`) para visualizar órdenes, cambiar estados, descargar STLs y desencriptar datos de envío únicamente al momento de despachar o facturar.

## 4. Estructura del Frontend Público (Vistas)
*   **Inicio:** Hero section impactante, explicación del servicio y propuestas de valor (ideal para estudiantes y cosplayers).
*   **Portafolio / Nuestro Trabajo:**
    *   Galería visual moderna usando CSS Grid.
    *   Filtros dinámicos por categorías: Accesorios, Anime, Cosplay, Hogar, Tecnología, Juguetes.
    *   Botón de llamada a la acción (CTA): "Cotizar algo similar" que redirige al formulario manual.
*   **Cotizador Automático (Núcleo):**
    *   Dropzone para subir archivo `.STL`.
    *   Visor 3D interactivo para previsualización espacial de la pieza.
    *   Selector de calidad (Básica, Media, Premium) y acabado (Crudo, Pintado).
    *   Cálculo dinámico: El archivo se envía al backend, se procesa, y el frontend muestra el Quote ID y el precio estimado. El precio sale del volumen, la calidad y el acabado. Las tarifas son números provisionales en un solo módulo de configuración.
*   **Cotizador Manual:** Formulario para proyectos personalizados, con descripción, imágenes de referencia y un STL opcional. Ese STL no calcula precio. El usuario elige WhatsApp o correo. Por correo deja su email, obligatorio, para que lo contacten de vuelta. Por WhatsApp se abre un mensaje al negocio con el Quote ID y la descripción; el número vive en configuración. Al enviarlo, la pantalla muestra el Quote ID. Este camino no cobra anticipo.
*   **Checkout y Pagos:**
    *   Se abre desde el cotizador automático, con su Quote ID.
    *   Resumen de la orden y opciones elegidas.
    *   Correo, siempre. Nombre y dirección de envío cuando la entrega es nacional.
    *   Selector de entrega: Local (San Juan del Río) o Envío Nacional.
    *   Casilla "¿Requieres factura?" que despliega el RFC y un input para subir el PDF de la Constancia de Situación Fiscal.
    *   Botón de pago con PayPal (cobro del 50% de anticipo) y, al confirmar, el Order ID.
*   **Rastreo de Pedido:** Order ID y correo. Si coinciden, una línea de tiempo con el estado: En cola -> Imprimiendo -> Terminado -> Enviado. Sin otros datos personales.

## 5. Backend (Endpoints Principales)
*   `POST /api/quote/calculate`: Recibe el archivo STL, calcula el volumen, lo guarda en el almacén local y devuelve el Quote ID y el precio estimado.
*   `POST /api/quotes/manual`: Recibe la descripción, las imágenes de referencia, un STL opcional y el canal, WhatsApp o correo. Si el canal es correo, el email es obligatorio y se guarda para la respuesta. Devuelve el Quote ID y, en WhatsApp, el enlace del mensaje. No calcula precio ni cobra.
*   `POST /api/checkout`: Recibe el Quote ID del cotizador automático, verifica el anticipo del 50% con PayPal, encripta los datos personales, guarda el PDF de la constancia en el almacén local y genera el Order ID.
*   `GET /api/orders/{order_id}`: Recibe también el correo. Si coinciden, devuelve el estado de la orden, sin otros datos personales.
*   `GET /api/portfolio`: Sirve la lista de imágenes y categorías para poblar dinámicamente la vista de "Nuestro Trabajo".
*   `PATCH /api/admin/orders/{order_id}`: Endpoint del panel de administración para actualizar estados. Fuera de las seis etapas; llega con `admin/`.

## 6. Notificaciones
*   Notificaciones transaccionales por correo electrónico enviadas desde el backend (Resend o SMTP) al cliente cada vez que el administrador actualice el estado de su orden.
*   En la cotización manual por correo, el backend envía la solicitud al negocio y conserva el email del cliente para responderle. En las etapas locales ese envío pasa por una interfaz falsa; Resend o SMTP lo reemplazan junto con el correo de cambio de estado.

## 7. Orden de construcción

El backlog es este orden, partido en tareas de una sesión. Cada etapa termina con algo que se puede ejecutar. La siguiente reemplaza una pieza y deja el resto en su lugar. No se abre una etapa hasta que la anterior tiene un resultado que se puede correr.

1. **Skeleton.** Carpetas, instalación, una prueba que pasa en el frontend y otra en el backend. Sin comportamiento de producto.
2. **Frontend con mock.** UI real del sitio público. Cada llamada al backend vive en una sola capa de servicio, y esa capa es falsa. El mock devuelve las mismas formas que devolverá la API.
3. **Contrato.** `openapi.yaml` en la raíz del repositorio, sacado de esa capa de servicio. Solo los endpoints que el mock ya llama.
4. **Backend en memoria.** API real que implementa el contrato y nada más. Los datos se pierden al reiniciar, a propósito. Sin base de datos.
5. **Conexión.** El frontend usa el cliente real. El mock se queda en el repo para las pruebas. El almacén sigue en memoria.
6. **Base de datos.** El almacén en memoria se reemplaza por PostgreSQL local y la carpeta de archivos. La UI y el contrato no cambian.

```text
spec
  |
  v
skeleton
  |
  v
frontend + mock
  |
  v
openapi.yaml
  |
  v
backend en memoria
  |
  v
frontend -> cliente real
  |
  v
base de datos
```

Layout de la primera versión:

```text
frontend/          sitio público
backend/           API y sus pruebas
_docs/             plan, proceso, roles
AGENTS.md
openapi.yaml       el contrato, desde la etapa 3
```

`admin/`, el correo al cambiar el estado y GCP no entran en estas seis etapas. En el backlog son las tareas 18, 19 y 20, y se parten en sesiones más chicas antes de implementarlas. La etapa 1 solo crea el esqueleto. El producto empieza en la etapa 2.
