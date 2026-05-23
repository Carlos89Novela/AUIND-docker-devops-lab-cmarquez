# AUIND-docker-devops-lab-cmarquez
#Autor: Carlos Márquez 7:55AM 21May26

Empezando la configuración del laboratorio 

Creación de carpetas: Principal laboratorio-docker-cmarquez y sub-carpetas: app, mailhos, portainer, redis y traefik
=========================================================================================================================================

#Autor Carlos Marquez 8:15AM 21May26
  #Carpeta Traefik
    Creación y habilitación de Traefik:
      comandos: "cd traefik", "docker network create proxy", "nano docker-compose.yml" y se agrega el siguiente codigo:
      services:
=========================================================================================================================================
        "traefik:
          image: traefik:v2.10

          command:
            - "--api.insecure=true"
            - "--api.dashboard=true"

            # ✅ ENTRYPOINT WEB (app)
            - "--entrypoints.web.address=:8080"

            # ✅ ENTRYPOINT TRAEFIK (dashboard)
            - "--entrypoints.traefik.address=:8081"

            - "--providers.file.directory=/etc/traefik"

          ports:
            - "8000:8080"   # app
            - "8081:8081"   # dashboard

          volumes:
            - ./dynamic.yml:/etc/traefik/dynamic.yml

          networks:
            - proxy

        networks:
          proxy:
            external: true"

  tambien se crea el documento "dynamic.yml" dentro de la carpeta de traefik con el siguienbte bloque de codigo:

    http:
      routers:

        
        mailhog:
          rule: "PathPrefix(`/mail`)"
          service: mailhog-service
          entryPoints:
            - web
          middlewares:
            - mailhog-strip
          priority: 100


        portainer:
          rule: "PathPrefix(`/portainer`)"
          service: portainer-service
          entryPoints:
            - web
          middlewares:
            - portainer-strip
          priority: 90

        web:
          rule: "PathPrefix(`/`)"
          service: web-service
          entryPoints:
            - web
          priority: 1

      services:

        web-service:
          loadBalancer:
            servers:
              - url: "http://web:80"

        portainer-service:
          loadBalancer:
            servers:
              - url: "http://portainer:9000"

        mailhog-service:
          loadBalancer:
            servers:
              - url: "http://mailhog:8025"

      middlewares:

        portainer-strip:
          stripPrefix:
            prefixes:
              - "/portainer"

        mailhog-strip:
          stripPrefix:
            prefixes:
              - "/mail"

==========================================================================================================================================

#Autor: Carlos Marquez 8:26AM 21May26

  #Carpeta APP
  Se crea dentro de la carpeta de app lo siguiente:  "index.html" y "docker-compose.yml"

