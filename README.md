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

## Implantación

## Mantenimiento

## Conclusiones

## Bibliografía



[jenkins-photo]: img/Jenkins.png
[jenkins-url]: https://www.jenkins.io/
[taiga-url]: https://tree.taiga.io/project/isx47328890-projecte-jenkins/timeline
[system-photo]: img/system_photo.png
[sonarqube-photo]: img/lenguajes_sonarqube.jpg

