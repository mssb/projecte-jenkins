# SonarQube

## ¿Qué es SonarQube?

SonarQube es una herramienta de revisión automática de código para detectar bugs, vulnerabilidades y *code smells* o *código que apesta*.

## SonarScanner

SonarScanner es el escáner que se utiliza cuando no hay un escáner específico en el sistema.

### Parámetros de análisis de SonarQube

Existen diferentes maneras de configurar los parámetros. La jerarquía es la siguiente:  

* **Propiedades globales**: Están definidas en la interfaz de usuario y se aplican a todos los proyectos. (Ir a `Administration > Configuration > General Settings`).  

* **Propiedades del proyecto**: Están definidas en la interfaz de usuario y anulan las **propiedades globales** (A nivel del proyecto, ir a `Project Settings > General Settings`).  

* **Parámetros de análisis del proyecto**: Están definidas en un fichero de configuración del análisis del proyecto o en un archivo de configuración de un scanner, anula los parámetros definidos en la UI.  

* **Línea de comandos de un parámetro**: Están definidas cuando se ejecuta un análisis (con la opción `-D` en la línea de comandos), anula los parámetros de análisis del proyecto.  

Solamente los parámetros de la UI se guardan en la base de datos. Por ejemplo, si sustituyes el parámetro `sonar.exclusions` a través de la línea de comandos, no se guardará en la base de datos.  

En este proyecto utilizaremos los **parámetros de análisis del proyecto**, es decir un archivo de configuración, ya que estamos utilizando un scanner. 
