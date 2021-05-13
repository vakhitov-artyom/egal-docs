| Метод | URI                                       |
|:-----:|:------------------------------------------|
|  GET  | `/[ServiceName]/[ModelName]/getItems`     |
|  GET  | `/[ServiceName]/[ModelName]/getItem/[id]` |

Пример запроса на получение всех работников
`domain/Service/Employee/getItems`

Запросы на чтение также поддерживают дополнительные возможности:

JSON payload

```json
{
  "per_page": 5,
  "page": 2,
  "filter": [
    ["full_name", "eq", "Иванов Иван Иванович"]
  ],
  "withs": [
    "departments"
  ],
  "order": [
    {"column": "id", "direction": "asc"},
    {"column": "email", "direction": "desc"}
  ]
}
```

Здесь

`per_page` - указывает кол-во записей на 1 страницу

`page` - указывает текущую страницу записей

`filter` - указывает фильтрацию по полям (см.
[Фильтры](/server/crud/filters.md))

`order` - указывает сортировку по полям

`withs` - указывает связанные сущности (relations)

