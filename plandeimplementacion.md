# 📋 PLAN DE IMPLEMENTACIÓN: `ZON skateshop`
**Stack:** Flutter (Dart) + Firebase (Auth + Firestore) + Provider  
**IDE recomendado:** Visual Studio Code (con extensiones oficiales de Flutter/Dart)  
**Nota:** `"antigravity"` no es un IDE reconocido para desarrollo móvil. Si te refieres a otra herramienta, por favor indícalo. VS Code es suficiente y ampliamente usado en este stack.

---

## 🗂️ ESTRUCTURA DEL PLAN
1. Configuración del entorno y herramientas
2. Diseño UI/UX y arquitectura de información
3. Dependencias y estructura de proyecto
4. Integración de Firebase (Auth + Firestore)
5. Desarrollo de funcionalidades core
6. Pruebas, optimización y despliegue
7. Cronograma sugerido y hitos

---

## 🔹 FASE 1: Configuración del Entorno y Herramientas
| Paso | Acción |
|------|--------|
| 1.1 | Instalar Flutter SDK, Dart SDK y Git |
| 1.2 | Configurar VS Code: extensiones `Flutter`, `Dart`, `Flutter Riverpod` (si se migra luego), `Error Lens`, `PubSpec Assist` |
| 1.3 | Crear emulador/simulador (Android Studio SDK o Xcode para iOS) y verificar `flutter doctor` |
| 1.4 | Crear proyecto Flutter: `zon_skateshop` |
| 1.5 | Crear proyecto en Firebase Console, registrar apps Android/iOS, descargar `google-services.json` y `GoogleService-Info.plist` |
| 1.6 | Instalar Firebase CLI, ejecutar `flutterfire configure` para generar `firebase_options.dart` |

---

## 🔹 FASE 2: Diseño UI/UX y Arquitectura de Información
### 🎨 Identidad Visual
- Paleta urbana/skate: neutros oscuros, acentos en neón o amarillo/rojo skate
- Tipografía: sans-serif moderna y legible (ej. `Inter`, `Poppins`, `Montserrat`)
- Iconografía: estilo lineal o filled, coherente con cultura streetwear

### 📱 Wireframes & Flujos (Figma/Adobe XD)
1. **Onboarding / Splash** → Logo animado, carga inicial
2. **Auth** → Login (email/password), Registro, Recuperar contraseña
3. **Home** → Banner destacado, categorías rápidas (Skateboards, Ropa, Piezas), productos trending
4. **Catálogo** → Grid/list, filtros por categoría, marca, tamaño, precio, stock
5. **Detalle Producto** → Galería, selector de variante, descripción, botón "Agregar al carrito", indicador de stock
6. **Carrito** → Items, cantidades, subtotal, botón checkout
7. **Checkout** → Dirección, método de envío, resumen, confirmación
8. **Perfil** → Datos personales, historial de pedidos, direcciones guardadas, cerrar sesión
9. **Admin (básico)** → Gestión de inventario, lista de pedidos, cambio de estado

### 🧭 Principios UX
- Navegación inferior: `Inicio | Catálogo | Carrito | Perfil`
- Proceso de compra ≤ 3 pasos después del carrito
- Mensajes de error/éxito claros y no intrusivos
- Accesibilidad: contrastes válidos, tamaños de toque ≥ 48x48px

---

## 🔹 FASE 3: Dependencias y Estructura de Proyecto
### 📦 `pubspec.yaml` (Dependencias principales)
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^latest
  firebase_auth: ^latest
  cloud_firestore: ^latest
  provider: ^latest
  go_router: ^latest          # Navegación declarativa
  cached_network_image: ^latest
  intl: ^latest               # Formato de fechas/moneda
  flutter_svg: ^latest        # Iconos y assets vectoriales
  shared_preferences: ^latest # Persistencia ligera
  flutter_secure_storage: ^latest # Tokens/sesiones si se requiere
  image_picker: ^latest       # (Futuro admin) subida de imágenes
  # Opcional según fase de pagos:
  # flutter_stripe: ^latest o pay: ^latest
