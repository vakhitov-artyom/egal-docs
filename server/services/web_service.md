# Web Service

Предназначен для адаптации из HTTP запросов в BUS сообщения.

#### Список версий адаптера

* latest (идентичен последней версии)
* 2
* 1
* 0

#### Как изменять версии адаптера

Нужно добавить `header` `Version`, где его значение будет равно одному
из [списка версий](#Список-версий-адаптера).

По стандарту заголовок равен значению `latest`.

## Version 2

#### Правила использования

* URL должен соответствовать [корректному виду](#Вид-url-запроса)
* Параметр `id` должен указываться единожды: в теле запроса или в URL.
* Тип тела запроса должен быть `JSON`.
* Все названия атрибутов должны быть указаны в
  [Camel Case](https://ru.wikipedia.org/wiki/CamelCase).

#### Вид URL запроса

```
{Domain}/{Service}/{Model}/{Action}/{?id}
```

Расшифровка URL запроса:

| Переменная |                                                           Значение                                                            | Обязательное? |
|:----------:|:-----------------------------------------------------------------------------------------------------------------------------:|:-------------:|
|   Domain   |                                          Доменное имя на котором работает приложение                                          |      Да       |
|  Service   |                                       Название сервиса к которому идет запрос действия                                        |      Да       |
|   Model    |                                        Название модели к которой идет запрос действия                                         |      Да       |
|     id     | Идентификатор, который будет передан в параметры запроса действия.<br>Может принимать как числовые, так и строковые значения. |      Нет      |

Примеры:
* egal.smw.tom.ru/auth/User/getItem/1
* egal.smw.tom.ru/notify/Message/getItems
* egal.smw.tom.ru/sms/Phone/create

#### Пример вызова действия:

Создадим действие `Example/testMessage`:

```php
namespace App\Models;

use Egal\Model\Model as EgalModel;

class Example extends EgalModel
{

    // ...

    /**
     * @param string $testMessage
     * @return string
     * @statuses-access guest,logged
     */
    public static function actionTestMessage(string $testMessage): string
    {
        return 'Message: ' . $testMessage;
    }

    // ...

}
```

Сформируем запрос:

```bash
curl --location --request GET 'domain/service/Example/testMessage' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "test_message": "Hello, John Doe!"
    }'
```

Получим ответ:

```json
{
    "action": {
        "type": "action",
        "service_name": "auth",
        "model_name": "User",
        "action_name": "testMessage",
        "parameters": {
            "test_message": "Hello, John Doe!"
        },
        "token": null,
        "uuid": "ebd0b008-e987-4e50-963e-8f115514a1fe"
    },
    "start_processing": {
        "type": "start_processing",
        "started_at": "2021-03-04T07:54:56.639985Z",
        "uuid": "6dcad2e2-dc03-4007-b787-2419ba8b25bd"
    },
    "action_result": {
        "type": "action_result",
        "data": "Message: Hello, John Doe!",
        "uuid": "33a055c7-065b-4596-8805-ff87eecdc7b7"
    },
    "action_error": null
}
```

## Version 1

### Адаптация под Latest:

#### Из GET Params в параметры действия:

|             GET Parameter              |                                                 Параметр действия                                                  | Стандартное значение | Комментарий                                                                                                                                     |
|:--------------------------------------:|:------------------------------------------------------------------------------------------------------------------:|:--------------------:|:------------------------------------------------------------------------------------------------------------------------------------------------|
|           `_count={number}`            |                                                `per_page={number}`                                                 |          10          |                                                                                                                                                 |
|            `_from={number}`            |                                                  `page={number}`                                                   |          -           | Посчитывается по следующей формуле:<br>`Округление в большую сторону({_from}/(_count))`                                                         |
| `_with=["{relation1}", "{relation2}"]` |                                       `withs=["{relation1}", "{relation2}"]`                                       |          -           |                                                                                                                                                 |
|    `_order={"field": "direction"}`     |                            `order=[{"column": "{field}", "direction": "{direction}"}]`                             |          -           |                                                                                                                                                 |
|           `{field}={value}`            |          `filter=[ ... , "AND", {"field": "{field}", "operator": "=", "value": "{value}"}, "AND", ... ]`           |          -           | Дополняется параметр filter с помощью объединителя `AND`.                                                                                       |
|             `_range_from`              |          `filter=[ ... , "AND", {"field": "{field}", "operator": ">=", "value": "{value}"}, "AND", ... ]`          |          -           |                                                                                                                                                 |
|              `_range_to`               |          `filter=[ ... , "AND", {"field": "{field}", "operator": "<=", "value": "{value}"}, "AND", ... ]`          |          -           |                                                                                                                                                 |
|    `_search={"{field}": "{value}"}`    |       `filter=[ ... , "AND", {"field": "{field}", "operator": "ILIKE", "value": "%{value}%"}, "AND", ... ]`        |          -           |                                                                                                                                                 |
|         `_full_search={value}`         | `filter=[ ... , "AND", [{"field": "field1", "operator": "ILIKE", "value": "%{value}%"}, "OR", ... ], "AND", ... ]` |          -           | Собираются все возможные поля у модели,<br>и дополняется фильтр одной частью с основным объединителем `AND`<br>и внутренними разделителями `OR` |

