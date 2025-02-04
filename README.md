# Craftech prueba tecnica 2025
---

Dentro de cada una de las carpetas se encuentra la resolución a las Pruebas 1,2 y 3 de la **prueba técnica de Craftech** para el puesto **DevOps Engineer Trainee**. Cada carpeta contiene su propio archivo *README* con enunciado, explicación de lo realizado y referencias.

A modo de resumen, haré una breve explicación de lo realizado en cada una de las pruebas. Para más detalles observar los README dentro de cada una de las carpetas.

---
## Prueba 1

Se diseñó una arquitectura en AWS con alta disponibilidad (HA), distribuyendo recursos en dos Availability Zones (AZs). El frontend estático se aloja en S3 con distribución mediante CloudFront y resolución DNS con Route 53. Se implementó Application Load Balancer (ALB) y Auto Scaling para balanceo de carga y escalabilidad.

El backend se ejecuta en EC2 con acceso a bases de datos Amazon RDS (SQL) y DynamoDB (NoSQL). La seguridad se maneja con GuardDuty y monitoreo con CloudWatch. El acceso a APIs externas se realiza a través de NAT Gateways y VPC Endpoints. AWS Backup se usa para copias de seguridad automatizadas.

---
## Prueba 2
Se desarrolló un despliegue con Docker y Docker Compose, integrando Django (backend) con Gunicorn, React (frontend) con Nginx, y PostgreSQL como base de datos.

Se implementó Multi-stage Building para reducir el peso de las imágenes y se definió una red interna para la comunicación entre servicios.

---
## Prueba 3

Se implementaron dos soluciones de Pipeline CI/CD para que, tras un cambio en index.html, buildee una imagen Docker y la suba a Docker Hub, para luego realiza el deploy de la nueva versión.

En la **Solución 1**, el deploy se hace en el propio **runner del pipeline** con Docker Compose.
En la **Solución 2**, el deploy se hace en una instancia **EC2 de AWS** usando SSH.

Los pipelines se dividen en dos etapas:
- ***Etapa 1 - Build:*** Obtiene el código, configura Docker Buildx, autentica en Docker Hub, buildea y sube la imagen.
- ***Etapa 2 - Deploy:*** Obtiene el código y realiza el despliegue.




## Conclusión

El desarrollo de esta prueba técnica me permitió aplicar todos los conocimientos adquiridos tras realizar el curso DevOps de Craftech, mejorando mi compresión sobre infraestructura en cloud, contenedor y automatización de deploys.

Cada desafío me permitió reforzar mis habilidades en escenarios que reflejan el día a día de un DevOps. Además, implementé una **metodología organizada**, utilizando un tablón de **Miro** para la planificación y una **bitácora de avances** para documentar el proceso.

El resultado final es un conjunto de soluciones funcionales que cumplen con los requerimientos planteados, dejándome con una gran satisfacción y motivación para seguir profundizando en el mundo DevOps.