```
*(Versiones `^latest` se reemplazarán con versiones estables al momento del desarrollo)*

### 📁 Estructura de carpetas (Feature-first + Provider)
```
lib/
├── core/
│   ├── constants/            # Colores, strings, rutas
│   ├── theme/                # ThemeData, tipografías
│   ├── utils/                # Formateadores, validadores
│   └── router/               # go_router config
├── features/
│   ├── auth/                 # Screens, viewmodels, providers
│   ├── catalog/              # Productos, filtros, detalle
│   ├── cart/                 # Carrito, checkout
│   ├── profile/              # Usuario, pedidos
│   └── admin/                # Gestión inventario/pedidos
├── services/
│   ├── firebase_auth_service.dart
│   └── firestore_service.dart
├── shared/
│   ├── widgets/              # Botones, cards, inputs reutilizables
│   └── models/               # Clases de datos (Product, User, Order)
└── main.dart
```

---

## 🔹 FASE 4: Integración de Firebase (Auth + Firestore)
### 🔐 Autenticación (Email/Password)
- Habilitar método en Firebase Console
- Flujo: `Email + Password → Validación local → signIn/createUser → Manejo de errores → Persistencia de sesión`
- Recuperación de contraseña vía `sendPasswordResetEmail`
- Separar roles: `customer` y `admin` (campo en documento `users`)

### 🗄️ Estructura Firestore
| Colección | Documentos / Campos clave |
|-----------|---------------------------|
| `users` | `uid`, `email`, `displayName`, `role`, `addresses[]`, `createdAt` |
| `products` | `id`, `name`, `category` (`skateboard`, `clothing`, `parts`), `brand`, `price`, `stock`, `variants[]`, `images[]`, `description`, `isActive` |
| `orders` | `id`, `userId`, `items[]`, `total`, `status` (`pending`, `processing`, `shipped`, `delivered`), `address`, `createdAt` |
| `categories` | `id`, `name`, `slug`, `icon` (para filtros dinámicos) |

### 🔒 Reglas de Seguridad (Firestore)
- `products`, `categories`: lectura pública, escritura solo admin
- `users/{uid}`: lectura/escritura solo si `request.auth.uid == uid`
- `orders/{id}`: lectura solo por creador o admin, escritura por app + validación de campos

### 🔌 Provider Architecture
- `AuthProvider`: maneja estado de sesión, login, registro, logout
- `ProductProvider`: carga catálogo, aplica filtros, paginación
- `CartProvider`: CRUD local de carrito, sincronización opcional con Firestore
- `OrderProvider`: creación y seguimiento de pedidos

---

## 🔹 FASE 5: Desarrollo de Funcionalidades Core
| Módulo | Tareas principales |
|--------|-------------------|
| **Auth** | Formularios con validación, manejo de estados (loading/success/error), redirección condicional, protección de rutas |
| **Catálogo** | Grid responsive, búsqueda en tiempo real (Firestore `where` + `>=`/`<=`), filtros combinados, lazy loading con `Pagination` |
| **Detalle** | Carrusel de imágenes, selector de variantes (talla/color/piezas), verificación de stock, botón "Agregar" con feedback |
| **Carrito** | `CartProvider` con `ChangeNotifier`, persistencia local inicial, cálculo de totales, eliminación/modificación de items |
| **Checkout** | Validación de dirección, resumen orden, creación de documento en `orders`, limpieza de carrito, pantalla de éxito |
| **Perfil** | Visualización de historial, edición de datos, logout seguro, sincronización de direcciones |
| **Admin (v1)** | Vista de pedidos, cambio de estado, edición de stock, toggle `isActive` para productos |

---

## 🔹 FASE 6: Pruebas, Optimización y Despliegue
| Actividad | Descripción |
|-----------|-------------|
| 🧪 Unit & Widget Tests | Proveedores, validadores, flujos UI clave, mocks de Firestore |
| 🔍 Integración Firebase | Modo prueba con datos ficticios, verificación de reglas de seguridad |
| ⚡ Optimización | `Provider.select` para rebuilds mínimos, caché de imágenes, lazy builders, Flutter DevTools (memory/CPU) |
| 📦 Preparación Stores | Iconos adaptativos, splash screen, `pubspec.yaml` metadata, signing (Android keystore / iOS certificates) |
| 🚀 Beta & Release | TestFlight (iOS) / Play Console internal testing (Android), recolección de feedback, hotfixes |
| 🔄 CI/CD (Opcional) | GitHub Actions o Codemagic para builds automáticos y despliegue en ramas |

---

## 📅 Cronograma Sugerido (Hitos)
| Semana | Hito |
|--------|------|
| 1 | Entorno, Firebase config, estructura base, tema UI |
| 2 | Auth completo, navegación, providers base |
| 3 | Catálogo + filtros + detalle producto |
| 4 | Carrito + checkout + creación de órdenes |
| 5 | Perfil + historial + panel admin básico |
| 6 | Pruebas, optimización, reglas Firestore, preparación stores |
| 7 | Beta testing, ajustes finales, despliegue |

*(Ajustable según disponibilidad y experiencia del equipo)*

---

## ✅ VALIDACIÓN ANTES DE ESCRIBIR CÓDIGO
- [ ] Diseño UI/UX aprobado en Figma
- [ ] Estructura de Firestore y reglas de seguridad definidas
- [ ] `pubspec.yaml` con versiones estables verificadas
- [ ] Proveedores y modelos de datos mapeados
- [ ] Flujo de compra validado (sin fricción, ≤3 pasos post-carrito)

---

🔜 **Siguiente paso:** Cuando este plan esté validado, puedo generar:
1. `pubspec.yaml` con versiones exactas y compatibilidad
2. Estructura de carpetas + `main.dart` inicializado
3. Modelos de datos (`Product`, `User`, `Order`)
4. Providers base (`AuthProvider`, `ProductProvider`, `CartProvider`)
5. Pantallas con UI estructurada (sin lógica compleja aún)
6. Configuración de Firestore + Reglas de seguridad listas para copiar/pegar

¿Deseas ajustar alguna fase, agregar integración de pagos específica, o definir roles de admin antes de pasar al código?
