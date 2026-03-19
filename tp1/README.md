# Part I : Docker basics

## 1. Install

### 🌞 Installer Docker votre machine Azure

Install de docker

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/usr/lib/systemd/system/docker.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-03-19 10:33:22 CET; 51min ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 1724 (dockerd)
      Tasks: 22
     Memory: 112.1M (peak: 115.3M)
        CPU: 640ms
     CGroup: /system.slice/docker.service
             └─1724 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

```
```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
```

Ajout au group

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ groups
leobln adm cdrom sudo dip plugdev users lpadmin vboxusers docker
```

🌞 Utiliser la commande docker run

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ docker run --name nginx -d -p 9999:80 nginx
1d25efdd30a8fd3e308a8f6d81c7846916a6239aac711d016820c4229ae8e214
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                     NAMES
1d25efdd30a8   nginx     "/docker-entrypoint.…"   13 seconds ago   Up 12 seconds   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp   nginx
f8791375ea49   debian    "sleep 99999"            12 minutes ago   Up 12 minutes                                             elated_jang
```

🌞 Rendre le service dispo sur internet

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ curl http://10.100.0.167:9999
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy, 
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional 
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

🌞 Custom un peu le lancement du conteneur

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ # 1. On supprime le cadavre du conteneur précédent
docker run -d \
  --name meow \
  -p 8888:7777 \
  --memory="512m" \
  -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf \
  -v $(pwd)/index.html:/var/www/tp_docker/index.html \
  nginx
meow
f5bf7d2d813800cedcf946ba6c3f29a78508821d5ed0411dfa24200352167365
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS                                                 NAMES
f5bf7d2d8138   nginx     "/docker-entrypoint.…"   10 seconds ago   Up 10 seconds   80/tcp, 0.0.0.0:8888->7777/tcp, [::]:8888->7777/tcp   meow
1d25efdd30a8   nginx     "/docker-entrypoint.…"   34 minutes ago   Up 34 minutes   0.0.0.0:9999->80/tcp, [::]:9999->80/tcp               nginx
f8791375ea49   debian    "sleep 99999"            46 minutes ago   Up 46 minutes                                                         elated_jang
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ curl http://localhost:8888 (ou: http://10.100.0.167:8888/)
<h1>salut ! Bienvenue sur mon serveur</h1>
```

🌞 Call me

```
leobln@leo-vivobook:~/Documents/travaille/parcloud/tp1$ curl http://10.100.0.167:8888
<h1>salut ! Bienvenue sur mon serveur</h1>
```

# Part II : Images

🌞 Construire votre propre image

On garde le meme index.html

On crée un fichier conf (my_apache.conf)
```
Listen 80
LoadModule mpm_event_module "/usr/lib/apache2/modules/mod_mpm_event.so"
LoadModule dir_module "/usr/lib/apache2/modules/mod_dir.so"
LoadModule authz_core_module "/usr/lib/apache2/modules/mod_authz_core.so"
DirectoryIndex index.html
DocumentRoot "/var/www/html/"
ErrorLog "/var/log/apache2/error.log"
LogLevel warn
```

on crée un Dockerfile
```
FROM debian:latest

RUN apt-get update && apt-get install -y apache2 && apt-get clean

RUN mkdir -p /var/log/apache2

COPY my_apache.conf /etc/apache2/apache2.conf

COPY index.html /var/www/html/index.html

EXPOSE 80

CMD ["apache2", "-DFOREGROUND"]
```

Puis on crée notre image a partir du Dockerfile(c'est le .)
```
docker build -t mon-apache-custom .
```

Enfin on lance le docker baser sur l'image crée précedament
```
docker run -d --name mon-serveur-apache -p 8080:80 mon-apache-custom
```

