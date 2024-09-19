
1. Будем использовать имеющийся свободный домен, купленный ранее
~~~
divanoblomova.ru
~~~
2. Создаём ВМ, подходящую под поставленные в задаче условия.
Для этого используем сервис firstvds.ru , где уже крутится пет проект.

3. Получим IP адрес 79.174.12.211

4. В Majordomo, куда делегировано управление DNS записями домена divanoblomova.ru создадим следующие 3 А записи, которые потребуются для выполнения задачи. 
~~~
А traefik.divanoblomova.ru 79.174.12.211
А site1.divanoblomova.ru 79.174.12.211
А site2.divanoblomova.ru 79.174.12.211
~~~
5. Задаём пароль для root, что бы иметь доступ к VM 
~~~
root
%PWD%
~~~
6. Проверяем доступность VM по FQDN, убеждаясь что распространение уже прошло. 
~~~
nslookup traefik.divanoblomova.ru
~~~
В ответ ожидаем получить
~~~
Non-authoritative answer:
Name:    traefik.divanoblomova.ru
Address:  79.174.12.211
~~~
8. Продолжаем используя FQDN
Подключаемся, по SSH, для удобства используем SSH клиент mRemoteNG
~~~
apt-get update
apt-get upgrade
~~~

9. Устанавливаем Docker Engine (https://docs.docker.com/engine/install/ubuntu/)
~~~
apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
apt-get update
apt-get install docker-ce docker-ce-cli containerd.io
~~~
10. Проверяем
~~~
docker run hello-world
~~~

11. Устанавливаем Docker Compose
~~~
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
~~~
Проверяем версию
~~~
docker-compose --version
~~~
12. Создаём сеть для Docker. (https://docs.docker.com/reference/cli/docker/network/create/)
~~~
docker network create web
~~~

13. Создаём директорию для проекта: (ChatGPT + https://habr.com/ru/articles/508636/)
~~~
mkdir traefik-nginx-setup
cd traefik-nginx-setup
~~~
14. Создай файл docker-compose.yml

~~~~
version: '3'

services:
  traefik:
    image: traefik:v2.9
    docker-compose up -d
    command:
      - "--api.insecure=true"
      - "--providers.file.directory=/etc/traefik"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.myresolver.acme.tlschallenge=true"
      - "--certificatesresolvers.myresolver.acme.email=alekskey87@gmail.com"
      - "--certificatesresolvers.myresolver.acme.storage=/letsencrypt/acme.json"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
      - "./letsencrypt:/letsencrypt"
      - "./traefik-config:/etc/traefik"
    networks:
      - web

  site1:
    image: nginx
    docker-compose up -d
    volumes:
      - ./site1:/usr/share/nginx/html
    networks:
      - web

  site2:
    image: nginx
    docker-compose up -d
    volumes:
      - ./site2:/usr/share/nginx/html
    networks:
      - web

networks:
  web:
    external: true


~~~~


15. В директории проекта создай папку traefik-config, и в ней файл traefik.yml:
~~~
mkdir /traefik-nginx-setup/traefik-config
cd /traefik-nginx-setup/traefik-config
~~~

Создаём файл конфигурации
~~~
nano traefik-config.yml
~~~
~~~
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"

http:
  routers:
    site1:
      rule: "Host(`site1.divanoblomova.ru`)"
      entryPoints:
        - websecure
      service: site1-service
      tls:
        certResolver: myresolver

    site2:
      rule: "Host(`site2.divanoblomova.ru`)"
      entryPoints:
        - websecure
      service: site2-service
      tls:
        certResolver: myresolver

  services:
    site1-service:
      loadBalancer:
        servers:
          - url: "http://site1:80"

    site2-service:
      loadBalancer:
        servers:
          - url: "http://site2:80"

~~~


16. Создаём папки для сайтов
~~~
cd /traefik-nginx-setup/
mkdir site1 site2
~~~

~~~
echo "Welcome to site 1" > site1/index.html
echo "Welcome to site 2" > site2/index.html
~~~

17. Запускаем контейнеры 
~~~
docker-compose up -d
~~~
Проверяем, что контейнеры запустились
~~~
docker ps
~~~

18. Проверяем, что сайт доступен извне
~~~
curl http://site1.divanoblomova.ru
curl -I http://site1.divanoblomova.ru
curl http://site1.divanoblomova.ru
curl -I http://site1.divanoblomova.ru
~~~
