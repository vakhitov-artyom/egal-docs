# Метаданные

## Описание метаданных у модели

Описание метаданных происходит путем прописывания phpDocs у модели.

Пример:

```php
/**
 * @property int $id            {@property-type field}  {@primary-key}
 * @property string $email      {@property-type field}  {@validation-rules required|string|email|unique:users|email}
 * @property string $password   {@property-type field}  {@validation-rules required|string}
 * @property Carbon $created_at {@property-type field}
 * @property Carbon $updated_at {@property-type field}
 *
 * @property Collection $roles          {@property-type relation}
 * @property Collection $permissions    {@property-type relation}
 *
 * @action getItems     {@statuses-access guest}
 * @action getItem      {@statuses-access guest}
 * @action getMetadata  {@statuses-access guest}
 * @action create       {@statuses-access guest}
 * @action update       {@statuses-access guest}
 * @action delete       {@statuses-access guest}
 */
class User extends Model
{

}
```

### Перечень всевозможных тегов

|         Тег         | Описание                                                            |                       Возможные значения                       |
|:-------------------:|:--------------------------------------------------------------------|:--------------------------------------------------------------:|
|      @property      | Указывает на то что данный элемент является полем                   |                                                                |
|   @property-type    | Описывает тип поля                                                  |                        field, relation                         |
|  @validation-rules  | Правила валидации для поля основанные на правилах laravel           | [Cм. документацию](https://laravel.com/docs/master/validation) |
|    @primary-key     | Описывает первичный ключ                                            |                         Без параметров                         |
|       @action       | Описывает endpoint                                                  |                                                                |
|  @statuses-access   | Описывает доступ к действию в зависимости от статуса аутентификации |                         logged, guest                          |
|    @roles-access    | Описывает доступ к действию в зависимости от релей                  |                 admin, задаются пользователем                  |
| @permissions-access | Описывает доступ к действию в зависимости от разрешений             |              authenticate, задаются пользователем              |


## Метаданные модели и её действий

Для получения можно обратиться к действию `getMetadata` у любой модели.

## Метаданные меню

Для получения можно обратиться к сервису `interface` к модели
`MenuMetadata` к действию `getItem`.

> Для получения древовидной структуры пунктов меню добавьте `with`
> `entries_tree`.

> Сопоставление `InterfaceMetadata` и `MenuEntryMetadata`:
> `InterfaceMetadata.id` = `MenuEntryMetadata.interface_metadata_id`

Для добавления пункта меню обратиться к сервису `interface` к модели
`MenuMetadata` к действию `create`.

> Для встраивания пункта меню в древовидную структуру ниже первого
> уровня указывайте атрибут `parent_menu_entry_metadata_id`, который
> указывает на родительский пункт меню.

## Метаданные интерфейсов

Для получения можно обратиться к сервису `interface` к модели
`InterfaceMetadata` к действию `getItem`.

Для добавления можно обратиться к сервису `interface` к модели
`InterfaceMetadata` к действию `create`.

> Для автоматической генерации интерфейса на клиенте, придусмотрены
> следующие автогенерируемые типы интерфейсов: `table`.

> При указывании типа не поддерживающего автогенерацию на клиенте,
> валидация `data` будет пропускаться. При указывании типа
> поддерживающего автогенерацию на клиенте, валидация `data` будет
> проходить в соответствии с требованиями типа к `data`.

