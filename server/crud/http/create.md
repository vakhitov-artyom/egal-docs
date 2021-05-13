| Метод | URI                                     |
|:-----:|:----------------------------------------|
| POST  | `/[ServiceName]/[ModelName]/create`     |
| POST  | `/[ServiceName]/[ModelName]/createMany` |

Пример создания нового работника `domain/Service/Employee/create`

JSON payload:

```json
{
  "attributes": {
    "full_name": "Иванов Иван Иванович",
    "email": "ivanov_ivan@domain.com"
  }
}
```

