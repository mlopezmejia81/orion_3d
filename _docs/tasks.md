# Tareas

El backlog activo son los issues de https://github.com/mlopezmejia81/orion_3d. Este archivo ya no manda. El número de la tarea es el número del issue.

Las tareas 1 a 17 caben en una sesión. Las tareas 18, 19 y 20 van después de las seis etapas y se parten en sesiones más chicas antes de implementarlas. No se abre la siguiente hasta que la anterior se puede ejecutar.

## 1. Esqueleto con una prueba que pasa
Goal: Un proyecto vacío instala y pasa una prueba en el frontend y otra en el backend.
Description: Crear `frontend/` con React, Vite y TypeScript, y `backend/` con FastAPI, cada uno con una prueba que pasa y sin pantallas ni endpoints de producto. Escribir en `AGENTS.md` los comandos reales para instalar, correr y probar cada lado. No añadir PostgreSQL, cotizador ni estilos de producto.

## 2. Cascarón, inicio y servicio falso
Goal: El sitio arranca, muestra el inicio y concentra todas las llamadas en un mock.
Description: En `frontend/`, instalar Tailwind y shadcn/ui, y construir layout, navbar, footer, la página de inicio (hero, servicio y propuesta para estudiantes y cosplayers) y las rutas vacías de portafolio, cotizador automático, cotizador manual, checkout y rastreo. Crear una sola capa de servicio, con mock y sin red, para portafolio, `POST /api/quote/calculate` (Quote ID y precio), `POST /api/quotes/manual`, `POST /api/checkout` (Order ID) y `GET /api/orders/{order_id}`. Probar el mock sin servidor.

## 3. Portafolio
Goal: La galería se filtra por categoría y lleva al formulario manual.
Description: Construir la vista con CSS Grid a partir del mock, con filtros de Accesorios, Anime, Cosplay, Hogar, Tecnología y Juguetes. El botón "Cotizar algo similar" abre el cotizador manual. Probar el filtro y la lista vacía sin servidor.

## 4. Cotizador automático
Goal: Un STL válido devuelve Quote ID y precio, todavía sin visor 3D.
Description: En el cotizador automático, el dropzone acepta solo `.stl`, y la calidad (Básica, Media, Premium) y el acabado (Crudo, Pintado) llaman a la capa de servicio. La pantalla muestra el Quote ID y el precio del mock, y ofrece seguir al checkout con ese Quote ID; un archivo inválido muestra error y no calcula. Probar el archivo válido, el inválido y las opciones vacías, sin servidor.

## 5. Visor 3D
Goal: La pieza del cotizador automático se puede girar antes de seguir al pago.
Description: Mostrar el STL cargado con `react-three-fiber`, con órbita, en la pantalla del cotizador automático. Si el archivo no se puede dibujar, la pantalla avisa y conserva el Quote ID y el precio que ya devolvió el servicio. Probar un STL dibujable y uno ilegible, sin servidor.

## 6. Cotizador manual
Goal: Una idea queda registrada con un Quote ID, sin cobro, y se manda por WhatsApp o por correo.
Description: El formulario pide la descripción, permite imágenes y un STL opcional, y exige elegir WhatsApp o correo; por correo el email es obligatorio para responderle, y por WhatsApp el servicio devuelve el enlace al negocio con el Quote ID y la descripción. El STL no calcula precio y este camino no cobra. Probar el correo con email, el rechazo si falta el email o la descripción, y el enlace de WhatsApp, sin servidor.

## 7. Checkout del anticipo
Goal: La cotización automática termina en un Order ID con el 50% simulado.
Description: La pantalla llega con el Quote ID, resume opciones y precio, pide correo, ofrece entrega local en San Juan del Río o envío nacional (nombre y dirección), y al marcar factura pide RFC y el PDF de la constancia. El botón de PayPal llama solo al servicio; el mock confirma el anticipo y devuelve el Order ID, sin contactar a PayPal. Probar factura, entrega nacional y el Order ID, sin servidor.

## 8. Rastreo de pedido
Goal: Order ID y correo muestran el estado, y un par incorrecto no muestra nada personal.
Description: El formulario consulta la capa de servicio y, si coinciden, marca el estado actual en la línea En cola, Imprimiendo, Terminado y Enviado. Si no coinciden, dice que no hay orden y no muestra nombre, dirección, RFC ni correo. Probar coincidencia, correo distinto e id desconocido, sin servidor.

## 9. Sistema de diseño
Goal: Las reglas visuales de las pantallas públicas quedan escritas.
Description: A partir de las pantallas ya construidas, escribir `_docs/design-system.md` con color, tipografía, espaciado y los componentes que el sitio ya usa, y enlazarlo desde `AGENTS.md`. No cambiar las pantallas en esta tarea.

