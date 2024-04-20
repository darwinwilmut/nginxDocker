# Nginx in Docker
### Prerequesites
- Docker Installed
### Step 1: Configure a domain
Configure a domain name for the IP. This IP should be attached to the VM.
Eg. vm0112.westeurope.cloudapp.azure.com --> 51.144.249.196
### Step 2: Create a docker network
```
sudo docker network create app-network
```
You can see the network created by running the following command
```
sudo docker network ls
```
### Step 3: Run a simple react application
Run the following command to run a simple react application, where we are about to expose this react application through nginx.
```
sudo docker run -d -p 3000:80 --name react --network app-network --restart always darwinwilmut/react-docker
```
### Step 4: Setup the Nginx Config
```
cd && mkdir nginx && cd nginx && touch default.conf
```
Open the default.conf and paste the following default SSL Configuration.

Note: Change your domain name. You can add any configuration or proxy pass.

```
server {

        root /var/www/html;

        index index.html index.htm index.nginx-debian.html;
        server_name vm0112.westeurope.cloudapp.azure.com; # managed by Certbot


        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                #try_files $uri $uri/ =404;
                proxy_pass http://react;
        }


    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/nginx/ssl/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/nginx/ssl/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = vm0112.westeurope.cloudapp.azure.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80 ;
        listen [::]:80 ;
    server_name vm0112.westeurope.cloudapp.azure.com;
    return 404; # managed by Certbot
}
```

### Step 5: Create SSL certificate for the domain using Certbot
Note: Change your domain name in the command
```
sudo docker run -it --rm --name certbot \
-p 80:80 -p 443:443 \
-v "/etc/letsencrypt:/etc/letsencrypt" \
-v "/var/lib/letsencrypt:/var/lib/letsencrypt" \
certbot/certbot certonly \
--standalone \
--preferred-challenges http \
-d vm0112.westeurope.cloudapp.azure.com
```

### Step 6: Run the nginx container
Note: Make sure you update the command.
```
sudo docker run -d -p 80:80 -p 443:443 --name nginx \
--network app-network \
-v "/home/darwinwilmut/nginx/conf:/etc/nginx/conf.d" \
-v "/etc/letsencrypt/live/vm0112.westeurope.cloudapp.azure.com:/etc/nginx/ssl" \
-v "/etc/letsencrypt:/etc/letsencrypt" \
nginx
```
