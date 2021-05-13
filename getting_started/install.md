# Установка

## Введение

Сейчас мы создадим структуру репозиториев, которая поможет легко и быстро разворачивать Egal Server.

## Первые шаги

1. Создаем новый репозитория
2. Клонируем его в нужную нам директорию и открываем в IDE
3. Создаем файл `.env`, содержанием которого на данном шаге будет:

```dotenv
PROJECT_NAME=project
```

4. Создаем файла `docker-compose.yml`, содержанием которого на данном шаге будет:

```yaml
version: "3.6"

services:

  # Нужен для общения с сервисами через AMQP, WS протоколы. 
  rabbitmq: 
    container_name: ${PROJECT_NAME}-rabbitmq
    image: ${RABBITMQ_TAG:-bitnami/rabbitmq:latest}
    environment:
      RABBITMQ_USERNAME: ${RABBITMQ_USERNAME:-admin}
      RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-password}
      RABBITMQ_PLUGINS: rabbitmq_management,rabbitmq_consistent_hash_exchange
    ports:
      - ${RABBITMQ_PORT:-5672}:5672
      - ${RABBITMQ_MANAGER_PORT:-15672}:15672
      
  # Нужен для общения с сервисами через HTTP протокол.
  web-service: 
    container_name: ${PROJECT_NAME}-web-service
    image: egalbox/web-service:2.0.0beta19
    ports:
      - ${WEB_SERVICE_PORT:-80}:8080
    environment:
      PROJECT_NAME: ${PROJECT_NAME}
      APP_SERVICE_NAME: web
      RABBITMQ_HOST: ${PROJECT_NAME}-rabbitmq
      RABBITMQ_USER: ${RABBITMQ_USERNAME:-admin}
      RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-password}
      WAIT_HOSTS: ${PROJECT_NAME}-rabbitmq:5672
    depends_on:
      - rabbitmq
```

5. Запустим сервисы:

```shell
dokcer-compose up -d
```

6. Проверим работоспособность `web-service`:

```shell
curl localhost:80
```

Мы должны получить примерно такой ответ:

```html
Hello, it's project/web-service!
```

> Если ответ не получен — дождитесь полного старта сервисов.
> Если в течение 1 минут сервисы не запустились — проверьте docker-compose logs и попробуйте решить проблему.
> Если проблема не решена — обратитесь к команде Egal.

## Добавление стандартных сервисов Egal

На примере `auth-service`, мы попробуем дополнить наше приложение новым сервисом.

1. Дополним `docker-compose.yml`, секцию `services` следующими сервисами:

```yaml
# Нужен для работы сервисов, которые взаимодействуют с базой данных
# На данном шаге нужен только для auth-service
database:
  container_name: ${PROJECT_NAME}-database
  image: egalbox/postgres:2.0.0
  restart: always
  ports:
    - ${RABBITMQ_PORT:-5432}:5432
  environment:
  POSTGRES_MULTIPLE_DATABASES: auth
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: ${DATABASE_PASSWORD:-password}

auth-service:
  container_name: ${PROJECT_NAME}-auth-service
  image: egalbox/auth-service:2.0.0beta20
  environment:
    APP_SERVICE_NAME: auth
    APP_SERVICE_KEY: uZsLnAJz35FWUTVx@eg#Xirv6I*jcw2Y
    DB_HOST: ${PROJECT_NAME}-database
    DB_PASSWORD: ${DATABASE_PASSWORD:-password}
    RABBITMQ_HOST: ${PROJECT_NAME}-rabbitmq
    RABBITMQ_USER: ${RABBITMQ_USERNAME:-admin}
    RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-password}
    WAIT_HOSTS: ${PROJECT_NAME}-rabbitmq:5672, ${PROJECT_NAME}-database:5432
  depends_on:
    - rabbitmq
    - database
```

2. Запустим сервисы:

```shell
dokcer-compose up -d
```

3. Проверим работоспособность `auth-service`

```shell
curl localhost:80/auth/User/test
```

Мы должны получить ответ, содержащий `action_result` или `action_error`, это будет означать, что наш сервис ответил.

## Добавление собственных сервисов

На примере `monolit-service`, мы попробуем дополнить наше приложение новым сервисом.

1. Инициализируем composer проект:

```shell
composer create-project egal/egal monolit-service
```

> Если вы получите ошибку `Could not find package egal/egal with stability stable.` - выполните следующую команду вместо предыдущей:
> ```shell
> composer create-project egal/egal monolit-service [VERSION] --stability dev
> ```
> Где `VERSION` - необходимая вам не стабильная версия.

2. Дополним `docker-compose.yml`, секцию `services` следующими сервисами:

```yaml
# Нужен для работы сервисов, которые взаимодействуют с базой данных
# На данном шаге нужен только для auth-service
# Если у вас уже есть database - можно просто добавить название новой базы данных в POSTGRES_MULTIPLE_DATABASES переменной через запятую
database:
  container_name: ${PROJECT_NAME}-database
  image: egalbox/postgres:2.0.0
  restart: always
  ports:
    - ${RABBITMQ_PORT:-5432}:5432
  environment:
  POSTGRES_MULTIPLE_DATABASES: monolit
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: ${DATABASE_PASSWORD:-password}

monolit-service:
  container_name: ${PROJECT_NAME}-auth-service
  build:
    context: monolit-service
  environment:
    APP_SERVICE_NAME: monolit
    APP_SERVICE_KEY: uiA3ZsU#x@ecwgJrv6ITjLV*nzX5FW2Y
    DB_HOST: ${PROJECT_NAME}-database
    DB_PASSWORD: ${DATABASE_PASSWORD:-password}
    RABBITMQ_HOST: ${PROJECT_NAME}-rabbitmq
    RABBITMQ_USER: ${RABBITMQ_USERNAME:-admin}
    RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-password}
    WAIT_HOSTS: ${PROJECT_NAME}-rabbitmq:5672, ${PROJECT_NAME}-database:5432
  depends_on:
    - rabbitmq
    - database
```

3. Запустим сервисы:

```shell
dokcer-compose up -d
```

4. Проверим работоспособность `monolit-service`

```shell
curl localhost:80/monolit/User/test
```

5. Превращаем `monolit-service` директорию в git submodule:

```shell
mv monolit-service/ temp/
```

```shell
git submodule init
```

```shell
git submodule add [URL] monolit-service
```

```shell
rsync -r ./temp/ ./monolit-service/
```

```shell
rm -r temp
```

На данном этапе, если вы используете IDE - её лучше перезапустить, чтобы проиндексировались git submodules.
Далее делаем commit и push всех новых файлов.

## Пояснения к установке

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
