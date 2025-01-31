# Diagrama de Red
Se seleccionó a **AWS** para realizar el diagrama de red.

Utilizo un **Bucket S3** para almacenar el contenido estático **(frontend)** de la aplicación web. ***S3 es altamente escalable y ofrece bajas latencias.***

Para distribuir el contenido estático a través de una red global de edge locations, se utiliza **CloudFront** en combinación con S3.

El **Auto Scaling** nos permite ajustar automáticamente el número de instancias EC2 dependiendo de la demanda.

El **balanceador de carga** va a distribuir el tráfico entre las instancias EC2. Esto mejora la ***disponibilidad del sistema, la tolerancia a fallos y a manejar cargas variables*** distribuyendo las solicitudes entre los servidores.

Utilizo una base de datos relacional **Amazon RDS** dado que provee de una **alta disponibilidad** al sistema al generar una réplica por cada AZ en caso de que la instancia principal se caiga ***(Multi AZ)***.

Para la base de datos no relacional, se optó por utilizar **DynamoDB**, una opción escalable y de alto rendimiento.

Para establecer la conexión con el servicio DynamoDB de AWS se utilizan **VCP Endpoints** de manera en que no sea necesario salir a internet para establecer conexión, dando seguridad.

Para el **consumo de APIs externas**, se dispuso de **NAT Gateways** en las subredes públicas de los AZ de manera en que los EC2 puedan tener comunicación con las APIs externas al pasar por el **Internet Gateway**.

**Posibles mejoras:** Si se desea incrementar la escalabilidad del sistema, se podría intercambiar la base de datos relacional Amazon RDS por una Aurora.