El archivo index.html solo tendra lo siguiente:


        <!DOCTYPE html>
        <html lang="es">
        <head>
          <meta charset="UTF-8">
          <title>Docker + Traefik</title>

          <style>
            body {
              margin: 0;
              font-family: Arial, Helvetica, sans-serif;
              height: 100vh;

              /* Fondo degradado azul y morado */
              background: linear-gradient(135deg, #2b6cb0, #6b46c1);

              display: flex;
              justify-content: center;
              align-items: center;
              color: white;
            }

            .container {
              text-align: center;
              background: rgba(255, 255, 255, 0.1);
              padding: 40px;
              border-radius: 15px;
              backdrop-filter: blur(10px);
              box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
            }

            h1 {
              font-size: 2.5rem;
              margin-bottom: 15px;
            }

            p {
              font-size: 1.2rem;
              margin: 8px 0;
            }

            .highlight {
              color: #90cdf4; /* azul claro */
              font-weight: bold;
            }
          </style>
        </head>

        <body>

          <div class="container">
            <h1>🚀 Hola los saludamos el equipo de Carlos Marquez</h1>

            <p>Estás dentro de <span class="highlight">Docker Compose</span></p>
            <p>Tu aplicación está funcionando correctamente ✅</p>

            <p>Utilizando <span class="highlight">Traefik</span> como Reverse Proxy</p>

            <p style="margin-top: 20px;">
              🎉 ¡Todo está configurado correctamente!
            </p>
          </div>

        </body>
        </html>

      
El archivo docker-compose.yml tendra lo siguiente:

        services:
          web:
            image: nginx:latest
            container_name: web

            volumes:
              - ./index.html:/usr/share/nginx/html/index.html

            networks:
              - proxy

        networks:
          proxy:
            external: true

Una vez creado los archivos procedemos a dar de alta los servicios con: "docker compose up -d" y probamos llendo al URL del puerto en el cual se habilito:
      ![alt text](image.png)
==========================================================================================================================================

#Autor: Carlos Marquez 8:47AM 21May26

#Integracion de Redis:

  Aqui lo que se hara es dentro de la carpeta de Redis, se generará el archivo "docker-compose.yml" con la siguiente configuración:


    services:

        redis:
          image: redis:7-alpine
          container_name: redis

          ports:
            - "6379:6379"

          volumes:
            - redis_data:/data

          networks:
            - proxy

    volumes:
      redis_data:

    networks:
      proxy:
        external: true

  Una vez hecho lo anterior solo damos de alta los servicios y probamos: docker compose up -d, docker exec -it redis redis-cli y escribimos PING y nos debe de responder un cun PONG
==========================================================================================================================================
#Autor: Carlos Marquez 9:18AM 21May26

#Integración de MailHog:

Para esto ingresaremos a MailHog y crearemos la carpeta "docker-compose.yml" y se agregaremos el siguiente bloque de codigo:

      services:

        mailhog:
          image: mailhog/mailhog
          container_name: mailhog
          ports:
            - "8025:8025"

          networks:
            - proxy

      networks: 
        proxy:
          external: true

  Una vez hecho lo anterior habilitamos los servicios con : docker compose up -d y probamos ingresando al URL del puerto en el que se habilito.
    ![alt text](image-3.png)
==========================================================================================================================================
#Autor: Carlos Marquez 9:43AM 21May26

Integración de Portainer:

Para esto haremos lo que hemos estado haciendo crear dentro de la carperta de portainer el documento: "docker-compose.yml" con el siguiente bloque de codigo:


      services:

        portainer:
          image: portainer/portainer-ce
          container_name: portainer

          volumes:
            - /var/run/docker.sock:/var/run/docker.sock

          ports:
            - "9000:9000"   # 🔥 ACCESO DIRECTO

          networks:
            - proxy

      networks:
        proxy:
          external: true
Y levantamos los servicios con: "docker compose up -d" y verificamos ingresando al URL del puerto que creamos:
    ![alt text](image-2.png)

==========================================================================================================================================

#Autor: Carlos Márquez 8:12AM 22May26

Habilitación de DNS local agregando en el archivo Hosts lo siguiente:
  Nos dirigimos a la carpeta C:\Windows\System32\drivers\etc e identificamos el archivo Hosts, abrimos un bloc de notas como administrador y damos a archivo, abrir y nos dirigimos a la carpeta antes mencionada y abrimos el archivo, ya una vez dentro nos dirigimos hasta el final y agregamos lo siguiente:

      127.0.0.1 app.midominio.com
      127.0.0.1 mail.midominio.com
      127.0.0.1 portainer.midominio.com
      127.0.0.1 dashboard.midominio.com
  
  Una vez hayamos agregado lo anterior, abriremos el archivo "docker-compose.yml" de cada una de las carpetas y agregamos lo siguiente:

    #APP:
      
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.web.rule=Host(`app.midominio.com`)"
        - "traefik.http.routers.web.entrypoints=web"

    #TRAEFIK:

      
      command:
        - "--api.dashboard=true"
        - "--api.insecure=true"

        - "--entrypoints.web.address=:80"

        - "--providers.docker=true"
        - "--providers.docker.exposedbydefault=false"

    #MAILHOG:

      
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.mailhog.rule=Host(`mail.midominio.com`)"
        - "traefik.http.routers.mailhog.entrypoints=web"

    #PORTAINER:

      
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.portainer.rule=Host(`portainer.midominio.com`)"
        - "traefik.http.routers.portainer.entrypoints=web"

  Se reinicia todo yo lo hago de la siguiente manera:

      cd ../mailhog
      docker compose down -v
      docker compose up -d

      cd ../portainer
      docker compose down -v
      docker compose up -d

      cd ../app
      docker compose down -v
      docker compose up -d

      cd ../traefik
      docker compose down -v
      docker compose up -d

y una vez hecho lo anterior se prueba con :

    http://app.midominio.com

  ![alt text](image-6.png)

    http://mail.midominio.com:8025

  ![alt text](image-5.png)

    http://portainer.dominio.com:9000

  ![alt text](image-4.png)

    http://dashboard.midominio.com:8081
    
  ![alt text](image-7.png)

==========================================================================================================================================
#Autor: Carlos Marquez 10:38AM 22May26

#Habilitación de Certificados con MKCERT y configuración de TLS a WEB, TREAFIK, MAILHOG Y PORTAINER:

  Primero descargamos de la pagina https://github.com/FiloSottile/mkcert/teleases en mi caso es en windows y descargue el archivo mkcert-v.1.4.4-windows-amd64.exe, una vez hecho esto renombramos a mkcert.exe y creamos una carpeta en C:/ llamado mkcert y lo movemos dentro, escribimos "cmd" en la barra donde nos dice la ubicacion del archivo, una vez que se habra el simbolo de sistema escribimos el comando: "mkcert.exe --install" ya instalado ejecutamos el comando "mkcert.exe app.midominio.com mail.midominio.com portainer.midominio.com", despues ejecutamos lo comandos: "app.midominio.com+2.pem" y "app.midominio.com+2-key.pem" para que se nos generen los certificados, los renombramos como "cert.pem y "key.pem", dentro de la carpeta de traefik agregaremos otra carpeta llamada cert y dentro de ella movemos los certificados recien hechos:

    #TRAEFIk:
      Dentro de Traefikañadiremos lo siguiente dentro del apartado command:

        
        - "--entrypoints.websecure.address=:443"
        - "--providers.docker=true"
        - "--providers.docker.exposedbydefault=false"

        Ports:
          -"443:443"

        volumes:
          
          - /var/run/docker.sock:/var/run/docker.sock
          - ./dynamic.yml:/etc/traefik/dynamic.yml
          - ./certs:/certs

      Sin salir de la carpeta de TRAEFIK abrimos el archivo dynamic.yml y agregamos lo siguiente hasta el final:

          tls:
            certificates:
              -certFile: /certs/cert.pem
              -keyFile: /certs/key.pem
    
    #APP:
      Abrimos el archivo "docker-compose.yml" de la carpeta APP y agregamos lo siguiente:

            - "traefik.http.routers.web.rule=Host(`app.midominio.com`)"
            - "traefik.http.routers.web.entrypoints=websecure"
            - "traefik.http.routers.web.tls=true"

    #MAILHOG:
      Abrimos el archivo "docker-compose.yml" de la carpeta MAILHOG y agregamos lo siguiente:

            - "traefik.http.routers.web.rule=Host(`mail.midominio.com`)"
            - "traefik.http.routers.web.entrypoints=websecure"
            - "traefik.http.routers.web.tls=true"

    #PORTAINER:
      Abrimos el archivo "docker-compose.yml" de la carpeta PORTAINER y agregamos lo siguiente:

            - "traefik.http.routers.web.rule=Host(`portainer.midominio.com`)"
            - "traefik.http.routers.web.entrypoints=websecure"
            - "traefik.http.routers.web.tls=true"

  Una vez hecho todo lo anterior Reiniciamos todo, como ya se ha explicado anteriormente, dentro de cada carpeta ejecutamos en terminar los comandos:

        docker compose down -v
        docker compose up -d

  Y probamos entrando a cada uno de las paginas agregando https a cada uno:

  ![alt text](image-9.png)

==========================================================================================================================================
#Autor: Carlos Marquez 06:54PM 22May26

#Se agrega POSTGRES, ADMINER, MYSQL y phpMYADMIN

#Se crean las carpetas de cada uno y dentro de cada una de las carpetas se agregan los archivos docker-compose.yml con los siguientes bloques de codigos respectivamente:

#POSTGRES:

      services:
        postgres:
          image: postgres:16
          container_name: postgres

          environment:
            POSTGRES_DB: laboratorio
            POSTGRES_USER: admin
            POSTGRES_PASSWORD: admin123

          volumes:
            - postgres_data:/var/lib/postgresql/data

          networks:
            - proxy

      volumes:
        postgres_data:

      networks:
        proxy:
          external: true

#ADMINER:


    services:
      adminer:
        image: adminer:latest
        container_name: adminer

        networks:
          - proxy

    networks:
      proxy:
        external: true


#MYSQL:


      services:
        mysql:
          image: mysql:8
          container_name: mysql

          environment:
            MYSQL_ROOT_PASSWORD: root123
            MYSQL_DATABASE: wordpress
            MYSQL_USER: wpuser
            MYSQL_PASSWORD: wp123

          volumes:
            - mysql_data:/var/lib/mysql

          networks:
            - proxy

      volumes:
        mysql_data:

      networks:
        proxy:
          external: true


#phpMYADMIN


    services:
      phpmyadmin:
        image: phpmyadmin/phpmyadmin
        container_name: phpmyadmin

        environment:
          PMA_HOST: mysql
          PMA_PORT: 3306

        networks:
          - proxy

    networks:
      proxy:
        external: true


#Modificamos el archivo docker-compose.yml de la carpeta de TRAEFIK y queda de la siguiente manera:


    http:
      routers:
        web:
          rule: "Host(`app.midominio.com`)"
          entryPoints:
            - websecure
          service: web-service
          tls: {}

        mailhog:
          rule: "Host(`mail.midominio.com`)"
          entryPoints:
            - websecure
          service: mailhog-service
          tls: {}

        portainer:
          rule: "Host(`portainer.midominio.com`)"
          entryPoints:
            - websecure
          service: portainer-service
          tls: {}

        adminer:
          rule: "Host(`adminer.midominio.com`)"
          entryPoints:
            - websecure
          service: adminer-service
          tls: {}

        phpmyadmin:
          rule: "Host(`phpmyadmin.midominio.com`)"
          entryPoints:
            - websecure
          service: phpmyadmin-service
          tls: {}

        wordpress:
          rule: "Host(`blog.midominio.com`)"
          entryPoints:
            - websecure
          service: wordpress-service
          tls: {}

        dashboard:
          rule: "Host(`dashboard.midominio.com`)"
          entryPoints:
            - websecure
          service: api@internal
          tls: {}

      services:
        web-service:
          loadBalancer:
            servers:
              - url: "http://web1:80"
              - url: "http://web2:80"
              - url: "http://web3:80"

        mailhog-service:
          loadBalancer:
            servers:
              - url: "http://mailhog:8025"

        portainer-service:
          loadBalancer:
            servers:
              - url: "http://portainer:9000"

        adminer-service:
          loadBalancer:
            servers:
              - url: "http://adminer:8080"

        phpmyadmin-service:
          loadBalancer:
            servers:
              - url: "http://phpmyadmin:80"

        wordpress-service:
          loadBalancer:
            servers:
              - url: "http://wordpress:80"

    tls:
      certificates:
        - certFile: /certs/cert.pem
          keyFile: /certs/key.pem


#WEB:

![alt text](image-14.png)
#DASHBOARD:

![alt text](image-15.png)

#MAILHOG:

![alt text](image-16.png)

#PORTAINER:

![alt text](image-17.png)
#ADMINER:

![alt text](image-18.png)
#phpMyAdmin:

![alt text](image-19.png)
#WordPress:

![alt text](image-20.png)
============================================================================
#Autor: Carlos Marquez 07:05PM 22May26

#Se agrega el Balanceo de carga y para esto se agregan 3 carpetas adicionales las cuales serán WEB1, WEB2 y WEB3 y en cada una de ellas se les agrega el archivo docker-compose.yml y en las cuales quedan de la siguiente manera:


#WEB1:


    services:
      web1:
        image: nginx:latest
        container_name: web1

        volumes:
          - ./index.html:/usr/share/nginx/html/index.html:ro

        networks:
          - proxy

    networks:
      proxy:
        external: true


#WEB2:


    services:
      web2:
        image: nginx:latest
        container_name: web2

        volumes:
          - ./index.html:/usr/share/nginx/html/index.html:ro

        networks:
          - proxy

    networks:
      proxy:
        external: true


#WEB3:


    services:
      web3:
        image: nginx:latest
        container_name: web3

        volumes:
          - ./index.html:/usr/share/nginx/html/index.html:ro

        networks:
          - proxy

    networks:
      proxy:
        external: true 

#Dentro de cada una de esas mismas carpetas aparte del archivo anterior agregaremos un archivo index.html en cada carpeta con el codigo siguiente en los cuales solo cambian los titulos y la leyendas de web1, web2 y web3 asi como los colores:


      <!DOCTYPE html>
      <html lang="es">
      <head>
        <meta charset="UTF-8">
        <title>Nginx 1.1 Docker + Traefik</title>

        <style>
          body {
            margin: 0;
            font-family: Arial, Helvetica, sans-serif;
            height: 100vh;

            /* Fondo degradado azul a morado */
            background: linear-gradient(135deg, #2563eb, #7c3aed);

            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
          }

          .container {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            padding: 40px;
            border-radius: 15px;
            backdrop-filter: blur(10px);
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
          }

          h1 {
            font-size: 2.5rem;
            margin-bottom: 15px;
          }

          p {
            font-size: 1.2rem;
            margin: 8px 0;
          }

          .highlight {
            color: #ddd6fe;
            font-weight: bold;
          }
        </style>
      </head>

      <body>

        <div class="container">
          <h1>🚀 Hola los saludamos el equipo de Carlos Marquez</h1>

          <p>Estás dentro de <span class="highlight">Docker Compose</span></p>
          <p>Tu aplicación está funcionando correctamente ✅</p>

          <p>Utilizando <span class="highlight">Traefik</span> como Reverse Proxy</p>

          <p style="margin-top: 20px;">
            🎉 ¡Todo está configurado correctamente!
          </p>

          <p style="margin-top: 20px;">
            🔹 <span class="highlight">Nginx 1.1 - Web 1.1</span>
          </p>
        </div>

      </body>
      </html>

#WEB1
![alt text](image-11.png)
#WEB2
![alt text](image-12.png)
#WEB3
![alt text](image-13.png)
============================================================================
#Autor: Carlos Marquez 07:12PM 22May26

#Se integra WordPress, se crea una carpeta con un archivo adentro docker-composer.yml con el siguiente bloque de codigo:


    services:
      wordpress:
        image: wordpress:latest
        container_name: wordpress

        environment:
          WORDPRESS_DB_HOST: mysql:3306
          WORDPRESS_DB_NAME: wordpress
          WORDPRESS_DB_USER: wpuser
          WORDPRESS_DB_PASSWORD: wp123

        volumes:
          - wordpress_data:/var/www/html

        networks:
          - proxy

    volumes:
      wordpress_data:

    networks:
      proxy:
        external: true
      wordpress:

![alt text](image-10.png)
============================================================================