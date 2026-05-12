Este nivel de documentación se denomina **Documento de Especificación de Requerimientos de Software (SRS) de Clase Empresarial**. Aquí no solo definimos el código, sino la gobernanza, el ciclo de vida de los datos y la estrategia de escalabilidad para **Wacamaya Sports**.

---

# 🏟️ Especificación Técnica de Ingeniería de Software: Wacamaya Sports v3.0

**Enfoque:** Arquitectura de Alta Disponibilidad y Resiliencia Multiplataforma.
**Plataformas de Despliegue:** Android (Play Store), iOS (App Store), Windows (MSI/Exe), Web (PWA).

---

## 🏛️ 1. Marco de Trabajo de Arquitectura (Framework)

Adoptamos una arquitectura **Feature-First con Inversión de Dependencias**. Esto significa que cada funcionalidad (Auth, Catálogo, Pagos) se trata como un módulo independiente que puede ser testeado y modificado sin afectar al resto de la aplicación.

### 1.1 Capas de Abstracción

* **Capa de Dominio (Domain Layer):** Contiene los modelos de negocio puros. Si el día de mañana decidimos que un "Jersey" ahora tiene "personalización de nombre", solo se modifica esta capa.
* **Capa de Datos (Data Layer):** Implementa el patrón *Repository*. Aquí gestionamos el caché local (SQLite o Hive) para que la app funcione sin internet en Windows o Móvil.
* **Capa de Presentación (Presentation Layer):** Utiliza el patrón **MVVM (Model-View-ViewModel)**. El ViewModel (Provider) procesa la información y la Vista solo se encarga de renderizar los colores Verde y Negro de la marca.

---

## 🗄️ 2. Modelo de Persistencia y Gobernanza de Datos (Firestore)

La base de datos se ha diseñado bajo un esquema de **Desnormalización Controlada** para optimizar el rendimiento de lectura y minimizar los costos de Firebase.

### 2.1 Diccionario de Datos Extendido (Español)

#### A. Colección: `usuarios`

* **id_usuario:** `String (UID)` -> Llave primaria única.
* **nombre_completo:** `String` -> Nombre para facturación y envíos.
* **correo:** `String` -> Único, validado por RegExp.
* **direccion_envio:** `Map` -> Contiene `{calle, numero, colonia, ciudad, estado, cp}`.
* **rol:** `String` -> Control de acceso (`cliente`, `admin`, `vendedor`).
* **lista_deseos:** `Array<String>` -> IDs de productos marcados como favoritos.
* **historial_busqueda:** `Array<String>` -> Para sugerencias personalizadas.
* **fecha_registro:** `Timestamp` -> Registro de auditoría.

#### B. Colección: `productos` (Jerseys, Tenis, Mochilas, etc.)

* **id_producto:** `String` -> ID alfanumérico generado.
* **nombre:** `String` -> Título comercial.
* **descripcion_tecnica:** `String` -> Materiales, tecnología (ej. Dri-FIT).
* **categoria:** `String` -> Filtro principal (`jerseys`, `tenis`, `accesorios`).
* **precio:** `Double` -> Valor monetario base.
* **oferta:** `Map` -> `{en_oferta: bool, precio_descuento: double, fin_oferta: timestamp}`.
* **existencia_total:** `Int` -> Stock global.
* **variantes:** `List<Map>` -> Cada mapa contiene `{talla: String, color: String, stock_especifico: Int}`.
* **imagenes:** `List<String>` -> URLs de Firebase Storage con Lazy Loading.

#### C. Colección: `pedidos` (Logs de Venta)

* **id_pedido:** `String` -> ID único de transacción.
* **id_cliente:** `String` -> Referencia al usuario comprador.
* **resumen_articulos:** `List<Map>` -> Copia de los datos del producto al momento de compra (para histórico de precios).
* **total_pago:** `Double` -> Monto final transaccionado.
* **metodo_pago:** `String` -> `tarjeta`, `transferencia`, `efectivo`.
* **estado_envio:** `String` -> `pendiente`, `en_preparacion`, `enviado`, `entregado`.
* **tracking_number:** `String` -> Número de guía de paquetería.

---

## 🎨 3. Design System: Identidad Visual Deportiva

La interfaz de Wacamaya Sports debe evocar velocidad, fuerza y profesionalismo.

* **Paleta Primaria:** Verde Wacamaya (`#00C853`). Evoca el césped del estadio. Usado en elementos interactivos de éxito.
* **Paleta Secundaria:** Negro Carbono (`#121212`). Fondo principal para modo oscuro (ahorro de batería en móviles y fatiga visual en Windows/Web).
* **Colores de Estado:**
* *Alerta (Naranja):* `#FF6D00` para stock bajo.
* *Error (Rojo):* `#D32F2F` para errores de pago.


