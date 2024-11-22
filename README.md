
# Nginx Certbot Docker SSL

A brief description of what this project does and who it's for


## 1. Verificar el dominio

Asegúrate de que tu dominio (dominio.com) esté apuntando correctamente a la IP pública de tu servidor. Usa un comando como este para verificar:
```javascript
ping dominio.com
```

## 2. Crear estructura del proyecto con Docker Compose
```javascript
version: '3.8'

services:
  nginx:
    image: nginx:latest
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
      - ./certs:/etc/letsencrypt
      - ./certs-data:/var/lib/letsencrypt
    restart: unless-stopped

  certbot:
    image: certbot/certbot
    container_name: certbot
    volumes:
      - ./certs:/etc/letsencrypt
      - ./certs-data:/var/lib/letsencrypt
      - ./html:/usr/share/nginx/html
    entrypoint: "/bin/sh -c 'trap exit TERM; while :; do sleep 1; done'"

```
## 3. Crear archivos necesarios
Carpeta para contenido web: Crea una carpeta llamada html en el mismo directorio donde está el archivo docker-compose.yml:
```javascript
mkdir html
echo "<h1>dominio está funcionando</h1>" > html/index.html
```
Archivo de configuración de Nginx:
Crea el archivo nginx.conf:
```javascript
events {}

http {
    server {
        listen 80;
        server_name ivanso.site www.ivanso.site;

        root /usr/share/nginx/html;
        index index.html;

        location /.well-known/acme-challenge/ {
            allow all;
        }
    }
}

```

## 4. Levantar los servicios

```javascript
docker-compose up -d
docker ps
```
## 5. Generar el certificado con Certbot
Ejecuta el siguiente comando para generar el certificado:
```javascript
docker run --rm -it \
    -v $(pwd)/certs:/etc/letsencrypt \
    -v $(pwd)/html:/usr/share/nginx/html \
    certbot/certbot certonly \
    --webroot \
    -w /usr/share/nginx/html \
    -d ivanso.site -d www.ivanso.site
```
## 6. Actualizar la configuración de Nginx

```javascript
events {}

http {
    server {
        listen 80;
        server_name ivanso.site www.ivanso.site;

        # Redirección a HTTPS
        return 301 https://$host$request_uri;
    }

    server {
        listen 443 ssl;
        server_name ivanso.site www.ivanso.site;

        ssl_certificate /etc/letsencrypt/live/ivanso.site/fullchain.pem;
        ssl_certificate_key /etc/letsencrypt/live/ivanso.site/privkey.pem;

        root /usr/share/nginx/html;
        index index.html;

        location / {
            try_files $uri $uri/ =404;
        }
    }
}

```

## 7. Verificar HTTPS

```javascript
curl -I https://dominio.com
```



## 8. Configurar la renovación automática
Certbot configurará automáticamente una tarea cron en el sistema host para renovar el certificado. Si prefieres manejarlo con Docker, puedes ejecutar este comando para renovar manualmente:
```javascript
docker run --rm -it \
    -v $(pwd)/certs:/etc/letsencrypt \
    -v $(pwd)/html:/usr/share/nginx/html \
    certbot/certbot renew

```