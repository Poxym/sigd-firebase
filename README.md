# SIGD · Base de datos Firebase

Configuración de Firestore y Cloud Storage del Sistema Integral de Gestión Docente (SIGD), derivada del mismo modelo relacional que la versión PostgreSQL.

## Contenido

- `firestore.rules`: deniega todo acceso directo desde clientes. Los datos solo se leen y escriben a través de la API de `sigd-sistema` con Admin SDK, que aplica la autorización del dominio.
- `storage.rules`: también deniega el acceso directo al bucket de evidencias; los archivos se suben y descargan a través de la API.
- `firestore.indexes.json`: índices compuestos que usan las consultas del sistema.
- `firebase.json`: emuladores de Firestore (8080), Storage (9199) y UI (4000).
- `docs/firebase-mapping.json`: correspondencia tabla → colección y FK → referencia.
- `docs/diagrama-relacional/`: diagrama relacional en PDF y SVG.

## Uso local con emuladores

```bash
cp .firebaserc.example .firebaserc   # pon el id de tu proyecto
cp .env.example .env
firebase emulators:start
```

Para un proyecto real, quita las variables `*_EMULATOR_HOST` y usa credenciales de servicio fuera del repositorio (`GOOGLE_APPLICATION_CREDENTIALS`). Para desplegar reglas e índices: `firebase deploy --only firestore,storage`.
