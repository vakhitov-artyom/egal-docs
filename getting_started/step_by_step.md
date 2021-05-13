# Пример по шагам

Рассмотрим пример создания части проекта Skilltoria.
Для простоты рассмотрим создание сущностей `WorkingTime` и `Speaker`.

> Сам проект представляет собой платформу для организации и проведения видео-уроков.

Проект будет состоять из:
- микросервиса monolit-service - основной микросервис в котором будет наше приложение.
- сервиса авторизации
- веб-сервиса

Статья состоит из следующих частей:
1. Разворот проекта
2. Создание сущностей
3. Авторизация

## 1. Разворот проекта

Инициализируем `monolit`.

<!-- TODO: Описать как инициализировать -->

В итоге имеем следующую структуру директории:

```text
monolit-service/
docker-compose.yml
assistent             скрипт помощник для работы с проектом
.gitignore
.gitmodules           файл в котором храниться описание существующих подмодулей
.git                  системная директория вашего репозитория
.env                  файл конфигурации в котором хранится часть переменных окружения
```

Установим зависимости. Заходим в директорию `monolit-service` и выполняем:

```shell
docker run --rm --interactive --tty --volume $PWD:/app --user $(id -u):$(id -g) composer install --ignore-platform-reqs
```

Настраиваем docker-compose.yml

Удаляем закомментированные сервисы в начале файле (такие, как pgadmin, k6, php-documentor), они нам сейчас не нужны.
Сейчас содержимое файла должно быть таким

```yaml
version: "3.6"

services:
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
```

И далее добавляем в конец то что нам пригодится.
Во-первых, нам нужна база данных.

```yaml
    database:
      container_name: ${PROJECT_NAME}-database
      image: egalbox/postgres:2.0.0 # указываем адрес, откуда подтягиваем образ контейнера
      restart: always # всегда перезапускаем контейнер
      ports:
        - ${RABBITMQ_PORT:-5432}:5432
      environment:
        POSTGRES_MULTIPLE_DATABASES: auth,monolit # через запятую указываем названия баз данных которые используются в нашем проекте. Эта переменная не стандартная и работает только с образом egalbox/postgres:2.0.0.
        POSTGRES_USER: postgres
        POSTGRES_PASSWORD: ${DATABASE_PASSWORD:-password}
```

Далее указываем сервисы поставляемые с `egal-box`

```yaml
 web-service:
   container_name: ${PROJECT_NAME}-web-service
   image: egalbox/web-service:2.0.0beta19
   ports:
     - ${WEB_SERVICE_PORT:-81}:8080
   environment:
     PROJECT_NAME: ${PROJECT_NAME}
     APP_SERVICE_NAME: web
     RABBITMQ_HOST: ${PROJECT_NAME}-rabbitmq
     RABBITMQ_USER: ${RABBITMQ_USERNAME:-admin}
     RABBITMQ_PASSWORD: ${RABBITMQ_PASSWORD:-password}
     WAIT_HOSTS: ${PROJECT_NAME}-rabbitmq:5672
   depends_on:
     - rabbitmq

 auth-service:
  container_name: ${PROJECT_NAME}-auth-service
  image: egalbox/auth-service:2.0.0beta20
  environment:
    APP_SERVICE_NAME: auth
    APP_SERVICE_KEY: uZn35FJAx@sg*eczrv6ITjLVWU#Xiw2Y
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

Осталось описать наш сервис `monolit`

```yaml
 monolit-service:
   container_name: ${PROJECT_NAME}-monolit-service
   build: monolit-service
   volumes:
     - ./monolit-service/:/app/
   environment:
     APP_SERVICE_NAME: monolit
     APP_SERVICE_KEY: B#J5mUWKh8FqzQ6Tj0XtYruIcSwpb@ed
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

### Настройка .env

Основное, что нам нужно настроить это переменная `PROJECT_NAME`, но она уже настроена на этапе установки через скрип install.

> PROJECT_NAME=testProject

### Миграции

