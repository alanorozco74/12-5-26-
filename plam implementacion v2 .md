Entendido. He ajustado el diseño de la base de datos para que todos los campos y la lógica interna de **Wacamaya Sports** estén en español, manteniendo el estándar profesional de nomenclatura para facilitar el desarrollo en **Antigravity / VS Code**.

Aquí tienes el plan de datos y el procedimiento actualizado para tu entorno multiplataforma.

---

# 📊 Estructura de Tablas (Colecciones Firestore)

### 1. Tabla: `usuarios` (Información de Clientes)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id_usuario` | String (PK) | ID único de Firebase Auth. |
| `nombre_completo` | String | Nombre y apellido del usuario. |
| `correo` | String | Email registrado. |
| `direccion_envio` | Map | Objeto con: `calle`, `ciudad`, `codigo_postal`. |
| `rol` | String | "cliente" o "administrador". |
| `fecha_registro` | Timestamp | Fecha en que creó la cuenta. |

### 2. Tabla: `productos` (Inventario Deportivo)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id_producto` | String (PK) | ID autogenerado. |
| `nombre` | String | Ejemplo: "Jersey Selección Mexicana 2026". |
| `descripcion` | String | Especificaciones de la prenda o accesorio. |
| `categoria` | String | jerseys, shorts, tenis, mochilas, termos. |
| `precio` | Double | Costo en moneda local. |
| `existencia` | Int | Cantidad de unidades en bodega. |
| `tallas` | Array | Lista de opciones (Ch, M, G / 25, 26, 27). |
| `imagenes` | Array | Lista de links de Firebase Storage. |

### 3. Tabla: `pedidos` (Ventas Realizadas)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id_pedido` | String (PK) | ID de transacción. |
| `id_cliente` | String (FK) | Relación con el usuario que compró. |
| `articulos` | Array | Detalle: `id_producto`, `cantidad`, `precio_unitario`, `talla_elegida`. |
| `total_pago` | Double | Monto final cobrado. |
| `estado_envio` | String | "pendiente", "enviado", "entregado". |
| `fecha_compra` | Timestamp | Momento exacto de la transacción. |

---

# 🚀 Procedimiento de Desarrollo Actualizado (Multiplataforma)

### Fase 1: Núcleo y Multiplataforma

* **Configuración Inicial:** Preparar el proyecto para soportar Web (CanvasKit), Windows (C++ nativo), Android (Kotlin) e iOS (Swift).
* **Estandarización:** Crear una carpeta `core/` donde se definan los colores oficiales (Verde Wacamaya) y tipografías (Oswald/Roboto) para que la marca se vea igual en una PC que en un iPhone.

### Fase 2: Lógica de Datos en Español

* **Mapeo de Modelos:** Crear las clases en Dart asegurando que los métodos `deMapa()` y `aMapa()` coincidan exactamente con los nombres de los campos de las tablas en español mencionadas arriba.
* **Seguridad Firestore:** Escribir las reglas de acceso prohibiendo la edición de la tabla `productos` a cualquier usuario que no tenga el `rol: "administrador"`.

### Fase 3: Interfaz de Usuario Adaptativa (Responsive)

* **Vistas Dinámicas:** Usar un sistema de rejilla (Grid) que muestre 2 columnas en móviles y hasta 5 columnas en la versión de Windows o Web.
* **Gestión de Estado:** Implementar el `CarritoProvider` para que, si un usuario agrega un termo desde la Web, la información se mantenga consistente durante la sesión.

---

# 🤖 Prompt Profesional Final (Optimizado para Antigravity)

Este es el prompt que debes usar para que la IA genere el código con los campos en español y la estructura profesional:

> "Actúa como un desarrollador experto en Flutter. Genera el código base para la aplicación 'Wacamaya Sports' siguiendo una arquitectura limpia (Clean Architecture).
> **Especificaciones Técnicas:**
> 1. **Tablas en Español:** Usa estrictamente los nombres de campos en español para los modelos de datos: `usuarios`, `productos` (con categorías: jerseys, shorts, tenis, mochilas, termos) y `pedidos`.
> 2. **Multiplataforma:** El código debe ser compatible con Android, iOS, Web y Windows. Asegúrate de incluir la inicialización de Firebase adaptable.
> 3. **Gestión de Estado:** Implementa `Provider`. Necesito un `AutenticacionProvider` para manejar el ingreso de usuarios y un `TiendaProvider` para gestionar el catálogo de productos desde Firestore.
> 4. **Seguridad y Estándares:** No utilices analíticas. Usa validaciones de tipos estrictos y manejo de excepciones (try-catch) en las peticiones a la base de datos.
> 5. **UI:** Crea el modelo de la clase `Producto` con los campos: `id_producto`, `nombre`, `precio`, `categoria`, `tallas` (lista) y `existencia`.
> 
> 
> Entrégame el archivo `pubspec.yaml`, la estructura de carpetas sugerida y el código del modelo de datos junto con el `main.dart` configurado con los Providers."

---
