# SIGD Firebase

Implementación del mismo modelo lógico con colecciones Firestore y archivos en Cloud Storage. Cada tabla se traduce a una colección raíz `{tabla}/{id}`; las relaciones mantienen IDs estables. `modelo/firebase-mapping.json` detalla las 58 colecciones y sus referencias. El diagrama relacional oficial (notación pata de gallo, vista general y una lámina por dominio) está en `modelo/diagrama_relacional/SIGD_Diagrama_Relacional.pdf`; las láminas `modelo/SIGD_*_CORREGIDO_v2.pdf` son una versión anterior.

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

## Estado de validación (23/11/2026)

Ejecutado contra Firebase Emulator Suite (Firestore 8080, Storage 9199, `firebase-tools` 15.37):

| Comprobación | Resultado |
|---|---|
| `npm test` (dominio compartido) | 26/26 |
| `npm run test:provider` (transacción, relaciones, reversión) | 1/1 |
| `npm run seed` y `npm run check` | 58 colecciones, relaciones válidas |
| Cliente sin credenciales lee `users` | 403 (reglas cerradas) |
| Cliente sin credenciales lista Storage | 403 |
| Recorrido completo en navegador con `sigd-sistema` sobre Firestore (plan, envío, aprobación, informe, evidencia, Administración, Reportes) | 11/11 pasos, sin errores de JavaScript |

Cada petición tarda entre 0.7 y 2.3 s en el emulador porque el adaptador lee todas las colecciones por transacción; el recorrido necesita tiempos de espera amplios. No se probó un proyecto Firebase real en la nube ni carga concurrente.
