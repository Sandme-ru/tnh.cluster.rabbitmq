# tnh.rabbitmq.cluster
## Конфигурируем docker-compose
Как всегда, конфигурация для docker-compose берется из файла docker-compose.yml. Им и займёмся:
```
version: '2'
services:
    rabbit:
        image: rabbitmq:management
        hostname: rabbit
        ports:
            - "15672:15672"
        environment:
            - RABBITMQ_ERLANG_COOKIE='secrets'
    hamster:
        image: rabbitmq:management
        hostname: hamster
        ports:
            - "15673:15672"
        environment:
            - RABBITMQ_ERLANG_COOKIE='secrets'
```
1. Два сервиса (контейнера), которые после запуска превратятся в RabbitMQ узлы с именами rabbit и hamster. Хомяк (hamster) имеет очень опосредованное отношение к кроликам (rabbit), но вы не представляете, как утомительно гуглить кроличьи семейства ночью.
2. Имена контейнерных хостов (hostname) заданы явно. Они будут фигурировать в именах RabbitMQ узлов и лучше бы им быть читабельнее.
3. Так как RabbitMQ вэб-админка будет выглядывать из контейнера через порт 15672, я открою его и со стороны хоста. Но контейнеров два, так что второй, с хомяком, будет доступен снаружи через порт 15673.
4. В официальный образ rabbitmq можно передать Erlang cookie в качестве переменой окружения — RABBITMQ_ERLANG_COOKIE, и тогда не нужно делать это руками через .erlang.cookie файл. Очень удобно. Свою cookie я назвал ‘mysecret’, но подошло бы что угодно.
## Запускаем сервисы
```
docker-compose -up -d --build
```
Порт мы уже знаем — 15672 для rabbit и 15673 для hamster, но IP адрес зависит от версии Docker и операционной системы. Это будет либо localhost, либо тот IP, который вернёт  docker-machine ip  или  boot2docker ip . Логин и пароль от админки — guest:guest.
## Запускаем кластер
Время делать кластер. Второй и последний шаг в создании кластера — вызов команды  rabbitmqctl join_cluster на всех брокерах, кроме первого. В нашем случае «все, кроме первого» — это hamster (или rabbit — смотря, кто больше нравится). docker-compose постарался сделать имя контейнера уникальным и добавил свой префикс, так что прежде чем зайти в него, нужно узнать настоящее имя:
```
docker-compose ps
#      Name                     Command                                                  
#---------------------------------------------------
#cluster_hamster_1   docker-entrypoint.sh rabbi ... 
#cluster_rabbit_1    docker-entrypoint.sh rabbi .
```
```
docker exec -ti cluster_hamster_1 bash
#root@hamster:/#
```
Делаем кластер:
```
rabbitmqctl stop_app                                                                                                                                                                                                                                         
#Stopping node rabbit@hamster ...
rabbitmqctl join_cluster rabbit@rabbit
#Clustering node rabbit@hamster with rabbit@rabbit ...
rabbitmqctl start_app
#Starting node rabbit@hamster ...                                                                                                                                                                                                                       
```
