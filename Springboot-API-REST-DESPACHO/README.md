## Cómo correr el proyecto
### Con docker-compose (recomendado)
Desde la carpeta raíz donde está el `docker-compose.yml`:
```bash
docker-compose up --build
```
El servicio queda en: `http://localhost:8081`

### Solo el contenedor
```bash
docker build -t smartlogix-back-despachos .
docker run -d --name back-despachos-app -p 8081:8081 \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://<HOST>:3306/smartlogix_db?useSSL=false \
  -e SPRING_DATASOURCE_USERNAME=<usuario> \
  -e SPRING_DATASOURCE_PASSWORD=<contraseña> \
  smartlogix-back-despachos
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
- **Etapa 1:** `maven:3.9.6-eclipse-temurin-17`, compila con `mvn package` y genera el `.jar`
- **Etapa 2:** `eclipse-temurin:17-jre-alpine`, copia solo el `.jar`. Crea el usuario `spring` y corre sin privilegios de root

## Persistencia
Los datos van a MySQL. El volumen `mysql_data` del `docker-compose.yml` garantiza que no se pierda información al reiniciar los contenedores.
Se optó por **named volume** porque es manejado directamente por Docker, no depende de rutas del host y es más fácil de manejar en producción que un bind mount.

## Pipeline CI/CD
Se activa con push a la rama `deploy`.
Pasos:
1. Checkout del código
2. Configura credenciales AWS
3. Login en Amazon ECR
4. Build y push de la imagen (repositorio: `back-despachos`, tag: `latest`)
5. Deploy en EC2 vía SSH

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