Миграции выполнять вручную не нужно т.к. они выполнятся при старте сервиса.

### Запуск проекта

Запуск производится одной простой командой

```shell
docker-compose up -d
```

Далее нужно проверить все ли запустилось. Для этого выполним команду вывода списка работающих контейнеров.

```shell
docker-compose ps
```

Должны получить что-то подобное

```text
            Name                           Command               State                                             Ports                                           
-------------------------------------------------------------------------------------------------------------------------------------------------------------------
testProject-auth-service        docker-php-entrypoint /bin ...   Up                                                                                                
testProject-database            docker-entrypoint.sh postgres    Up      5432/tcp                                                                                  
testProject-monolit-service     docker-php-entrypoint /bin ...   Up                                                                                                
testProject-rabbitmq            /opt/bitnami/scripts/rabbi ...   Up      15671/tcp, 0.0.0.0:15672->15672/tcp, 25672/tcp, 4369/tcp, 5671/tcp, 0.0.0.0:5672->5672/tcp
testProject-web-service         /bin/sh -c /wait && /usr/b ...   Up      0.0.0.0:81->8080/tcp                                                                      

```

Сделаем запрос для проверки.

```shell
curl http://localhost:81
```
В ответ получим

```text
Hello, it's testProject/web-service!
```
На этом разворот можно считать законченным.

## 2. Создание сущностей

Начнем с генерации модели `Speaker`.
Зайдем в контейнер и создадим модель

```shell
docker-compose exec monolit-service bash
php artisan egal:make:model Speaker
```

После выполнения этой команды у нас сгенерировалась модель со следующим содержимым

```php
<?php

namespace App\Models;

use Egal\Model\Model as EgalModel;

/**
 * @property $id
 * @property $name {@validation-rules required|string}
 * @property $created_at
 * @property $updated_at
 *
 * @action getMetadata {@statuses-access guest,logged}
 * @action getItem {@statuses-access guest,logged}
 * @action getItems {@statuses-access logged} {@roles-access user}
 * @action create {@statuses-access logged} {@permissions-access super_permission}
 * @action update {@statuses-access logged} {@permissions-access super_permission}
 * @action delete {@statuses-access logged} {@permissions-access super_permission}
 */
class Speaker extends EgalModel
{

}
```
Аналогично создадим сущность `WorkingTime`.

```shell
php artisan egal:make:model WorkingTime
```

Внесем следующие правки.

- пока что для удобства сделаем публичными все роуты.
- проставим типы полей и валидацию
- Укажем связанные сущности. Для `WorkingTime` это `speaker`, для `Speaker` это `workingTimes`.

В итоге получаем

`app/Models/Speaker.php`

```php
<?php

namespace App\Models;

use Carbon\Carbon;
use Egal\Model\Model as EgalModel;
use Illuminate\Database\Eloquent\Collection;
use Illuminate\Database\Eloquent\Relations\HasMany;

/**
 * @property integer $id           {@property-type field}  {@primary-key}
 * @property uuid $uid             {@property-type field}  {@validation-rules required|uuid|unique:speakers,uid|unique:schools,uid}
 * @property string $name          {@property-type field}  {@validation-rules required|string}
 * @property string $surname       {@property-type field}  {@validation-rules required|string}
 * @property string $avatar        {@property-type field}  {@validation-rules string}
 * @property string $video         {@property-type field}  {@validation-rules string}
 * @property Carbon $created_at    {@property-type field}
 * @property Carbon $updated_at    {@property-type field}
 *
 * @property Collection|WorkingTime[] $workingTimes     {@property-type relation}
 *
 * @action getMetadata  {@statuses-access guest,logged}
 * @action getItem      {@statuses-access guest,logged}
 * @action getItems     {@statuses-access guest,logged}
 * @action create       {@statuses-access guest,logged}
 * @action update       {@statuses-access guest,logged}
 * @action delete       {@statuses-access guest,logged}
 */
class Speaker extends EgalModel
{

    protected $fillable = [
        'uid',
        'name',
        'surname',
        'avatar',
        'video',
    ];

    protected $hidden = [
        'created_at',
        'updated_at',
    ];
    
    public function workingTimes(): HasMany
    {
        return $this->hasMany(WorkingTime::class);
    }

}

```

