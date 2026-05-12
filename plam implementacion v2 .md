Esta es la **Específicación Técnica Definitiva de Ingeniería de Software** para **Wacamaya Sports**. Este documento no es solo un plan, es el manual de arquitectura que define cómo se construye una aplicación de clase mundial, robusta, escalable y mantenible para el año 2026.

---

# 🏟️ Documento de Arquitectura y Especificación: Wacamaya Sports

**Proyecto:** Sistema de Comercio Electrónico Deportivo Multiplataforma.
**Ingeniero Responsable:** Gemini (Software Architect Mode).
**Ecosistema:** Flutter SDK 3.x | Firebase Suite | Dart 3.x (Null-Safety Estricto).

---

## 🏛️ 1. Paradigma de Arquitectura: Clean Architecture & DDD Lite

Para que la aplicación soporte cambios de tecnología en el futuro (ej. cambiar Firebase por Supabase o una API propia en Rust), el código se divide en capas con **Inversión de Dependencias**.

### 1.1 Capa de Dominio (Domain Layer)

Es la capa más pura. No tiene dependencias externas. Contiene la "verdad del negocio".

* **Entidades (Entities):** Clases simples en español (`Producto`, `Usuario`, `Pedido`, `CarritoItem`).
* **Repositorios (Interfaces):** Contratos que dicen *qué* debe hacer la base de datos (ej. `consultarProductos()`), pero no *cómo*.
* **Casos de Uso (Use Cases):** Lógica específica, por ejemplo: `CalcularDescuentoPorVolumen` o `VerificarDisponibilidadStock`.

### 1.2 Capa de Datos (Data Layer)

Donde vive la implementación técnica.

* **Modelos (Models):** Clases que extienden a las entidades y añaden métodos `deMapa()` y `aMapa()` para hablar con **Cloud Firestore**.
* **Data Sources:** Clientes específicos de Firebase.
* **Mapeadores:** Transforman los datos crudos de la red en objetos de dominio.

### 1.3 Capa de Presentación (Presentation Layer)

Lo que el usuario toca y ve.

* **Providers (State Management):** Gestores de estado reactivos que notifican a la UI cuando hay cambios.
* **Vistas Adaptativas:** Código que detecta si estás en un **Windows Surface**, un **iPad** o un **Android** y ajusta el layout automáticamente.

---

## 🗄️ 2. Diseño de Base de Datos (Cloud Firestore)

Utilizaremos un modelo **NoSQL orientado a documentos**, optimizado para lectura rápida y baja latencia en redes móviles.

### 2.1 Tabla/Colección: `usuarios`

| Atributo | Tipo | Función Técnica |
| --- | --- | --- |
| `id_usuario` | String (UID) | Llave Primaria vinculada a FirebaseAuth. |
| `nombre_completo` | String | Almacena el nombre legal del cliente. |
| `correo_electronico` | String | Identificador único de comunicación. |
| `rol_usuario` | Enum | "cliente", "vendedor", "administrador". |
| `preferencias` | Map | Almacena tallas usuales y equipos favoritos. |
| `fecha_creacion` | Timestamp | Metadato de auditoría del sistema. |

### 2.2 Tabla/Colección: `productos`

Diseñada para un catálogo deportivo dinámico (Jerseys, Tenis, Mochilas, etc.).

| Atributo | Tipo | Función Técnica |
| --- | --- | --- |
| `id_producto` | String | Identificador alfanumérico único (SKU). |
| `nombre_articulo` | String | Título comercial (ej: "Jersey Wacamaya Retro"). |
| `descripcion_larga` | String | Soporta formato Markdown para descripciones detalladas. |
| `categoria_id` | String | Indexado para filtrado rápido (tenis, mochilas). |
| `precio_actual` | Double | Valor monetario con precisión decimal. |
| `stock_disponible` | Int | Contador de inventario en tiempo real. |
| `galeria_imagenes` | List | URLs de Firebase Storage con optimización WebP. |
| `especificaciones` | Map | Atributos dinámicos (Material, Peso, Tecnología). |

### 2.3 Tabla/Colección: `pedidos`

| Atributo | Tipo | Función Técnica |
| --- | --- | --- |
| `id_pedido` | String | Referencia de rastreo única. |
| `referencia_cliente` | String | Llave foránea al UID del usuario. |
| `lista_articulos` | List | Snapshot de productos, precios y tallas al comprar. |
| `monto_total` | Double | Sumatoria final (incluye impuestos y envío). |
| `estado_logistica` | String | "procesando", "enviado", "entregado", "cancelado". |
| `timestamp_pago` | Timestamp | Registro de la confirmación de la pasarela. |

---

## 🎨 3. Design System & UX Multiplataforma

Wacamaya Sports no es solo una app; es una identidad visual deportiva.

### 3.1 Paleta de Colores (Wacamaya Branding)

