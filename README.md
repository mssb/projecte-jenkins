# Proyecto Jenkins

[![Jenkins photo][jenkins-photo]][jenkins-url]

<br><br>

**Autores:**  Christian Manalo y Mark Santiago  

**Curso:** 2º de Administración de sistemas informáticos en redes

**Centro:** INS Escola del Treball


### [Proyecto en GitHub](https://github.com/isx47328890/projecte-jenkins)

### [Proyecto en Taiga](https://tree.taiga.io/project/isx47328890-projecte-jenkins/timeline)

### [Servidor Jenkins](http://3.213.6.243:8080/)


<br><br>

# Índice

- **[Estudio inicial](#estudio-inicial)**
- **[Diseño del sistema](#diseño-del-sistema)**
- **[Análisis del sistema y conceptos](#análisis-del-sistema-y-conceptos)**
  - **[Jenkins](#que-es-jenkins)**
  - **[SonarQube](#que-es-sonarqube)**
- **[Instalación](#instalación)**
  - **[Docker Compose](#docker-compose)**
  - **[Configuración Jenkins](#configuración-jenkins)**
  - **[Configuración Sonarqube](#configuración-sonarqube)**
  - **[Conectar Jenkins y Sonarqube](#conectar-jenkins-y-sonarqube)**
- **[Implantación](#implantación)**
- **[Mantenimiento](#mantenimiento)**
- **[Conclusiones](#conclusiones)**
- **[Bibliografía](#bibliografía)**

<br><br>

## Estudio inicial

El proyecto consistirá en el desarrollo de un sistema automatizado de entrega continua con **Jenkins**. El trabajo o "*job*" de Jenkins desplegará un repositorio **git** en un servidor y dentro de este proceso habrá diferentes etapas o "*stages*" de la cual cogerá los diferentes códigos del repositorio y será evaluada a través de **SonarQube** para saber la calidad del código, el cual devolverá un feedback.

<br><br>

## Diseño del sistema

![System photo][system-photo]

<br><br>

## Análisis del sistema y conceptos

El sistema automatizado de entrega continua estara montado en tres contenedores docker de la cual será **Jenkins**, **Sonarqube** y **Postgressql**. 

**Jenkins** cogerá los codigos de **Git** y contactará con Sonarqube para evaluarlos. **Sonarqube** tiene un servidor web para poder visualizar los escaneos.

Dentro de **Postgressql** se guardaran los escaneos de código hechos por **Sonarqube**, y dentro de este tambien estara la configuración de la instancia del servidor web.

<br><br>

#### Que es Jenkins?
Es un servidor automatizado de integración continua de código abierto, es capaz de organizar una cadena de acciones que ayudan a lograr el proceso de integración continua de manera automatizada,  está completamente escrito en **Java**.

Jenkins puede instalarse a través de un sistema de gestión de paquetes, Docker o incluso ejecutarse de forma independiente en cualquier máquina que tenga instalado Java Runtime Environment (JRE).

Las compañías de software pueden acelerar su proceso de desarrollo del código, ya que este puede automatizar, agilizar y aumentar el ritmo de toda la compilación y las pruebas de los proyectos.

<br><br>

#### Que es SonarQube? 
Es una plataforma de software libre para evaluar la calidad de nuestro código, realizando un análisis estático sobre dicho código, con el objetivo de advertirnos sobre diferentes puntos a mejorar.

Estos son los lenguajes soportados:

![Sonarqube photo][sonarqube-photo]

## Instalación

El proyecto estara montado en 3 containers docker diferentes, uno para Jenkins, otro para Sonarqube y otro para Postrgessql. 

Antes de arrancar los containers debemos establecer unos parametros para que el Sonarqube no esté limitado ya que utiliza `Elasticsearch`.

```
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
```

### Docker Compose
Para arrancar los dockers hemos hecho un docker-compose para que encienda todo lo necesario para que funcione.

```
docker-compose -f docker/Jenkins-Sonar/docker-compose.yaml up -d
```

El contenido de este docker-compose es lo siguiente:

```
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

Para conseguir la contraseña tenemos dos opciones, entrar en el docker para hacer un `cat` del siguiente archivo `/var/jenkins_home/secrets/initialAdminPassword` o hacer un `docker logs`.

```bash
docker logs jenkins-sonar_jenkins_1
```

Solo debemos buscar la passwd, nos saldrá lo siguiente:
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
docker exec -it -u root jenkins-sonar_jenkins_1 mkdir /opt/sonarqube
```

![Jenkin Sonarqube Home Photo][Jenkins-SonarInstall]


Para la integración de estos dos servidores se necesita un token de autenticación. Tenemos que asegurarnos de guardar el token ya que mas adelante lo necesitaremos.

Dentro de Sonarqube `My Account > Security > Generate Token`.

![Sonarqube Token Photo][Sonarqube-Token]


Ahora toca añadir el token dentro del servidor de Jenkins. `Manage Jenkins > Configure Systems > Sonarqube Servers`. 

```
Kind: Secret Text
Secret: token_generado_en_sonarqube
Description: SonarQube Server
```

![Jenkins Token Integration Photo][Jenkins-Token]


Seleccionar el authentication token `SonarQube Server`
```
Name: SonarQube
Server URL: 
```
![Jenkins Token Integration Photo 2][Jenkins-Token2]

## Implantación

## Mantenimiento

## Conclusiones

## Bibliografía



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