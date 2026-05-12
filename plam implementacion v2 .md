Entendido. Aquí tienes el **Plan de Implementación Maestro** consolidado en un solo bloque de texto técnico, estructurado para ser copiado y utilizado directamente como tu guía de ingeniería en **Antigravity / VS Code**.

---

# 🏆 Plan Maestro: Wacamaya Sports

**Objetivo:** Aplicación E-commerce de artículos de fútbol (Jerseys, Tenis, Accesorios).
**Plataformas:** Android, iOS, Windows, Web.
**Stack:** Flutter + Firebase + Provider.

---

## 📂 1. Estructura de Proyecto (Clean Architecture)

Para garantizar que la app funcione igual en Windows que en un móvil, usaremos esta estructura:

* **`lib/core/`**: Configuración global, constantes y **manejo de colores**.
* **`lib/data/`**: Modelos con campos en español y repositorios de Firebase.
* **`lib/domain/`**: Entidades lógicas y contratos.
* **`lib/presentation/`**: La capa de UI (Screens y Widgets) y los **Providers**.

---

## 🗄️ 2. Estructura de Tablas (Firestore en Español)

### A. Tabla: `usuarios`

* `id_usuario`: (String) UID de Firebase Auth.
* `nombre_completo`: (String) Nombre del cliente.
* `correo`: (String) Email de registro.
* `direccion_envio`: (Map) `{ calle, ciudad, codigo_postal }`.
* `rol`: (String) "cliente" o "administrador".

### B. Tabla: `productos`

* `id_producto`: (String) ID autogenerado.
* `nombre`: (String) Nombre del artículo.
* `categoria`: (String) jerseys, shorts, tenis, mochilas, termos.
* `precio`: (Double) Costo unitario.
* `existencia`: (Int) Stock disponible.
* `tallas`: (Array) Opciones disponibles (Ej: ["M", "G", "27"]).

### C. Tabla: `pedidos`

* `id_pedido`: (String) ID de transacción.
* `id_cliente`: (String) Relación con el usuario.
* `articulos`: (Array) `[{ id_producto, cantidad, precio_unitario }]`.
* `total_pago`: (Double) Monto final.
* `estado_envio`: (String) "pendiente", "enviado", "entregado".

---

## 🧠 3. Arquitectura de Providers (Gestión de Estado)

Centralizaremos la lógica en 4 Providers principales inyectados en el `main.dart`:

1. **`AutenticacionProvider`**: Controla el estado de sesión (login/registro) y persiste el usuario en Android/iOS/Web/Windows.
2. **`FirestoreProvider`**: Encargado de leer y escribir en las tablas de Firebase con métodos asíncronos en español.
3. **`TiendaProvider`**: Maneja el catálogo, los filtros por categoría y la búsqueda de productos.
4. **`CarritoProvider`**: Gestiona la lógica de suma, resta y cálculo de totales de la compra actual.

---

## 🎨 4. Identidad Visual y Colores (Theme)

Definiremos un archivo `colores.dart` con la siguiente paleta deportiva:

* **Principal:** `#00C853` (Verde Wacamaya - Energía).
* **Fondo:** `#121212` (Negro Carbón - Elegancia).
* **Contraste:** `#FFFFFF` (Blanco Hielo - Claridad).
* **Acento:** `#FF6D00` (Naranja Balón - Acción).

---

## 🚀 5. Procedimiento de Desarrollo Paso a Paso

1. **Configuración Inicial:** Crear el proyecto Flutter y habilitar plataformas con `flutter config --enable-windows-desktop --enable-web`.
2. **Vínculo Firebase:** Usar FlutterFire CLI para generar la configuración de las 4 plataformas.
3. **Definición de Modelos:** Crear las clases `Usuario`, `Producto` y `Pedido` con sus métodos `deMapa` y `aMapa`.
4. **Inyección de Providers:** Envolver el `MaterialApp` en un `MultiProvider`.
5. **Desarrollo de UI Responsive:** Crear el `Home` usando `LayoutBuilder` (Móvil: Listado vertical; Windows/Web: Grid de 4-5 columnas).
6. **Lógica de Negocio:** Conectar los botones de compra con el `CarritoProvider` y finalizar con `FirestoreProvider`.

---

## 📦 6. Dependencias Estándar (`pubspec.yaml`)

```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^2.30.0
  firebase_auth: ^4.19.0
  cloud_firestore: ^4.17.0
  provider: ^6.1.2
  google_fonts: ^6.2.1
  cached_network_image: ^3.3.1
  intl: ^0.19.0 # Para precios y fechas

```

---

## 🤖 Prompt Profesional para Generar el Código

> "Actúa como un desarrollador Flutter Senior. Genera el código base para la aplicación **'Wacamaya Sports'** compatible con Android, iOS, Web y Windows.
> **Especificaciones:**
> 1. Implementa **Clean Architecture** (carpetas: data, domain, presentation, core).
> 2. Usa estrictamente nombres en español para los campos de las tablas: **usuarios, productos, pedidos**.
> 3. Configura el `main.dart` con un **MultiProvider** que incluya: `AutenticacionProvider`, `FirestoreProvider` y `CarritoProvider`.
> 4. Define una clase de colores global con: Verde (#00C853) y Negro (#121212).
> 5. Crea el modelo de datos para `Producto` con: `id_producto`, `nombre`, `precio`, `categoria`, `tallas` y `existencia`.
> 6. El código debe ser limpio, sin analíticas, con tipado estricto y manejo de errores (try-catch)."
> 
> 

¿Deseas que comencemos generando el archivo **`main.dart`** con esta configuración de Providers o el **modelo de datos** del catálogo?
