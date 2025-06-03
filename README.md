# IAC
El componente de IaC se fue desarrollado en formato yaml para posteriormente ser desplegado en CloudFormation. Para su desarrollo se tuvo en cuenta el diagrama anteriormente presentado. Entre los aspectos clave se encuentran:

## SUBSECCIÓN DE PARÁMETROS
En esta subsección se definen los diferentes parámetros requeridos para el montaje de la infraestructura. Entre estos aspectos se encuentran:

-	CIRD de VPC: 10.0.0.0/16
-	CIRD de subnets: Siendo 5, van desde CIRD de VPC: 10.0.1.0/24 a 10.0.5.0/24
-	Tipos de instancias: 
•	db.t3.micro para base de datos
•	t2.medium para instancias del webserver.
-	Tamaño mínimo y máximo del Autoscaling group: De 2 a 6 instancias.
-	Nombre y motor de base de datos: aplicaciodb, soportando mysql y postgres. 
-	Usuario y contraseña de la base de datos: admin, password.
-	Keypair: Usada para el acceso al bastión host y las instancias webserver. Se uso el key-pair provisto por el laboratorio (vockey).
 
## SUBSECCIÓN DE RECURSOS
En esta subsección se detallan los componentes necesarios para la infraestructura usada (security groups, route tables, targets groups, etc), que permiten el correcto desenvolvimiento de la solución, entre estos se encuentran:

-	Creación de la VPC: Requiriendo el parámetro CIRD.
-	Creación del InternetGateway
-	Asociación de InternetGateway con la VPC.
-	Creación de subnets de acuerdo con CIRDs.
-	Creación de rutas públicas para el acceso de subnets públicas a internet.
-	Creación de rutas privadas para las subnets privadas.
-	Creación del NatGateway (Asociandolo con las subnets privadas).
-	Asociación del NatGateway con InternetGateway para el acceso de las instancias privadas a internet. 
-	Creación del grupo de seguridad del Bastión Host: Permitiendo el acceso mediante ssh (22) desde cualquier ip.
-	Creación del grupo de seguridad del webserver: Permitiendo tráfico http (80), https (443), ssh (22) únicamente desde el bastión host.
-	Creación del grupo de seguridad del Aplication Load Balancer: Permitiendo tráfico http (80), https (443).
-	Creación del grupo de seguridad de base de datos: Permitiendo tráfico Mysql (3306) y Postgres (5432).
-	Creación del Application Load Balancer con parámetros ya definidos.
-	Creación del Listener del ALB: Para escuchar a los webservers (ngnix puerto 80).
-	Creación del Target Group: Grupo de instancias de EC2 a las que apunta el ALB, realiza chequeos para verificar cuando las instancias se encuentren healthy.
-	Creación de la plantilla del web server: Plantilla donde se configura la aplicación (webserver).
-	Creación del Auto Scaling group del web server: Donde crean las instancias de la aplicación web (web server), usando los parámetros definida y la plantilla anteriormente configurada.
-	Creación de la política de escalado: Para definir cuando se deben escalar las instancias (en nuestro caso uso promedio de CPU>50%)
-	Creación de la base de datos: Es asociada a 2 subnets debido las restricciones de AWS. Sin embargo, se crea únicamente una instancia en la primera zona de disponibilidad; para esto último, se emplean los parámetros previamente definidos.
-	Creación del bastión host: Se crea el bastión host usando un tipo de instancia t3.micro y parámetros ya definidos.
-	Creación de alarmas de CloudWatch: Se configuran 2 alarmas:
•	La primera está atenta al Autoscaling Webserver por si el uso de CPU excede el 50% en un periodo de un minuto.
•	La segunda está atenta al Target Group del ALB por si el numero de instancias unhealthy es mayor que 0.
-	Creación de SNS: Se configura un tópico y una subscripción de email a SNS, para enviar notificaciones en caso de que alguna de las anteriores alarmas se active.
-	Creación y configuración de bucket de CloudTrail: Se crea un bucket donde se almacene toda la información referente a los logs de la cuenta actual recopilados por CloudTrail.
-	Creación de dashboard de CloudWatch: Se crea dashboards personalizados que buscan analizar el comportamiento de las instancias de EC2 y base de datos, así como como el ALB.

## SUBSECCIÓN DE SALIDAS
En esta subsección, se encuentran las diferentes salidas que en caso de ser requeridas pueden ser usadas por otros recursos de AWS, entre esas se encuentran:

-	Id de la VPC.
-	Id de las subnets (Privadas y públicas).
-	Id del grupo de seguridad del webserver.
-	URL del ALB.
-	Endpoint de la base de datos.
-	Ip del bastión host.

