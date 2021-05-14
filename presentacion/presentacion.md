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
def listUsers = "alumne1 alumne2 alumne3"
def user = listUsers.split(" ")

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
                SCANNER_HOME = tool 'sonarqube'
            }
            steps {
                withSonarQubeEnv(installationName: 'sonarqube', credentialsId: 'sonarqube-token'){
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