## 10. Contrato OpenAPI
Goal: `openapi.yaml` en la raíz describe solo lo que el mock ya llama.
Description: Leer la capa de servicio de `frontend/` y especificar método, ruta, cuerpo, respuesta y errores de portafolio, cálculo de cotización, cotización manual, checkout (con la confirmación de pago) y consulta de orden por Order ID y correo. No incluir el panel admin, GCP ni endpoints que el mock no usa. Si el mock y `_docs/plan.md` no coinciden, corregir el spec antes de dar por cerrado el archivo.

## 11. API en memoria para portafolio y orden
Goal: Esas dos lecturas responden desde un almacén que se vacía al reiniciar.
Description: Implementar en `backend/` únicamente esas operaciones de `openapi.yaml`, en módulos, con almacén en memoria y portafolio sembrado. Las pruebas cubren la lista, la orden que coincide y la que no; reiniciar el proceso borra los datos. No añadir PostgreSQL ni el cálculo del STL.

## 12. Cálculo de volumen y precio
Goal: El precio automático sale del volumen del STL.
Description: Implementar `POST /api/quote/calculate` con `trimesh`, guardar el archivo en memoria y devolver Quote ID y precio según calidad y acabado. Las tarifas provisionales viven en un solo módulo de configuración. Probar un STL de volumen conocido, un archivo que no es STL y que un reinicio borra la cotización.

## 13. Checkout y cotización manual en memoria
Goal: El anticipo simulado y la solicitud manual dejan registros que el rastreo puede leer.
Description: Implementar esas dos operaciones del contrato sobre el almacén en memoria. En la cotización manual guardar el canal y, si es correo, el email de respuesta, y avisar por una interfaz de correo falsa; en el checkout guardar correo, entrega y, cuando vengan, nombre, dirección, RFC y la referencia al PDF. El Order ID se crea solo después de que una interfaz de pago falsa confirme el 50%, sin llamar a PayPal, sin enviar un correo real y sin escribir en disco o PostgreSQL.

## 14. PayPal sandbox del anticipo
Goal: El servidor real confirma el 50% con PayPal antes de crear la orden.
Description: Cambiar la implementación de pago del proceso que corre por la del sandbox, con credenciales en variables de entorno, sin modificar `openapi.yaml` ni el frontend. Las pruebas siguen usando el proveedor falso y no contactan a PayPal. Sin credenciales en el entorno, el servidor que corre localmente conserva el proveedor falso.

## 15. Conectar el frontend a la API
Goal: Cotizar, pagar y rastrear funciona en el navegador contra el backend en memoria.
Description: Sustituir el mock de la capa de servicio por el cliente HTTP real y dejar el mock en el repo para las pruebas sin servidor. El cliente real pide la aprobación de PayPal sandbox y después llama a los endpoints del contrato; la URL de la API y el id público de PayPal van en variables de entorno. No cambiar la API ni añadir base de datos.

## 16. PostgreSQL y cifrado
Goal: Un reinicio conserva cotizaciones y órdenes, con los datos personales cifrados.
Description: Reemplazar el almacén en memoria por PostgreSQL local, detrás de la misma interfaz, con la conexión en una variable de entorno. Cifrar nombre, dirección y RFC en la aplicación antes de guardarlos; la llave va en otra variable de entorno. Las pruebas no dependen de una base en la máquina, y esta tarea no cambia el contrato, el frontend ni mueve los archivos a disco.

## 17. Archivos en disco
Goal: Un reinicio conserva los STL, las imágenes de referencia y las constancias.
Description: Guardar esos archivos en una carpeta configurada por variable de entorno y persistir en la base solo la referencia; el backend es el único que lee y escribe esa carpeta. Las pruebas no dependen de archivos que ya estuvieran en la máquina. La UI y el contrato se quedan igual.

## 18. Panel de administración
Goal: Una app separada lista las órdenes, cambia su estado y descarga los archivos.
Description: Crear `admin/` con React, Vite, TypeScript, Tailwind y shadcn/ui. Desde ahí se ven las órdenes, se actualiza el estado con `PATCH /api/admin/orders/{order_id}`, se descarga el STL y se descifran nombre, dirección y RFC solo al despachar o facturar. El endpoint queda protegido y el sitio público no cambia.

## 19. Correo al cambiar el estado
Goal: El cliente recibe un correo cada vez que el admin actualiza su orden.
Description: Al completar el PATCH de estado, el backend envía un correo con Resend o SMTP al correo de la orden, con el Order ID y el estado nuevo. El mismo proveedor envía la cotización manual al negocio y usa el email del cliente como dirección de respuesta. Las pruebas no envían un correo real.

## 20. Paso a GCP
Goal: Archivos, base y llave de cifrado salen de la máquina sin cambiar el producto.
Description: Cloud Storage reemplaza la carpeta de archivos, Cloud SQL reemplaza PostgreSQL local y Secret Manager o Cloud KMS reemplaza la llave de cifrado del entorno. Se hace cuando cotizar, pagar y rastrear ya funcionan en local. La interfaz de almacén y el contrato se quedan igual.
