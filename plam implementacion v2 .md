Como creador de software senior, entiendo que para un proyecto de la magnitud de **Wacamaya Sports**, la documentación debe ser exhaustiva para evitar "deuda técnica" y garantizar que cualquier desarrollador pueda unirse al equipo y entender la lógica en minutos.

Aquí tienes la **Especificación Técnica Extendida y Profesional**, diseñada para un entorno de alto rendimiento.

---

# 🏆 Especificación Técnica Maestra: Wacamaya Sports

**Estatus:** Documento de Arquitectura de Software (SAD)
**Entorno de Desarrollo:** VS Code / Antigravity con Flutter SDK 3.x
**Arquitectura:** Clean Architecture (Domain-Driven Design Lite)

---

## 🏛️ 1. Filosofía de Arquitectura: "The Clean Way"

Para que Wacamaya Sports sea escalable en **Windows, Web y Móvil**, dividimos el sistema en círculos concéntricos de dependencia:

### A. Capa de Dominio (El Corazón)

Es la capa más interna. No sabe nada de Firebase, de la web o de Windows.

* **Entidades:** Objetos de negocio puros en español (`Producto`, `Usuario`, `Pedido`).
* **Casos de Uso:** La lógica de lo que hace la app (ej. `RealizarCompra`, `ConsultarStock`).
* **Interfaces de Repositorio:** Contratos que definen qué datos necesitamos, sin decir de dónde vienen.

### B. Capa de Datos (La Infraestructura)

Aquí es donde ocurre la magia de **Firestore**.

* **Modelos:** Extensiones de las entidades que incluyen `deMapa()` y `aMapa()` para hablar con Firebase.
* **Repositorios Implementados:** El código real que usa `cloud_firestore` para traer los jerseys o termos.
* **Manejo de Errores:** Conversión de códigos de error de Firebase a excepciones personalizadas de Wacamaya Sports.

### C. Capa de Presentación (La Interfaz)

* **Providers (State Management):** Los "cerebros" de las pantallas. Transforman flujos de datos en estados de UI (Cargando, Éxito, Error).
* **Widgets Adaptativos:** Componentes que cambian su forma. En Windows usan `Hover` y en Móvil usan `LongPress`.

---

## 🗄️ 2. Diccionario de Datos (Firestore - Nomenclatura Estándar)

He expandido los campos para cubrir necesidades reales de un e-commerce profesional:

### 2.1 Colección: `usuarios`

| Campo | Tipo | Restricción | Descripción |
| --- | --- | --- | --- |
| `id_usuario` | String | Unique, PK | UID de Firebase Authentication. |
| `nombre_completo` | String | Not Null | Nombre legal del cliente. |
| `correo` | String | Unique | Email verificado. |
| `telefono` | String | Optional | Para contacto de mensajería (envíos). |
| `direccion_envio` | Map | Required | `{calle, ext, int, colonia, ciudad, cp, estado}`. |
| `rol` | String | Enum | `['cliente', 'vendedor', 'admin']`. |
| `favoritos` | List | Index | IDs de productos que el usuario marcó. |
| `fecha_registro` | Timestamp | Server side | Fecha de creación del perfil. |

### 2.2 Colección: `productos`

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `id_producto` | String | ID alfanumérico único. |
| `nombre` | String | Título comercial del artículo. |
| `sku` | String | Código de inventario para almacén. |
| `descripcion` | String | Formato enriquecido (Markdown) de características. |
| `categoria` | String | `jerseys`, `shorts`, `tenis`, `mochilas`, `termos`. |
| `etiquetas` | List | `['oferta', 'nuevo', 'edición limitada']`. |
| `precio` | Double | Precio base de venta. |
| `descuento` | Double | Porcentaje de descuento (0.0 a 1.0). |
| `existencia` | Int | Stock físico total sincronizado. |
| `variantes` | List | `{talla: 'M', color: 'Verde', stock: 10}`. |
| `imagenes` | List | URLs de Firebase Storage (WebP optimizado). |

---

## 🧠 3. Gestión de Estado Avanzada (Logic & Providers)

Para Wacamaya Sports, no basta con mostrar datos; hay que gestionar el estado de forma profesional:

