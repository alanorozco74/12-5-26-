# 📱 Plan de Implementación: Wacamaya Sports
**Tienda multiplataforma de artículos deportivos de fútbol (jerseys, shorts, tenis, mochilas y termos)**  
*Framework: Flutter + Dart | Backend: Firebase (Auth, Firestore, Storage) | Gestión de estado: Provider | IDE: VS Code*

---

## 🛠️ Fase 1: Preparación del Entorno y Herramientas
1. **SDKs y Core**
   - Instalar Flutter SDK (rama estable) y Dart SDK.
   - Configurar variables de entorno para `flutter` y `dart`.
2. **IDE y Extensiones (VS Code)**
   - Extensiones recomendadas: `Flutter`, `Dart`, `Firebase`, `Pubspec Assist`, `Error Lens`, `GitLens`, `Flutter Riverpod/Provider Snippets` (opcional).
   - Configurar formateo automático (`flutter format`) y linting (`flutter analyze`).
3. **Control de Versiones**
   - Inicializar repositorio Git.
   - Configurar `.gitignore` oficial para Flutter y Firebase.
   - Establecer rama `main`, `develop` y `feature/*`.
4. **Emuladores y Compilación**
   - Instalar Android Studio (solo para emuladores y SDK de Android).
   - Instalar Xcode (solo macOS, para simulador iOS y builds nativos).
   - Verificar con `flutter doctor` que todo esté verde antes de iniciar.
5. **Nota sobre "Antigravity"**
   - No existe un IDE llamado "Antigravity" en el ecosistema Flutter. Si te refieres a un editor ligero alternativo, VS Code es la opción recomendada y oficialmente soportada por el equipo de Flutter.

---

## 🎨 Fase 2: Diseño UI/UX
1. **Investigación y Definición**
   - Público objetivo: aficionados al fútbol, jugadores amateurs, padres, coleccionistas.
   - Benchmark de tiendas deportivas y apps de e-commerce.
   - Definir tono de marca: dinámico, confiable, deportivo.
2. **Wireframing y Prototipado**
   - Herramienta: Figma (recomendado) o Adobe XD.
   - Crear flujos clave: Onboarding → Login/Registro → Home → Catálogo → Detalle → Carrito → Checkout → Perfil.
   - Diseñar vistas adaptativas (mobile first, luego tablet/web).
3. **Guía de Estilo**
   - Paleta de colores: primario (energía/deporte), secundario (contraste), neutros (fondos/textos).
   - Tipografía: sans-serif legible, jerarquía clara (H1, H2, cuerpo, botones).
   - Componentes reutilizables: cards de producto, botones CTA, campos de formulario, barras de navegación, badges de oferta.
4. **Principios UX**
   - Navegación intuitiva (máximo 3 taps para llegar a un producto).
   - Estados de carga y error visibles.
   - Accesibilidad: contraste WCAG, tamaños de texto ajustables, soporte para lectores de pantalla.
   - Microinteracciones: hover en web, ripple en botones, transiciones suaves entre pantallas.

---

## 🏗️ Fase 3: Arquitectura y Gestión de Estado
1. **Patrón Arquitectónico**
   - Estructura modular por capas: `presentation/`, `domain/`, `data/`, `core/`, `utils/`.
   - Separación clara entre UI, lógica de negocio y acceso a datos.
2. **State Management con Provider**
   - `ChangeNotifier` para estado global (autenticación, carrito, preferencias).
   - `MultiProvider` en el root para inyectar servicios y viewmodels.
   - `Consumer`/`Selector` en widgets para reconstruir solo lo necesario.
   - Evitar lógica de negocio en widgets; mantener en viewmodels o servicios.
3. **Navegación**
   - Uso de enrutamiento declarativo (ej. `go_router` conceptualmente).
   - Definir rutas protegidas (solo accesibles autenticados).
   - Manejo de deep links para compartir productos.
4. **Manejo de Errores y Logs**
   - Captura centralizada de excepciones.
   - Registro con Firebase Crashlytics (fase posterior).
   - Mensajes de error amigables y localizables.

---

## 📦 Fase 4: Dependencias Requeridas (`pubspec.yaml`)
*(Lista conceptual con propósito de cada paquete)*