* **Primary (Verde Estadio):** `#00C853` - Usado para Call-to-Actions (Comprar, Confirmar).
* **Background (Negro Noche):** `#121212` - Base visual para resaltar los colores de los jerseys.
* **Surface (Gris Carbón):** `#1E1E1E` - Para cards de producto y menús elevados.
* **Accent (Naranja Fuego):** `#FF6D00` - Para ofertas relámpago y badges de notificación.

### 3.2 Adaptabilidad de Interfaz

* **Mobile (Android/iOS):** Navegación por pulgar, gestos de swipe para borrar del carrito.
* **Desktop (Windows):** Atajos de teclado (Ctrl+F para buscar), menús desplegables y vista multicanal.
* **Web:** Optimización SEO, carga perezosa (lazy loading) de imágenes y compatibilidad con todos los navegadores modernos.

---

## 📦 4. Stack de Dependencias (pubspec.yaml)

Seleccionamos solo librerías de alto rendimiento y mantenimiento activo:

```yaml
dependencies:
  flutter:
    sdk: flutter
  
  # INFRAESTRUCTURA FIREBASE
  firebase_core: ^2.32.0         # Núcleo de conexión
  firebase_auth: ^4.20.0         # Seguridad y Usuarios
  cloud_firestore: ^4.17.5       # Base de datos en tiempo real
  firebase_storage: ^11.6.0      # Almacenamiento de imágenes de alta calidad

  # GESTIÓN DE ESTADO (PROVIDER)
  provider: ^6.1.2               # Inyección de dependencias y reactividad

  # INTERFAZ Y RECURSOS
  google_fonts: ^6.2.1           # Tipografía Oswald y Urbanist
  cached_network_image: ^3.4.0   # Caché de imágenes para ahorro de datos
  flutter_svg: ^2.0.10           # Iconos vectoriales de marcas
  intl: ^0.19.0                  # Localización de moneda y fechas
  font_awesome_flutter: ^10.7.0  # Iconografía deportiva extra

  # UTILIDADES DE DESARROLLO
  logger: ^2.4.0                 # Debugging profesional en consola
  uuid: ^4.4.0                   # Generación de identificadores únicos locales

```

---

## 🚀 5. Procedimiento de Implementación Paso a Paso

1. **Fase 0: Boilerplate Multiplataforma:** Configuración de los runners nativos de Windows y habilitación de Firebase Web.
2. **Fase 1: Capa Core:** Implementación del sistema de colores, temas oscuros y utilidades de red en `lib/core`.
3. **Fase 2: Modelado de Datos:** Escritura de los modelos en Dart con soporte para los campos en español definidos.
4. **Fase 3: Firebase Auth & Usuarios:** Creación del `AutenticacionProvider` y flujo de persistencia de sesión.
5. **Fase 4: Catálogo Firestore:** Implementación del `TiendaProvider` con lógica de "Streams" para actualizaciones en vivo de jerseys y tenis.
6. **Fase 5: Lógica del Carrito:** Desarrollo del `CarritoProvider` con persistencia en disco local (para que no se pierda al cerrar la app).
7. **Fase 6: UI Adaptativa:** Construcción de pantallas usando `LayoutBuilder` y `OrientationBuilder`.
8. **Fase 7: Seguridad:** Configuración de las "Security Rules" en Firebase para proteger los datos de los usuarios.

---

## 🤖 Prompt Maestro de Ingeniería (Uso en Antigravity)

Este prompt es la síntesis de todo el conocimiento anterior. Úsalo para que la IA genere el código más limpio posible:

> "Actúa como un **Arquitecto de Software Senior**. Genera la base de código para la aplicación e-commerce **'Wacamaya Sports'** compatible con **Android, iOS, Windows y Web**.
> **Requerimientos Obligatorios:**
> 1. **Arquitectura:** Estructura de carpetas bajo **Clean Architecture** (core, data, domain, presentation).
> 2. **Base de Datos:** Implementa modelos en Dart con campos en **ESPAÑOL** (usuarios, productos, pedidos) según la especificación técnica.
> 3. **Gestión de Estado:** Configura el `main.dart` utilizando `MultiProvider` que incluya: `AuthProvider`, `FirestoreProvider` y `CarritoProvider`.
> 4. **Catálogo:** El `FirestoreProvider` debe gestionar las categorías: jerseys, shorts, tenis, mochilas y termos.
> 5. **Tema:** Clase `AppTheme` con colores primarios Verde (#00C853) y Fondo Negro (#121212).
> 6. **Calidad:** Código profesional, tipado estricto, manejo de errores asíncronos con try-catch, sin analíticas y con comentarios técnicos.
> 
> 
> **Entregables:** Pubspec.yaml, estructura de directorios, modelos de datos y el esqueleto funcional de los Providers y el Main."

¿Deseas que desgloce la **lógica matemática interna del CarritoProvider** (cálculo de impuestos y descuentos) o prefieres que genere las **reglas de seguridad de Firebase** para proteger la tabla de productos?
