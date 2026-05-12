Esta es una expansión masiva del documento de especificación, elevándolo a un nivel de **Arquitectura de Sistemas Distribuidos**. Hemos pasado de una app de comercio electrónico a una plataforma escalable con auditoría, infraestructura en la nube y optimización de rendimiento.

---

# 🏟️ Especificación Técnica de Ingeniería de Software: Wacamaya Sports v3.5

**Nivel:** Documentación de Grado Industrial (Enterprise-Grade)

**Alcance:** Multiplataforma Total (Desktop, Mobile, Web)

---

## 🏛️ 1. Ecosistema de Arquitectura y Patrones Avanzados

Para garantizar que Wacamaya Sports soporte picos de tráfico (como un lanzamiento de edición limitada de jerseys), no solo usamos **Clean Architecture**, sino que implementamos **Event-Driven UI**.

### 1.1 Estructura de Directorios (Modularización)

```text
lib/
├── core/               # Lógica transversal (Formatters, Errores, Temas)
│   ├── network/        # Client-side caching & Interceptors
│   ├── constants/      # AppColors, AssetsPath, APIEndpoints
│   └── errors/         # Failure classes para manejo de excepciones
├── features/           # Módulos independientes (Feature-First)
│   ├── catalog/        # UI, Modelos, Logic de Productos
│   ├── auth/           # Login, Registro, Recuperación
│   ├── orders/         # Historial, Seguimiento, Facturación
│   └── cart/           # Gestión de bolsa de compras
├── shared/             # Widgets reutilizables entre módulos
└── main.dart           # Punto de entrada y composición de inyección

```

### 1.2 Inversión de Dependencias (Dependency Injection)

Utilizaremos contratos (clases abstractas) en la capa de dominio. Esto permite que, si en el futuro Wacamaya Sports migra de Firebase a un backend propio en Go o Node.js, **solo cambiamos la implementación en la capa de datos**, sin tocar un solo píxel de la interfaz.

---

## 🗄️ 2. Gobernanza de Datos y Persistencia Híbrida

### 2.1 Estrategia de Caché "Offline-First"

Para la versión de **Windows (Desktop)**, es vital la fluidez. Implementamos un sistema de sincronización bidireccional:

1. **Local:** `Sembast` o `Hive` para persistencia NoSQL ultra rápida en disco local.
2. **Remoto:** `Firestore` con `Snapshot Listeners` para actualizaciones en tiempo real de stock.

### 2.2 Diccionario de Datos Profundo (Entidades Core)

| Entidad | Campo | Tipo | Función de Negocio |
| --- | --- | --- | --- |
| **Usuario** | `metadata_seguridad` | Map | Almacena el `last_login`, `ip_registro` y `dispositivo`. |
| **Producto** | `score_relevancia` | Double | Algoritmo simple para posicionar productos en el home. |
| **Pedido** | `bitacora_estados` | List | Historial: `[{"status": "creado", "hora": "..."}, {"status": "pagado", "hora": "..."}]` |
| **Stock** | `reserva_temporal` | Int | Bloquea el artículo por 10 min mientras el usuario está en el checkout. |

---

## 🛡️ 3. Protocolos de Seguridad y Reglas de Negocio (Backend-less)

No confiamos en la lógica del cliente para datos sensibles. Aplicamos **Firebase Security Rules** robustas:

* **Integridad de Precios:** Los usuarios tienen permiso de `read` en productos, pero el campo `precio` solo puede ser modificado por UIDs con rol `admin`.
* **Privacidad:** Las colecciones de `pedidos` tienen una regla: `allow read: if request.auth.uid == resource.data.id_cliente`.
* **Validación de Stock:** Usamos **Cloud Functions** (Node.js) para que, al momento de crear un pedido, se verifique si hay `existencia_total` real. Si no hay, la transacción se revierte automáticamente (Atomicidad).

---

## 🎨 4. Design System "Athletic-Dark"

La experiencia de usuario (UX) se centra en la **fricción mínima**.

### 4.1 Componentes de UI Adaptativos

* **Desktop (Windows):** Navegación lateral extendida. Uso de atajos de teclado (ej. `Ctrl + F` para buscar productos).
* **Mobile (iOS/Android):** Navegación por gestos. Soporte para **Biometría** (FaceID/Huella) para confirmar compras rápidamente.
* **Web:** Optimización de SEO mediante meta-tags dinámicos para que los productos de Wacamaya aparezcan en Google.

---

## 🚀 5. Roadmap de Implementación Técnica (Detallado)

### Fase V: Inteligencia y Analítica (Semana 5-6)

1. **Event Tracking:** Implementación de `Firebase Analytics` para medir qué jerseys se ven más pero se compran menos (Embudo de conversión).
2. **Push Notifications:** Configuración de `FCM` (Firebase Cloud Messaging) para avisar al usuario: *"¡Tu pedido de Wacamaya Sports va en camino! 🚚"*.
3. **Deep Linking:** Permitir que un link compartido por WhatsApp (`wacamaya.com/producto/jersey-verde`) abra directamente la app instalada en el teléfono.

### Fase VI: DevSecOps (Semana 7)

1. **CI/CD Pipeline:** Uso de *GitHub Actions* para que, al hacer push a la rama `main`, la app se compile y se suba automáticamente a la Play Store (Internal Test) y a Firebase Hosting para la Web.
2. **Unit Testing:** Pruebas unitarias para la lógica del `CarritoProvider` (asegurar que $1+1$ sea $2$ tras aplicar descuentos).

---

## 🤖 Master Prompt Nivel "Principal Engineer"

Este prompt es una versión evolucionada para generar un código que no solo funcione, sino que sea **escalable y profesional**:

> "Actúa como un **Principal Software Engineer** especializado en Flutter y Firebase. Desarrolla el núcleo de la plataforma **Wacamaya Sports** siguiendo estos lineamientos técnicos de alto nivel:
> 1. **Arquitectura:** Implementa una estructura **Feature-First**. Crea una carpeta `core/` para el `Theme` y `Models/`, y una carpeta `features/catalog/` para la lógica de productos.
> 2. **Modelos con Null-Safety:** Crea clases Dart para `Producto` y `Variante`. El modelo `Producto` debe incluir un método `fromFirestore()` que maneje errores de mapeo y valores por defecto.
> 3. **Estado Global (Multi-Device):** Configura `ChangeNotifierProvider` para el `Carrito`. La lógica del carrito debe persistir los datos localmente usando `shared_preferences` para que no se pierdan al cerrar la app en Windows.
> 4. **Interfaz de Alto Rendimiento:** >    - Crea un `CustomScrollView` para el catálogo con un `SliverGrid` que cambie el `crossAxisCount` dinámicamente según el ancho de la pantalla (Desktop vs Mobile).
> * Aplica la identidad visual: Fondos `#121212`, acentos `#00C853` y fuentes Oswald/Urbanist.
> 
> 
> 5. **Lógica de Checkout:** Crea una función asíncrona `procesarPago()` que simule la integración con una pasarela, incluya manejo de estados (loading, success, error) y actualice la colección `pedidos` en Firestore.
> 6. **Calidad de Código:** Usa `final` para variables inmutables, `const` para widgets estáticos y añade documentación triple slash (///) en los métodos críticos de negocio.
> 
> 
> Entrega el código fuente organizado, el `pubspec.yaml` con las dependencias necesarias (provider, cloud_firestore, firebase_core, google_fonts) y un ejemplo de la pantalla principal adaptativa."

---

**¿Te gustaría que ahora desarrollemos el código específico de las Cloud Functions para el control de inventario o prefieres ver la implementación del Theme adaptativo en Flutter?**
