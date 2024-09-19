
Добавление третьего сайта в конфигурацию Traefik и Nginx

0. В Majordomo, куда делегировано управление DNS записями домена divanoblomova.ru создадим следующие 3 А записи, которые потребуются для выполнения задачи. 
~~~
А site3.divanoblomova.ru 79.174.12.211
~~~

1. Создайте папку для site3 и добавьте содержимое:

~~~
mkdir site3
echo "Welcome to site 3" > site3/index.html
~~~


2. Обновление traefik.yml: Добавьте новый роутер и сервис для site3 в файл traefik-config/traefik.yml:

~~~
http:
  routers:
    site3:
      rule: "Host(`site3.divanoblomova.ru`)"
      entryPoints:
        - websecure
      service: site3-service
      tls:
        certResolver: myresolver

  services:
    site3-service:
      loadBalancer:
        servers:
          - url: "http://site3:80" #Container Name
~~~

3. Обновление docker-compose.yml: Добавьте новый сервис для site3 в файл docker-compose.yml:

~~~
  site3:
    image: nginx
    docker-compose up -d
    volumes:
      - ./site3:/usr/share/nginx/html
    networks:
      - web
~~~

4. Создаём папку для сайта
~~~
cd /traefik-nginx-setup/
mkdir site3
~~~
~~~
echo "Welcome to site 3" > site3/index.html
~~~


5. Перезапуск контейнеров: Перезапустите контейнеры для применения изменений:

~~~
docker-compose down
docker-compose up -d
~~~

5. Проверка доступа: 
~~~
curl http://site3.divanoblomova.ru
curl -I http://site3.divanoblomova.ru
~~~