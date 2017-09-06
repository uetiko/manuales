Indicaciones para instalar jenkins.
===========================

Requerimientos:
- java 8
- Nginx
- supervisor

##Contenido
- commons
- Instalando python
- Instalado java
- Instalar Jenkins
- Configurando Nginx
- Configurando supervisor 

## commons
```sh
yum install openssl-libs
yum insall alternatives
yum install nginx
yum install supervisor
```

##instalando python

[Instalar python 3.6 sobre centos](python/install_python3_6.md)

## Instalando java


```sh
wget http://download.oracle.com/otn-pub/java/jdk/8u131-b11/d54c1d3a095b4ff2b6607d096fa80163/jdk-8u131-linux-x64.rpm
rpm -Uvh jdk-8u131-linux-x64.rpm
alternatives --install /usr/bin/java java /usr/java/latest/jre/bin/java 200000
alternatives --install /usr/bin/javaws javaws /usr/java/latest/jre/bin/javaws 200000
alternatives --install /usr/bin/javac javac /usr/java/latest/bin/javac 200000
alternatives --install /usr/bin/jar jar /usr/java/latest/bin/jar 200000
alternatives --config java
alternatives --config javaws
alternatives --config javac
```
Agrega `JAVA_HOME` a /etc/profile

```sh
export JAVA_HOME="/usr/java/jdk1.8.0_131"
```

## Instalando Jenkins

```sh
wget http://mirrors.jenkins.io/war-stable/latest/jenkins.war
mv jenkins.war /usr/share/jenkins/
```

## Configurando supervisor
```conf
[program:jenkins]
command=java -jar /usr/share/jenkins/jenkins.war
user=jenkins
environment=
    HOME="/var/lib/jenkins"
autostart=true
autorestart=true
stdout_logfile=/var/log/jenkins.log
redirect_stderr=true
stopsignal=QUIT
```

## config nginx


```conf
mkdir -p /etc/nginx/sites-available
mkdir -p /etc/nginx/sites-enabled
```

Se debe abrir el archivo /etc/nginx/nginx.conf

```conf
include /etc/nginx/sites-enabled/*.conf;
server_names_hash_bucket_size 64;
```


```conf
location / {
  proxy_pass http://127.0.0.1:8080;
  proxy_redirect off;
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
}
```
## Configurando el firewall en centOS

```sh
firewall-cmd --zone=public --permanent --add-service=http
firewall-cmd --reload
```

## Regla para SELinux

```sh
setsebool -P httpd_can_network_connect 1
```