| Categoría | Paquete | Propósito |
|-----------|---------|-----------|
| **Core** | `cupertino_icons`, `intl` | Iconos y formateo de fechas/moneda |
| **Firebase** | `firebase_core`, `firebase_auth`, `cloud_firestore`, `firebase_storage` | Inicialización, autenticación, base de datos, almacenamiento de imágenes |
| **Estado** | `provider` | Gestión de estado reactivo y reutilizable |
| **Navegación** | `go_router` | Enrutamiento declarativo y protegido |
| **UI/UX** | `cached_network_image`, `flutter_svg`, `shimmer`, `google_fonts`, `carousel_slider` | Carga de imágenes, iconos vectoriales, placeholders, tipografía, sliders |
| **Formularios** | `flutter_form_builder` o validación nativa | Validación segura y experiencia de entrada consistente |
| **Utilidades** | `shared_preferences`, `uuid`, `equatable` | Persistencia local, identificadores únicos, comparación de objetos |
| **Pagos (futuro)** | `stripe_flutter` / `mercadopago_sdk` | Integración de checkout seguro |
| **Dev/Testing** | `flutter_test`, `mockito`, `firebase_auth_mocks` | Pruebas unitarias, de widget y simulación de Firebase |

> ⚠️ Mantén versiones compatibles entre sí. Usa `flutter pub outdated` y `flutter pub upgrade` regularmente. Evita paquetes abandonados o sin mantenimiento activo.

---

## 🔐 Fase 5: Autenticación (Email/Password)
1. **Configuración Firebase Console**
   - Crear proyecto Firebase.
   - Habilitar Authentication → Proveedor `Email/Password`.
   - Configurar verificación por correo (opcional pero recomendada).
2. **Flujos de Usuario**
   - Registro con validación en tiempo real.
   - Inicio de sesión con manejo de errores (contraseña incorrecta, email no verificado, cuenta deshabilitada).
   - Recuperación de contraseña vía email.
   - Cierre de sesión seguro y limpieza de estado local.
3. **Gestión de Estado de Sesión**
   - Escuchar `authStateChanges` para detectar login/logout automático.
   - Persistir token de sesión de forma segura (Firebase lo maneja nativamente).
   - Redirigir a rutas protegidas o públicas según el estado.
4. **Seguridad**
   - Validar longitud y complejidad de contraseñas en UI.
   - Implementar rate-limiting conceptual (Firebase ya aplica límites nativos).
   - Nunca almacenar credenciales en `SharedPreferences` o archivos locales.

---

## 🗄️ Fase 6: Base de Datos Firestore
1. **Estructura de Colecciones**
   - `users`: perfil, historial de compras, direcciones, rol (cliente/admin).
   - `categories`: nombre, slug, imagen, orden de visualización.
   - `products`: nombre, descripción, precio, stock, categoría, imágenes, etiquetas, disponibilidad.
   - `cart` (o carrito embebido en `users`): items, cantidades, totales parciales.
   - `orders`: estado, items, total, fecha, dirección, método de pago, ID de usuario.
2. **Optimización y Buenas Prácticas**
   - Índices compuestos para filtrado por categoría + precio + disponibilidad.
   - Paginación con `startAfterDocument` para listados largos.
   - Evitar documentos >1MB; separar subcolecciones si es necesario.
   - Uso de timestamps nativos (`FieldValue.serverTimestamp()`).
3. **Reglas de Seguridad (Conceptual)**
   - Lectura pública para catálogo y categorías.
   - Escritura solo para usuarios autenticados con rol `admin` o `owner` del recurso.
   - Validación de tipos y rangos en reglas (ej. precio > 0, stock ≥ 0).
4. **Imágenes**
   - Subir a Firebase Storage.
   - Guardar URL pública o firmada en Firestore.
   - Optimizar tamaño y formato (WebP preferido) antes de la subida.

---

## 📝 Fase 7: Procedimiento Paso a Paso (Sin Código)
1. **Inicialización del Proyecto**
   - Crear app Flutter con nombre `wacamaya_sports`.
   - Configurar Firebase (`flutterfire configure` conceptual).
   - Verificar compilación en emulador Android e iOS.
2. **Estructura de Carpetas**
   - Crear directorios por capa y por feature (auth, catalog, cart, profile).
   - Definir archivos de configuración global y temas.
3. **Configuración de Dependencias**
   - Añadir paquetes al `pubspec.yaml`.
   - Ejecutar `flutter pub get` y validar resolución sin conflictos.
4. **Componentes UI Reutilizables**
   - Construir botones, cards, campos de texto, barras de navegación, placeholders.
   - Aplicar guía de estilo Figma a nivel de widgets.
5. **Integración de Provider**
   - Crear notifiers para auth, carrito y catálogo.
   - Inyectar en el root con `MultiProvider`.
   - Conectar consumidores en vistas específicas.
