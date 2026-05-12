Este es el plan de ejecución profesional para **Wacamaya Sports**. He estructurado el documento como una hoja de ruta de ingeniería de software, enfocada en escalabilidad, seguridad y rendimiento.

---

# 🏆 Plan de Implementación: Wacamaya Sports

**Proyecto:** E-commerce Multiplataforma de Artículos de Fútbol.
**Stack:** Flutter, Dart, Firebase (Auth, Firestore, Storage), Provider.

---

## 🛠️ Fase 1: Ingeniería de Infraestructura y Entorno

1. **Configuración del Workspace:**
* Uso estricto de **VS Code** con el pack de extensiones de Flutter/Dart.
* Configuración de `analysis_options.yaml` con reglas de linting estrictas para asegurar código limpio.


2. **Ecosistema Firebase:**
* Creación de proyectos en Firebase Console (Desarrollo y Producción).
* Vinculación mediante `FlutterFire CLI` para generar `firebase_options.dart`.


3. **Arquitectura de Software:**
* Implementación de **Clean Architecture** (Capas: Data, Domain, Presentation).
* Estructura de carpetas por *Features* (ej: `features/auth`, `features/shop`, `features/cart`).



## 🎨 Fase 2: Diseño UI/UX y Sistema de Diseño

1. **Atomic Design:** Crear una biblioteca de widgets base (botones, inputs, cards de producto) para mantener consistencia visual.
2. **Tematización (Theming):**
* `ThemeData` personalizado con colores corporativos (Verde césped, Negro deportivo, Blanco).
* Soporte nativo para **Dark Mode**.


3. **Experiencia de Usuario:**
* Skeleton loaders (shimmer effect) para carga de productos.
* Navegación mediante `BottomNavigationBar` persistente.



## 📦 Fase 3: Gestión de Dependencias (pubspec.yaml)

Antes de programar, se deben integrar estas librerías clave:

* **Firebase Core/Auth/Firestore:** Infraestructura backend.
* **Provider:** Inyección de dependencias y gestión de estado.
* **Google Fonts:** Tipografía profesional (ej: Montserrat o Roboto).
* **Cached Network Image:** Optimización de memoria al cargar fotos de jerseys.
* **Flutter SVG:** Para logos de marcas y equipos sin pérdida de calidad.
* **Intl:** Formateo de moneda (MXN/USD) y fechas.

## 🔐 Fase 4: Sistema de Autenticación y Seguridad

1. **Lógica de Acceso:**
* Servicio de Auth independiente que gestione: Registro, Login y Recuperación.
* Validaciones de formularios mediante RegEx (Email válido, contraseña de +8 caracteres).


2. **Persistencia:**
* Uso de `StreamBuilder` para escuchar el estado de la sesión (`authStateChanges`) y redirigir al usuario automáticamente.



## 🗄️ Fase 5: Modelado de Datos (Firestore)

Estructura NoSQL optimizada:

* **Collection `products`:** Documentos con campos: `uid`, `name`, `category` (jersey/tenis), `price`, `stock`, `sizes` (array), `imageUrl`.
* **Collection `users`:** Perfiles con historial de compras y direcciones.
* **Collection `orders`:** Registro de transacciones para el administrador.

## 🚀 Fase 6: Procedimiento de Desarrollo Paso a Paso

1. **Sprint 1 (Base):** Configuración de Firebase y creación de Modelos de Datos (Clases Dart + JSON mapping).
2. **Sprint 2 (Auth):** UI de Login/Registro y vinculación con `FirebaseAuth`.
3. **Sprint 3 (Catálogo):** Implementación de `GridView` para productos con filtrado por categoría (Jerseys, Tenis, etc.).
4. **Sprint 4 (Carrito):** Lógica de `Provider` para sumar/restar productos y calcular totales en tiempo real.
5. **Sprint 5 (Detalle y Perfil):** Vistas de producto con selector de tallas y gestión de datos del usuario.

---



---

### 💡 Consejo Pro de Wacamaya Sports:

Para los **Termos y Mochilas**, asegúrate de que en Firestore el campo `sizes` sea opcional o cambie a `capacity` (litros), ya que la lógica de tallas de un Jersey no aplica a un accesorio. Esto evitará errores de renderizado en la UI.

## 🤖 Prompt Profesional para Generación de Código

Copia y pega el siguiente prompt en tu IA de confianza una vez estés listo para empezar a programar. Está diseñado para obtener resultados de nivel Senior.

> **Prompt:**
> "Actúa como un Desarrollador Senior de Flutter & Dart. Necesito iniciar la codificación de la aplicación 'Wacamaya Sports'. El objetivo es construir la base técnica siguiendo Clean Architecture y utilizando Provider como gestor de estado.
> **Requerimientos técnicos:**
> 1. Configura el archivo `pubspec.yaml` con las dependencias necesarias para: Firebase (Auth y Firestore), Provider, Google Fonts y Cached Network Image.
> 2. Proporciona la estructura de carpetas profesional sugerida (Capas: data, domain, presentation).
> 3. Genera el modelo de datos `Product` en Dart, incluyendo un método `fromFirestore` y `toMap`, con campos para: id, nombre, descripción, precio, categoría (jersey, shorts, tenis, mochilas, termos), imagenUrl y stock.
> 4. Crea un `AuthService` utilizando `firebase_auth` que incluya métodos para login, registro y logout.
> 5. Implementa el `ChangeNotifier` de Provider para manejar el estado de la autenticación en el `main.dart`.
> 
> 
> El código debe ser limpio, con comentarios explicativos, manejo de errores mediante bloques try-catch y siguiendo las mejores prácticas de tipado fuerte en Dart. No generes la UI completa aún, enfócate en la lógica del core y la conexión con Firebase."