1. **`AutenticacionProvider`**:
* **Estado:** `SinAutenticar`, `Autenticando`, `Autenticado`, `Error`.
* **Lógica:** Gestiona el token de sesión y la redirección automática a la pantalla de bienvenida o a la tienda.


2. **`FirestoreProvider` (Gestor de Datos)**:
* Implementa **Streams** para que, si tú cambias el precio de un termo en la consola de Firebase, el usuario lo vea cambiar en su pantalla de Windows o Android al instante sin refrescar.


3. **`CarritoProvider`**:
* **Persistencia:** Guarda el carrito en `LocalStorage` (usando `shared_preferences`) para que si el usuario cierra la app en su Mac/Windows, sus productos sigan ahí al volver.
* **Validación:** Consulta la `existencia` antes de permitir el pago final.



---

## 🎨 4. Design System: Wacamaya Visual Identity

Definiremos un `ThemeData` robusto que use los colores deportivos solicitados:

* **Paleta Primaria:** Verde Esmeralda (`0xFF00C853`). Usado en botones de "Comprar" y éxitos.
* **Paleta de Fondo:** Negro Profundo (`0xFF121212`). Fondo de la app para resaltar las imágenes de los jerseys.
* **Acentos de Alerta:** Naranja Intenso (`0xFFFF6D00`). Para liquidaciones y stock bajo.
* **Tipografía:**
* **Headlines:** *Oswald* (Bold) para ese look de revista deportiva.
* **Body:** *Roboto* o *Urbanist* para legibilidad en descripciones largas.



---

## 🚀 5. Procedimiento de Desarrollo Paso a Paso (Full Stack)

1. **Fase 1: Bootstrap Multiplataforma**:
* Configurar `flutter create` con soporte para `web`, `windows`, `android` e `ios`.
* Configurar el soporte nativo de Windows (Visual Studio Build Tools).


2. **Fase 2: Capa de Datos (Modelado)**:
* Crear clases en Dart con `json_serializable`.
* Implementar métodos `deMapa()` para recibir de Firestore y `aMapa()` para enviar.


3. **Fase 3: Implementación de Providers**:
* Configurar `MultiProvider` en la raíz.
* Crear los servicios de Auth y Firestore.


4. **Fase 4: UI Adaptativa (Responsive)**:
* Crear el `LayoutBuilder` maestro.
* Diseñar la "Card de Producto" que sea táctil en móviles y con efectos de `hover` (ratón) en Windows/Web.


5. **Fase 5: Seguridad y Reglas de Firestore**:
* Escribir las reglas: `allow read: if true; allow write: if request.auth != null && get(/databases/$(database)/documents/usuarios/$(request.auth.uid)).data.rol == 'admin';`



---

## 🤖 Prompt Maestro Definitivo (Senior Engineer Level)

Copia este prompt para obtener el código estructural completo:

> "Actúa como un **Principal Software Architect** experto en Flutter. Necesito la base técnica de **'Wacamaya Sports'**, un e-commerce multiplataforma.
> **Instrucciones Estrictas:**
> 1. **Estructura:** Implementa **Clean Architecture** con carpetas: `core`, `data`, `domain`, `presentation`.
> 2. **Base de Datos:** Crea modelos Dart con campos en **ESPAÑOL** (id_producto, nombre, precio, existencia, categoria, tallas) para las tablas `usuarios`, `productos` y `pedidos`.
> 3. **Estado:** Genera el archivo `main.dart` inyectando mediante `MultiProvider`: `AutenticacionProvider`, `FirestoreProvider` y `CarritoProvider`.
> 4. **Providers:** El `FirestoreProvider` debe manejar métodos CRUD en español para obtener productos por categoría (jerseys, shorts, tenis, mochilas, termos).
> 5. **UI & Colores:** Crea una clase `WacamayaTheme` con colores primarios Verde (#00C853) y Fondo Negro (#121212). Implementa un widget de navegación adaptativo para Windows (lateral) y Móvil (inferior).
> 6. **Calidad:** Código robusto, manejo de errores asíncronos, sin analíticas y siguiendo las guías de estilo de Google.
> 
> 
> **Entrega:** Estructura de carpetas, `pubspec.yaml`, modelos de datos en español y el esqueleto funcional de los Providers."

¿Deseas que añada la sección de **Estrategia de Despliegue (CI/CD)** para Windows y Web o pasamos directamente a generar los **modelos de datos**?
