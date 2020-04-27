# Docker - Apache - Certbot

Esta configuración consta de un contenedor apache con php y otro con cerbot para la obtención y renovación de los certificados.
Docker debe de estar ya instalado en el servidor.

---

## Preparación en el servidor

- Creación de las carpetas necesarias para Let's Encrypt
```
mkdir -p /home/letsencrypt/data/.well-known/acme-challenge
mkdir /home/letsencrypt/certs
```

- Descarga de la imagen de cerbot
```
docker pull certbot/certbot
```
El contenedor solo será activado para la generación y la renovación de los certificados.

- Modificación ficheros configuración Apache
Adaptar ficheros a tu entorno, nombre dominio y otros cambios que desees realizar


- Para comprobación de conexión con Let’s Encrypt
```
echo -n "Test conexion a let's encrypt correcto" > /home/letsencrypt/data/.well-known/acme-challenge/test
```

- Modificar ficheros de configuración apache, para la obtención de los primeros certificados
    - `000-default.con`: quitar o comentar las líneas que hacen referencia al redireccionamiento https.
    - `default-ssl`: este fichero no es necesario para la primera vez que se obtienen los certificados

---

## Ejecución

- Levantar el docker compose
```
cd PathFicheroDocker-compose.yml
docker-compose up --build -d
```
En futuros inicios:
```
docker-compose start
docker-compose stop
```

- Comprobar acceso web a la carpeta data de Let's Encrypt
```
http://TuDomnio/.well-known/acme-challenge/test
```
Debes cambiar `TuDomnio` por el tuyo real (www.TuDominio).

- Comprobar si la obtencion de certificados es correcta
```
docker run -it --rm -v /home/letsencrypt/certs:/etc/letsencrypt -v /home/letsencrypt/data/:/data/letsencrypt certbot/certbot certonly --webroot --webroot-path=/data/letsencrypt -d TuDominio -d www.TuDominio --email info@TuDominio --agree-tos --dry-run
```
Esto lanzará el contenedor certbot y simulará la obtención de los certificados (--dry-run).
Debes cambiar `TuDomnio` por el tuyo real.
Si todo es correcto volver a lanzar el mismo comando pero eliminando "--dry-run".

- Modificar ficheros de configuración apache
    - `000-default.con`: recuperar las líneas que hacen referencia al redireccionamiento https.
    - `default-ssl`: modificar las referencias a `TuDominio` por el que corresponda

---

## Renovación
Los certificados de Let’s encrypt son válidos por 90 días, así que tendremos que renovarlos antes de que caduquen.
Para hacerlo lo automatizaremos con la utilidad cron del sistema. En nuestro caso lo haremos cada 2 meses.

- Abrir la utilidad
```
crontab -e
```

- Añadir al final del fichero cualquiera de estas dos sequencias
```
0 0 0 */2 * docker run -it --rm -v /home/letsencrypt/certs:/etc/letsencrypt -v /home//letsencrypt/data/:/data/letsencrypt certbot/certbot renew 
```
```
0 0 0 */2 * docker run -it --rm -v /home/letsencrypt/certs:/etc/letsencrypt -v /home/letsencrypt/data/:/data/letsencrypt certbot/certbot certonly --webroot --webroot-path=/data/letsencrypt -d TuDominio -d www.TuDominio --email info@TuDominio --agree-tos
```

---

## Posdata
Esta configuración es mas didáctica que otra cosa, ya que no soy ningún experto ni en docker, ni en apache.
Cualquier consejo o mejora serán bien recibidos.