# SIGD Firebase

Implementación del mismo modelo lógico con colecciones Firestore y archivos en Cloud Storage. Cada tabla se traduce a una colección raíz `{tabla}/{id}`; las relaciones mantienen IDs estables. `modelo/firebase-mapping.json` detalla las 58 colecciones y sus referencias. `modelo/SIGD_Modelado_BD_CORREGIDO_v2.pdf` contiene el diagrama corregido.

## Ejecutar con emuladores

Requiere Node.js 24 o posterior, Java compatible con Firebase Emulator Suite y Firebase CLI:

```powershell
Copy-Item .env.example .env
npm.cmd install
npx.cmd firebase-tools emulators:start --project demo-sigd
```

En otra terminal, dentro de este proyecto:

```powershell
npm.cmd run seed
npm.cmd run check
npm.cmd start
```

La API abre en `http://127.0.0.1:4204/api/health`. El frontend completo se encuentra en el proyecto `sigd-sistema` y puede usar este proveedor con las mismas variables de entorno.

Para un proyecto real, quitar las variables de emulador, indicar su ID y bucket, y usar Application Default Credentials o una cuenta de servicio fuera del repositorio. Cambiar la configuración de desarrollo antes de desplegar. No se ejecutó ningún despliegue real.

## Reglas y almacenamiento

Firestore y Storage están cerrados a acceso directo desde clientes. El backend utiliza Admin SDK con IAM y autoriza cada operación a través del servicio de dominio. Esta configuración corresponde al patrón de [backend exclusivo documentado por Firebase](https://firebase.google.com/docs/rules/insecure-rules). Una cuenta de Firebase conectada al asistente no implica que este proceso tenga una credencial IAM para el proyecto.

Los archivos se guardan en `evidence/{documentId}/{evidenceId}`. En Firestore se conservan nombre, MIME, tamaño, SHA-256 y ruta. La API acepta PDF, PNG y JPEG de hasta 5 MiB y valida su firma. Las reglas denegadas no sustituyen la autorización del servidor: Admin SDK no se rige por las mismas reglas del cliente.

Las transacciones leen antes de escribir y verifican referencias y unicidad desde el contrato común. Los IDs y las versiones son iguales a los del proveedor PostgreSQL. Este adaptador limita las operaciones a 400 cambios y 800 KiB por documento. No se guardan binarios en Firestore.

## Autenticación y límites

La API incluida usa credenciales con hash y sesiones del servidor. `users.auth_uid` permite vincular posteriormente Firebase Authentication, pero no se afirma que esa integración esté implementada. No hay autorregistro de docentes.

El repositorio central lee el conjunto de colecciones por transacción. Es una implementación del contrato para verificar el modelo y el flujo; antes de producción deben medirse coste y concurrencia y sustituir las lecturas globales por consultas acotadas. Las versiones históricas nunca se actualizan mediante el servicio.

```powershell
npm.cmd test
npm.cmd run test:provider
```

Las pruebas del dominio compartido fueron ejecutadas. Las pruebas contra emuladores o un proyecto Firebase real quedaron pendientes por falta de emuladores y credenciales en el entorno de ejecución. El código, reglas, índices y semilla están preparados; no se declara una base desplegada ni validada en la nube.
