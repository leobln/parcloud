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