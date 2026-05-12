Esta es la culminación del **SRS (Software Requirements Specification)** para **Wacamaya Sports v4.0**. En este nivel, la documentación no es solo una guía de desarrollo, sino un **Contrato de Arquitectura** que asegura la viabilidad técnica, comercial y operativa del sistema a largo plazo.

Hemos añadido secciones de **Infraestructura de Red**, **Estrategia de DevOps**, **Manejo de Errores a nivel de Kernel de App** y un **Modelo de Escalabilidad Horizontal**.

---

# 🏟️ Especificación Técnica de Ingeniería de Software: Wacamaya Sports v4.0

**Enfoque:** Ecosistema Digital Unificado con Resiliencia Crítica.

---

## 🏛️ 1. Filosofía de Ingeniería y Patrones de Diseño

Para Wacamaya Sports, no basta con que el código "funcione". Debe ser **mantenible por equipos distribuidos**.

### 1.1 Inyección de Dependencias y desacoplamiento

Utilizaremos un contenedor de dependencias (como `GetIt` o `Riverpod`) para separar la interfaz de usuario de la lógica de servicios. Esto permite realizar **Unit Testing** sin necesidad de conectarse a la base de datos real.

### 1.2 Flujo de Datos Unidireccional (UDF)

Para evitar estados inconsistentes (ej. que el carrito muestre un precio y el total otro), implementamos un flujo unidireccional:

1. **Evento:** El usuario presiona "Agregar al carrito".
2. **Estado:** El `CartProvider` procesa la lógica, valida stock y emite un nuevo *Estado Inmutable*.
3. **UI:** La pantalla se reconstruye basándose exclusivamente en ese nuevo estado.

---

## 🗄️ 2. Modelo de Datos Avanzado y Estrategia NoSQL

En Firestore, el costo se mide por lectura/escritura. Nuestra estrategia es la **Agregación de Datos**.

### 2.1 Esquema de Colecciones "High-Performance"

#### A. Colección: `configuracion_global` (Documento Único)

Contiene parámetros dinámicos que la app descarga al iniciar sin consultar toda la base de datos:

* `banner_promocional_url`: String.
* `version_minima_requerida`: String (fuerza la actualización si hay cambios críticos).
* `mantenimiento_activo`: Boolean (bloquea la app en caso de despliegue crítico).

#### B. Colección: `inventario_logico` (Subcolección de Productos)

Separamos el stock de la descripción para evitar lecturas innecesarias:

* `id_variante`: String (SKU).
* `stock_reservado`: Int (artículos en carritos activos).
* `stock_disponible`: Int (existencia física menos reservados).

---

## 🛡️ 3. Seguridad, Cumplimiento y Middleware

### 3.1 Cifrado y Tránsito

* **SSL Pinning:** Para las versiones de iOS y Android, implementamos SSL Pinning para evitar ataques de *Man-in-the-Middle* en redes Wi-Fi públicas (estadios, centros comerciales).
* **Ofuscación de Código:** Durante el build de producción para Windows (.exe), se aplicará ofuscación de símbolos para dificultar la ingeniería inversa.

### 3.2 Firebase Cloud Functions (El "Backend" Invisible)

Lógica que **nunca** debe estar en el celular del cliente:

* **`onOrderCreated`**: Al crear un pedido, esta función se dispara, valida el pago con la API de Stripe/PayPal y genera la factura en PDF enviándola por correo automáticamente.
* **`cleanTempCarts`**: Tarea programada (Cron Job) que libera el stock de carritos abandonados por más de 24 horas.

---

## 🚀 4. DevOps y Ciclo de Vida del Software (SDLC)

### 4.1 Entornos de Ejecución

1. **Development (DEV):** Sandbox para programadores.
2. **Staging (STG):** Réplica exacta de producción para pruebas de QA y demos.
3. **Production (PRD):** Entorno final con escalado automático.

### 4.2 Estrategia de Despliegue (Blue-Green Deployment)

Al lanzar una nueva versión de Wacamaya Sports en la Web, no reemplazamos el código viejo de inmediato. Mantenemos ambas versiones corriendo y desviamos el 10% del tráfico a la nueva para asegurar que no haya errores (Canary Releases).

---

## 📉 5. Plan de Contingencia y Observabilidad (SRE)

### 5.1 Monitoreo de Errores (Sentry/Crashlytics)

Cada vez que la app falle en la computadora de un cliente en Windows, recibiremos un log detallado que incluye:

* Especificaciones de hardware (RAM, CPU).
* Traza del error (Stacktrace).
* Últimas 5 acciones del usuario antes del crash.

---

## 🤖 Master Prompt Nivel "Software Architect & Product Owner"

Este prompt es el más completo hasta la fecha. Úsalo para generar el esqueleto lógico de toda la plataforma:

> "Genera la arquitectura base para **Wacamaya Sports v4.0**, una solución de e-commerce de alto rendimiento.
> **Especificaciones de Código:**
> 1. **Patrón de Arquitectura:** Clean Architecture dividida en `Data`, `Domain` y `Presentation`.
> 2. **Modelado de Datos (Dart):** Crea entidades inmutables usando `freezed` o `equatable`. Define el modelo `Product` con soporte para `variantes` (talla/color) y el modelo `Order` con un `StatusEnum` (Pendiente, Pagado, Enviado).
> 3. **Gestión de Estado Pro:** Configura un `MultiProvider` que incluya un `ThemeProvider` (para cambio dinámico de Verde Wacamaya a Modo Oscuro) y un `InventoryProvider` que use `Streams` para actualizar el stock en tiempo real.
> 4. **Capa de Servicio (API):** Crea una clase abstracta `BaseRepository` y su implementación `FirebaseRepository`. Incluye manejo de errores personalizado con una clase `AppException`.
> 5. **UI Responsiva:** Diseña un Widget `WacamayaScaffold` que detecte automáticamente si la app corre en un monitor 4K (Windows) o en un iPhone SE, ajustando los márgenes y el tamaño de la tipografía Oswald.
> 6. **Internacionalización básica:** Configura el soporte para español (es) como idioma nativo.
> 
> 
> **Resultado esperado:** Estructura de archivos completa, `pubspec.yaml` optimizado, y la implementación del `AuthService` con manejo de persistencia de sesión."

---

### 💡 Próximos Pasos Sugeridos:

1. **Módulo de Analítica:** ¿Quieres que desgloce cómo medir el ROI (Retorno de Inversión) dentro de la app?
2. **Módulo de Logística:** ¿Deseas la integración técnica con APIs de paquetería (FedEx/DHL) para el rastreo en tiempo real?
3. **Código UI:** ¿Prefieres que empiece a generar el código de la pantalla de inicio con el efecto de "Parallax" en los jerseys?