`app/Models/WorkingTime.php`

```php
<?php

namespace App\Models;

use Carbon\Carbon;
use Egal\Model\Model as EgalModel;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

/**
 * @property integer $id           {@property-type field}  {@primary-key}
 * @property integer $speaker_id   {@property-type field}  {@validation-rules required|exists:speakers,id}
 * @property Carbon $starts_at     {@property-type field}  {@validation-rules required|date|before:ends_at}
 * @property Carbon $ends_at       {@property-type field}  {@validation-rules required|date|after:starts_at}
 * @property Carbon $created_at    {@property-type field}
 * @property Carbon $updated_at    {@property-type field}
 *
 * @property Speaker $speaker {@property-type relation}
 *
 * @action getMetadata          {@statuses-access guest,logged}
 * @action create               {@statuses-access guest,logged}
 * @action update               {@statuses-access guest,logged}
 * @action delete               {@statuses-access guest,logged}
 * @action actionCreateMany     {@statuses-access guest,logged}
 * @action actionDeleteManyRaw  {@statuses-access guest,logged}
 * @action getItem              {@statuses-access guest,logged}
 * @action getItems             {@statuses-access guest,logged}
 */
class WorkingTime extends EgalModel
{

    protected $fillable = [
        'speaker_id',
        'starts_at',
        'ends_at',
    ];

    protected $hidden = [
        'created_at',
        'updated_at',
    ];

    public function speaker(): BelongsTo
    {
        return $this->belongsTo(Speaker::class);
    }
    
}

```

Далее нам нужно сгенерировать миграции для этих сущностей

```shell
php artisan egal:make:migration-create Speaker
php artisan egal:make:migration-create WorkingTime
```

Немного подправим их, т.к. нужно прописать связи на таблицы

`database/migrations/2021_04_27_064152_create_speakers_table.php`
```php
class CreateSpeakersTable extends Migration
{
    public function up()
    {
        Schema::create('speakers', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('surname');
            $table->string('avatar')
                ->nullable();
            $table->string('video')
                ->nullable();
            $table->timestamps();
        });
    }
/** ... */
}
```

`database/migrations/2021_04_27_064229_create_working_times_table.php`

```php
class CreateWorkingTimesTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('working_times', function (Blueprint $table) {
            $table->id();
            $table->unsignedBigInteger('speaker_id');
            $table->foreignId('speaker_id')
                ->constrained()
                ->onDelete('cascade');
            $table->timestamp('starts_at');
            $table->timestamp('ends_at');
            $table->timestamps();
        });
    }
/** ... */
}
```

Осталось перезапустить микросервис `monolit`, чтобы выполнились миграции и применился новый код.

```shell
docker-coompose restart monolit-service
```

Для проверки работоспособности сделаем пару запросов на создание спикеров и рабочего времени.

```shell
curl -iLXPOST -H "Content-Type: application/json" -d '{"attributes": {"name": "Ivan", "surname": "Ivanov"}}' http://localhost:81/monolit/Speaker/create
curl -iLXPOST -H "Content-Type: application/json" -d '{"attributes": {"speaker_id": "1", "starts_at": "2021-04-27 03:23:57", "ends_at": "2021-04-28 03:23:57"}}' http://localhost:81/monolit/WorkingTime/create
```

Сделаем запросы на получение созданных данных

```shell
curl http://localhost:81/monolit/Speaker/getItems
```

Результат

