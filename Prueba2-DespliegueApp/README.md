# Prueba 2 -  Despliegue de una aplicación Django y React.js
---
## Enunciado
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
- **5.** Abrir el **navegador** y dirigirse a "**http://localhost**"

---
## Requisitos
Tener una cuenta en **AWS**  para utilizar **AWS Console**.
Instructivo basado en el *free trier de AWS.*

## Instructivo para el Deploy en AWS

- **1.** **Crear una instancia EC2 en AWS**
  - Acceder a la cuenta de AWS
  - Ir a la consola EC2
  - Hacer click en "Launch Instance"
  - LLenar con los datos del EC2 
    - Nombre
    - OS (Para este instructivo elegí ubuntu)
    - t2.micro (free tier)
    - Crear ssh (.pem)
    - Almacenamiento (Default 8gb)
    - Grupos de seguridad (*puertos 80, 443, 22*)
- **2.** **Conectarse desde la terminal utilizando ssh**
  - *ssh -i "Craftech_Prueba2.pem" ubuntu@ec2-54-166-14-178.compute-1.amazonaws.com*
- **3. Instalar Docker y Docker Compose**
  - >sudo apt install -y docker.io
  - >sudo systemctl enable docker
  - >sudo systemctl start docker
  - >sudo usermod -aG docker $USER
  - >newgrp docker
  - >sudo apt install -y docker-compose

- **4.** **Clonar el repositorio y configurar el entorno**
  - >cd /home/ubuntu
  - >git clone https://github.com/LucianoLoyola/Craftech-prueba-tecnica-2025.git
  - >cd Craftech-prueba-tecnica-2025/Prueba2-DespliegueApp
- **5. Buildear la imagen y correr docker-compose**
  - >docker-compose up --build -d
- **6. Obtener dirección del sitio en AWS**
  - Lanzar el siguiente comando junto con la dirección IPv4 asignada por AWS
  - >host 54.166.14.178
  *178.14.166.54.in-addr.arpa domain name pointer ec2-54-166-14-178.compute-1.amazonaws.com.*
  - Es probable que el firewall bloquee el tráfico tcp, recomiendo ejecutar este comando para habilitarlo
  - >sudo ufw allow 5432/tcp

---
## Explicación de la solución propuesta

El frontend en React se levanta en **Nginx**, que servirá los archivos estáticos en el puerto **80** dentro del contenedor. 

El backend en Django se ejecuta en un contenedor utilizando **Gunicorn**, que escucha en el puerto **8000** dentro del contenedor.

Existía un archivo "docker-compose.yml" en "backend" (que borré del proyecto dado que integré todo en un solo docker-compose) que contenía información sobre que debe levantar un container para una base de datos postgres, por lo que respetaré esto y lo voy a mantener para la resolución del ejercicio.

Para el ejercicio voy a utilizar Docker para encapsular los servicios de la aplicación (frontend, backend y base de datos). Voy a utilizar un solo archivo docker-compose de manera en que todos los servicios se ejecuten con una sola línea en la terminal. (*Esta es la mayor ventaja que nos da docker-compose*).

Estuve analizando las posibles imagenes que puedo utilizar para buildear la aplicación. Comencé a indagar en **Docker Hub** para ver las diferentes alternativas.

Para **Node**, leí la documentación de este link (https://hub.docker.com/_/node) y determiné que, para un ambiente de pruebas, la versión **node:alpine** para el buildeo de la aplicación React es la mejor opción a utilizar por su bajo peso. *Posible mejora:* Si se tratase de un ambiente productivo, quizás utilizaría una versión de node con Debian. Aunque utilizar alpine hará que los tiempos de buildeo sean mucho menores.
Para montar el servidor del front voy a utilizar **Nginx-alpine** con  ya que es un **servidor web ligero y eficiente**.

En el archivo **Dockerfile** para el caso del **frontend** con React, voy a aprovechar la estrategia **Multi-stage Building** para reducir el tamaño final de la imagen resultante. Para eso, voy a utilizar **node:18-alpine** para hacer el build de la aplicación react y luego una imagen **nginx-alpine** para que el servidor levante los archivos estáticos generados por la imagen de node. Esto hará que en el entorno de producción solo se levante la imagen de nginx con el archivo estático almacenado en su filesystem. En el dockerfile, divido la construcción de la imagen final en dos etapas. Etapa 1: Utiliza Node Alpine para compilar la aplicación React y en la etapa 2 utilizo una imagen Nginx Alpine que obtiene los archivos generados por la etapa 1 y los provee en el servidor.

En el archivo **Dockerfile** para Django en el **backend**, decidí optar por utilizar **python:3.10-slim**, una versión ligera de Python para mejorar los tiempos de buildeo. El archivo dockerfile instalará las dependencias necesarias del sistema (por ejemplo: el cliente PostreSQL) y luego instalará las dependencias de la aplicación Django almacenadas en requirements.txt. Tras ello, expondrá el puerto **8000** y ejecutará **Gunicorn** para levantar el servidor de la aplicación. 

Para el archivo **docker-compose**...
Defino los tres servicios: Para el backend la aplicación Django utilizando Gunicorn para el servidor. Para el frontend la aplicación React con nginx. Para la base de datos un servicio PostgreSQL.
Para que los tres contenedores se comuniquen, voy a necesitar crear una **red** que todos tengan compartida. Esta red de tipo bridge se llama app_network.
Para persistir los datos de la base de datos PostgreSQL voy a crear el **volumen** **postgres_data**.

*Nota:* Quité de los archivos .gitignore los archivos de entorno (.env) para que puedan replicar la ejecución de docker-compose.

## Referencias
- *Clase Docker Craftech:* https://youtu.be/EkJXacEKNpA?si=JRweSL3L8dSSpJLv
- *Clase Docker-Compose Craftech:* https://youtu.be/to1ZUIjbNjs?si=FjgmMdzi8EipJwt4
- https://hub.docker.com/
- https://hub.docker.com/_/node
- https://hub.docker.com/_/python
- https://hub.docker.com/_/nginx