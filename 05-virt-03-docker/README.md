# Домашнее задание к занятию "5.3. Введение. Экосистема. Архитектура. Жизненный цикл Docker контейнера"
## Как сдавать задания

Обязательными к выполнению являются задачи без указания звездочки. Их выполнение необходимо для получения зачета и диплома о профессиональной переподготовке.
Задачи со звездочкой (*) являются дополнительными задачами и/или задачами повышенной сложности. Они не являются обязательными к выполнению, но помогут вам глубже понять тему.
Домашнее задание выполните в файле readme.md в github репозитории. В личном кабинете отправьте на проверку ссылку на .md-файл в вашем репозитории.
Любые вопросы по решению задач задавайте в чате учебной группы.

---

## Задача 1

Сценарий выполения задачи:

- создайте свой репозиторий на https://hub.docker.com;
```buildoutcfg
   Создал тестовой репозиторий:
   https://hub.docker.com/repository/docker/aturganov/test2_netology    
Поставил докер на Ubuntu 20.04
https://docs.docker.com/engine/install/ubuntu/
```
- выберете любой образ, который содержит веб-сервер Nginx;
- создайте свой fork образа;
```buildoutcfg
Сделал проект
Добавил dockerfile:

FROM nginx
COPY wwww /usr/share/nginx/www

создание образа
docker build -t aturganov/nginx:v1 .

turganovai@vds2260027:~/git/nginx_netology$ docker build -t aturganov/nginx:v1 .
Sending build context to Docker daemon  41.98kB
Step 1/1 : FROM nginx
 ---> fa5269854a5e
Successfully built fa5269854a5e
Successfully tagged turganovai/nginx:v1

Подключаемся к хабу
turganovai@vds2260027:~/git/nginx_netology$ docker login --username aturganov
Password: 
WARNING! Your password will be stored unencrypted in /home/turganovai/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

Закидываем образ
turganovai@vds2260027:~/git/nginx_netology$ docker push aturganov/nginx:v1
The push refers to repository [docker.io/aturganov/nginx]
b6812e8d56d6: Pushed 
7046505147d7: Pushed 
c876aa251c80: Pushed 
f5ab86d69014: Pushed 
4b7fffa0f0a4: Pushed 
9c1b6dd6c1e6: Pushed 
v1: digest: sha256:1d7ac80b420030c9669bea15d01116e43a402f4760c160f73123cead2b423894 size: 1570
```
- реализуйте функциональность:
запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m DevOps Engineer!</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на https://hub.docker.com/username_repo.
```buildoutcfg
Заходите на сайт через NGINX
http://95.214.8.211:5000/

образы:
https://hub.docker.com/repository/docker/aturganov/nginx_netology_nginx
https://hub.docker.com/repository/docker/aturganov/www

проект:
https://github.com/aturganov/nginx_netology
```

## Задача 2

Посмотрите на сценарий ниже и ответьте на вопрос:
"Подходит ли в этом сценарии использование Docker контейнеров или лучше подойдет виртуальная машина, физическая машина? Может быть возможны разные варианты?"

Детально опишите и обоснуйте свой выбор.
```buildoutcfg
На мой вгляд докеризация используется при небольших приложениях, когда нужно на одной/нескольких машинах
создать несколько изолированных сервисов.
Боевые системы премущественно VM/физические сервера. Один нод системы по сути занимает одни серевер.
А масштабирование, это увеличение VM серверов для одной боевой системы.

https://habr.com/ru/post/474068/
```
--

Сценарий:

- Высоконагруженное монолитное java веб-приложение; 
```
VM/физ.сервер, поскольку потребуется маштабирование при увеличении нагрузки.
```
- Nodejs веб-приложение;
````buildoutcfg
Если приложение небольшое, потребуется к примеру развернуть и другие сервисы, то можно отдокеризировать.
````
- Мобильное приложение c версиями для Android и iOS;
```buildoutcfg
    VM/физический сервер, IOS вроде не докеризируется.
```
- Шина данных на базе Apache Kafka;
```buildoutcfg
  VM/физический сервер. Потребуется масштабирование системы. 
```
- Elasticsearch кластер для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana;
```buildoutcfg
Кластерная машина. Здесь подойдет VM, либо набор физ.серверов.
```
- Мониторинг-стек на базе Prometheus и Grafana;  
```
Ну, чтобы не захламлять операционку различными конфигами и т.д., можно быстро докеризировать сервиc.
Но как правило такие сервисы ставятся сразу на операционку VM/физ. сервер.
```
- MongoDB, как основное хранилище данных для java-приложения;
```buildoutcfg
Если база и приложение предполагаются на одной VM/физ.сервере, то можно докеризировать.
```
- Gitlab сервер для реализации CI/CD процессов и приватный (закрытый) Docker Registry.
```buildoutcfg
VM/физ. сервер. В том числе по причине безопасности.
```

## Задача 3

- Запустите первый контейнер из образа ***centos*** c любым тэгом в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
```buildoutcfg
docker run -dt -v ./data:/data centos:8
```
- Запустите второй контейнер из образа ***debian*** в фоновом режиме, подключив папку ```/data``` из текущей рабочей директории на хостовой машине в ```/data``` контейнера;
```buildoutcfg
docker run -dt -v ./data:/data debian:latest
turganovai@vds2260027:~/git/nginx_netology$ docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED              STATUS              PORTS                                               NAMES
ffd5def4677a   debian:latest          "bash"                   18 seconds ago       Up 17 seconds                                                           elegant_feynman
3e22778ab042   centos:8               "bash"                   About a minute ago   Up About a minute    
```
- Подключитесь к первому контейнеру с помощью ```docker exec``` и создайте текстовый файл любого содержания в ```/data```;
```buildoutcfg
docker exec -d ffd5def4677a bash -c "shuf -i 1-10000 -n 1 -o /data/centos.txt"
Содержание файла:
root@ffd5def4677a:/data# cat centos.txt
3985
```
- Добавьте еще один файл в папку ```/data``` на хостовой машине;
```buildoutcfg
docker exec -d ffd5def4677a bash -c "shuf -i 1-10000 -n 1 -o /data/centos2.txt"
```  
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в ```/data``` контейнера.
```buildoutcfg
Второй контейнер 
turganovai@vds2260027:~/git/nginx_netology$ docker exec ffd5def4677a ls -a /data
.
..
centos.txt
centos2.txt
```
## Задача 4 (*)

Воспроизвести практическую часть лекции самостоятельно.

Соберите Docker образ с Ansible, загрузите на Docker Hub и пришлите ссылку вместе с остальными ответами к задачам.


---

### Как cдавать задание

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---
