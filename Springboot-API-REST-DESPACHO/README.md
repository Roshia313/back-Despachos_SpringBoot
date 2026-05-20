# Backend - Despachos SmartLogix

API REST hecha con Spring Boot para gestionar los despachos de SmartLogix. Se conecta a una base de datos MySQL y se despliega como contenedor en una EC2 en la subred privada de AWS.

## Tecnologías

- Java 17 + Spring Boot 3
- Maven 3.9
- Spring Data JPA / Hibernate
- MySQL 8
- SpringDoc (Swagger UI)
- Docker

## Estructura

```
Springboot-API-REST-DESPACHO/
├── .github/workflows/deploy.yml
├── src/main/java/com/citt/
│   ├── controller/DespachoController.java
│   ├── persistence/
│   │   ├── entity/Despacho.java
│   │   ├── repository/
│   │   └── services/
│   ├── config/CorsConfig.java
│   └── exceptions/
├── src/main/resources/application.properties
├── Dockerfile
└── pom.xml
```

## Cómo correr el proyecto

### Con docker-compose (recomendado)

Desde la carpeta raíz donde está el `docker-compose.yml`:

```bash
docker-compose up --build
```

El servicio queda en: `http://localhost:8081`

### Solo el contenedor

Necesitas una instancia MySQL corriendo:

```bash
docker build -t smartlogix-back-despachos .

docker run -d --name back-despachos-app -p 8081:8081 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://<HOST>:3306/smartlogix_db?useSSL=false \
  -e SPRING_DATASOURCE_USERNAME=<usuario> \
  -e SPRING_DATASOURCE_PASSWORD=<contraseña> \
  smartlogix-back-despachos
```

### Swagger UI

Con el servicio corriendo:

```
http://localhost:8081/swagger-ui.html
```

## Variables de entorno

| Variable | Descripción |
|---|---|
| `SPRING_DATASOURCE_URL` | URL de conexión a MySQL |
| `SPRING_DATASOURCE_USERNAME` | Usuario de la base de datos |
| `SPRING_DATASOURCE_PASSWORD` | Contraseña de la base de datos |

Puerto expuesto: `8081`

## Dockerfile

Multi-stage build en dos etapas:

- **Etapa 1:** `maven:3.9.6-eclipse-temurin-17`, compila el proyecto con `mvn package` y genera el `.jar`
- **Etapa 2:** `eclipse-temurin:17-jre-alpine`, copia solo el `.jar`. Crea el usuario `spring` y corre el proceso sin privilegios de root

## Persistencia

Los datos se guardan en MySQL. El volumen `mysql_data` definido en el `docker-compose.yml` asegura que la base de datos no se pierda si se reinician los contenedores.

Se usó **named volume** en lugar de bind mount porque Docker lo gestiona de forma independiente al sistema de archivos del host, lo que facilita el manejo en producción y evita problemas de permisos.

## Pipeline CI/CD

Se activa con push a la rama `deploy`.

Pasos:
1. Checkout del código
2. Configura credenciales AWS
3. Login en Amazon ECR
4. Build y push de la imagen (repositorio: `back-despachos`, tag: `latest`)
5. SSH a la EC2 del backend
6. Pull y reemplazo del contenedor con las variables de BD

Secrets necesarios:

| Secret | Descripción |
|---|---|
| `AWS_ACCESS_KEY_ID` | Credencial AWS Academy |
| `AWS_SECRET_ACCESS_KEY` | Credencial AWS Academy |
| `AWS_SESSION_TOKEN` | Token de sesión AWS Academy |
| `EC2_BACKEND_HOST` | IP de la EC2 del backend |
| `EC2_SSH_KEY` | Clave privada PEM para SSH |
| `DB_URL` | URL de la base de datos en producción |
| `DB_USERNAME` | Usuario de la BD en producción |
| `DB_PASSWORD` | Contraseña de la BD en producción |

## Notas

- El proceso corre como usuario `spring`, no como root
- Las credenciales van solo en GitHub Secrets, nunca en el código
- El servicio no es accesible desde internet, solo desde el frontend según los Security Groups