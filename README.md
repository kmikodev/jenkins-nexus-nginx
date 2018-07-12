# Infraestructura
## Tabla de contenidos
- [Introducción](https://github.com/kmikodev/jenkins-nexus-nginx#introducci%C3%B3n)
- [Servicios](https://github.com/kmikodev/jenkins-nexus-nginx#servicios)
    - [Git (gogs)](https://github.com/kmikodev/jenkins-nexus-nginx#gogs)
    - [Nexus](https://github.com/kmikodev/jenkins-nexus-nginx#nexus)
    - [Jenkins](https://github.com/kmikodev/jenkins-nexus-nginx#jenkins)
    - [Nginx](https://github.com/kmikodev/jenkins-nexus-nginx#nginx)

---
<div style="text-align:center">
    <img src="https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/devops_logo.png" style="max-width:40%;"/>
</div>

### Introducción
El objetivo de este repositorio es ejemplificar un entorno de **devops**  con el que tener una **integración continua** de nuestro código.

La estructura de nuestro ejercicio será el siguiente

<div style="text-align:center">
    <img src="https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/infrauml.png" style="max-width:80%;"/>
</div>

Se compone por un servidor **nginx** para que sirva los diferentes servicios y asi poder acceder a los mismos mediante el puerto 80 y/o 443. A parte también habrá 3 servicios que describiremos a continuación.

### Servicios


#### Gogs

![Gogs](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/gogs.jpg)


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


#### Nexus

![Nexus](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/nexus.png)


Nexus será nuestro repositorio de artefactos de todo tipo:

- npm 
- maven
- docker hub (en desarrollo)

Una vez tengamos el servicio arrancado podemos crear los repositorios usando el api de nexus.
  - create-repositories.sh
  - repos.json
  - Ejecución

```bash
#!/bin/bash

jsonFile=$1

printf "Creating Integration API Script from $jsonFile\n\n"
cat $jsonFile

curl -v -u user:pass --header "Content-Type: application/json" 'http://localhost:8081/service/rest/v1/script/' -d @$jsonFile

```

```json
{
  "name": "repositories",
  "type": "groovy",
  "content": "repository.createMavenHosted('maven-library-release'); repository.createMavenHosted('maven-library-snapshot'); repository.createNpmHosted('npm-snapshot'); repository.createNpmHosted('npm-release'); repository.createNpmProxy('npmjs-org','https://registry.npmjs.org'); repository.createNpmGroup('npm-all-pro',['npmjs-org', 'npm-release']); repository.createNpmGroup('npm-all-dev',['npmjs-org','npm-snapshot', 'npm-release'])"
}
```

```bash
$ sh create-repositories.sh repos.json
```

Con el script y el json anterior crearemos repositorios de desarrollo y de producción de npm, y un repositorio proxy que apunta al registry por defecto. Dos grupos uno para desarrollo que contiene el propio de desarrollo (snapshot) y el proxy; otro con el de produccion (release) y el proxy.

- ¿Cómo exponer nexus con nginx?

```conf
server {
  listen   80;
  server_name nexus.midominio.com;

  # allow large uploads of files
  client_max_body_size 1G;

  # optimize downloading files larger than 1G
  #proxy_max_temp_file_size 2G;

  location / {
    proxy_pass http://10.5.0.4:8081/;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }

  /*
  Con ssl
    location / {
      proxy_pass http://10.5.0.4:8081/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto "https";
    }
  */
}
```

#### Jenkins

![Jenkins](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/jenkins.png)

Jenkins será una de las piezas claves de nuestro sistema de ALM, ya que será el encargado de escuchar los cambios en nuestro repositorio y ejecutar una serie de tareas.  

Durante el proceso de instalación nos pedirá un secret-key y procederemos con la instalación.

- Plugins
- Ejemplos de pipeline

#### Plugins

- [Gogs plugin](https://wiki.jenkins.io/display/JENKINS/Gogs+Webhook+Plugin): Será el que usemos para la integración con gogs (repositorio git).

![Gogs plugin](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/gogs.png)

- [JUnit Plugin](https://wiki.jenkins.io/display/JENKINS/JUnit+Plugin): Nos mostrará la tendencia de los test en jenkins

![JUnit plugin](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/junit.png)

- [NodeJS plugin](https://wiki.jenkins.io/display/JENKINS/NodeJS+Plugin): Este plugin nos ayuda a la hora de crear entornos de ejecución y poder escoger que versión de node y npm vamos a usar así como las diferentes dependencias que queramos tener en el path, @ngular/cli, rimraf...

![NodeJS plugin](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/nodejs.png)


- [Slack Notification Plugin](https://wiki.jenkins.io/display/JENKINS/Slack+Plugin): Este plugin nos permite enviar notificaciones a slack.

![Slack](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/slack.png)


![Slack](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master//readme_assets/slack-jenkins.png)


#### Nginx
![Nginx](https://raw.githubusercontent.com/kmikodev/jenkins-nexus-nginx/master/readme_assets/nginx.png)

Contenedor que se usará como punto de entrada para exponer los servicios al exterior. Desde aquí podemos exponer tanto por HTTP cómo por HTTPS.  

Dispone de los volumenes 
- ./nginx/nginx.conf:/etc/nginx/nginx.conf
- ./nginx/sites-enable:/etc/nginx/sites-enable
- ./nginx/sites-ssl:/etc/ssl



##### ./nginx/nginx.conf

Archivo que usaremos para configurar el servicio que hay dentro del contenedor de nginx

##### ./nginx/sites-enable

Volumen en el que incluiremos los diferentes virtual host mediante ficheros tipo: 


##### ./nginx/sites-ssl

Volumen que usaremos para añadir los ficheros necesarios para usar los certificados.

  - ¿Cómo generar un certificado autofirmado con openssl?

  ```sh
    $ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout cert.key -out cert.crt

    $ sudo openssl dhparam -out dhparam.pem 2048
  ```

  - ¿Cómo añadir los certificados a un virtual host?

  ```conf
  server {
    listen 443 http2 ssl;
    listen [::]:443 http2 ssl;

    server_name miweb.com;
    // Ruta hasta el directorio donde está el certificado
    ssl_certificate /etc/ssl/certs/cert.crt;
    // Ruta hasta el directorio donde está el key
    ssl_certificate_key /etc/ssl/private/cert.key;
    // Ruta hasta el directorio donde está el dhparam.pem    
    ssl_dhparam /etc/ssl/certs/dhparam.pem;
 
    ########################################################################
    # from https://cipherli.st/                                            #
    # and https://raymii.org/s/tutorials/Strong_SSL_Security_On_nginx.html #
    ########################################################################

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_ecdh_curve secp384r1;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    # Disable preloading HSTS for now.  You can use the commented out header line that includes
    # the "preload" directive if you understand the implications.
    #add_header Strict-Transport-Security "max-age=63072000; includeSubdomains; preload";
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;

    ##################################
    # END https://cipherli.st/ BLOCK #
    ##################################
   
    location / {
      root   /usr/share/nginx/html/directorio_de_mi_web;
      index  index.html index.htm;
    }
   }
  ```