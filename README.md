# Backend académico — PharmaSoft

API REST utilizada por PharmaMobile en la Guía Práctica de Laboratorio N.º 07. Esta copia se basa en [dreyna/pharmaSoft](https://github.com/dreyna/pharmaSoft); consulta [la procedencia del código](PROCEDENCIA.md). Luciana Gutiérrez mantiene este repositorio y documenta el entorno y las verificaciones realizadas.

## Requisitos y entorno local

- Java 21 (verificación realizada con 21.0.12).
- Wrapper Maven incluido: no requiere Maven instalado globalmente.
- Oracle disponible en `localhost:1522/FREEPDB1`, esquema `PHARMADB`.
- Imagen utilizada: `gvenzl/oracle-free:23.26.1-slim-faststart`, Oracle AI Database 26ai Free release 23.26.1.
- Contenedor utilizado: `pharmasoft-oracle`, con volumen persistente y puerto local 1522 hacia el puerto 1521 del contenedor.

Las migraciones incluyen columnas SQL BOOLEAN: no sustituir la imagen por una edición antigua que no admita este tipo. El contenedor y el volumen no se incluyen en Git; clonar el repositorio no instala Oracle ni recupera sus datos. Tampoco hay Docker Compose en este repositorio.

## Arranque y pruebas (PowerShell)

Antes de iniciar la API, arrancar Docker y el contenedor existente:

```powershell
docker start pharmasoft-oracle
docker exec pharmasoft-oracle healthcheck.sh
```

Esperar a que la comprobación termine correctamente y confirmar que Oracle admite la conexión del esquema configurado. Las tablas pertenecientes al proyecto se crean mediante Flyway, no manualmente.

```powershell
$env:JAVA_HOME = 'C:\Program Files\Java\jdk-21.0.12'
$env:PATH = "$env:JAVA_HOME\bin;C:\Windows\System32\WindowsPowerShell\v1.0;C:\Windows\System32;$env:PATH"
.\mvnw.cmd -version
.\mvnw.cmd test
.\mvnw.cmd spring-boot:run
```

Adaptar la ruta del JDK a la instalación local. El perfil por defecto es `dev`, con puerto 8080. Las pruebas de contexto requieren Oracle disponible. Flyway aplica `V1__esquema_inicial.sql`; Hibernate valida las entidades con `ddl-auto: validate`.

La configuración `dev` contiene credenciales académicas locales. No reutilizarlas ni exponer esta instancia en producción. El perfil `prod` recibe `DB_URL`, `DB_USERNAME` y `DB_PASSWORD` mediante variables de entorno. No incluir credenciales reales en documentación, capturas o commits.

## Endpoints de la práctica

- Health: http://localhost:8080/api/health
- Swagger: http://localhost:8080/swagger-ui.html
- Productos: `GET /api/v1/productos?pagina=0&tamanio=20&ordenarPor=id&direccion=asc`
- Categorías: `GET /api/v1/categorias`

El listado de productos devuelve un objeto paginado con `contenido`, `pagina`, `tamanio`, `totalElementos`, `totalPaginas` y `ultima`. No es una lista simple. Cada producto incluye `id`, `nombre`, `precio`, `stock`, `estado`, `categoriaId`, `categoriaNombre` y fechas.

Los datos de la práctica se crean desde Swagger: primero una categoría mediante POST, luego tres productos usando su ID. No ejecutar esos POST repetidamente: pueden producir conflictos o duplicados. Los datos viven en Oracle, no en el repositorio.

## Verificaciones y evidencias

El 4 de octubre de 2026 se verificaron compilación, 21 pruebas sin fallos ni errores, Flyway V1, health 200 y listado 200 con tres productos. Son verificaciones históricas de esa sesión, no una garantía de disponibilidad actual.

Las capturas y el cuerpo JSON se incorporan en `evidencias/sesion07/`, con su informe de resultados. La aplicación móvil y sus pruebas se mantienen en el repositorio independiente [PharmaMobile](https://github.com/LucianaGutierrez-049/PharmaMobile/tree/feature/ktor-client-Gutierrez). La ejecución iOS quedó pendiente de macOS/Xcode según el reporte de la integración.

La guía requiere consumo GET móvil; el POST móvil corresponde a la sesión 8. Este repositorio documenta el backend, no acredita por sí solo la práctica completa.

## Informe integrado

La copia `evidencias/Informe_Guia_Practica_07_Integrado.docx` reúne el informe móvil y la documentación del backend, con once imágenes de evidencia. Conserva los originales y corrige la fecha de verificación a 4 de octubre de 2026 según el índice de evidencias y el registro Ktor. iOS permanece pendiente.

El contenido y las imágenes de la copia se comprobaron estructuralmente. La revisión visual de sus páginas está pendiente: el renderizador empaquetado no pudo ejecutarse porque falta LibreOffice en el entorno de herramientas. Revisar el documento en Word antes de entregarlo al docente.
