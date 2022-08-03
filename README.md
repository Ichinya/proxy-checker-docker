# proxy-checker-docker

Комплект по проверке прокси адресов. Включает в себя:

* [proxy-checker](https://github.com/Ichinya/proxy_cheker) - проверка прокси, потребляем примерно 15мб ОЗУ на каждый
* [proxy-producer](https://github.com/Ichinya/proxy_producer) - собирает списки прокси и отправляет на проверку, потребляет примерно 12мб ОЗУ. При импорте и отправки в очередь грузит максимум ОЗУ и неплохо нагружает процессор. 
* rabbitmq - менеджер очередей, через него происходит отправка на проверку. Уходит на работа 150Мб
* postgresql - база данных для хранения списков и результатов. Нужно для работы 100-300Мб

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

Для подключения к RabbitMQ нужно добавить порты к сервису:
```yaml
    ports:
      - 5672:5672
      - 15672:15672
```

Для редактирования и просмотра базы данных можно открыть порт 5432 в сервисе или использовать допонительный образ Docker SQLPad, просто нужно добавить в композицию новый сервис, логин и пароль some@think.ru и пароль password:
```yaml

  sqlpad:
    image: sqlpad/sqlpad:latest
    container_name: sqlpad
    ports:
      - 3000:3000
    environment:
      SQLPAD_ADMIN: 'some@think.ru'
      SQLPAD_ADMIN_PASSWORD: 'password'
      SQLPAD_APP_LOG_LEVEL: info
      SQLPAD_WEB_LOG_LEVEL: warn
      SQLPAD_CONNECTIONS__sqlserverdemo__name: DB
      SQLPAD_CONNECTIONS__sqlserverdemo__driver: postgres
      SQLPAD_CONNECTIONS__sqlserverdemo__host: database
      SQLPAD_CONNECTIONS__sqlserverdemo__database: homestead
      SQLPAD_CONNECTIONS__sqlserverdemo__username: user
      SQLPAD_CONNECTIONS__sqlserverdemo__password: pass
    volumes:
      - ./sqlpad-volume:/var/lib/sqlpad
    depends_on:
      - database

```


## Примечания
В файле `docker-compose.yaml` можно найти у proxy-checker `replicas: 2` - это количество чекеров. Каждый потребляем примерно 15-20мегабайт ОЗУ. Но из-за небольшого количество сайтов проверок ip адресов получается большое количество ошибок. Нормально работает примерно 5 чекеров.

Также открыты порты для подключения к БД и RabbitMQ. К mysql подключится можно по адресу 127.0.0.1:3601 используя логин user и пароль pass. А к кролику подключаемся по адресу http://localhost:15672/ с логином и паролем guest

## Возможно появится:
* Работа без mysql (для уменьшения потребляемой памяти), возможно, будет использован sqlite3
* Работа с файлами. Закидываем файлы со списками, а после проверки - получаем готовый список
* Будут сделаны готовые образы
* Будет добавлен Gearman, так как кролик немало жрёт ОЗУ.
* Команды для управления producer.
* Планировщик. Либо cron у producer, либо отдельный образ.