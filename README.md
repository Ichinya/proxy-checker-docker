# proxy-checker-docker

Комплект по проверке прокси адресов. Включает в себя:

* [proxy-checker](https://github.com/Ichinya/proxy_cheker) - проверка прокси, потребляем примерно 15мб ОЗУ на каждый
* [proxy-producer](https://github.com/Ichinya/proxy_producer) - собирает списки прокси и отправляет на проверку, потребляем примерно 12мб ОЗУ
* rabbitmq - менеджер очередей, через него происходит отправка на проверку. Уходит на работа 150Мб
* mysql - база данных для хранения списков и результатов. Нужно для работы 300-500Мб

## Запуск

Клонируем исходники с гитхаб. Потом будет сделаны отдельные образы, чтобы запускалось двумя командами.
Запускаем сборку.
После запуска отправляем команды на сбор прокси и, вторую команду, на отправку собранных прокси на проверку

```shell
git clone https://github.com/Ichinya/proxy-checker-docker.git .
cd proxy-checker-docker
git clone https://github.com/Ichinya/proxy_producer.git .
git clone https://github.com/Ichinya/proxy_cheker.git .

docker compose -f "docker-compose.yaml" up -d --build 

docker exec proxy_producer python3 update_list_proxy.py
docker exec proxy_producer python3 send_to_mq.py
```

## Примечания
В файле `docker-compose.yaml` можно найти у proxy-checker `replicas: 2` - это количество чекеров. Каждый потребляем примерно 15-20мегабайт ОЗУ. Но из-за небольшого количество сайтов проверок ip адресов получается большое количество ошибок. Нормально работает примерно 5 чекеров.

## Возможно появится:
* Работа без mysql (для уменьшения потребляемой памяти), возможно будет использован sqlite3
* Работа с файлами, закидываем файлы со списками. После проверки - получаем готовый список
* Будут сделаны готовые образы
* Будет добавлен Gearman, так как кролик немало жрёт ОЗУ.