```json
{
  "action": {
    "response": {},
    "connection": {},
    "type": "action",
    "service_name": "monolit",
    "model_name": "Speaker",
    "action_name": "getItems",
    "parameters": [],
    "token": null,
    "uuid": "710b36c0-77fb-4906-a4c2-234bf53fb09a"
  },
  "start_processing": {
    "type": "start_processing",
    "started_at": "2021-04-28T03:38:18.464676Z",
    "uuid": "5ae27e60-d05b-4b11-97eb-80ea0a81ecac",
    "action_message": {
      "type": "action",
      "service_name": "monolit",
      "model_name": "Speaker",
      "action_name": "getItems",
      "parameters": [],
      "token": null,
      "uuid": "710b36c0-77fb-4906-a4c2-234bf53fb09a"
    }
  },
  "action_result": {
    "type": "action_result",
    "data": {
      "current_page": 1,
      "total_count": 1,
      "per_page": 10,
      "items": [
        {
          "id": 1,
          "name": "Ivan",
          "surname": "Ivanov",
          "avatar": null,
          "video": null
        }
      ]
    },
    "uuid": "42e389bb-30d6-4483-bd84-7be78a5654dc",
    "action_message": {
      "type": "action",
      "service_name": "monolit",
      "model_name": "Speaker",
      "action_name": "getItems",
      "parameters": [],
      "token": null,
      "uuid": "710b36c0-77fb-4906-a4c2-234bf53fb09a"
    }
  },
  "action_error": null
}
```

Создание рабочего времени
```shell
curl -iLXPOST -H "Content-Type: application/json" -d '{"attributes": {"speaker_id": "1", "starts_at": "2021-04-27 03:23:57", "ends_at": "2021-04-28 03:23:57"}}' http://localhost:81/monolit/WorkingTime/create
```

Получаем результат
```shell
curl http://localhost:81/monolit/WorkingTime/getItems
```

## 3. Авторизация

Теперь настало время прикрыть доступ для незарегистрированных пользователей.

Уберем у каждой модели разрешения для guest. Соответственно меняем 
`{@statuses-access guest,logged}` на `{@statuses-access logged}`

Далее нам нужно зарегистрировать нового пользователя.

```shell
curl -iLXPOST -H "Content-Type: application/json" -d '{"email": "ivan@mail.ru", "password": "qazwsx"}' http://localhost:81/auth/User/register
```

Т.к. это "чистый" проект, то сервис авторизации не знает о существовании сервиса `monolit`.

Заходим в контейнер
```shell
docker-compose exec auth-service bash
```

Регистрируем `monolit`
```shell
php artisan egal:register:service monolit B#J5mUWKh8FqzQ6Tj0XtYruIcSwpb@ed
```

Получаем master token 
```shell
curl -iLXPOST -H "Content-Type: application/json" -d '{"email": "ivan@mail.ru", "password": "qazwsx"}' http://localhost:81/auth/User/loginByEmailAndPassword
```

Ответ:
```json
{
  "action": {
    "response": {},
    "connection": {},
    "type": "action",
    "service_name": "auth",
    "model_name": "User",
    "action_name": "loginByEmailAndPassword",
    "parameters": {
      "email": "ivan@mail.ru",
      "password": "qazwsx"
    },
    "token": null,
    "uuid": "dec38c4e-c09c-412b-8225-6038aa39258d"
  },
  "start_processing": {
    "type": "start_processing",
    "started_at": "2021-04-28T06:17:27.493838Z",
    "uuid": "9be6e7b9-2f31-4062-b5c6-5f3413882e86",
    "action_message": {
      "type": "action",
      "service_name": "auth",
      "model_name": "User",
      "action_name": "loginByEmailAndPassword",
      "parameters": {
        "email": "ivan@mail.ru",
        "password": "qazwsx"
      },
      "token": null,
      "uuid": "dec38c4e-c09c-412b-8225-6038aa39258d"
    }
  },
  "action_result": {
    "type": "action_result",
    "data": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4",
    "uuid": "2cb792f4-6a42-4cc8-ac20-9f9d24c09354",
    "action_message": {
      "type": "action",
      "service_name": "auth",
      "model_name": "User",
      "action_name": "loginByEmailAndPassword",
      "parameters": {
        "email": "ivan@mail.ru",
        "password": "qazwsx"
      },
      "token": null,
      "uuid": "dec38c4e-c09c-412b-8225-6038aa39258d"
    }
  },
  "action_error": null
}
```

