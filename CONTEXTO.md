# Contexto del proyecto ConstruAliado

## Propósito

ConstruAliado es un prototipo de tienda web de materiales de construcción para Guatemala. La aplicación tiene un catálogo público y una sección de administración de inventario. Este documento resume su estado actual para orientar los siguientes cambios sin tener que volver a recorrer todo el HTML.

## Archivos y tecnologías

- `index.html`: contiene la estructura, estilos CSS, interfaz y lógica JavaScript de la aplicación.
- `assets/construaliado-logo.png`: logo utilizado en la pantalla de login, la barra superior y el pie de página.
- `CONTEXTO.md`: contexto funcional y técnico del proyecto.
- HTML, CSS y JavaScript nativo; Bootstrap 5.3.3 y Bootstrap Icons se cargan desde CDN.
- No hay servidor, API ni base de datos remota.
- La pantalla inicial requiere autenticación local. En el primer inicio se configura una cuenta administradora.

## Funcionalidad actual

### Tienda pública

- Catálogo de 63 productos en cuatro categorías: obra gris, electricidad, plomería y herramientas.
- Búsqueda por nombre y filtros por categoría.
- Muestra precio de venta cuando está definido; de lo contrario, muestra un precio de referencia estimado en quetzales (Q).
- Las referencias incluyen unidad y son orientativas: pueden variar según marca, medida, zona y flete; no son cotizaciones en tiempo real.
- Imágenes de categoría ilustrativas por defecto. El administrador puede cargar una foto propia para un producto.

### Administración

- Agregar, editar y eliminar productos.
- Incluye una pestaña de pedidos realizados, con búsqueda y detalle de datos de facturación y líneas del pedido. Solo se muestra dentro de la vista de administración.
- Incluye un módulo local de usuarios: crear cuentas, asignar acceso a Tienda/Inventario/Pedidos/Usuarios, editar permisos y restablecer contraseña con una clave temporal.
- Las cuentas nuevas y las que reciben una contraseña restablecida deben cambiarla en su siguiente inicio de sesión.
- Desde el login se puede recuperar el acceso olvidado restableciendo las cuentas locales; después se crea un administrador nuevo. Esto elimina todos los usuarios y permisos, pero conserva productos, carrito y pedidos. No verifica identidad y solo es apropiado para la demostración local.
- Cada producto puede tener nombre, categoría, precio de venta, existencias, unidad e imagen.
- El formulario ofrece precio de referencia para los productos iniciales y permite copiarlo al campo del precio de venta.
- Las imágenes admitidas son JPG, PNG y WebP, con límite de 8 MB antes de procesar. Se redimensionan y comprimen como JPEG para almacenarlas.
- La administración requiere iniciar sesión en esta demostración local, pero no ofrece protección de acceso real.

### Carrito / Mi pedido

- Permite agregar materiales, ver imágenes, cambiar cantidades, quitar productos y revisar cantidad, precio unitario y total por línea.
- El formulario de registro local incluye número correlativo de comprobante, empresa/NIT de empresa y cliente/NIT/dirección/teléfono.
- La dirección de entrega se ingresa como texto en el campo del cliente; no se solicita ubicación, coordenadas ni permisos del navegador.
- Al registrar la compra, el público ve un modal de confirmación con sus datos de facturación y el detalle del pedido. Es un comprobante interno imprimible, no una factura fiscal.
- El detalle de la venta y sus datos se guardan en el navegador; los productos sin precio de venta guardan su referencia por separado y no se suman al total confirmado.
- Para productos iniciales sin precio configurado se permite agregar a cotización mostrando su precio de referencia.
- Para productos con precio y existencias definidos, el carrito limita la cantidad a las existencias registradas.
- El registro es local y de demostración: no cobra, no envía datos a un negocio y no produce una factura fiscal.

## Persistencia actual

La aplicación guarda automáticamente en `localStorage` del navegador:

| Clave | Contenido |
|---|---|
| `construaliado_products_v1` | Lista de productos, precios de venta, existencias, unidades e imágenes cargadas. |
| `construaliado_cart_v1` | Cantidades de productos del carrito. |
| `construaliado_sales_v1` | Ventas locales de demostración, sus datos de empresa/cliente y líneas de producto. |
| `construaliado_users_v1` | Usuarios, hash PBKDF2 de contraseña, permisos por módulo y estado de cambio obligatorio. |
- `sessionStorage` usa `construaliado_session_v1` para mantener la sesión en la pestaña actual; al cerrar la pestaña se solicita iniciar sesión otra vez.

- En la primera carga se siembra el catálogo inicial y se guarda en `construaliado_products_v1`.
- Los cambios del inventario y del carrito se escriben en `localStorage`.
- Al recargar, los datos se recuperan y los precios de referencia de los 63 productos iniciales se completan desde el catálogo base.
- Se validan los datos leídos; si hay contenido inválido o falla la lectura, se registra el error en consola y se muestra un aviso.
- Si el almacenamiento se llena, el guardado de productos muestra un aviso; las imágenes están optimizadas para reducir el uso de espacio.
- `localStorage` es local al navegador, perfil y origen/dispositivo. No sincroniza información entre equipos. Borrar los datos del navegador puede eliminar los datos guardados.
- Contraseñas del prototipo se guardan como hash PBKDF2 con sal aleatoria; no se guarda texto plano. El cliente y la base de permisos siguen siendo modificables por quien controla el navegador.

## Modelo de producto

Los productos iniciales y creados usan estos campos principales:

```text
id, name, category, price, stock, unit, referencePrice, referenceUnit, image
```

- `price`: precio de venta configurado en administración.
- `referencePrice` y `referenceUnit`: guía referencial para los materiales precargados; productos nuevos no tienen referencia configurada.
- `image`: imagen del producto en formato Data URL después de optimizarla; vacío si no se cargó una imagen propia.
- `stock`: existencias; en el catálogo inicial permanece sin definir hasta que se administre.

Las ventas incluyen número `CA-######`, fecha, empresa (nombre/NIT), cliente (nombre/NIT/dirección/teléfono), líneas con nombre/cantidad/precio unitario/total y total de venta. Ventas históricas guardadas anteriormente pueden incluir coordenadas, pero ya no se usan en la aplicación.

## Límites y decisiones para futuras tareas

- Mantener HTML, CSS y JavaScript dentro de `index.html`, salvo que se acuerde separar archivos.
- Usar `localStorage` para continuar el prototipo local. Para una tienda multiusuario, sincronizada o con pedidos reales será necesario añadir backend y almacenamiento remoto.
- No tratar los precios de referencia como precios verificados o actuales. Si se requiere exactitud comercial, definir fuentes, fecha de consulta, marca, presentación y ubicación.
- No presentar la simulación del carrito como pago o compra confirmada.
- La pestaña de pedidos solo está oculta en la interfaz pública; como el proyecto no tiene backend ni autenticación, no es un área privada segura. No usar para almacenar datos reales sensibles ni publicar el sitio como sistema de producción sin implementar autenticación y almacenamiento protegido.
- El login/administración de usuarios es solo una demostración escolar en `localStorage`; no protege la aplicación frente a inspección o modificación del navegador. La seguridad real requiere backend, sesiones de servidor y almacenamiento de credenciales protegido.
- La recuperación de administrador también es local y no es una recuperación segura: cualquier persona con acceso al navegador puede borrar las cuentas y crear otra.
- Las dependencias CDN e imágenes ilustrativas requieren conexión a Internet.
