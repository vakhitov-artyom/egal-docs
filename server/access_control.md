# Разграничения доступа

## Разграничение доступа до действий

### Статусы аутентификации

При наличии UST во время обращения к действию пользователь имеет статус
аутентификации - `logged`, при отсутствии - `guest`. Для указания
доступа к действию в зависимости от статуса аутентификации используйте
следующий код в блоке описания действия:

```php
/**
 * @statuses-access guest
 */
public static function actionExample()
{
    // ...
}
```

Либо в блоке описания модели:

```php
/**
 * @action example {@statuses-access guest}
 */
class Example extends EgalModel
{

    // ...

}
```


### Роли

Для указания доступа к действию в зависимости от ролей используйте
следующий код в блоке описания действия:

```php
/**
 * @roles-access example_role
 */
public static function actionExample()
{
    // ...
}
```

Либо в блоке описания модели:

```php
/**
 * @action example {@roles-access example_role}
 */
class Example extends EgalModel
{

    // ...

}
```

> При указании ролевого доступа до действия, автоматически проверяется
> статус авторизации пользователя, он должен быть `logged` (см.
> [Статусы аутентификации](#Статусы-аутентификации)).

### Permission'ы

Для указания доступа к действию в зависимости от разрешений используйте
следующий код в блоке описания действия:

```php
/**
 * @permissions-access example_permissions
 */
public static function actionExample()
{
    // ...
}
```

Либо в блоке описания модели:

```php
/**
 * @action example {@permissions-access example_permissions}
 */
class Example extends EgalModel
{

    // ...

}
```

> При указании permissions до действия, автоматически проверяется статус
> авторизации пользователя, он должен быть `logged` (см.
> [Статусы аутентификации](#Статусы-аутентификации)).

### Динамическое изменение информации о текущем пользователе сессии

При появлении в сессии UST - публикуется `UserServiceTokenDetectedEvent`
событие. В реакции на который можно изменить информацию о пользователе.

Пример добавления роли пользователю:
1. Создадим обработчика события `AddingRoleToUserServiceToken`.

```bash
php artisan egal:make:listener AddingRoleToUserServiceToken
```

2. Зарегистрируем обработчика события в `EventServiceProvider`

```php
namespace App\Providers;

use Egal\Core\Events\UserServiceTokenDetectedEvent;
use App\Listeners\AddingRoleToUserServiceToken;

class EventServiceProvider extends ServiceProvider
{

    // ...

    protected $listen = [
        UserServiceTokenDetectedEvent::class => [
            AddingRoleToUserServiceToken::class
        ]
    ];

    // ...
    
}
```

> Если нет класса `EventServiceProvider` его можно создать командой
>
> ```bash
> php artisan egal:make:event-service-provider
> ```

3. Реализуем в обработчике добавление роли пользователю

```php
namespace App\Listeners;

use App\Models\Speaker;
use Egal\Core\Listeners\EventListener;
use Egal\Core\Session\Session;

class AdditionUserServiceTokenListener extends EventListener
{

    public function handle(): void
    {
        Session::getUserServiceToken()->addRole('example_role');
    }

}
```