Нам нужен токен который находится в поле `data`:

```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4
```

Теперь нам нужно получить service token для того чтобы мы могли делать запросы на защищенные роуты нешего микросервиса
```shell
curl -iLXPOST -H "Content-Type: application/json" -d '{"token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4", "service_name": "monolit"}' http://localhost:81/auth/User/loginToService
```

Ответ

```json
{
  "action": {
    "response": {},
    "connection": {},
    "type": "action",
    "service_name": "auth",
    "model_name": "User",
    "action_name": "loginToService",
    "parameters": {
      "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4",
      "service_name": "monolit"
    },
    "token": null,
    "uuid": "bc4eb176-6a98-4deb-a4c4-be372864c4dc"
  },
  "start_processing": {
    "type": "start_processing",
    "started_at": "2021-04-28T08:01:35.214688Z",
    "uuid": "a36d9b2c-b288-404b-8017-051990c04784",
    "action_message": {
      "type": "action",
      "service_name": "auth",
      "model_name": "User",
      "action_name": "loginToService",
      "parameters": {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4",
        "service_name": "monolit"
      },
      "token": null,
      "uuid": "bc4eb176-6a98-4deb-a4c4-be372864c4dc"
    }
  },
  "action_result": {
    "type": "action_result",
    "data": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidXN0IiwiYXV0aF9pbmZvcm1hdGlvbiI6eyJpZCI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImVtYWlsIjoiaXZhbkBtYWlsLnJ1IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsInJvbGVzIjpbXSwicGVybWlzc2lvbnMiOltdfSwiYWxpdmVfdW50aWwiOiIyMDIxLTA0LTI4VDA4OjExOjM1LjIyMzUzNFoifQ.oonpQvvOAycWg5W2Bkh_fvETQofLs6Wc0J3Y5eT3c04",
    "uuid": "4e520fb5-9b23-4f40-be76-6ac826380b35",
    "action_message": {
      "type": "action",
      "service_name": "auth",
      "model_name": "User",
      "action_name": "loginToService",
      "parameters": {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidW10IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImFsaXZlX3VudGlsIjoiMjAyMS0wNC0yOVQwODowMDoyMi4zMjEyNzJaIn0.8MNzIB137LC1QYIOt7Io3zTfSO9xUbklaTn5xB_7yP4",
        "service_name": "monolit"
      },
      "token": null,
      "uuid": "bc4eb176-6a98-4deb-a4c4-be372864c4dc"
    }
  },
  "action_error": null
}
```

Теперь попробуем сделать запрос в наш сервис с полученным токеном:

```shell
curl -H "Authorization: eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0eXBlIjoidXN0IiwiYXV0aF9pbmZvcm1hdGlvbiI6eyJpZCI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsImVtYWlsIjoiaXZhbkBtYWlsLnJ1IiwiYXV0aF9pZGVudGlmaWNhdGlvbiI6IjU1MDcwNjQzLTMzOTAtNDM4My1hYzA0LWM3ODM4NzdmZGYwMiIsInJvbGVzIjpbXSwicGVybWlzc2lvbnMiOltdfSwiYWxpdmVfdW50aWwiOiIyMDIxLTA0LTI4VDA4OjExOjM1LjIyMzUzNFoifQ.oonpQvvOAycWg5W2Bkh_fvETQofLs6Wc0J3Y5eT3c04" http://localhost:81/monolit/Speaker/getItems
```
Если все хорошо, то вы увидите тот же самый результат, как и в прошлый раз.
Иначе, если запрос отрабатывает с ошибками, то проверьте каждый шаг с начала.

## Заключение

Мы с вами прошли основные шаги создания проекта с нуля и как итог получили "заготовку" микросервиса с настроенной авторизацией.
