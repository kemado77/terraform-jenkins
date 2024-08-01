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