6. **Desarrollo de Autenticación**
   - Implementar pantallas de login, registro y recuperación.
   - Conectar con `firebase_auth` a través de un servicio abstracto.
   - Manejar redirección post-auth y persistencia de sesión.
7. **Conexión con Firestore**
   - Crear repositorio para productos y categorías.
   - Implementar streams para listados en tiempo real.
   - Añadir paginación y manejo de estados (cargando, éxito, error, vacío).
8. **Carrito y Checkout**
   - Diseñar flujo: añadir → modificar cantidad → eliminar → resumen.
   - Persistir carrito localmente + sincronización opcional con Firestore.
   - Preparar estructura para futura pasarela de pago.
9. **Perfil y Pedidos**
   - Pantalla de usuario con historial, direcciones, preferencias.
   - Listado de órdenes con estados (pendiente, enviado, entregado).
10. **Pulido Final**
    - Transiciones, animaciones suaves, manejo de orientación.
    - Localización básica (español/inglés).
    - Optimización de imágenes y reducción de reconstrucciones innecesarias.

---

## 🧪 Fase 8: Pruebas y Optimización
1. **Pruebas Automatizadas**
   - Unit tests para viewmodels y servicios.
   - Widget tests para componentes críticos (formulario, card, carrito).
   - Integration tests para flujos completos (login → compra → cierre).
2. **Emuladores Firebase**
   - Usar Local Emulator Suite para Auth y Firestore.
   - Validar reglas de seguridad y flujos offline.
3. **Testing en Dispositivos Reales**
   - Pruebas en Android (varias versiones/API).
   - Pruebas en iOS (simulador + dispositivo físico si es posible).
   - Validar rendimiento en redes lentas y modo avión.
4. **Optimización**
   - Revisar con Flutter DevTools (FPS, memoria, rebuilds).
   - Implementar caché inteligente para imágenes y datos estáticos.
   - Minimizar listeners de Firestore y usar snapshots puntuales cuando sea posible.
5. **Accesibilidad y UX Final**
   - Verificar contraste, tamaños de texto, navegación por teclado/voz.
   - Ajustar microcopys y mensajes de error para claridad.

---

## 🚀 Fase 9: Despliegue y Mantenimiento
1. **Preparación de Builds**
   - Configurar `pubspec.yaml` para release (iconos, splash, permisos, versiones).
   - Firmar APK/AAB y IPA con keystore/certificados.
   - Validar con `flutter build` y pruebas de smoke en dispositivos.
2. **Publicación**
   - Google Play Console: crear app, subir AAB, llenar ficha, configurar pruebas internas/cerradas.
   - Apple App Store Connect: certificados, provisioning profiles, revisión de cumplimiento.
3. **Monitoreo Post-Lanzamiento**
   - Activar Firebase Crashlytics y Analytics.
   - Configurar alertas de errores críticos.
   - Revisar métricas de retención, conversión y rendimiento.
4. **Mantenimiento y Escalabilidad**
   - Actualizar dependencias mensualmente.
   - Implementar CI/CD básico (GitHub Actions, Fastlane o Codemagic).
   - Planificar futuras features: pasarela de pagos, notificaciones push, panel de administración web, catálogo con IA/recomendaciones.
5. **Backups y Cumplimiento**
   - Exportación periódica de Firestore (consola o scripts).
   - Revisar políticas de privacidad y GDPR/CCPA según región objetivo.
   - Documentar flujos críticos para nuevos colaboradores.

---

## ✅ Checklist de Validación Final
- [ ] Entorno Flutter + Firebase configurado y verificado  
- [ ] UI/UX validado en Figma con flujo completo  
- [ ] Arquitectura limpia y Provider integrado  
- [ ] Dependencias añadidas y compatibles  
- [ ] Auth email/password funcional con manejo de errores  
- [ ] Firestore estructurado, seguro y optimizado  
- [ ] Catálogo, carrito y perfil implementados  
- [ ] Pruebas unitarias, de widget e integración pasadas  
- [ ] Builds de release generados y firmados  
- [ ] Monitoreo activo y plan de mantenimiento definido  

---
💡 **Recomendación estratégica:** Comienza con un MVP que incluya catálogo, autenticación, carrito y checkout simulado. Valida con usuarios reales antes de integrar pagos reales. Itera con base en métricas de conversión y feedback.

¿Deseas que profundice en alguna fase específica (por ejemplo, estructura de carpetas, diseño de reglas de Firestore, o estrategia de pruebas) antes de comenzar la codificación?
