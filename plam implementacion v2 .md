¡Entendido! Como arquitecto de software, he diseñado una hoja de ruta técnica de nivel industrial. He integrado el concepto de **Antigravity** como tu entorno de alto rendimiento y he estructurado las bases de datos con un enfoque relacional/NoSQL optimizado para una tienda deportiva multiplataforma.

Aquí tienes el plan de implementación extendido y el **Prompt Maestro** final.

---

# 🏆 Plan de Implementación Maestro: Wacamaya Sports

**Alcance:** Android, iOS, Windows y Web.
**Stack:** Flutter SDK + Dart + Firebase (Sin Analíticas para máxima privacidad y velocidad).

## 📊 1. Arquitectura de Datos (Tablas/Colecciones)

Para Firestore, utilizaremos una estructura de colecciones optimizada para escalabilidad y ahorro de lecturas.

### Tabla: `users` (Usuarios)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `uid` | String (PK) | ID generado por Firebase Auth. |
| `name` | String | Nombre completo del cliente. |
| `email` | String | Correo electrónico de contacto. |
| `address` | Map | Objeto con calle, ciudad, CP para envíos. |
| `role` | String | "client" o "admin" (para gestión de stock). |
| `createdAt` | Timestamp | Fecha de registro. |

### Tabla: `products` (Catálogo)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `pid` | String (PK) | ID único del producto. |
| `name` | String | Nombre (Ej: Jersey Real Madrid 2024). |
| `description` | String | Detalles técnicos y material. |
| `category` | String | jerseys, shorts, tenis, mochilas, termos. |
| `price` | Double | Precio de venta. |
| `stock` | Int | Unidades disponibles. |
| `sizes` | Array | Lista de tallas disponibles (S, M, L / 27, 28, 29). |
| `images` | Array | Lista de URLs de Firebase Storage. |

### Tabla: `orders` (Pedidos)

| Campo | Tipo | Descripción |
| --- | --- | --- |
| `oid` | String (PK) | ID del pedido. |
| `userId` | String (FK) | Relación con el comprador. |
| `items` | Array | Lista de productos (id, qty, price, size). |
| `total` | Double | Monto total de la transacción. |
| `status` | String | "pending", "shipped", "delivered". |
| `timestamp` | Timestamp | Fecha y hora de compra. |

---

## 🛠️ 2. Procedimiento de Desarrollo Extendido

### Fase 1: Entorno y Core (Semana 1)

1. **Configuración Antigravity/VS Code:** Instalación de linters estándar de Google para evitar errores de compilación en Windows/Web.
2. **Inicialización Multiplataforma:** Ejecutar `flutter create .` y asegurar que las carpetas `android`, `ios`, `windows` y `web` estén correctamente configuradas.
3. **Firebase Core:** Configurar `firebase_options.dart` para las 4 plataformas.

### Fase 2: Capa de Datos y Providers (Semana 2)

1. **Modelado:** Creación de clases Dart con serialización `Map<String, dynamic>` para Firestore.
2. **Implementación de Providers:**
* `AuthProvider`: Lógica de persistencia de sesión.
* `ShopProvider`: Gestión del catálogo y filtros.
* `CartProvider`: Lógica de cálculo de precios y stock temporal.



### Fase 3: UI/UX Multiplataforma (Semana 3)

1. **Responsive Design:** Implementar `LayoutBuilder` para que la tienda se vea como una Web en escritorio y como App nativa en móvil.
2. **Navegación:** Uso de `NavigationRail` para Windows/Web y `BottomNavigationBar` para móviles.

### Fase 4: Integración y Seguridad (Semana 4)

1. **Reglas de Firestore:** Configurar seguridad para que solo el dueño vea sus pedidos pero cualquiera vea el catálogo.
2. **Optimización:** Implementación de `CachedNetworkImage` para no consumir datos innecesarios en móviles.

---

## 📦 3. Dependencias Estándar (`pubspec.yaml`)

```yaml
dependencies:
  flutter:
    sdk: flutter
  # Backend
  firebase_core: ^2.30.0
  firebase_auth: ^4.19.0
  cloud_firestore: ^4.17.0
  # Estado
  provider: ^6.1.2
  # Estética y Funcionalidad
  google_fonts: ^6.2.1
  cached_network_image: ^3.3.1
  intl: ^0.19.0
  font_awesome_flutter: ^10.7.0 # Iconos deportivos profesionales

```

---

## 🚀 . Prompt Profesional Maestro (Final)

Este prompt está diseñado para que la IA genere el código más robusto y limpio posible para tu proyecto:

> "Actúa como un Ingeniero de Software Senior experto en Flutter y Firebase. Necesito desarrollar la arquitectura base para 'Wacamaya Sports', una app multiplataforma (Android, iOS, Web, Windows).
> **REQUERIMIENTOS TÉCNICOS:**
> 1. **Estructura de Carpetas:** Implementa una arquitectura de capas (Clean Architecture): `data/models`, `data/repositories`, `presentation/providers`, `presentation/screens` y `core/theme`.
> 2. **Base de Datos:** Utiliza las tablas `users`, `products` (con categorías: jerseys, shorts, tenis, mochilas, termos) y `orders` definidas en el plan.
> 3. **Gestión de Estado:** Configura `MultiProvider` en el `main.dart`. Incluye `AuthProvider` para Firebase Auth y `CartProvider` para la lógica del carrito.
> 4. **UI Multiplataforma:** Crea un widget de navegación adaptativo que funcione bien en móvil y escritorio.
> 5. **Modelado:** Genera el modelo `Product` con soporte para tallas (array) y precios (double), incluyendo métodos `fromFirestore` y `toFirestore`.
> 6. **Estándares:** Usa código limpio, tipado estricto (no dynamic), manejo de errores con try-catch y **no incluyas analíticas ni configuraciones de producción**.
> 
> 
> **ENTREGABLE:** Proporciona la estructura del proyecto, el archivo `pubspec.yaml` completo, el `main.dart` con la configuración de Providers y los modelos de datos principales."

---

