# Plan de Desarrollo - Plataforma de Impresión 3D

## 1. Visión General del Proyecto
Desarrollar una plataforma web profesional para ofrecer servicios de impresión 3D bajo demanda en San Juan del Río, Querétaro, con alcance de envíos a todo México. El sistema debe permitir cotizaciones automáticas basadas en el análisis de archivos STL y cotizaciones manuales para proyectos complejos. Se integrará un flujo de pagos para cobrar el 50% de anticipo y un sistema seguro de gestión de órdenes.

## 2. Stack Tecnológico
*   **Frontend (Público y Admin):** React, Vite, Tailwind CSS, shadcn/ui.
*   **Visor 3D Frontend:** `react-three-fiber` o `three.js`.
*   **Backend:** Python con FastAPI.
*   **Procesamiento 3D (Backend):** `trimesh` o `numpy-stl` (para calcular el volumen del STL de forma segura en el servidor).
*   **Base de Datos:** PostgreSQL (alojado en GCP Cloud SQL).
*   **Almacenamiento y Nube:** Google Cloud Platform (GCP Cloud Storage para archivos STL y Constancias Fiscales, Secret Manager o Cloud KMS para llaves de encriptación).
*   **Pagos:** Integración con API de PayPal.

## 3. Arquitectura y Seguridad
*   **Gestión de Usuarios (Guest Mode):** No hay registro ni login para clientes. Se usa un `Quote ID` para la etapa de cotización que transiciona a un `Order ID` cuando se confirma el pago. Los clientes rastrean su pedido usando su `Order ID` y su correo electrónico.
*   **Privacidad (PII):** Los datos sensibles (nombre, dirección de envío, RFC) se guardan encriptados a nivel de aplicación (FastAPI) en la base de datos PostgreSQL.
*   **Panel de Administración:** Aplicación frontend completamente separada para visualizar órdenes, cambiar estados, descargar STLs y desencriptar datos de envío únicamente al momento de despachar o facturar.

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
    *   Cálculo dinámico: El archivo se envía al backend, se procesa, y el frontend muestra el precio estimado.
*   **Cotizador Manual:** Formulario limpio para proyectos personalizados, permitiendo subir imágenes de referencia y describir el proyecto.
*   **Checkout y Pagos:**
    *   Resumen de la orden y opciones elegidas.
    *   Selector de entrega: Local (San Juan del Río) o Envío Nacional.
    *   Casilla "¿Requieres factura?" que despliega un input para subir el PDF de la Constancia de Situación Fiscal.
    *   Botón de pago con PayPal (Cobro automático del 50% de anticipo).
*   **Rastreo de Pedido:** Input simple para ingresar el `Order ID` y visualizar una línea de tiempo (timeline) con el estado: En cola -> Imprimiendo -> Terminado -> Enviado.

## 5. Backend (Endpoints Principales)
*   `POST /api/quote/calculate`: Recibe el archivo STL, calcula el volumen, lo guarda en un bucket temporal de GCP y devuelve el precio estimado.
*   `POST /api/checkout`: Procesa el pago de PayPal, encripta datos PII, guarda el PDF de la constancia en el bucket, mueve el STL a un bucket definitivo y genera el `Order ID`.
*   `GET /api/orders/{order_id}`: Devuelve el estado actual de la orden (sin exponer PII).
*   `GET /api/portfolio`: Sirve la lista de imágenes y categorías para poblar dinámicamente la vista de "Nuestro Trabajo".
*   `PATCH /api/admin/orders/{order_id}`: Endpoint protegido para uso exclusivo del Panel de Administración para actualizar estados.

## 6. Notificaciones
*   Notificaciones transaccionales por correo electrónico enviadas desde el backend (ej. usando `aiosmtplib` o Resend) al cliente cada vez que el administrador actualice el estado de su orden.

## 7. Flujo de Trabajo para Cursor AI
1.  **Fase 1:** Inicializar proyecto Vite con React, configurar Tailwind CSS y shadcn/ui. Construir la UI base (Layout, Navbar, Footer y la galería estática del Portafolio).
2.  **Fase 2:** Construir el componente del Cotizador Automático (Dropzone) y el Visor 3D utilizando `react-three-fiber`.
3.  **Fase 3:** Configurar el entorno backend con FastAPI, conectar a PostgreSQL (local o GCP) y crear la lógica de cálculo volumétrico de STLs usando `trimesh`.
4.  **Fase 4:** Conectar Frontend y Backend para completar el flujo del cotizador automático.
5.  **Fase 5:** Integrar la pasarela de pago (PayPal), el sistema de encriptación de datos PII y la conexión con GCP Storage.
6.  **Fase 6:** Construir la aplicación separada para el Panel de Administración.