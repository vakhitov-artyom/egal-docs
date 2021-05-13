| Меread.md |                                            |
|:---------:|:-------------------------------------------|
|    PUT    | `/[ServiceName]/[ModelName]/update`        |
|    PUT    | `/[ServiceName]/[ModelName]/updateMany`    |
|    PUT    | `/[ServiceName]/[ModelName]/updateManyRaw` |

Пример обновления сущности работник `domain/Service/Employee/update`

JSON payload:

```json
{
  "id": "ea806608-71bb-4279-84f1-6d8473b9076c",
  "attributes": {
    "full_name": "Иванов Иван Иванович",
    "email": "ivanov_ivan@domain.com"
  }
}
```

