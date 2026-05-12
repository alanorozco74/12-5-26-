Como creador de software senior, he rediseñado el plan maestro de **Wacamaya Sports** para que sea un documento de especificación técnica de alto nivel. Este plan está diseñado para ser la base de una aplicación escalable, con estándares de arquitectura empresarial, optimizada para **Antigravity / VS Code** y con soporte total multiplataforma.

---

# 📝 Documento de Especificación Técnica: Wacamaya Sports

**Versión:** 2.0 (Enterprise Ready)

**Entorno de ejecución:** Multiplataforma (Android, iOS, Windows Desktop, Web)

**Enfoque:** Clean Architecture & Reactive State Management

---

## 🛠️ 1. Infraestructura y Configuración de Entorno

Para garantizar la paridad entre plataformas (que el código se comporte igual en Windows que en Android), estableceremos un núcleo sólido:

* **SDK:** Flutter 3.22+ (o la versión estable más reciente en 2026) con canal `stable`.
* **Motor de Renderizado Web:** `CanvasKit` para asegurar que las sombras y animaciones se vean fluidas en navegadores.
* **Windows Runner:** Configuración de C++ en VS Code para compilación nativa de escritorio.
* **Linter:** Configuración de `analysis_options.yaml` con `package:flutter_lints` para prohibir el uso de `dynamic` y forzar tipos de datos estrictos.

---

## 📂 2. Arquitectura de Carpetas (Layered Architecture)

Dividiremos el código en módulos para que el mantenimiento no sea una pesadilla cuando la tienda crezca:

```text
lib/
├── core/                        # Núcleo de la aplicación
│   ├── theme/                   # Gestión de colores (Wacamaya Palette) y fuentes
│   ├── constants/               # Endpoints, rutas de navegación y strings
│   └── utils/                   # Validadores de formularios y formateadores
├── data/                        # Capa de infraestructura (Acceso a datos)
│   ├── models/                  # Clases Dart con mapeo en ESPAÑOL
│   ├── repositories/            # Implementación de los contratos de Firebase
│   └── sources/                 # Servicios directos (FirebaseAuth, Firestore)
├── domain/                      # Lógica de negocio (Contratos y Entidades)
│   ├── entities/                # Objetos limpios (POJOs)
│   └── repositories/            # Interfaces para inversión de dependencias
└── presentation/                # Capa visual y de estado
    ├── providers/               # Lógica reactiva (Autenticacion, Tienda, Carrito)
    ├── screens/                 # Vistas adaptativas (Mobile/Desktop)
    └── widgets/                 # Componentes atómicos (Botones, Cards)

```

---

## 🗄️ 3. Ingeniería de Datos: Tablas Firestore (Nomenclatura en Español)

### A. Colección: `usuarios`

Gestión de identidad y permisos.

| Campo | Tipo | Función |
| --- | --- | --- |
| `id_usuario` | String (UID) | Llave primaria ligada a Firebase Auth. |
| `nombre_completo` | String | Nombre para etiquetas de envío. |
| `correo` | String | Email de contacto y facturación. |
| `direccion_envio` | Map | {calle, ciudad, codigo_postal, estado}. |
| `rol` | String | "cliente" o "administrador" (acceso a stock). |
| `fecha_registro` | Timestamp | Auditoría de creación. |

### B. Colección: `productos`

Inventario centralizado.

| Campo | Tipo | Función |
| --- | --- | --- |
| `id_producto` | String | ID autogenerado para cada SKU. |
| `nombre` | String | Nombre comercial (Ej: Jersey Wacamaya Home). |
| `descripcion` | String | Características, telas y tecnologías. |
| `categoria` | String | jerseys, shorts, tenis, mochilas, termos. |
| `precio` | Double | Valor unitario en moneda local. |
| `existencia` | Int | Stock físico disponible. |
| `tallas` | Array | [ "S", "M", "L" ] o [ "26", "27", "28" ]. |
| `imagenes` | Array | Lista de URLs de Firebase Storage. |

### C. Colección: `pedidos`

Historial de transacciones y logística.

| Campo | Tipo | Función |
| --- | --- | --- |
| `id_pedido` | String | ID de seguimiento único. |
| `id_cliente` | String | Referencia al `id_usuario`. |
| `articulos` | List | Sub-objeto: {id_producto, cantidad, talla}. |
| `total_pago` | Double | Sumatoria total de la orden. |
| `estado_envio` | String | pendiente, procesando, enviado, entregado. |
| `fecha_compra` | Timestamp | Timestamp generado por el servidor. |

