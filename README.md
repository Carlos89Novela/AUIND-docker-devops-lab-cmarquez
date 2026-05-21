# AUIND-docker-devops-lab-cmarquez

#Empezando la configuración del laboratorio Carlos Márquez 7:55AM 21May26
    #Creación de carpetas: Principal laboratorio-docker-cmarquez y sub-carpetas: app, mailhos, portainer, redis y traefik


#Autor Carlos Marquez 8:15AM 21May26
#Carpeta Traefik
  #Creación y habilitación de Traefik:
      #comandos: "cd traefik", "docker network create proxy", "nano docker-compose.yml" y se agrega el siguiente codigo:
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

  #tambien se crea el documento "dynamic.yml" dentro de la carpeta de traefik
==========================================================================================================================================

#Autor: Carlos Marquez 8:26AM 21May26
#Carpeta APP
#Se crea dentro de la carpeta de app lo siguiente:  "index.html" y "docker-compose.yml"

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

  #Una vez creado los archivos procedemos a dar de alta los servicios con: "docker compose up -d"
==========================================================================================================================================