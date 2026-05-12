Excelente. Has planteado los cimientos de un sistema de élite. Ahora, tomemos ese **Contrato de Arquitectura Técnica** y lo elevemos a un **Blueprint de Ejecución de Misión Crítica**. Vamos a expandir cada capa con directrices ejecutables, casos de borde resueltos de antemano y una estrategia de caos controlado.

Aquí tienes la **expansión definitiva** del manifiesto, transformando principios en acciones concretas y abundante detalle técnico.

---

# 🧠 Manifiesto de Ingeniería de Grado Empresarial: Wacamaya Sports v4.0 (Expansión Total)

**Documento:** Especificación de Requisitos de Software (SRS) – **Nivel Platino**

**Versión:** 4.0.1-Enterprise

**Audiencia:** Arquitectos de Software, Líderes Técnicos, SREs, Desarrolladores Senior.

## 🌟 0. Visión Ejecutiva del Sistema

Wacamaya Sports v4.0 no es una aplicación de comercio electrónico convencional. Es un **ecosistema transaccional en tiempo real**, diseñado para manejar picos de demanda de 10,000 solicitudes por segundo durante el lanzamiento de un jersey edición limitada, con una tolerancia a fallos de partición (P en el teorema CAP) que prioriza la disponibilidad y la experiencia del usuario sobre la consistencia fuerte en microsegundos. El sistema operará bajo el lema: *"El usuario siempre ve el producto, aunque el backend esté en llamas"*.

---

## 🏛️ 1. Filosofía de Ingeniería: De la Teoría a la Práctica Quirúrgica

### 1.1 Clean Architecture Hexagonal – Expansión del Andamiaje

La capa de Dominio no puede conocer los detalles de infraestructura. Para enforced esto, usamos **paquetes de Dart modulares** dentro del monorepo:

```text
lib/
├── domain/               # CERO dependencias externas (ni Flutter, ni Firebase)
│   ├── entities/
│   ├── repositories/     # (interfaces abstractas)
│   ├── use_cases/        # lógica de negocio pura
│   └── value_objects/    # Email, PhoneNumber, PositivePrice (con validación en el constructor)
├── data/
│   ├── datasources/      # remote (Firebase) y local (Hive/Isar)
│   ├── repositories/     # implementaciones concretas de las interfaces del dominio
│   └── models/           # anotaciones @freezed, @JsonSerializable
├── presentation/
│   ├── providers/        # Riverpod o BLoC (inyección de casos de uso)
│   ├── pages/
│   └── widgets/
└── core/
    ├── di/               # inyección de dependencias (get_it + injectable)
    ├── network/          # clientes http, interceptores, manejo de tokens
    ├── errors/           # clases AppException, Failure, Either<Failure, Success>
    └── utils/
```

**Ejemplo de Value Object con validación estricta (Domain Layer):**
```dart
// lib/domain/value_objects/positive_price.dart
import 'package:equatable/equatable.dart';

class PositivePrice extends Equatable {
  final double value;
  
  PositivePrice(this.value) {
    if (value <= 0) throw ArgumentError('El precio debe ser positivo: $value');
    if (value > 1_000_000) throw ArgumentError('Precio fuera de rango permitido');
  }

  @override
  List<Object?> get props => [value];
}
```

### 1.2 Gestión de Estado - Más Allá del BLoC Básico

No solo usaremos BLoC, sino que implementaremos el patrón **Event Sourcing** para el carrito de compras. Cada acción (agregar, eliminar, actualizar cantidad) es un evento inmutable que se almacena en una lista local. Esto permite:
- **Time Travel Debugging** en desarrollo.
- **Reintentos inteligentes:** Si falla la sincronización con Firestore, el evento se guarda en una cola local y se reintenta con backoff exponencial.
- **Deshacer/Rehacer** nativo en la UI.

**Estado del Carrito (freezed + EventSourcing):**
```dart
@freezed
class CartState with _$CartState {
  const factory CartState.initial() = _Initial;
  const factory CartState.loading() = _Loading;
  const factory CartState.success({
    required List<CartItem> items,
    required List<CartEvent> eventLog, // historial de eventos
    @Default(false) bool isSyncing,
  }) = _Success;
  const factory CartState.error({required String message}) = _Error;
  const factory CartState.updating({required String itemId}) = _Updating;
}
```

