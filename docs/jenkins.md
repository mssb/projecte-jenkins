# Jenkins

## CI/CD

La **CI/CD** es un método para distribuir aplicacions a los clientes con frecuencia mediante el uso de la **automatización** en las etapas del desarrollo de aplicacions.

Los principales conceptos que se atribuyen a la CI/CD son la **integración continua**, la **distribución continua** y la **implementación continua**.


## ¿Qué es Jenkins?

Es un servidor de automatización de código abierto que se puede utilizar para automatizar todo tipo de tareas relacionadas con la creación, prueba y entrega o implementación de software.

## Características

* **Instalación sencilla**: Jenkins es una plataforma independiente, programa autónomo basado en Java, que está listo para instalar a través de paquetes para Windows, MAC OS, y sistemas UNIX.

* **Configuración sencilla**: Jenkins es configurado y puesto en marcha facilmente gracias a su interfaz web. 

* **Plugins disponibles**: Hay cientos de plugins disponibles en el centro de actualización, integrando cada herramienta en la cadena de herramientas de CI y CD.

* **Extensible**: Jenkins se puede extender por medio de su arquitectura de plugins.

* **Distribución**: Jenkins puede facilmente distribuir trabajo a través de máquinas para hacer _builds_ , _test_ y despliegues de manera más rápida.

* **Free Open Source**: Jenkins es un recurso de código abierto que tiene el soporte de una comunidad muy grande.

## Jobs

Los **jobs** son unas tareas para conseguir un objetivo en Jenkins. 

Hay diferentes maneras de crear los _jobs_:

* **Freestyle project**
* **Pipeline**
* **Multi-configuration project**
* **Folder**
* **GitHub Organization**
* **Multibranch Pipeline**

En este proyecto nos centraremos en utilizar pipelines, porque tiene muchas ventajas como las siguientes:

* **Código**: Los pipelines se implementan con código.
* **Durabilidad**: Los pipelines pueden aguantar reinicios planeados o no planeados del controlador de Jenkins.
* **Pausable**: Los pipelines pueden opcionalmente pausarse y esperar a un input humano o aprobación antes de continuar con el Pipeline.
* **Versátil**: Los pipelines soportan requisitos de desarrollo continuo del mundo real bastante complejos, incluyendo la habilidad de bifurcar/unir, bucles, y trabajar en paralelo.
* **Extensible**: El plugin Pipeline soporta extensions personalizadas y múltiples opcions para integrarse con otros plugins.

### Pipeline

**Jenkins Pipeline** o simplemente **Pipeline**  es un paquete de plugins que soporta la implementación e integración de _pipelines de entrega continua_ en Jenkins. 

#### Ejemplo de pipeline

```pipeline
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```