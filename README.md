# hello-hetzner
A simple blazor webapp running behind [Traefik](https://traefik.io/traefik/), supporting http->https redirection and SSL certificates using [Let's Encrypt](https://letsencrypt.org/).

All services run inside [Docker](https://www.docker.com/), so the server must have it installed.

Ther server also needs to export ports 80 and 443.

## Build all images locally
```pwsh
docker compose up -d --build
```

## Copy hellowworld image to the server
```pwsh
docker save helloworld:latest | gzip | ssh username@hostname docker load
```
> [!NOTE]
> You may have to install gzip (windows: `winget install GnuWin32.Gzip`) and add it to the PATH.

## Copy docker-compose.prod.yml to server
```pwsh
scp .\docker-compose.prod.yml username@hostname:/opt/hello-hetzner/docker-compose.yml
```
> [!NOTE]
> On docker-compose.yml, you can uncomment the `caServer` command if you want to use Let's Encrypt staging environment.

## Create .env file in /opt/hello-hetzner
```pwsh
echo "DOMAIN=yourdomain`nEMAIL=youremail`nAUTH=username:apr1-md5-hashed-password" | ssh username@hostname -T "cat | sed 's/\r$//' > /opt/hello-hetzner/.env"
```
> [!NOTE]
> You can get the password in the right format by running:
>  ```bash
> echo $(htpasswd -nb username password) | sed -e s/\\$/\\$\\$/gH`
>  ```
> The basic authentation is only needed to access Traefik dashboards.  
> You also need to configure the "traefik" subdomain by having A and AAAA records pointing to your server.

## Start containers
On the remove server, navigate to `/opt/hello-hetzner/` and run the following command to start the containers:
```bash
docker compose up -d --build
```

## Test
Access https://yourdomain.com and check that no errors occur.  
Access http://yourdomain.com and check that you're redirected to https://yourdomain.com.
>[!NOTE]
> If using the staging environment, you will get an SSL error, but if you display the certificate and see it was emitted by `Fake LE Intermediate X1` then it means all is good. (It is the staging environment intermediate certificate used by Let's Encrypt). 