---

## 🧠 4. Sistema de Gestión de Estado (Providers)

Utilizaremos `MultiProvider` en el punto de entrada para desacoplar la UI de la lógica:

1. **`AutenticacionProvider`**: Maneja el ciclo de vida de la sesión. Escucha cambios en tiempo real de Firebase Auth y actualiza la UI automáticamente si el usuario cierra sesión.
2. **`FirestoreProvider`**: Es la capa de servicio. Contiene los métodos asíncronos para leer productos, crear pedidos y actualizar el inventario.
3. **`TiendaProvider`**: Lógica de navegación del catálogo. Filtra productos por categoría (Jerseys, Tenis, etc.) y gestiona la búsqueda.
4. **`CarritoProvider`**: Lógica puramente aritmética. Calcula subtotales, IVA, costos de envío y valida que no se agreguen más productos de los que hay en `existencia`.

---

## 🎨 5. Diseño Visual y Tematización (Multiplataforma)

### Paleta de Colores Corporativa

Configurada en `lib/core/theme/colores.dart`:

* **Verde Wacamaya (`primary`):** `#00C853` - Representa el campo de juego y dinamismo.
* **Negro Carbón (`background`):** `#121212` - Elegancia y alto contraste para pantallas OLED.
* **Blanco Hielo (`surface`):** `#FFFFFF` - Tipografía y elementos limpios.
* **Naranja Balón (`secondary`):** `#FF6D00` - Botones de acción y notificaciones (badges).

### Adaptabilidad (Responsiveness)

* **Móvil (Android/iOS):** Vista de lista o grid de 2 columnas. Navegación mediante `BottomNavigationBar`.
* **Windows/Web:** Vista expandida de 4 a 6 columnas. Navegación mediante `NavigationRail` lateral para aprovechar el ancho de pantalla.

---

## 📦 6. Dependencias Estratégicas (`pubspec.yaml`)

```yaml
dependencies:
  flutter:
    sdk: flutter
  
  # Backend & Auth
  firebase_core: ^2.32.0
  firebase_auth: ^4.20.0
  cloud_firestore: ^4.17.5
  
  # Gestión de Estado
  provider: ^6.1.2
  
  # UI & Recursos
  google_fonts: ^6.2.1
  cached_network_image: ^3.4.0
  flutter_svg: ^2.0.10
  intl: ^0.19.0
  font_awesome_flutter: ^10.7.0
  
  # Utilidades
  logger: ^2.4.0          # Para depuración profesional en consola
  uuid: ^4.4.0            # Generación de IDs locales si es necesario

```

---

## 🚀 7. Master Prompt Profesional (Uso Final)

Copia y pega este prompt en tu entorno de generación para obtener el código base con los más altos estándares:

> "Actúa como un **Ingeniero de Software Senior experto en Flutter**. Genera la arquitectura base para la aplicación **'Wacamaya Sports'** que debe ser compatible con **Android, iOS, Windows y Web**.
> **Especificaciones de Código:**
> 1. **Arquitectura:** Aplica **Clean Architecture** separando estrictamente en carpetas: data, domain, presentation y core.
> 2. **Base de Datos:** Implementa modelos de datos en Dart con campos en **ESPAÑOL** basados en las tablas: `usuarios`, `productos` (categorías: jerseys, shorts, tenis, mochilas, termos) y `pedidos`.
> 3. **Estado:** Configura un `MultiProvider` en `main.dart` que integre: `AutenticacionProvider`, `FirestoreProvider` y `CarritoProvider`.
> 4. **Multiplataforma:** Asegúrate de que la inicialización de Firebase sea compatible con Windows y Web.
> 5. **Tema:** Crea una clase `AppColores` con Verde (#00C853) y Negro (#121212).
> 6. **Calidad:** Usa tipado estricto, evita 'dynamic', implementa manejo de errores asíncronos con try-catch y formatea el código según los estándares oficiales de Dart. No incluyas analíticas.
> 
> 
> **Entregables:** El archivo `pubspec.yaml`, la estructura de archivos sugerida y la lógica del `main.dart` con la inyección de providers."

¿Deseas que profundice en la **lógica de las reglas de seguridad de Firestore** o prefieres que genere el **modelo de datos del Producto** con sus validaciones?