---

## 🗄️ 2. Estrategia de Datos: Persistencia, Consistencia Eventual y Guerra de Latencia

### 2.1 Denormalización Inteligente – Esquema Concreto para Firestore

La colección `users/{userId}` tendrá este documento (simplificado pero real):

```json
{
  "uid": "abc123",
  "email": "fan@wacamaya.com",
  "displayName": "Leonel Fanático",
  "lastPurchasesPreview": [  // array fijo de 3 elementos (denormalización)
    { "id": "ord_001", "total": 89.99, "date": "2025-01-15T20:00:00Z" }
  ],
  "cartSummary": {  // cache denormalizado del carrito
    "itemCount": 2,
    "totalAmount": 179.98,
    "lastUpdated": "2025-01-20T10:30:00Z"
  },
  "preferences": {
    "favoriteTeam": "Águilas Reales",
    "mfaEnabled": false
  }
}
```

**Reglas de seguridad de Firestore (mínimas pero letales):**
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
      allow read: if request.auth.uid == userId;
      allow write: if request.auth.uid == userId 
                  && request.resource.data.cartSummary.itemCount is int
                  && request.resource.data.cartSummary.totalAmount is double;
    }
    match /products/{productId} {
      allow read: if true;  // productos públicos en caché
      allow write: if request.auth.token.isAdmin == true;
    }
  }
}
```

### 2.2 Caché de Nivel 2 – Estrategia Offline-First con Isar

Usaremos **Isar** (base de datos NoSQL ultrarrápida para Flutter) en modo **híbrido**:
- **Al iniciar:** La UI se suscribe a un Stream de Isar (carga instantánea).
- **En segundo plano:** Firestore escucha cambios y actualiza Isar mediante `onSnapshot`.
- **En modo offline:** Todas las escrituras van a una cola FIFO persistente. Al recuperar la conexión, se reproduce la cola con un mecanismo de **idempotencia** para evitar duplicados.

**Ejemplo de cola offline:**
```dart
class OfflineQueue {
  final List<Map<String, dynamic>> _pendingOps = [];
  
  void enqueue(String operation, Map<String, dynamic> data, {String? idempotencyKey}) {
    _pendingOps.add({
      'op': operation,
      'data': data,
      'idempotencyKey': idempotencyKey ?? uuid.v4(),
      'timestamp': DateTime.now().toIso8601String(),
    });
    _persistToIsar();
  }
  
  Future<void> replay() async {
    for (var op in _pendingOps) {
      try {
        await _executeWithRetry(op, maxRetries: 3);
      } catch (e) {
        _moveToDeadLetterQueue(op, error: e.toString());
      }
    }
  }
}
```

---

## 🛡️ 3. Seguridad Perimetral – Defensa en Profundidad

### 3.1 Custom Claims – Permisos a Nivel de Token

En una Cloud Function (o backend propio), asignamos claims personalizados:

```javascript
// Firebase Cloud Function - onUserCreate
exports.addAdminClaim = functions.auth.user().onCreate(async (user) => {
  if (user.email === 'admin@wacamaya.com') {
    await admin.auth().setCustomUserClaims(user.uid, { isAdmin: true, tier: 'gold' });
  } else {
    await admin.auth().setCustomUserClaims(user.uid, { tier: 'bronze' });
  }
});
```

Luego en la app:
```dart
final claims = await FirebaseAuth.instance.currentUser?.getIdTokenResult();
final isAdmin = claims?.claims?['isAdmin'] == true;
if (isAdmin) showAdminDashboard();
```

### 3.2 SSL Pinning con Certificados en Tiempo de Compilación

Usamos `dio` + `dio_cookie_manager` + un **interceptor personalizado** que valida el certificado contra un hash SHA-256 incrustado en el binario:

```dart
class PinningInterceptor extends Interceptor {
  @override
  void onRequest(RequestOptions options, RequestInterceptorHandler handler) {
    // El pinning real se configura en el HttpClientAdapter
    (options.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate = (client) {
      client.badCertificateCallback = (cert, host, port) {
        // Compara el hash del certificado con el hardcodeado
        final pem = cert.pem;
        final sha256 = sha256.convert(utf8.encode(pem)).toString();
        return sha256 == 'EB:5A:...'; // hash del certificado esperado
      };
    };
    handler.next(options);
  }
}
```

---

## 🚀 4. Ecosistema DevOps – Entrega Continua con Gobernanza

### 4.1 Terraform – Infraestructura Reproducible

Un fragmento real de `main.tf` para Firebase y Cloud Run:

```hcl
resource "google_firebase_project" "wacamaya" {
  project = var.project_id
}

resource "google_firestore_database" "database" {
  project     = var.project_id
  name        = "(default)"
  location_id = "nam5" # multi-región
  type        = "FIRESTORE_NATIVE"
}

resource "google_cloud_run_service" "algolia_sync" {
  name     = "wacamaya-algolia-sync"
  location = "us-central1"
  
  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/algolia-sync:latest"
        env {
          name  = "ALGOLIA_APP_ID"
          value = var.algolia_app_id
        }
      }
    }
  }
}
```

### 4.2 Pipeline CI/CD – Flujo Detallado con GitHub Actions

Un `.github/workflows/release.yml` completo incluye:

```yaml
name: Wacamaya CD Pipeline

