¡Excelente iniciativa! Vamos a estructurar la habilidad global **.agents** para automatizar el desarrollo de tu aplicación de venta de teléfonos. Esta configuración permitirá que el agente no solo entienda el código, sino la arquitectura y el flujo de trabajo en Flutter y Firebase.

---

## 1. Estructura de la Habilidad `.agents`

Primero, creamos la base del agente dentro de tu directorio raíz.

### `SKILL.md` (Definición del Agente)
```markdown
# Skill: Agente Global de Ventas de Teléfonos
**Descripción:** Especialista en automatización de Flutter, Scraping de catálogo y Código Dart.
**Stack:** Flutter, Dart, Firebase Firestore.

## Capacidades
- **Diseño:** Generación de UI responsiva con Material 3.
- **Código:** CRUD funcional con Clean Architecture (opcional) o MVC.
- **Scraping:** Automatización para poblar el catálogo de teléfonos.

## Estructura de Carpetas
- /scripts: Automatización de deploys y limpieza.
- /ejemplos: Snippets de UI y lógica.
- /resources: Assets, iconos y temas.
```

---

## 2. Preparación del Entorno (Prerrequisitos)

Antes de codificar, debemos asegurar que las herramientas estén listas. Ejecuta estos comandos en tu terminal de **VS Code** o **Antigravity**:

### Verificación e Instalación
1. **Flutter:** `flutter --version` (Si no está, descarga el SDK de flutter.dev).
2. **Firebase CLI:** * Instalar: `npm install -g firebase-tools`
   * Login: `firebase login` (Se abrirá el navegador).
   * FlutterFire CLI: `dart pub global activate flutterfire_cli`

### Configuración del Proyecto en Firebase
1. Ve a [Firebase Console](https://console.firebase.google.com/).
2. Crea el proyecto: **"proyectoventatelefonos"**.
3. Habilita **Cloud Firestore** en modo prueba.
4. Vincula la app: `flutterfire configure`.

---

## 3. Creación del Proyecto Flutter

Ejecuta en la terminal:
```bash
flutter create proyectoventatelefonos
cd proyectoventatelefonos
```

### Configuración de `pubspec.yaml`
Añade las dependencias necesarias:
```yaml
dependencies:
  flutter:
    sdk: flutter
  firebase_core: ^3.0.0
  cloud_firestore: ^5.0.0
  cupertino_icons: ^1.0.6
```
Luego corre: `flutter pub get`.

---

## 4. Estructura de Código y CRUD

Vamos a crear los archivos esenciales para el proyecto.

### A. Modelo de Datos (`lib/models/telefono.dart`)
```dart
class Telefono {
  String id;
  String marca;
  String modelo;
  double precio;

  Telefono({required this.id, required this.marca, required this.modelo, required this.precio});

  Map<String, dynamic> toMap() => {
    "marca": marca,
    "modelo": modelo,
    "precio": precio,
  };

  factory Telefono.fromFirestore(Map<String, dynamic> data, String id) {
    return Telefono(
      id: id,
      marca: data['marca'] ?? '',
      modelo: data['modelo'] ?? '',
      precio: (data['precio'] ?? 0).toDouble(),
    );
  }
}
```

### B. Servicio Firestore (`lib/services/firebase_service.dart`)
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/telefono.dart';

class FirebaseService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('ventas');

  // Create
  Future<void> addVenta(Telefono t) => _db.add(t.toMap());

  // Read
  Stream<List<Telefono>> getVentas() {
    return _db.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Telefono.fromFirestore(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // Update
  Future<void> updateVenta(String id, Telefono t) => _db.doc(id).update(t.toMap());

  // Delete
  Future<void> deleteVenta(String id) => _db.doc(id).delete();
}
```

### C. Pantalla de Listado (UI - `lib/screens/home_screen.dart`)


```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/telefono.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("Venta de Teléfonos - CRUD")),
      body: StreamBuilder<List<Telefono>>(
        stream: _service.getVentas(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          final ventas = snapshot.data!;
          return ListView.builder(
            itemCount: ventas.length,
            itemBuilder: (context, i) {
              return ListTile(
                title: Text("${ventas[i].marca} ${ventas[i].modelo}"),
                subtitle: Text("\$${ventas[i].precio}"),
                trailing: IconButton(
                  icon: Icon(Icons.delete, color: Colors.red),
                  onPressed: () => _service.deleteVenta(ventas[i].id),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () => _mostrarDialogo(context),
      ),
    );
  }

  void _mostrarDialogo(BuildContext context) {
    // Aquí iría el formulario para Crear (TextFields para marca, modelo, precio)
  }
}
```

---

## 5. Secuencia Lógica del Agente `.agents`

Para que tu agente funcione correctamente, sigue este orden:
1.  **Sync:** Conectar con Firebase (`firebase login`).
2.  **Scraping:** (Opcional) Ejecutar scripts en `/scripts` para obtener precios de Amazon/MercadoLibre.
3.  **Build:** Generar las pantallas de UI en `lib/screens/`.
4.  **Test:** Probar el CRUD desde el emulador.

¿Te gustaría que desarrolle el código completo del formulario de inserción (Create) o prefieres configurar primero la navegación entre páginas?
