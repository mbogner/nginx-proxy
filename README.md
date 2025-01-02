# nginx Reverse Proxy Example

This is a sample SSL enabled reverse proxy. It terminates SSL and allows forwarding to other containers. Of course you can use this setup for non proxy setups as well.

The included [docker-compose.yml](docker-compose.yml) file and provided folder structure make it easier to work
with the container. In this compose config `nginx` container is proxying requests to <https://localhost.mbo.dev> to `nginx2` which is just a default nginx without extra config.

`localhost.mbo.dev = entry in /etc/hosts for 127.0.0.1`

It displays the default nginx start page when accessed.

Without domain name (direct access to proxy): Requests to port 80 of the proxy (<http://localhost>) display the included [index.html](docker/nginx/html/default/index.html) file under [docker/nginx/html/default](docker/nginx/html/default). Requests to port 443 to the default virtual host aren't answered (<https://localhost>) by the default config because this would require another certificate and this was only tested for local environment with localhost for direct access. If the default server isn't listening for something nginx takes the first virtual host that is capable of serving the request. So access to <https://localhost> could end up on nginx2 with SSL error on your client.

## Folder Structure

The folder structure [docker/nginx](docker/nginx) holds files used in this example:

- [certificates](docker/nginx/certificates): Place your certificates into this folder which is mapped to `/etc/ssl/certs/nginx` in the provided [docker-compose.yml](docker-compose.yml) file.
- [conf.d](docker/nginx/conf.d): Folder for virtual host configurations. It is mentioned in nginx.conf which is not part of this repo - the default one is used. [example.conf](docker/nginx/conf.d/example.conf) also contains an example proxy configuration. You can have multiple proxy configs inside the same container as long as they don't have overlapping server names. I left my `localhost.mbo.dev` which I used for testing. This way I can verify functionality without any changes. Of course you need to change the domain when using this.
- [html](docker/nginx/html): Mapped to `/var/www/html` inside the container. Place your files here to use them from your virtual hosts.
- [includes](docker/nginx/includes): Some reusable configurations that can be shared between virtual hosts
  - [ssl.conf](docker/nginx/includes/ssl.conf): State of the art SSL configuration with TLSv1.3
  - [proxy.conf](docker/nginx/includes/proxy.conf): Some defaults for a proxy
  - [headers.conf](docker/nginx/includes/headers.conf): Default headers including security
  - [compression.conf](docker/nginx/includes/compression.conf): Content compression config
  - [defaults.conf](docker/nginx/includes/defaults.conf): Common settings for all vhosts - enable UTF-8, compression.conf and headers.conf

## SSL

The repository doesn't include any certificates in the [certificates](docker/nginx/certificates) folder by default. For being able to run the container with the provided [docker-compose.yml](docker-compose.yml) file
you need to place your own files into this folder. The [example.conf](docker/nginx/conf.d/example.conf) uses files provided from letsencrypt.

To receive a wildcard certificate without the need of running a webserver you can run the command below. Before you start make sure to replace the placeholders with your own values.

```shell
DOMAIN=example.com
MAIL=your-email@example.com
certbot certonly --manual --preferred-challenges=dns --email "$MAIL" --agree-tos -d "$DOMAIN" -d "*.$DOMAIN"
```

You can the copy the created files into the certificates folder. Make sure to copy the real files and not just the softlinks.
