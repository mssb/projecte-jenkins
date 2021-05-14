### Objetivo

- Sistema automatizado de integración continua con **Jenkins**.
- Códigos provenientes de **Git** evaluados con **Sonarqube** para saber la calidad de código.

### Infraestructura
---
![Infraestructura](../img/system_photo.png)

- **Instancia AWS**
- **Docker**
- **Volúmenes**

---



### Jenkins

![Jenkins Logo](../img/Jenkins.png)

#### CI/CD

- **CI:** Continuous Integration
- **CD:** Continuous Deployment.


La CI/CD es un método para distribuir aplicaciones a los clientes mediante el uso de la automatización en las etapas del desarrollo de aplicaciones:

- **Integración continua**
- **Distribución continua**
- **Implementación continua**

![Integracion_Continua](../img/integracion_continua.png)

#### ETAPAS CI/CD
![ci_cd](../img/ci_cd.png)

#### Caracteristicas
* **Instalación sencilla**

* **Configuración sencilla**

* **Plugins disponibles**

* **Extensible**

* **Distribución**

* **Free Open Source**


---

### Sonarqube

![Sonarqube](../img/sonarqube-logo.png)

- **Lenguajes disponibles:**

![Sonarqube photo](../img/lenguajes_sonarqube.jpg)

---

### Integración Jenkins y Sonarqube

**Jenkins**
![Jenkins-Token2](../img/ConfigJenkins/Jenkins-SonarToken2.png)
![Jenkins-Token](../img/ConfigJenkins/Jenkins-SonarToken.png)

**Sonarqube**
![Token](../img/ConfigSonarqube/SonarqubeToken.png)

---

### Pipeline Jenkins

```java
def users = "alumne1 alumne2 alumne3"
def listUsers = users.split(" ")

pipeline {
    agent any
    stages {
        // Clonación de repositorio
        stage ('Clone repositories'){
            steps {
                script {
                    for (user in listUsers) {
                        sh """
                            if [ -d ${user} ]; then
                                cd ${user}
                                git pull
                            else
                                git clone https://gitlab.com/2daw2020/${user}.git ${user}/
                            fi
                        """
                    }
                }
            }
        }
        // Analisis del repositorio
        stage ('Analysis'){
            environment {
                SCANNER_HOME = tool 'sonarqube'
            }
            // Propiedades Server Sonarqube
            steps {
                withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonarqube-token'){
                    script {
                        for (user in listUsers){
                            sh """
                                ${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=${user} \
                                -Dsonar.projectName=${user} \
                                -Dsonar.sources=/var/jenkins_home/workspace/daw/${user} \
                                -Dsonar.css.node=. \
                                -Dsonar.host.url=http://3.213.6.243:9000 \
                            """                    
                        }
                    }                        
                }
            }
        }
    }
}
```


### Mantenimiento
- **Actualización de la imagen jenkins**
- **Añadir nuevos repositorios**

---

### Cosas a mejorar
- **Output Mail**
- **Detectar los commits y se ejecute automáticamente**
- **Detectar plagios**

---

### Conclusiones

---

## FIN

Gracias por vuestra atención!