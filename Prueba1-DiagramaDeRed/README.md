# Prueba 1 - Diagrama de Red
---
## Enunciado
*Prueba 1 - Diagrama de Red Produzca un diagrama de red (puede utilizar
lucidchart) de una aplicación web en GCP o AWS y escriba una descripción de
texto de 1/2 a 1 página de sus elecciones y arquitectura.
El diseño debe soportar:
• Cargas variables
• Contar con HA (alta disponibilidad)
• Frontend en Js
• Backend con una base de datos relacional y una no relacional
• La aplicación backend consume 2 microservicios externos
El diagrama debe hacer un mejor uso de las soluciones distribuidas.*

---

## Descripción de la arquitectura

Se seleccionó a **AWS** para realizar el diagrama de red.
Se distribuyó la arquitectura en dos Availability Zones (AZs) dentro de la misma región para tolerancia a fallos, esto provee al sistema de **alta disponibilidad** **(HA)**. Si una AZ falla, la otra puede seguir operando sin interrupciones.

### - Almacenamiento y distribución de contenido estático

Utilizo un **Bucket S3** para almacenar el contenido estático **(frontend)** de la aplicación web. ***S3 es altamente escalable y ofrece baja latencia.***

Utilizo **DNS Resolution** con **Amazon Route 53** para mapear los recursos de la aplicación a los recursos de AWS.

Para distribuir el contenido estático a través de una red global de edge locations, se utiliza **CloudFront** en combinación con S3.

La combinación de Route 53 con CloudFront permite que Route 53 dirija el tráfico a CloudFront para que este sirva el contenido desde los edge locations más cercanos.

### - Balanceo de carga y escalabilidad

Para implementar **Escalabilidad**, utilizo **ALB (Application Load Balancer)** dentro de subredes públicas para balancear la carga en nuestros EC2. Esto permite **crecimiento horizontal** de nuestras instancias (Auto Scaling), como también seguridad al ejecutar servidores web en subredes privadas.

El **Auto Scaling** nos permite ajustar automáticamente el número de instancias EC2 dependiendo de la demanda.
### - Infraestructura de Red

Toda la arquitectura está contenida dentro de una **VPC** con un rango de red privado (10.0.0.0/16) para mayor seguridad y control.

**Subredes públicas:** Contienen el ALB y los NAT Gateways.

**Subredes privadas:** Albergan las instancias EC2 (backend) y la base de datos Amazon RDS, garantizando aislamiento.

Para el **consumo de APIs externas**, se dispuso de **NAT Gateways** en las subredes públicas de los AZ de manera en que los EC2 puedan tener comunicación con las APIs externas al pasar por el **Internet Gateway**.

Para establecer la conexión con el servicio DynamoDB de AWS se utilizan **VCP Endpoints** de manera en que no sea necesario salir a internet para establecer conexión, dando seguridad.

### - Bases de datos SQL y NoSQL

Utilizo una base de datos relacional **Amazon RDS** dado que provee de una **alta disponibilidad** al sistema al generar una réplica por cada AZ en caso de que la instancia principal se caiga ***(Multi AZ)***. Es redimensionable y automatiza tareas de aprovisionamiento de hardware, configuración de BD, parcheos, backups.

Para la base de datos no relacional, se optó por utilizar **DynamoDB**, una opción escalable y de baja latencia.

### - Seguridad y Monitoreo

Para tener **seguridad** frente a actividad maliciosa, proteger cuentas y datos utilizo el servicio de **Amazon GuardDuty**, el cual permite detectar anomalías y amenazas en mi red.

Utilizo **AWS Systems Manager** para ver, administrar y operar nodos de forma centralizada. En combinación con este, decidí instalar un agente de monitoreo de **Amazon CloudWatch Logs** para poder tomar los archivos de logeo de la aplicación y enviarlos a CloudWatch, de manera en que se puedan ver os recursos consumidos en tableros de control.
Tomando en cuenta que tenemos el conocimiento de como están nuestros recursos operacionales en la nube, decidí usar un **Amazon CloudWatch Alarms** para que nos alerten cuando se sobrepasa cierto nivel de consumo de los recursos.

### - Backups

Para automatizar y centralizar la gestión de las **copias de seguridad** utilizo **AWS Backup**. Asegura protección de los datos y disponibilidad al almacenar los datos en varios servicios de AWS.

---

## Referencias
- *Curso craftech:* https://youtu.be/ekvpWRqneDo?si=uR-eETpAO9rrR-Tc
- https://aws.amazon.com/es/blogs/aws-spanish/
- https://aws.amazon.com/es/blogs/aws-spanish/well-architected-transformando-una-arquitectura-tradicional-a-una-optimizada-para-computo-en-la-nube/
- https://aws.amazon.com/es/blogs/aws-spanish/entender-los-patrones-de-resiliencia-y-las-consideraciones-claves-para-arquitecturar-eficientemente-en-la-nube/
- https://aws.amazon.com/es/microservices/
- https://aws.amazon.com/es/cloudwatch/
- https://aws.amazon.com/es/route53/
- https://aws.amazon.com/es/rds/
- https://aws.amazon.com/es/dynamodb/
- https://aws.amazon.com/es/s3/
- https://aws.amazon.com/es/cloudfront/
- https://aws.amazon.com/es/guardduty/
- https://aws.amazon.com/es/backup/
- https://aws.amazon.com/es/ec2/

