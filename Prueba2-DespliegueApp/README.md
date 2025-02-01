# Prueba 2
---
## - Enunciado
**Despliegue de una aplicación Django y React.js** 
*Elaborar el deployment dockerizado de una aplicación en django (backend) con frontend en React.js contenida en el repositorio. Es necesario desplegar todos los servicios en un solo docker-compose.
Se deben entregar los Dockerfiles pertinentes para elaborar el despliegue y justificar la forma en la que elabora el deployment (supervisor, scripts, dockercompose, kubernetes, etc).
Subir todo lo elaborado a un repositorio (github, gitlab, bitbucket, etc). En el repositorio se debe incluir el código de la aplicación y un archivo README.md con instrucciones detalladas para compilar y desplegar la aplicación, tanto en una PC local como en la nube (AWS o GCP).*

---
## Requisitos
Tener instalado **Docker** y **Docker-Compose** en la **PC local** donde se ejecute el proyecto.

## Instructivo para el Deploy en PC local

- **1.** **Clonar** el repositorio "**Craftech-prueba-tecnica-2025**" -> *git clone url*
- **2.** Abrir la **terminal** y posicionarse en "**Prueba2-DespliegueApp**" --> *cd Prueba2-DespliegueApp*
- **3.** Ejecutar el comando **docker-compose build**
- **4.** Ejecutar el comando **docker-compose up -d**
- **5.** Abrir el **navegador** y dirigirse a "**http://localhost:3000/**"

---
## Requisitos
Tener AWS CLI instalado y configurado con *aws configure*

## Instructivo para el Deploy en AWS
Para el deploy en **AWS** se utilizará **Amazon Elastic COntainer Registry** (ECR) porque tiene integración directa con AWS ECS, lo que facilita el deploy y la configuración sin credenciales externas.

- **1.**  **Autenticarse** en AWS ECR
    >*aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com*
- **2.** Crear un **repositorio** en ECR
    >aws ecr create-repository --repository-name mi-app-frontend
    aws ecr create-repository --repository-name mi-app-backend
- **3.** **Construir** imágenes
    >docker build -t mi-app-frontend -f frontend/Dockerfile .
    docker build -t mi-app-backend -f backend/Dockerfile .
- **4.** **Etiquetar** imágenes
    >docker tag mi-app-frontend:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-frontend
    docker tag mi-app-backend:latest <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-backend
- **5.** **Subir** imágenes a ECR
    >docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-frontend
    docker push <aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-backend
- **6** Crear **amazon RDS** para la base de datos PostgreSQL
  - Ir a AWS RDS -> Crear base de datos
  - Seleccionar PostgreSQL
  - Definir nombre, usuario y contraseña.
  - Habilitar acceso público y guardar el endpoint que se genera.
- **7.** **Crear y Configurar Amazon ECS**
  - Ir a AWS ECS -> Crear cluster -> Elegir Fargate
  - Ir a Definiciones de Tareas -> Crear una nueva con 2 containers
  - Backend puerto 8000
    - Imagen: *<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-backend*
    - Variable de entorno:
    - >DATABASE_URL=postgresql://myuser:mypassword@mydatabase.abc123.us-east-1.rds.amazonaws.com:5432/mydatabase
  - Frontend puerto 80
    - Imagen: *<aws_account_id>.dkr.ecr.us-east-1.amazonaws.com/mi-app-frontend*
  - **Crear un servicio EC2 con Fargate**
- **8.** Copiar la URL generada por ECS y abrirla en el navegador
---
## Explicación de la solución propuesta

El frontend en React se levanta en **Nginx**, que servirá los archivos estáticos en el puerto **80** dentro del contenedor. Sin embargo, en el docker-compose.yaml, este puerto se expone como 3000 en el host.

El backend en Django se ejecuta en un contenedor utilizando **Gunicorn**, que escucha en el puerto **8000** dentro del contenedor.

Existía un archivo "docker-compose.yml" en "backend" (que borré del proyecto dado que integre todo en un solo docker-compose) que contenía información sobre que debe levantar un container para una base de datos postgres, por lo que respetaré esto y lo voy a mantener para la resolución del ejercicio.

Para el ejercicio voy a utilizar Docker para encapsular los servicios de la aplicación (frontend, backend y base de datos). Voy a utilizar un solo archivo docker-compose de manera en que todos los servicios se ejecuten con una sola línea en la terminal. (*Esta es la mayor ventaja que nos da docker-compose*).

Estuve analizando las posibles imagenes que puedo utilizar para buildear la aplicación. Comencé a indagar en **Docker Hub** para ver las diferentes alternativas.

Para **Node**, leí la documentación de este link (https://hub.docker.com/_/node) y determiné que, para un ambiente de pruebas, la versión **node** con **alpine** es la mejor opción para utilizar por su bajo peso. Posible mejora: Si se tratase de un ambiente productivo, quizás utilizaría una versión de node con Debian. Aunque utilizar alpine hará que los tiempos de buildeo sean mucho menores.
Para montar el servidor del front voy a utilizar **Nginx** ya que es un **servidor web ligero y eficiente**.

En el archivo **Dockerfile** para el caso del **frontend** con React, voy a aprovechar la estrategia **Multi-stage Building** para reducir el tamaño final de la imagen resultante. Para eso, voy a utilizar **node:18-alpine** para hacer el build de la aplicación react y luego una imagen **nginx-alpine** para que el servidor levante los archivos estáticos generados por la imagen de node. Esto hará que en el entorno de producción solo se levante la imagen de nginx con el archivo estático almacenado en su filesystem. En su dockerfile, divido la construcción de la imagen final en dos etapas. Etapa 1: Utiliza Node Alpine para compilar la aplicación React y en la etapa 2 utilizo una imagen Nginx que obtiene los archivos generados por la etapa 1 y los provee en el servidor.

En el archivo **Dockerfile** para Django en el **backend**, decidí optar por utilizar **python:3.10-slim**. El archivo dockerfile instalará las dependencias necesarias del sistema (el cliente PostreSQL) y luego instalará las dependencias de la aplicación Django almacenadas en requirements.txt. Tras ello, expondrá el puerto **8000** y ejecutará **Gunicorn** para levantar el servidor de la aplicación. 

Para el archivo **docker-compose**...
Defino los tres servicios: Para el backend la aplicación Django utilizando Gunicorn para el servidor. Para el frontend la aplicación React con nginx. Para la base de datos un servicio PostgreSQL.
Para que los tres contenedores se comuniquen, voy a necesitar crear una **red** que todos tengan compartida. Esta red de tipo bridge se llama app_network.
Para persistir los datos de la base de datos PostgreSQL voy a crear el **volumen** **postgres_data**.

*Nota:* Quité de los archivos .gitignore los archivos de entorno (.env) para que puedan replicar la ejecución de docker-compose.
