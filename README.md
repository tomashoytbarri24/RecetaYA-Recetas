# Recetas — Receta Ya

Repositorio local independiente del microservicio de recetas médicas. Usa Java 21, Spring Boot 4.1.1, H2, Flyway y JWT emitidos por Auth. Escucha en `127.0.0.1:8082`.

## Requisitos de la evaluación

La pauta pide mostrar rutas REST, capas internas, entidades JPA y relaciones, dependencias de `pom.xml`, ramas Git, pruebas en Postman, conexión a la base relacional, comandos Maven y arranque reproducible. El video conjunto debe durar de 3 a 8 minutos y mostrar resultados reales con audio claro.

Este servicio permite crear, listar, consultar, actualizar y anular recetas. La anulación es una operación de negocio; no hay un `DELETE` físico de recetas. En la demostración local con SQS deshabilitado, una receta creada queda `ACEPTADA_PENDIENTE_RESERVA`; esto no significa que se haya reservado stock.

## Obtener el código

Este repositorio **aún no tiene remoto de GitHub**. Desde su raíz local se puede comprobar la clonación con los siguientes comandos. Tras publicarlo, sustituir `.` por su URL real.

### Windows (PowerShell)

```powershell
git clone . ..\recetas-copia
Set-Location ..\recetas-copia
git switch develop
java -version
```

### macOS y Linux (Bash)

```bash
git clone . ../recetas-copia
cd ../recetas-copia
git switch develop
java -version
```

Se necesita JDK 21, Git, puerto `8082` libre y acceso a Maven Central para descargar dependencias la primera vez. Maven Wrapper está incluido; en macOS y Linux se invoca con `bash ./mvnw` porque no está versionado como ejecutable.

## Compilar, probar y empaquetar

Desde la raíz de este repositorio, donde está `pom.xml`:

```powershell
# Windows
.\mvnw.cmd clean verify
Get-Item .\target\recetas-0.0.1-SNAPSHOT.jar
```

```bash
# macOS y Linux
bash ./mvnw clean verify
ls -lh target/recetas-0.0.1-SNAPSHOT.jar
```

`verify` compila, ejecuta las pruebas y empaqueta. Para la evidencia IE7 también se puede ejecutar `clean package`. Mostrar `BUILD SUCCESS` y el JAR generado. Detener el servicio antes de `clean` si el JAR está en uso.

## Configurar y ejecutar

Auth y Recetas deben usar el **mismo** emisor y secreto JWT. Generar un secreto local aleatorio de al menos 32 caracteres UTF-8 fuera de cámara; no incluirlo en archivos versionados. Para la demostración local, SQS permanece deshabilitado.

```powershell
# Windows: definir en la terminal de Recetas
$env:RECETAYA_JWT_EMISOR = 'http://localhost:8081'
$env:RECETAYA_JWT_SECRETO = Read-Host 'El mismo secreto JWT usado en Auth'
$env:RECETAYA_SQS_HABILITADA = 'false'
java -jar .\target\recetas-0.0.1-SNAPSHOT.jar
```

```bash
# macOS y Linux: definir en la terminal de Recetas
export RECETAYA_JWT_EMISOR='http://localhost:8081'
read -r -s -p 'El mismo secreto JWT usado en Auth: ' RECETAYA_JWT_SECRETO; printf '\n'
export RECETAYA_JWT_SECRETO
export RECETAYA_SQS_HABILITADA=false
java -jar target/recetas-0.0.1-SNAPSHOT.jar
```

Ejecutar desde la raíz del repositorio: `jdbc:h2:file:./data/recetas-recetaya` es relativa al directorio actual. Verificar `GET http://localhost:8082/actuator/health`: debe responder `200` y `UP`.

## Pruebas en Postman

Primero iniciar Auth, obtener un JWT de un usuario `MEDICO` y preparar en Inventario una sucursal y un medicamento con existencias. Usar solo datos ficticios y ocultar el token.

| Método | Ruta | Resultado esperado |
|---|---|---|
| `POST` | `/api/v1/recetas` | `201`, receta creada. |
| `GET` | `/api/v1/recetas` | `200`, recetas propias del médico. |
| `GET` | `/api/v1/recetas/{id}` | `200` o `404`. |
| `PUT` | `/api/v1/recetas/{id}` | Actualiza una receta cuando sus reglas de estado lo permiten. |
| `POST` | `/api/v1/recetas/{id}/anular` | Anula una receta cuando sus reglas de estado lo permiten. |

Ejemplo de cuerpo para `POST`, reemplazando los UUID por los devueltos por Inventario:

```json
{
  "paciente": {"documento": "PAC-001", "nombre": "Paciente ficticio"},
  "sucursalId": "<UUID de sucursal>",
  "medicamentos": [{"medicamentoId": "<UUID de medicamento>", "cantidad": 1}]
}
```

Mostrar la respuesta `201` y consultar la receta con `GET` (`200`). Con SQS deshabilitado, comprobar el estado `ACEPTADA_PENDIENTE_RESERVA`. Mostrar también una solicitud inválida o un identificador inexistente y su código/mensaje real.

## Base de datos y estructura

- `pom.xml` incluye Web MVC, Security, JPA, H2, Flyway, Validation, Actuator, Jackson, SQS y pruebas.
- `src/main/resources/application.yaml` configura la URL H2 de archivo, Flyway y `ddl-auto: validate`.
- `src/main/resources/bd/migracion/` crea pacientes, recetas, detalles y tablas de integración.
- `entidad/Receta.java` se relaciona con `Paciente` y sus detalles mediante JPA; `repositorio/`, `servicio/` y `controlador/` separan persistencia, reglas y API.

La consola H2 de este servicio está deshabilitada en la configuración actual. Para mostrar persistencia, crear una receta por la API, consultarla por ID y explicar la configuración y migraciones. No presentar la reserva pendiente como confirmada.

## Ramas y entrega

Las ramas locales `main`, `develop` y `feature/recetas-service` se crearon para continuar el desarrollo separado. El historial del código se extrajo del repositorio original; no se ha configurado remoto ni se ha publicado este repositorio. Antes de entregar, completar y comprobar su URL de GitHub, mostrar `git branch -a` y `git log --oneline`, y aportar el enlace real en AVA y al docente.
