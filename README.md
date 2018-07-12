# Infraestructura
## Tabla de contenidos
- Introducción
- Servicios
    - Jenkins
    - Nexus
    - Nginx
    - Git (gogs)
    - Problemas

---
<div style="text-align:center">
    <img src="https://i.imgur.com/rzqo4Rp.png" style="max-width:40%;"/>
</div>

### Introducción
El objetivo de este repositorio es ejemplificar un entorno de **devops**  con el que tener una **integración continua** de nuestro código.

La estructura de nuestro ejercicio será el siguiente

<div style="text-align:center">
    <img src="https://i.imgur.com/IaoOKhU.png" style="max-width:80%;"/>
</div>

Se compone por un servidor **nginx** para que sirva los diferentes servicios y asi poder acceder a los mismos mediante el puerto 80 y/o 443. A parte también habrá 3 servicios que describiremos a continuación.

### Servicios

#### Jenkins

<img src="https://jenkins.io/sites/default/files/jenkins_logo.png"/>