* **Tipografía:**
* *Titulares:* **Oswald** (Google Fonts). Fuente robusta y atlética.
* *Cuerpo:* **Urbanist**. Fuente moderna y legible para descripciones de productos.



---

## 🧠 4. Lógica de Negocio y Gestión de Estado (Providers)

Implementamos un sistema de **Single Source of Truth (SSOT)**.

1. **`AutenticacionProvider`:** Gestiona el token de sesión. Implementa persistencia para que el usuario no tenga que loguearse cada vez que abre la app en Windows o Android.
2. **`FirestoreProvider`:** Centraliza las peticiones a la base de datos. Implementa un sistema de "Caché de Catálogo" para que la navegación sea instantánea.
3. **`CarritoProvider`:** Gestiona la lógica de la canasta.
* *Lógica de Impuestos:* Cálculo automático de IVA.
* *Validación de Tallas:* No permite agregar artículos sin seleccionar talla.
* *Sincronización:* Capacidad de guardar el carrito en la nube para recuperarlo en cualquier dispositivo.



---

## 🚀 5. Plan de Implementación por Fases (12 Pasos)

### Fase I: Cimientos (Semana 1)

1. **Instanciación:** Configuración de Flutter para soporte de Windows (Visual Studio) y Web (CanvasKit).
2. **Firebase Multi-Project:** Vinculación de los 4 IDs de plataforma (Android, iOS, Web, Windows).
3. **Core Theme:** Implementación de la clase `AppTheme` con los colores Verde/Negro.

### Fase II: Data & Auth (Semana 2)

4. **Modelado en Español:** Creación de las entidades y modelos con serialización JSON.
5. **Servicio de Auth:** Implementación de login con correo y manejo de excepciones de Firebase.
6. **Firestore Services:** Programación de los métodos asíncronos para CRUD de productos.

### Fase III: Experiencia de Usuario (Semana 3)

7. **Catálogo Adaptativo:** Implementación de GridView dinámico (2 columnas en móvil, 5 en PC).
8. **Detalle de Producto:** Widget de slider de imágenes y selector de variantes (tallas/colores).
9. **Carrito de Compras:** Lógica de persistencia local y cálculo de totales.

### Fase IV: Finalización y Seguridad (Semana 4)

10. **Proceso de Checkout:** Formulario de dirección de envío y confirmación de pedido.
11. **Security Rules:** Escritura de reglas en Firebase para evitar acceso no autorizado a los datos de otros usuarios.
12. **Optimización:** Minificación de código y generación de binarios para Windows (.exe) y Web.

---

## 🤖 Master Prompt Profesional (Nivel Arquitecto Senior)

Copia este prompt final para que la IA genere el código con esta profundidad técnica:

> "Actúa como un **Arquitecto de Soluciones Senior** experto en sistemas e-commerce. Necesito la implementación base para **'Wacamaya Sports'**, una solución multiplataforma (Android, iOS, Windows, Web).
> **Requerimientos Técnicos Obligatorios:**
> 1. **Arquitectura:** Estructura de carpetas bajo **Clean Architecture** (core, data, domain, presentation).
> 2. **Modelado en Español:** Genera los modelos Dart para `usuarios`, `productos` y `pedidos` con los campos detallados en el plan maestro. Incluye mapeo de variantes (tallas y colores).
> 3. **Gestión de Estado:** Configura el `main.dart` con `MultiProvider` inyectando: `AutenticacionProvider`, `FirestoreProvider` y `CarritoProvider`.
> 4. **Identidad Visual:** Implementa una clase `EstiloWacamaya` que defina colores Primario Verde (#00C853) y Fondo Negro (#121212) con tipografía Oswald.
> 5. **Multiplataforma:** Asegura que el código sea responsivo, usando `LayoutBuilder` para diferenciar la navegación de Windows (sidebar) de la de Android (bottom bar).
> 6. **Calidad:** Código robusto, manejo de errores asíncronos mediante bloques try-catch, uso de nulabilidad segura y comentarios explicativos.
> 
> 
> **Entrega:** Estructura completa de archivos, archivo `pubspec.yaml`, modelos de datos, inyección de Providers y el esqueleto de la pantalla de catálogo."

¿Deseas que profundice en el **diseño detallado de la pantalla de pagos** o que desglose las **reglas de seguridad de Firebase** para proteger la integridad de los pedidos?
