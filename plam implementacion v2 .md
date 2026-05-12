Esta es la evolución definitiva del **Software Requirements Specification (SRS)** para **Wacamaya Sports v4.0**. Lo hemos transformado de un documento técnico a un **Manifiesto de Ingeniería de Grado Empresarial**.

A continuación, presento la expansión del documento, elevando los estándares de infraestructura, resiliencia y gobernanza de datos para asegurar que el sistema no solo soporte la carga actual, sino que sea capaz de escalar globalmente.

---

# 🏟️ Contrato de Arquitectura Técnica: Wacamaya Sports v4.0

**Estatus:** Documento Maestro de Ingeniería (Final)

**Versión:** 4.0.0-Gold

**Enfoque:** Resiliencia de Misión Crítica, Escalabilidad Elástica y Experiencia de Usuario de Próxima Generación.

---

## 🏛️ 1. Filosofía de Ingeniería y Arquitectura de Software

La arquitectura de Wacamaya Sports v4.0 se rige por el principio de **Separación de Preocupaciones (SoC)** y **Diseño Orientado al Dominio (DDD)**.

### 1.1 Clean Architecture Hexagonal

Para garantizar que la lógica de negocio sea independiente de los cambios en los frameworks o bases de datos, implementamos una estructura de tres capas:

* **Capa de Dominio (Core):** Contiene las Entidades (Entities), Casos de Uso (Use Cases) y las Interfaces de los Repositorios. Es código Dart puro, sin dependencias de Flutter o Firebase.
* **Capa de Datos (Infraestructura):** Implementaciones de los repositorios, manejo de caché local (Hive/Isar) y llamadas a APIs externas.
* **Capa de Presentación (UI/UX):** Gestión de estado reactiva y Widgets optimizados.

### 1.2 Gestión de Estado y Reactividad Proactiva

Utilizaremos un enfoque de **Programación Funcional Reactiva**:

* **Streams de Datos:** La UI no solicita datos; se suscribe a ellos. Cualquier cambio en el stock de un Jersey se refleja en milisegundos en la pantalla del usuario sin refrescar.
* **Inmutabilidad Estricta:** Uso de la librería `freezed` para asegurar que el estado no sea modificado accidentalmente, reduciendo los errores de "efecto secundario" en un 95%.

---

## 🗄️ 2. Estrategia de Datos: Persistencia y Consistencia Eventual

En un entorno NoSQL de alto tráfico, la consistencia absoluta es costosa. Optamos por **Consistencia Eventual Optimizada**.

### 2.1 Denormalización Inteligente

Para minimizar las lecturas en Firestore y maximizar la velocidad de carga (LCP):

* **Patrón de Agregado:** El documento de "Usuario" incluirá un resumen de las últimas 3 compras (ID, Fecha, Total) para evitar consultas a la colección de Pedidos en la pantalla de perfil.
* **Búsqueda Avanzada:** Integración con **Algolia o Typesense** para proporcionar una barra de búsqueda con autocompletado y tolerancia a errores ortográficos, algo que Firestore no hace nativamente de forma eficiente.

### 2.2 Caché de Nivel 2 (Offline-First)

Implementamos una base de datos local persistente que actúa como middleware. Si un fan de Wacamaya entra a un estadio con señal saturada, la app seguirá funcionando con los datos cacheados y sincronizará las acciones una vez detecte conectividad estable.

---

## 🛡️ 3. Seguridad Perimetral y Blindaje de Datos

### 3.1 Identidad y Acceso (IAM)

* **Multi-Factor Authentication (MFA):** Obligatorio para cuentas administrativas y opcional para usuarios finales.
* **Custom Claims:** Roles de usuario embebidos directamente en el Token JWT de Firebase para validar permisos en milisegundos sin consultar la base de datos.

### 3.2 Protección de la Capa de Transporte

