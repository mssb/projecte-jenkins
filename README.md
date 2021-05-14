# Proyecto Jenkins

[![Jenkins photo][jenkins-photo]][jenkins-url]

<br><br>

**Autores:**  Christian Manalo y Mark Santiago  

**Curso:** 2º de Administración de sistemas informáticos en redes

**Centro:** INS Escola del Treball


### [Proyecto en GitHub](https://github.com/isx47328890/projecte-jenkins)

### [Proyecto en Taiga](https://tree.taiga.io/project/isx47328890-projecte-jenkins/timeline)

### [Servidor Jenkins](http://3.213.6.243:8080/)

### [Servidor SonarQube](http://3.213.6.243:9000/)


---

# Índice

- **[Estudio inicial](#estudio-inicial)**
- **[Diseño del sistema](#diseño-del-sistema)**
- **[Análisis del sistema y conceptos](#análisis-del-sistema-y-conceptos)**
  - **[Jenkins](#qué-es-jenkins)**
  - **[SonarQube](#qué-es-sonarqube)**
- **[Instalación](#instalación)**
  - **[Docker Compose](#docker-compose)**
  - **[Configuración Jenkins](#configuración-jenkins)**
  - **[Configuración Sonarqube](#configuración-sonarqube)**
  - **[Integración Jenkins y Sonarqube](#integración-jenkins-y-sonarqube)**
- **[Mantenimiento](#mantenimiento)**
- **[Conclusiones](#conclusiones)**
- **[Bibliografía](#bibliografía)**

---

## Estudio inicial

El proyecto consistirá en el desarrollo de un sistema automatizado de integración continua con **Jenkins**. La tarea o "*job*" de Jenkins desplegará un repositorio **Git** en un servidor y dentro de este proceso habrá diferentes etapas o "*stages*" de la cual cogerá los diferentes códigos del repositorio y será evaluada a través de **SonarQube** para saber la calidad del código, el cual devolverá un feedback.

---

## Diseño del sistema

![System photo][system-photo]

---

## Análisis del sistema y conceptos

El sistema automatizado de entrega continua estará montado en tres contenedores Docker de las cuales serán **Jenkins**, **Sonarqube** y **PostgreSQL**. 

**Jenkins** clonará los códigos de **Git** y contactará con Sonarqube para evaluarlos. **Sonarqube** tiene un servidor web para poder visualizar los escaneos.

Dentro de **PostgreSQL** se guardaran los escaneos de código hechos por **Sonarqube**, y dentro de este también estará la configuración de la instancia del servidor web.

<br><br>

#### ¿Qué es Jenkins?
Es un servidor automatizado de integración continua de código abierto, es capaz de organizar una cadena de acciones que ayudan a lograr el proceso de integración continua de manera automatizada,  está completamente escrito en **Java**.

Jenkins puede instalarse a través de un sistema de gestión de paquetes, Docker o incluso ejecutarse de forma independiente en cualquier máquina que tenga instalado Java Runtime Environment (JRE).

Las compañías de software pueden acelerar su proceso de desarrollo del código, ya que este puede automatizar, agilizar y aumentar el ritmo de toda la compilación y las pruebas de los proyectos.

Más información sobre Jenkins [aquí](https://github.com/isx47328890/projecte-jenkins/blob/main/docs/jenkins.md)

<br><br>

#### ¿Qué es SonarQube? 
Es una plataforma de software libre para evaluar la calidad de nuestro código, realizando un análisis estático sobre dicho código, con el objetivo de advertirnos sobre diferentes puntos a mejorar.

Estos son los lenguajes soportados:

![Sonarqube photo][sonarqube-photo]

Más información sobre SonarQube [aquí](https://github.com/isx47328890/projecte-jenkins/blob/main/docs/sonarqube.md)

---

## Instalación

El proyecto estará montado en 3 containers Docker diferentes, uno para Jenkins, otro para SonarQube y otro para PostgreSQL. 

Antes de arrancar los containers debemos establecer unos parámetros para que el Sonarqube no esté limitado ya que utiliza `Elasticsearch`.

```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

### Docker

Para instalar Docker y todos sus componentes acceder a este [enlace](https://docs.docker.com/engine/install/)

### Docker Compose
Para arrancar los contenedores utilizado la herramienta docker-compose para poner en marcha todo lo necesario para que funcione.

```shell
docker-compose -f docker/Jenkins-Sonar/docker-compose.yaml up -d
```

El contenido de este `docker-compose.yaml` es el siguiente:

```yaml
version: "2"
services:
  jenkins:
    image: jenkins/jenkins:lts
    ports:
      - "8080:8080"
    networks:
      - hisx2_net
    volumes:
      - jenkins-data:/var/jenkins_home
  
  sonarqube:
    image: sonarqube:latest
    depends_on:
      - db
      - jenkins
    ports:
      - "9000:9000"
    networks:
      - hisx2_net
    environment:
      - SONAR_JDBC_URL=jdbc:postgresql://db:5432/sonar
      - SONAR_JDBC_USERNAME=sonar
      - SONAR_JDBC_PASSWORD=sonar
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_temp:/opt/sonarqube/temp
    
  db:
    image: postgres:latest
    networks:
      - hisx2_net
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    volumes:
      - postgresql:/var/lib/postgreql
      - postgresql_data:/var/lib/postgresql/data

networks:
  hisx2_net:

volumes:
  jenkins-data:

  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_temp:

  postgresql:
  postgresql_data:
```

Comprobamos que estan todas encendidas con `docker ps`.
```
CONTAINER ID        IMAGE                 COMMAND                  CREATED             STATUS              PORTS                               NAMES
b21b62117152        sonarqube:latest      "bin/run.sh bin/sona…"   7 seconds ago       Up 6 seconds        0.0.0.0:9000->9000/tcp              jenkins-sonar_sonarqube_1
f52323e5d55a        jenkins/jenkins:lts   "/sbin/tini -- /usr/…"   35 seconds ago      Up 7 seconds        0.0.0.0:8080->8080/tcp, 50000/tcp   jenkins-sonar_jenkins_1
e6bb9a5763e8        postgres:latest       "docker-entrypoint.s…"   35 seconds ago      Up 31 seconds       5432/tcp                            jenkins-sonar_db_1

```

### Configuración Jenkins

Desde el navegador accedemos en la siguiente URL y nos aparecerá la siguiente página:
```
http://localhost:8080
```

![Jenkins Admin Password photo][Jenkins-passwd]

Para conseguir la contraseña tenemos dos opciones, entrar en el contenedor para hacer un `cat` del siguiente archivo `/var/jenkins_home/secrets/initialAdminPassword` o hacer un `docker logs`.

```shell
docker logs jenkins-sonar_jenkins_1
```

Solo debemos buscar la password, nos saldrá lo siguiente:
```
*************************************************************
*************************************************************
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

87396ae5e3c449adb588f0401804924a

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

Lo siguiente es instalar los plugins de Jenkins, podemos customizarlo a nuestro gusto o simplemente elegir `Install suggested plugins`. 

Para que sea una instalación rapida elegiré `Install suggested plugins`.

![Jenkins Custom photo][Jenkins-custom]

Empezará a instalarse los plugins.

![Jenkins Plugins photo][Jenkins-plugins]

Lo siguiente nos pedirá la nueva contraseña del administrador, podemos poner lo que queramos.

![Jenkins Custom Passwd photo][Jenkins-AdminPasswd]

Y por último nos saldrá la configuracion del DNS, pero en nuestro caso lo dejaremos como está.

![Jenkins URL photo][Jenkins-DNS]

Finalizar instalación Jenkins.

![Jenkins Ready photo][Jenkins-ready]

### Configuración Sonarqube

Desde el navegador accedemos en la siguiente URL y nos aparecerá la siguiente página:
```
http://localhost:9000
```

El usuario y la contraseña es `admin` ya que es la que está por defecto.

![Sonarqube Admin Password photo][Sonarqube-admin]

Nos pedirá en cambiar la contraseña por cuestiones de seguridad. Y la configuración ya estaria hecha.

![Sonarqube Admin New Password photo][Sonarqube-newAdminPasswd]


### Integración Jenkins y Sonarqube

Primero de todo debemos instalar el plugin de Sonarqube en Jenkins, para ello debemos ir a `Dashboard > Manage Jenkins > Manage Plugins`.

![Jenkin Sonarqube Plugin Photo][Jenkins-SonarPlugin]


Configurar la ruta Home de Sonarqube `Dashboard > Manage Jenkins > Global tool configuration > Sonarqube Scanner`. Pero antes debemos crear una carpeta dentro del container de jenkins.

```
docker exec -it -u root jenkins-sonar_jenkins_1 mkdir -p /var/jenkins_home/tools/sonar-scanner
```

![Jenkin Sonarqube Home Photo][Jenkins-SonarInstall]

Instalamos el sonar-scanner en del contenedor de Jenkins.

Descargamos el archivo zip.
```
docker exec -it -u root jenkins-sonar_jenkins_1 wget -P /var/jenkins_home/ https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.1.2450-linux.zip

```

Descomprimimos el archivo en `/var/jenkins_home/tools/`.
```
docker exec -it -u root jenkins-sonar_jenkins_1 unzip /var/jenkins_home/sonar-scanner-cli-4.6.1.2450-linux.zip -d /var/jenkins_home/tools/
```

Movemos todos los archivos de `sonar-scanner-4.6.1.2450-linux` a `sonar-scanner`.
```
docker exec -it -u root jenkins-sonar_jenkins_1 mv /var/jenkins_home/tools/sonar-scanner-4.6.1.2450-linux/* /var/jenkins_home/tools/sonar-scanner/.

```


Para la integración de estos dos servidores se necesita un token de autenticación. Tenemos que asegurarnos de guardar el token ya que mas adelante lo necesitaremos.

Dentro de Sonarqube `My Account > Security > Generate Token`.

![Sonarqube Token Photo][Sonarqube-Token]


Ahora toca añadir el token dentro del servidor de Jenkins. `Manage Jenkins > Configure Systems > Sonarqube Servers`.
Pulsamos en el icono de `Add`.  

```
Kind: Secret Text
Secret: token_generado_en_sonarqube
Description: SonarQube Server
```

![Jenkins Token Integration Photo][Jenkins-Token]


Y por último seleccionar el Server authentication token `SonarQube Server`
```
Name: SonarQube
Server URL: 
```
![Jenkins Token Integration Photo 2][Jenkins-Token2]

---

### Pipelines

Ahora crearemos la tarea que consistirá en un pipeline. En la página principal accedemos a `New item`, como vemos en la imagen.

![Jenkins New Item][Jenkins-newitem]

A continuación, seleccionaremos el tipo de tarea que queremos. En este caso, utilizaremos un Pipeline como hemos comentado anteriormente.

![Jenkins Job][Jenkins-job]

En la zona inferior de la página de configuración del pipeline podremos introducir nuestro script, el cual será el siguiente:

```java
def listusers = "alumne1 alumne2 alumne3"
def user = listusers.split(" ")

pipeline {
    agent any
    stages {
        stage ('Clone repositories'){
            steps {
                script {
                    for (i in user) {
                        def exists = fileExists "${i}"
                        if (!exists){
                            new File("${i}").mkdir()
                            dir ("${i}") {
                            git url: "https://gitlab.com/2daw2020/${i}"
                            }
                        }
                        else {
                            dir ("${i}"){
                                sh "git pull origin master"
                            }
                        }

                    }
                }
            }
        }
        stage ('Analysis'){
            environment {
                SCANNER_HOME = tool 'SonarQube'
            }
            steps {
                withSonarQubeEnv(installationName: 'SonarQube', credentialsId: 'sonarqube-token'){
                    script {
                        for (i in user){
                            sh """
                                ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=${i} \
                                -Dsonar.projectName=${i} \
                                -Dsonar.sources=/var/jenkins_home/workspace/${env.JOB_NAME}/${i} \
                                -Dsonar.host.url=http://3.213.6.243:9000  \
                                -Dsonar.scm.disabled=true
                            """                    
                        }
                    }                        
                }
            }
        }
    }
}
```
Para ejecutar nuestro job, tendremos que darle a *apply* y después *save* y nos enviará a otra página para poder ejecutar nuestra tarea.  

![Jenkins pipeline][Jenkins-pipeline]  
  
![Jenkins estatus][Jenkins-estatus]

El botón `Build Now` inicia la tarea, nos saldrá un estado de nuestra *build* en la misma página.

![Jenkins build][Jenkins-build]

Después de ver que se ha completado nuestra *build*, podemos acceder a nuestra página de SonarQube.

![Sonarqube projects][Sonar-projects]


---
## Mantenimiento

Para el mantenimiento de este proyecto solo debemos que tener en cuenta las actualizaciones de las nuevas versiones de Jenkins.

Para ello, antes de ejecutar el comando `docker-compose -f docker/Jenkins-Sonar/docker-compose.yaml up -d` debemos borrar la imagen Jenkins con el comando `docker rmi jenkins/jenkins:lts`, y despues ejecutar el comando de docker-compose y automaticamente se descargara la nueva imagen de jenkins.

Tenemos que tener en cuenta que no perderemos la configuración ya que todo esta montado con volumenes y esto hará que no perdamos ningún dato.

Otra de las cosas a tener en cuenta es a la hora de añadir nuevos repositorios, para ello solamente tenemos que editar el Pipeline anteriormente mencionado.

---
## Conclusiones

Gracias a este proyecto se facilitará el trabajo de personas que tienen que pasar horas corrigiendo un programa, con un solo botón se podrá ver el resultado de evaluación de cada código.

---
## Bibliografía

[Red-hat](https://www.redhat.com/es/topics/devops/what-is-ci-cd)
[Jenkins](https://www.jenkins.io/doc/book/)
[SonarQube](https://docs.sonarqube.org/latest/)
[Docker installation](https://docs.docker.com/engine/install/ubuntu/)
[Dockerhub SonarQube](https://hub.docker.com/_/sonarqube)
[Jenkins pipeline](https://www.jenkins.io/doc/book/pipeline/)



[jenkins-photo]: img/Jenkins.png
[jenkins-url]: https://www.jenkins.io/
[taiga-url]: https://tree.taiga.io/project/isx47328890-projecte-jenkins/timeline
[system-photo]: img/system_photo.png
[sonarqube-photo]: img/lenguajes_sonarqube.jpg

[Jenkins-passwd]: img/ConfigJenkins/Jenkins_passwd.png
[Jenkins-custom]: img/ConfigJenkins/CustomJenkins.png
[Jenkins-plugins]: img/ConfigJenkins/InstallPlugins.png
[Jenkins-AdminPasswd]: img/ConfigJenkins/AdminPasswd.png
[Jenkins-DNS]: img/ConfigJenkins/JenkinsURL.png
[Jenkins-ready]: img/ConfigJenkins/JenkinsReady.png
[Jenkins-SonarPlugin]: img/ConfigJenkins/SonarqubePlugins.png
[Jenkins-SonarInstall]: img/ConfigJenkins/Jenkins_SonarqubeHomePath.png
[Jenkins-Token]: img/ConfigJenkins/Jenkins-SonarToken.png
[Jenkins-Token2]: img/ConfigJenkins/Jenkins-SonarToken2.png

[Sonarqube-admin]: img/ConfigSonarqube/SonarqubeAdmin.png
[Sonarqube-newAdminPasswd]: img/ConfigSonarqube/Sonarqube_newPasswd.png
[Sonarqube-Token]: img/ConfigSonarqube/SonarqubeToken.png

[Jenkins-newitem]: img/Pipeline/Jenkins-newitem.png
[Jenkins-job]: img/Pipeline/Jenkins-job.png
[Jenkins-estatus]: img/Pipeline/Jenkins-estatus.png
[Jenkins-pipeline]: img/Pipeline/Jenkins-pipeline.png
[Jenkins-build]: img/Pipeline/Jenkins-build.png
[Sonar-projects]: img/Pipeline/Sonar-projects.png