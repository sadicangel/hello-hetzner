# hello-hetzner
A webapi that says hello.

Configured using [Traefik](https://traefik.io/traefik/) to redirect http to https and generate SSL certificates with [Let's Encrypt](https://letsencrypt.org/).  

All services run inside [Docker](https://www.docker.com/), so the server must have it installed.

Both 80 and 443 ports should be exposed by the server.

## Build all images locally
```pwsh
docker compose up -d --build
```

## Copy all images to the server
```pwsh
docker save webapi | gzip | ssh root@cerquido.net docker load
```
> [!NOTE]
> You may have to install gzip (windows: `winget install GnuWin32.Gzip`) and add it to the PATH.

## Copy docker-compose.prod.yml to server
```pwsh
scp .\docker-compose.prod.yml username@hostname:/opt/hello-hetzner/docker-compose.yml
```
> [!NOTE]
> On traefik server, uncomment the caServer command if you want to use Let's Encrypt staging environment.

## Create .env file in /opt/hello-hetzner
```env
DOMAIN=yourdomain.com
EMAIL=username@email.com
CERT_RESOLVER=letsencrypt
TRAEFIK_LOG_LEVEL=
```
> [!NOTE]
> [TRAEFIK_LOG_LEVEL](https://doc.traefik.io/traefik/observability/logs/#level) should only be set when debugging errors, otherwise it should be left empty.

## Start containers
```bash
docker compose up -d --build
```

## Test
Access https://yourdomain.com and check that no errors occur.  
Access http://yourdomain.com and check that you're redirected to https://yourdomain.com.
>[!NOTE]
> If using the staging environment, you will get an SSL error, but if you display the certificate and see it was emitted by `Fake LE Intermediate X1` then it means all is good. (It is the staging environment intermediate certificate used by Let's Encrypt). 
