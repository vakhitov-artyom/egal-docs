# Установка

## Введение

Для удобства мы предлагаем использовать подход с подмодулями git (см.
[Git Tools Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules)).

Это позволяет быстро разворачивать проект и в дальнейшем комфортно
работать.

## Скрипт установки

В разработке.

### Пояснения к установке

### Web service

На первом этапе вам будет предложено создать `web-service`. Он нужен
если вы собираетесь использовать `HTTP` запросы. В противном случае
используется подход работы на `WebSocket`.

### Auth service

У вас будет выбор установить поставляемый нами, либо сделать `fork` и
внести необходимые изменения.

### Микросервис

Мы поставляем шаблон сервиса
который предназначен для ваших целей. Вам будет предложено вставить
ссылку на уже существующий репозитории (fork от шаблона сервиса). Скрипт
добавит его как подмодуль в ваш новый проект.

Либо все это вы можете сделать самостоятельно. Главное — вам необходимо
будет убедиться что ваш сервис привязан к конкретному репозиторию и вы
добавили его в подмодули.

Добавить репозитории как подмодуль можно следующим образом:

```shell
git submodule add REPOSITORY_ORIGIN_URL
```

где REPOSITORY_ORIGIN_URL - ссылка в виде `HTTP` или `SSH`.

Посмотреть установленные подмодули можно в `.gitmodules`.

### Проект

В итоге в директории с проектом появится следующая структура файлов и
папок.

Базовый проект будет иметь следующую структуру:

```text
web-service/          веб-сервис, нужен если вы используете http запросы на клиенте
auth-service/         сервис авторизации, по умолчанию поставляемый нами, но вы можете сделать свой
your-service/         ваш сервис который вы будете разрабатывать
docker-compose.yml    файл YAML, определяющий как будут запускаться ваши сервисы, сеть между ними и тома для приложения Docker
.gitmodules           файл в котором храниться описание существующих подмодулей
.git                  системная директория вашего репозитория
```

## Настройка, переменные окружения и файлы конфигурации

Конфигурирование каждого сервиса зашито в docker-compose.yml файле через
переменные окружения. Пример auth-service

```yaml
...
  auth-service:
    container_name: my_project-${AUTH_SERVICE_NAME:-auth}-service
    build: ${AUTH_SERVICE_FOLDER:-./auth-service}
    volumes:
      - ${AUTH_SERVICE_FOLDER:-./auth-service}/:/app/
    environment:
      APP_NAME: ${AUTH_SERVICE_NAME:-auth}
      APP_ENV: local
      DB_CONNECTION: pgsql
      DB_HOST: tt-${AUTH_SERVICE_NAME:-auth}-database
      DB_PORT: 5432
      DB_DATABASE: ${AUTH_SERVICE_NAME:-auth}
      DB_USERNAME: postgres
      DB_PASSWORD: ${AUTH_SERVICE_DATABASE_PASSWORD:-auth-service}
      RABBITMQ_HOST: tt-rabbitmq
      RABBITMQ_PORT: ${RABBITMQ_PORT:-5672}
      RABBITMQ_USER: ${RABBITMQ_USERNAME:-admin}
      RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-admin}
    command: ./artisan egal:run
...
```

Здесь с `APP_NAME` по `RABBITMQ_PASSWORD` переменные окружения, которые
при обычном подходе находятся в файле `.env` в корне микросервиса.
Подробнее можно посмотреть здесь
[Lumen Configuration](https://lumen.laravel.com/docs/8.x/configuration).

Все переменные окружения которые указаны как
`${AUTH_SERVICE_FOLDER:-./auth-service}` и т.д. можно задать в файле
`.env` корня проекта рядом с `docker-compose.yml`. Либо будет
использоваться значение по умолчанию, в данном примере это
`./auth-service`.


## Запуск проекта

Запуск осуществляется путем выполнения команды

```shell
docker-compose up -d
```

## Ручная установка

Так-же вы можете собрать ваш рабочий проект по частям не используя
подмодули git.

Для этого нужно отдельно сделать клон каждый сервис отдельно и потом
настроить docker-compose.yml.
