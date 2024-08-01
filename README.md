# terraform-jenkins

OPENSHIFT

- Plataforma oriendata al desarrollo, con caracteristicas de Plataform as a Service.
- utiliza internamente Docker y Kubernetes
- Es una version de Kubernetes con mas funcionalidades. 
- Permite la integracion continua ya que permite mover contedores de un entorno a otro 

    PAAS: El usuario solo se encarga de la aplicacion y la data.
          el paas (runtime,middleware,OS,virtualizacion,networking,storage,servers)

- Ventajas sobre kubernetes: 
        - Source to image: Crear imagen que se puede desplegar por medio de codigo
        - Simplifica el ciclo de vida de las aplicaciones.
        - siempre el proyecto se guarda en repositorio
        - inyecta el codigo fuente en una img de docker en una nueva img
        - tolerancia a errores
        - escalabilidad dinamica
        - actualizaciones continuas
        - despleigues automaticos
        - enrutamiento a nuestras aplicaciones
        - balanceo de carga
        - volumenes persistentes
        - gestion mas sencilla de proyectos y usuarios
        - muchas imagenes base
        - flujos de ci / CD integradas
        - metricas y monitorizacion

- Version Enterprise: instalacion en privado, aws,azure, google, ibm cloud
- Version Comunity: okd 

ARQUITECTURA:
    Nodos: gestionan los pods
    master: se encargan de gestionar  seguridad, schedulling, replication, 
    registro: donde se guardan las imagenes
    almacenamiento:
    enrutamiento: conectar cluster con el mundo exterior







- Crear Security group que permita la entrada desde internet al elb .
			ec2 ---> security groups ---> crear :  
					INTERNET-TO-ELB , PERMITIR SOLICITUDES DESDE INTERNET. 
					TYPE: HTTP, ORIGEN : 00000	

- Crear Security Group   que permita la entrada desde ELB  al ECS 
			ec2 ---> security groups ---> crear :  
					EBL-TO-ECS, PERMITIR  ALL REQUEST DESDE ELB,
					Type: ALLTRAFIC, Origen: SELECCIONA EL SG  INTERNET-TO-ELB
					Crear otra regla para acceso SSH , 0000,

- Crear SG  desde mysql a ecs 
			ECS-TO-RDS, PERTMITIR REQUEST DESDE ECS, DESDE MYSQL,
			Typo:MYSQL,  Origen: ESCOGEMOS EL SG DEL ELB-TO-ECS
			Typo: MYSQL, Origen: 0.0.0.0 para acceso desde el bastion 

- Crear SG desde ecs to efs
			ECS-TO-EFS,perimit trafico desde ecs a efs, nfs al sg del ecs 
			type: NFS , origen: ELB-TO-ECS  
			type: NFS , origen: 0.0.0.0
			 
- Crear una instancia 
			BASTION-HOST
			ubuntu 22.04,micro, slecciona un sg que permita conectar por ssh 

- Crear un EFS
	 
	customize :   
			EFS-ECS
			standar
			deschuleo backups, encripcion
			netxt 
			escoger las AZ 1a,1b,1c   en SG seleccionamos ECS-TO-EFS 
			next
			create
			
