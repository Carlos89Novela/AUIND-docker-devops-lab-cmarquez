# AUIND-docker-devops-lab-cmarquez

#Empezando la configuración del laboratorio Carlos Márquez 7:55AM 21May26
    #Creación de carpetas: Principal laboratorio-docker-cmarquez y sub-carpetas: app, mailhos, portainer, redis y traefik


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

  tambien se crea el documento "dynamic.yml" dentro de la carpeta de traefik
==========================================================================================================================================

#Autor: Carlos Marquez 8:26AM 21May26

  #Carpeta APP
  Se crea dentro de la carpeta de app lo siguiente:  "index.html" y "docker-compose.yml"

      El archivo index.html solo tendra lo siguiente:
        <h1>Hola desde Docker Compose 🚀 Elaborado por Carlos Marquez</h1>
        <p>Aplicación funcionando con Traefik</p>
      
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
  #Integración de MailHog
    Para esto ingresaremos a MailHog y crearemos la carpeta "docker-compose.yml" y se agregaremos el siguiente bloque de codigo:

      services:

        mailhog:
          image: mailhog/mailhog
          container_name: mailhog

          networks:
            - proxy

      networks: 
        proxy:
          external: true

    Una vez hecho lo anterior habilitamos los servicios con : docker compose up -d y probamos ingresando al URL del puerto en el que se habilito.
    ![alt text](image-1.png)
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

