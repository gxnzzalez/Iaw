2. Pasos que tendrás que hacer mañana en el nuevo equipo (después del git pull):

Cuando llegues al otro ordenador y descargues el proyecto, tendrás que seguir estos pasos rápidos para levantarlo de nuevo:

    Instalar los servicios:
    Asegúrate de tener instalados Nginx, Apache y OpenSSL en el nuevo entorno (o contenedor):
    Bash

    apt update && apt install -y nginx apache2 openssl

    Volver a generar o copiar los certificados TLS:
    Como los certificados autofirmados suelen estar excluidos de Git, tendrás que volver a lanzar el comando de OpenSSL para crearlos en la carpeta certs/ del nuevo equipo (o llevártelos copiados en un pendrive si los guardaste).

    Recrear los enlaces simbólicos:
    Git guarda tus ficheros de configuración en las carpetas sites-available, pero debes volver a enlazarlos para que los servicios los lean:

        Para Apache:
        Bash

        ln -s /ruta/a/tu/repo/configuracion.virtualhost/nimbusbotas.conf /etc/apache2/sites-available/
        ln -s /ruta/a/tu/repo/configuracion.virtualhost/nimbusguantes.conf /etc/apache2/sites-available/
        a2ensite nimbusbotas.conf nimbusguantes.conf
        a2dissite 000-default.conf

        Para Nginx:
        Bash

        ln -s /ruta/a/tu/repo/multi-marca-proxy.conf /etc/nginx/sites-available/
        ln -s /etc/nginx/sites-available/multi-marca-proxy.conf /etc/nginx/sites-enabled/
        rm -f /etc/nginx/sites-enabled/default

    Actualizar el /etc/hosts del nuevo equipo:
    No olvides añadir de nuevo la línea en el archivo de hosts de la nueva máquina para que resuelva los dominios localmente:
    Bash

    echo "127.0.0.1 nimbusbotas.com nimbusguantes.com" >> /etc/hosts

Con estos pequeños ajustes tras el git pull, el entorno funcionará al 100% de la misma manera que lo tienes ahora. ¡Buen trabajo hoy y descansa!