on:
  push:
    branches: [ main, release/* ]
  pull_request:
    branches: [ main ]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: subosito/flutter-action@v2
        with:
          flutter-version: '3.16.x'
      - run: flutter pub get
      - run: dart run custom_lint  # linting severo
      - run: flutter test --coverage --test-randomize-ordering-seed=random
      - run: dart run dependency_validator  # detecta dependencias no usadas

  build-and-deploy-android:
    if: github.ref == 'refs/heads/release/production'
    runs-on: macos-latest  # necesita macOS para firmar
    steps:
      - run: flutter build appbundle --release
      - uses: r0adkll/upload-google-play@v1
        with:
          serviceAccountJson: ${{ secrets.PLAY_CONSOLE_SA }}
          releaseFile: build/app/outputs/bundle/release/app-release.aab
          track: internal
```

---

## 📉 5. Observabilidad – SRE en la Práctica

### 5.1 Instrumentación con Sentry + Firebase Performance + Datadog

Cada repositorio emite **spans** personalizados:

```dart
final span = tracer.startSpan('fetch_product_list');
try {
  final products = await firestore.collection('products').get();
  span.setAttribute('product_count', products.docs.length);
} catch (e) {
  span.setStatus(StatusCode.error, description: e.toString());
  throw AppException('No se pudo cargar inventario: $e');
} finally {
  span.end();
}
```

### 5.2 SLOs y Alertas – Ejemplo Concreto

- **SLO 1 (Latencia de Checkout):** 99% de peticiones < 2.5s.
- **SLO 2 (Disponibilidad de búsqueda):** 99.9% de respuestas exitosas de Algolia.
- **Error Budget:** 0.1% de downtime mensual permitido (43.8 minutos).
- **Alerta crítica:** Si el error budget se consume al 80% → el equipo de guardia (on-call) recibe un SMS.

**Cloud Function de verificación de SLO:**
```javascript
exports.verifyLatencySLO = functions.pubsub.schedule('every 5 minutes').onRun(async () => {
  const recentOps = await firestore.collection('performance_logs')
      .where('timestamp', '>', Date.now() - 3600000)
      .get();
  let slowRequests = recentOps.docs.filter(doc => doc.data().duration > 2500).length;
  let percentSlow = slowRequests / recentOps.size * 100;
  if (percentSlow > 1.0) {  // más del 1% superan 2.5s
    await sendAlertToPagerDuty(`SLO violado: ${percentSlow}% de requests lentos`);
  }
});
```

---

## 🤖 Master Prompt Evolucionado – Instrucción para IA de Nivel Principal

> **"Eres un Principal Software Engineer con 15 años de experiencia en sistemas de alta concurrencia. Vas a generar el esqueleto completo de Wacamaya Sports v4.0. Requisitos:**
>
> **1. Estructura de módulos:** Usa un enfoque de **Sharding por funcionalidad** (`auth/`, `cart/`, `checkout/`, `inventory/`, `profile/`). Cada módulo tiene su propia capa de dominio, datos y presentación (Diseño basado en características).
>
> **2. Patrón de estado:** Implementa **Riverpod 2.0** con `StateNotifier` y `AsyncValue`. El carrito debe ser un `StateNotifier<CartState>` que emite eventos a un `Repository` que maneja la fuente de la verdad (Firestore + Isar).
>
> **3. Modelos con `freezed` y `json_serializable`:** Incluye validaciones de negocio en el factory constructor (ej. `Product.stock` no puede ser menor a 0). Genera también extensiones `CopyWith` para actualizaciones inmutables.
>
> **4. Manejo de errores global:** Crea un `ErrorHandler` que capte excepciones no manejadas, las clasifique (`NetworkException`, `AuthException`, `BusinessException`) y muestre un Snackbar con un mensaje amigable. Todas las llamadas a repositorios deben retornar `Either<Failure, Success>`.
>
> **5. Servicio de caché inteligente:** Implementa un `CacheManager` con estrategia **TTL (Time To Live)**: los productos se cachean por 5 minutos, las órdenes por 1 hora, y el catálogo de estilos (estático) por 7 días.
>
> **6. Seguridad:** Genera un interceptor `AuthInterceptor` que refresque el token de Firebase automáticamente cuando queden menos de 5 minutos para expirar (usando un `Timer` o un `Isolate`). Incluye Storage Emulación para pruebas locales.
>
> **7. UI resiliente:** Todos los widgets de lista (`ListView.builder`) deben ser **lazy** (cargar imágenes con `cached_network_image`) y mostrar skeletons personalizados (shimmer con el color verde Wacamaya). Los botones de compra deben tener un estado `disabled` mientras se procesa el pago y además un `Overlay` que bloquee la interacción con el resto de la pantalla.
>
> **8. Documentación:** Genera comentarios `///` para cada clase pública y un archivo `README.md` con instrucciones para ejecutar el emulador de Firebase local y poblar la base de datos con datos de prueba (seeds).
>
> **Salida final:** Proporciona:
> - El `pubspec.yaml` completo (con dependencias: hooks_riverpod, freezed, isar, dio, sentry_flutter, firebase_core, cloud_firestore, etc.)
> - La estructura de directorios completa (comando `tree` simulado).
> - El código del `CartNotifier` (Riverpod), el `CartRepository` (con manejo offline-first) y un widget de ejemplo `CartScreen` que reaccione a los estados `AsyncValue.loading`, `data`, `error`.
> - **Bonus:** Incluye una prueba unitaria para el `CartNotifier` que verifique que agregar un producto dos veces incremente la cantidad en lugar de duplicar el ítem."

---

## 💡 Hoja de Ruta de Próximos Pasos de Alto Nivel (Ejecutables)

1. **Módulo de Inteligencia de Negocios (BI) en Tiempo Real:**
   - Crear una Cloud Function que cada 10 minutos agregue datos de ventas a **BigQuery**.
   - Desde la app (solo admins), mostrar un dashboard con gráficos de **Apache ECharts** (integración webview o paquete nativo) que muestre:
     - Mapa de calor de ventas por zona geográfica (basado en geohash de la IP del usuario).
     - Productos más vistos vs más comprados (ratio de conversión).
   - Si el stock de un producto estrella baja del 10%, enviar una notificación push automática a los usuarios que lo marcaron como favorito: *"¡Últimas unidades del jersey edición limitada!"*.

2. **Infraestructura Global y Edge Computing:**
   - Desplegar imágenes de productos (jerseys, gorras) en **Cloudflare R2** o **Google Cloud CDN** con URLs firmadas (expiran en 1 hora para evitar hotlinking).
   - Configurar **Firebase Remote Config** para cambiar URLs de CDN en caliente sin actualizar la app.
   - Implementar un **Worker de Cloudflare** que sirva imágenes WebP redimensionadas según el DPR del dispositivo (ahorro del 60% en ancho de banda).

3. **Estrategia de Backoff y Resiliencia ante Fallos Catastróficos:**
   - Si Firestore está completamente caído (disaster scenario), la app pasa a **modo solo lectura local**.
   - Los pedidos se guardan en una cola con **persistencia en el sistema de archivos** (usando `path_provider`).
   - Cuando se restaura el servicio, la app no solo reproduce la cola, sino que usa **idempotency keys** (un hash de la compra + timestamp) para evitar duplicados en el backend.
   - **Simulación de caos:** Ejecutar pruebas de GameDay donde se mata intencionalmente el servicio de autenticación durante 10 segundos para verificar que la app con cache offline sigue permitiendo navegar el catálogo.

4. **Cumplimiento Normativo y Auditoría (GDPR / CCPA):**
   - Añadir un botón *"Exportar mis datos"* que genere un JSON con todo el historial de compras y preferencias.
   - Implementar TTL automático en Firestore: los logs de navegación anónima se borran después de 30 días mediante una Cloud Function programada.
   - Opción de *"cuenta fantasma"*: permite comprar sin registro, pero con un token efímero. Los datos se eliminan automáticamente a las 48 horas.

---

### 🔥 Resumen Ejecutivo para Stakeholders Técnicos

| Componente | Decisión Técnica | Justificación |
|------------|------------------|----------------|
| **Arquitectura** | Clean Architecture + DDD | La lógica de negocio sobrevive a cambios de base de datos o framework. |
| **Estado** | Riverpod + StateNotifier + Event Sourcing | Reactividad predecible, fácil testing y time-travel debugging. |
| **Persistencia** | Isar (local) + Firestore (remoto) con cola offline | Experiencia "offline-first", sin pantallas de carga falsas. |
| **Seguridad** | Custom Claims + SSL Pinning + AES-256 en disco | Mitiga ataques MITM y escalada de privilegios. |
| **Observabilidad** | OpenTelemetry + Sentry + Datadog | Detectamos anomalías antes que los usuarios. |
| **DevOps** | Terraform + GitHub Actions + Feature Flags | Rollbacks en menos de 2 minutos, despliegue azul/verde. |

---

¿Listo para que empecemos a generar el código real de este monolito bien estructurado? Podemos comenzar con el archivo `pubspec.yaml` y el `CartNotifier` con su prueba unitaria, o prefieres que profundice en el **módulo de pagos con Stripe Connect** (incluyendo manejo de webhooks y reconciliación de fondos). 

**Dime la siguiente capa que deseas materializar en código.**

Prompt:
"Actúa como un Arqitecto de Software Senior experto en Flutter y Firebase. Tu tarea es generar la estructura inicial y la configuración base para la aplicación multiplataforma 'Wacamaya Sports' (tienda de artículos de fútbol).
Stack Tecnológico:
Framework: Flutter (Dart)
Backend: Firebase (Auth, Firestore, Storage)
State Management: Provider
IDE: VS Code
Navegación: go_router (recomendado) o Navigator 2.0
Requisitos Funcionales Clave:
Autenticación completa (Login/Registro con Email/Password, Recuperación de contraseña).
Catálogo de productos (Jerseys, shorts, tenis, mochilas, termos) conectado a Firestore.
Carrito de compras persistente.
Perfil de usuario e historial de pedidos.
Entregables solicitados (SOLO configuración y estructura, sin lógica de negocio compleja aún):
Archivo pubspec.yaml completo: Incluye todas las dependencias necesarias (firebase_core, firebase_auth, cloud_firestore, firebase_storage, provider, go_router, cached_network_image, flutter_svg, intl, google_fonts, etc.) con versiones compatibles estables.
Estructura de Carpetas (Árbol de directorios): Muestra una arquitectura limpia escalable (ej. /lib/core, /lib/features/auth, /lib/features/catalog, /lib/providers, /lib/services, /lib/utils).
Configuración de Firebase (firebase_options.dart conceptual): Indica dónde colocar las claves y cómo inicializar FirebaseApp.
Setup de Providers (main.dart): Escribe el código del main.dart configurando MultiProvider con los notifiers básicos necesarios (AuthProvider, CartProvider, ProductProvider) y la inicialización de Firebase.
Reglas de Seguridad Firestore (Borrador): Un esquema básico de reglas de seguridad para firestore.rules que permita lectura pública de productos pero escritura solo para admins/usuarios autenticados en sus propios datos.
Nota: No escribas la UI de las pantallas ni la lógica detallada de los servicios todavía. Concéntrate en la infraestructura, dependencias y arquitectura base para que el proyecto compile y esté listo para desarrollar features."