-  entramos a la instancia BASTION-HOST 
				subir los archivos del wordpress 
				
				
				attachd efs al bastion
				   seleccionamos el efs, view details --> attach -- user guide ---> manually installing  --> linux distrubtions ----< ubuntu 
						sseguir pasos de instalar paquetes 
						     
						clonar proyecto de github
						cd efs-utils
						ejecutar comando
						  ****
						      
									To build and install a Debian package:

								$ sudo apt-get update
								$ sudo apt-get -y install git binutils rustc cargo pkg-config libssl-dev
								$ git clone https://github.com/aws/efs-utils
								$ cd efs-utils
								$ ./build-deb.sh
								$ sudo apt-get -y install ./build/amazon-efs-utils*deb 



								***
						
			montar el efs 
				mkdir /mnt/efs 


			montar el el fs (se saca el comando del details del efs.  " attach" --> algo parecido a: sudo mount -t efs -o tls fs-074ab342765c73d2f:/ efs
			
			
			copiar el archivo al /mnt/efs 
			cd /mnt/efs
			crear carpeta proyecto 
			descomprimir archov de wordpress y dejar todo en "wordpress"
			instalar el cliente mysql  : apt -y install mysql-common mysql-client 
			
			
- entrar a RDS 
		crear una bd 
				standart, mysql , free tieer, RDS-ECS, admin, Nequi2024, quitarautoscaling,  selecciono sg ECS-TO-RDS ,  
				CREATE DB 

- crear balanceador de carga 
		create alb 
				ALB-ECS 
				
				escojo las 3 zonas de disponibilidad
				en SG  INTERNET-TO-ELB  
				listenr puerto 80 
				target gropu : TG-ECS   ---< create target gropu 
				create load balancer 
				
				
				eliminar listener, y target gropu, ya que eso lo administra el propio ecs 
				
- ir a ECS ,  crear cluster ec2 linux 
       CLUSTER ECS 
	   t3.small
	   2
	   vpc por defecto
	   seleccione las 3 subnets 
	   sg: elb-to-ecs
		---> enable ip publica 
		
 ***** CREAR CLUSTER 
			seleccionamos fargate , ponemos el nombre y crear .
			
			
**** crear task definition
			vamos a definicion de tareas
			ponemos el nombre
			tipo: fargate 
			* en rol de tarea, creamos uno neuvi que permita ejecutar task de ecs 
			En Contenedor: 
					nombre: wordpress , imagen: wordpress:5.6.0-php7.4 
					variables de entorno: 
							WORDPRESS_DB_HOST
							WORDPRESS_DB_USER
							WORDPRESS_DB_PASSWORD
							WORDPRESS_DB_NAME
					
					En almacenamiento:		
								nombre:EFS-ECS
								Tipo de volumen: EFS 
								id del sistema de archivos: Seleccionamos el EFS que montamos 
								directorio raiz: /wordpress
								
					selecciono Mount points, selecciono el volumen y /var/www/html
					
					
					
	

	
- crear task dedinition  fagate  ,  TD- , network bridge 
		 add container WORDPRESS    , iamgen wordpress numero , ports 0 a 80,  variables de entorno:  WORDPRESS_DB_HOST, WORDPRESS_DB_PASSWORD,WORDPRESS_DB_NAME, WORDPRESS_DB_USER.
		 add 
		 en VLOLUMS : EFS-ECS type:EFS , seleccionamos  el EFS que creamos , path : /proyecto , add
		 
		 vuelvo al contendor y selecciono Mount points, selecciono el volumen y /var/www/html




- entrar a la RDS  y  copiar la url
        entrar al bastion  
		nos conectamos  a la rds : mysql -u admin -p -h "aca pego el dns de la rds" , entro el password
		
		creo la BD
				CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
				CREATE USER 'wordpressuser'@'%' IDENTIFIED BY 'password';
				FLUSH PRIVILEGES; 
		
		importar la bd 
			cd /mtf/efs/proyecto
			
			y donde esta el sql 
			
			mysql -u admin -p -h "dns de la rds" wordpress < archivo.sql 
			 
		    cambiar en la bd : 
			
			Cambio del WP OPTIONS en Wordpress:

SELECT * FROM wp_options WHERE option_name = 'home';
UPDATE wp_options SET option_value="ALB-ECS-1948917671.us-east-1.elb.amazonaws.com" WHERE option_name = "home";

SELECT * FROM wp_options WHERE option_name = 'siteurl';
UPDATE wp_options SET option_value="ALB-ECS-1948917671.us-east-1.elb.amazonaws.com" WHERE option_name = "siteurl";

		
-  volvemos a ecs  y llenamos los datos en la var de enviroment 


- crear tareas 

		
	   
				
				
		
		
		
		
		
		
		
		
			
			
			
			
			

							
						
						
						 
				
				
				
				
				
				
				
				
			
			
			
										
