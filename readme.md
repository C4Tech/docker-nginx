# Nginx container services

A basic public nginx 1.x container. It is configured to provide both HTTP
and/or HTTPS connections.


## Configuration

1. `/app/public` must be a mounted volume and be the webroot.
2. `fpm` must be a linked container providing the PHP-FPM service for the `php` and `php-ssl` images.
3. `/etc/ssl/public/app.crt` must be a mounted volume for the `php-ssl` and `static-ssl` images.
4. `/etc/ssl/private/app.key` must be a mounted volume for the `php-ssl` and `static-ssl` images.
5. `/etc/ssl/private/dhparam.pem` must be a mounted volume for the `php-ssl` and `static-ssl` images.

We use the `c4tech/generic-data` image to provide `app` as a linked volume.
Also, `c4tech/laravel-fpm` is what we use for our PHP-FPM service.

To generate the `dhparam.pem` file, run:
```
openssl dhparam -out dhparam.pem 4096
```

For the SSL certificate and key, you can generate a CSR and key, then submite
the CSR to a certificate provider who will provide the certificate.
```
openssl req -nodes -new -newkey rsa:4096 -keyout server.key -out server.csr -sha256
```

Alternatively, you can create a self-signed SSL certificate:
```
openssl req -x509 -newkey rsa:4096 -keyout server.key -out server.crt -days 3650 -nodes -sha256
```


## Docker Compose example

```
www:
  image: c4tech/nginx:php-ssl
  volumes:
    - ./:/app
    - ./ssl/server.crt:/etc/ssl/public/app.crt
    - ./ssl/server.key:/etc/ssl/private/app.key
    - ./ssl/dhparam.pem:/etc/ssl/private/dhparam.pem
  links:
    - fpm
  ports:
    - "80:80"
    - "443:443"
```
