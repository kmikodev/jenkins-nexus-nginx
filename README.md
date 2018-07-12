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
    <img src="https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/devops_logo.png" style="max-width:40%;"/>
</div>

### Introducción
El objetivo de este repositorio es ejemplificar un entorno de **devops**  con el que tener una **integración continua** de nuestro código.

La estructura de nuestro ejercicio será el siguiente

<div style="text-align:center">
    <img src="https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/Mapa de flujo de valor.png" style="max-width:80%;"/>
</div>

Se compone por un servidor **nginx** para que sirva los diferentes servicios y asi poder acceder a los mismos mediante el puerto 80 y/o 443. A parte también habrá 3 servicios que describiremos a continuación.

### Servicios


#### Gogs

![Gogs](/readme_assets/gogs.jpg)


Servicio que vamos a usar para tener un repostorio de fuentes privado. Este repositorio es muy sencillo de configurar y no tiene nada que enviadarle a otras solucciones como pueden ser gitlab.

En la solucción que se está implementando en este caso está compuesto de:

- gogsData: Servicio que contiene los volumenes
- gogs: Es nuestro servicio de git en si
- mysql: Sistema de base de datos que va a acompañar al servicio.

A la hora de instalarlo le indicaremos la ruta y puerto de mysql que en este caso es `10.5.0.8:3306`.

Para configurarlo podemos usar el [cheatset](https://gogs.io/docs/advanced/configuration_cheat_sheet), el que se recomienda para este caso es el siguiente:

```bash
...

[service]
REGISTER_EMAIL_CONFIRM = false
ENABLE_NOTIFY_MAIL     = false
ENABLE_CAPTCHA         = true
REQUIRE_SIGNIN_VIEW    = false
DISABLE_REGISTRATION   = true

...
```

#### Jenkins

<img src="https://jenkins.io/sites/default/files/jenkins_logo.png"/>

