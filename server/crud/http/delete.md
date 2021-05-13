| Метод  | URI                                        |
|:------:|:-------------------------------------------|
| DELETE | `/[ServiceName]/[ModelName]/delete`        |
| DELETE | `/[ServiceName]/[ModelName]/deleteMany`    |
| DELETE | `/[ServiceName]/[ModelName]/deleteManyRaw` |

Пример удаления работника `domain/Service/Employee/delete`

JSON payload:

```json
{
  "attributes": {
    "id": 1
  }
}
```