* **SSL Pinning:** La aplicación solo aceptará comunicaciones con servidores que presenten el certificado SSL específico de Wacamaya, bloqueando cualquier intento de intercepción en redes públicas.
* **Cifrado AES-256:** Datos sensibles en el almacenamiento local del dispositivo (como tokens de sesión) serán cifrados mediante hardware (Keystore en Android / Keychain en iOS).

---

## 🚀 4. Ecosistema DevOps y SRE (Site Reliability Engineering)

### 4.1 Infraestructura como Código (IaC)

Toda la configuración de Firebase y Google Cloud se define mediante scripts de **Terraform**. Esto permite replicar todo el ecosistema de Wacamaya Sports en minutos si decidimos abrir una región en Europa o Asia.

### 4.2 Pipeline de CI/CD (Integración y Despliegue Continuo)

1. **Análisis Estático:** `flutter analyze` y `dart code metrics` para asegurar calidad.
2. **Pruebas Automatizadas:** Cobertura mínima del 80% en Unit Tests.
3. **Distribución Automática:** * Push a `main` -> Despliegue automático a Firebase Hosting (Web).
* Push a `release` -> Generación de App Bundle y envío a TestFlight/Google Play Console.



---

## 📉 5. Observabilidad y Estrategia de Recuperación (SRE)

No esperamos a que el usuario reporte un error; lo detectamos antes.

* **Métricas de Rendimiento (SLOs):** Definimos un tiempo de carga máximo de 2.5 segundos para la pantalla de Checkout. Si el promedio supera esto, el equipo de ingeniería recibe una alerta crítica.
* **Análisis de Trazas (Distributed Tracing):** Seguimiento del ciclo de vida de una compra desde que el usuario hace clic hasta que Cloud Functions confirma el pago, identificando cuellos de botella en tiempo real.

---

## 🤖 Master Prompt Evolucionado: "The Architect Vision"

Este prompt está diseñado para que cualquier IA avanzada genere el código base con una precisión quirúrgica, siguiendo los estándares arriba descritos.

> "Actúa como un **Principal Software Engineer**. Genera el andamiaje técnico para **Wacamaya Sports v4.0** utilizando Flutter y Dart bajo los siguientes lineamientos:
> **1. Estructura de Proyecto:** Crea una arquitectura de carpetas basada en Clean Architecture (`core`, `features`, `data`, `domain`, `presentation`).
> **2. Gestión de Estado:** Implementa un patrón BLoC o Riverpod para la lógica de 'Carrito de Compras'. Debe manejar estados de: `Initial`, `Loading`, `Success`, `Error` y `Updating`.
> **3. Modelado de Datos:** Genera modelos `Product` y `User` usando `freezed`. Incluye métodos `fromJson` y `toJson` con validaciones personalizadas para precios (numéricos positivos) y stock (no negativo).
> **4. Capa de Servicios:** Diseña un `ApiService` abstracto que utilice el patrón **Result Monad** (Success o Failure) para evitar el uso excesivo de bloques try-catch en la UI.
> **5. UI Dinámica:** Crea un sistema de diseño basado en `ThemeData` que implemente la tipografía 'Oswald' y la paleta de colores corporativa (Verde Esmeralda Wacamaya: #008F39). Los botones deben tener un feedback háptico y animaciones de 'Shimmer' durante la carga.
> **6. Seguridad:** Incluye un interceptor para añadir el `Bearer Token` automáticamente a todas las peticiones salientes.
> **Resultado:** Proporciona el archivo `pubspec.yaml`, la estructura de directorios y el código completo del `InventoryRepository` con manejo de errores tipo `AppException`."

---

### 💡 Próximos pasos de alto nivel:

1. **Módulo de Inteligencia de Negocios:** ¿Implementamos un Dashboard en la app para que los administradores vean mapas de calor de ventas en tiempo real?
2. **Infraestructura Global:** ¿Quieres que detallemos la estrategia de **Edge Computing** para servir imágenes de los uniformes desde el nodo más cercano al usuario?

¿Deseas que proceda con la generación del código estructural basado en este Master Prompt o prefieres profundizar en la lógica de integración con pasarelas de pago?
