# Implementacion de Aplicaciones Web 

#Practica 1

# |. Diseño y Requisitos

#Los dos dominios ficticios elegidos (uno por marca) y por qué los habéis nombrado así.

He elegido estas dos marcas porque no se me ocurria nada, asi qué elegi dos sitios qué vendieran cosas relacionadas con el futbol, uno botas y otro guantes.
nimbusbotas.com
nimbusguantes.com
 
#El MPM de Apache elegido y la justificación técnica de esa elección (repasad UD2,
bloque de MPM).

He elegido MPM event ya qué a diferencia de modelos tradicionales como prefork, el MPM event destaca optimizando el rendimiento frente a conexiones persistentes, ya que delega la gestión de las conexiones ociosas o en espera a hilos oyentes dedicados (listener threads), evitando el bloqueo de recursos y reduciendo de forma drástica el consumo de memoria RAM. Esto garantiza una comunicación altamente eficiente y fluida entre Nginx y el backend de Apache en el puerto 8080,complementando perfectamente el filtrado y la seguridad perimetral de la arquitectura.

#II. Fase 1 — Backend Apache Multi-Marca

#Demostrad con curl, usando la cabecera Host explícita, que cada dominio devuelve el
contenido correcto.

Todos los Curls me llevan al dominio correcto.

#La marca "interna" (la que decidáis que no debe ser pública) tiene que proteger un
fichero sensible simulado — con nombre propio, no wp-config.php — y desactivar el
listado de directorios en toda su ruta.

La marca qué he elegido qué sea la interna cerrada al publico es nimbusguantes.com , lo elegi porque soy jugador de campo y los guantes me importan menos.
Cree un archivo sensible con nombre archivo_privado.conf (elegi .conf porque ya qué los he usado durante la practica seria mas sencillo ) y desactive el dictado de directorios con el atributo:
<Directory /var/www/html/guantes> 
Options -Indexes +FollowSymLinks 
AllowOverride 
None Require all granted 
</Directory> 

#Un dominio que no coincida con ningún Virtual Host debe caer en un comportamiento
controlado por vosotros (no el primero al azar) — explicad en el README por qué pasa
eso.

Yo cree un virtualhost nuevo para ponerlo predeterminado con el nombre pordefecto.com y su configuracion  virtualhostpordefecto.conf.
Cambiamos el virtualhost por defecto para qué si en caso de caer en un dominio qué no puede localizar, la maquina nos envie al dominio por defecto qué yo elegi.
Si no lo cambiara me llevaria al virtualhost por defecto de apache ya puesto.

#IV. Fase 2 — Seguridad y Límite de Peticiones

#Cabeceras de seguridad mínimas: Strict-Transport-Security,
X-Content-Type-Options y una tercera que elijáis y justifiquéis vosotros (repasad UD1,
tabla de cabeceras)
Strict-Transport-Security): Obliga a los navegadores a comunicarse exclusivamente mediante HTTPS, previniendo ataques de interceptación y la degradación del protocolo.

X-Content-Type-Options (nosniff): Evita la autodetección de tipos MIME, impidiendo que el navegador ejecute archivos con un formato distinto al declarado por el servidor.

X-Frame-Options (SAMEORIGIN): Protege contra ataques de clickjacking al impedir que las páginas de nuestra plataforma sean incrustadas en marcos (<iframe>) de sitios web externos.

#Demostrad con una ráfaga de peticiones repetidas (un bucle for con curl, o ab desde
un contenedor cliente) que el límite calculado realmente bloquea, adjuntando la salida.

Para ello le ponemos este comando al contenedor:
for i in {1,2,3,4,5,6,7,8,9,10,11,12}; do curl -s -k -L -o /dev/null -w "Peticion $i -> HTTP Status: %{http_code}\n" -H "Host: nimbusbotas.com" https://127.0.0.1/; done

Y nos devolvera:
Peticion 1 -> HTTP Status: 200
Peticion 2 -> HTTP Status: 200
Peticion 3 -> HTTP Status: 200
Peticion 4 -> HTTP Status: 200
Peticion 5 -> HTTP Status: 200
Peticion 6 -> HTTP Status: 200
Peticion 7 -> HTTP Status: 503
Peticion 8 -> HTTP Status: 503
Peticion 9 -> HTTP Status: 503
Peticion 10 -> HTTP Status: 503
Peticion 11 -> HTTP Status: 503
Peticion 12 -> HTTP Status: 503



Ya qué el rate limit está puesto en 3, solo nos van a dar efectivas 1 por el tiempo, pero al poner un burst o buffer, nos permite 5 peticiones mas temporales.


#V. Fase 3 — Diagnóstico Provocado

#1 fallo relacionado con la parte Apache (Fase 1: Virtual Host, protección de recursos).


Componente: Apache(Fase 1)

Síntoma Observado: Al realizar una petición HTTP especificando la cabecera Host de una marca, Apache responde sirviendo la página de otra marca o la del directorio por defecto.

Hipótesis Inicial: Error de sintaxis en el archivo del VirtualHost o ruta de DocumentRoot mal especificada.

Comando de Diagnóstico: apache2ctl -S

Causa Real y Solución Conceptual: Causa: Omisión de la directiva ServerName dentro del bloque <VirtualHost *:8080>, impidiendo que Apache empareje el dominio y haciendo que caiga por defecto en el primer VirtualHost cargado alfabéticamente.
Solución: Declarar ServerName nimbusguantes.com en el fichero correspondiente y recargar el servicio.





#1 fallo relacionado con la parte Nginx (Fase 2: TLS, comunicación con Apache, o rate
limiting).


Componente: Nginx(Fase 2)

Síntoma observado: Al someter el servicio a ráfagas masivas de peticiones, Nginx procesa todo el tráfico sin bloquear nada, devolviendo HTTP 200 OK en todas las peticiones.

Hipótesis inicial: La zona de memoria del rate limiting se ha corrompido o el servicio Nginx no ha leído la configuración.

Comando de Diagnóstico: nginx -t y auditoría de la directiva dentro del bloque location.

Causa Real y Solución Conceptual: Causa: La directiva limit_req_zone estaba declarada en el bloque global http, pero se omitió activar la directiva limit_req dentro del bloque location / del servidor virtual.               Solución: Añadir limit_req zone=doc_limit burst=8 nodelay; dentro del bloque location / y reiniciar Nginx.











