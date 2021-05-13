# Валидация

## Список правил валидации

### upper_case

Атрибут введен в верхнем регистре.

### lower_case

Атрибут введен в нижнем регистре.

### Правила валидации Laravel

[Cм. документацию](https://laravel.com/docs/master/validation).


## Пользовательские правила валидации

> В проекте должна использоваться библиотека `egal/validation`.

1. Сгенерируйте файл правила с помощью команды:

```bash
php artisan egal:make:rule ExampleRule
```

2. Реализуйте метод `validate`

```php
namespace App\Rules;

use Egal\Validation\Rules\Rule as EgalRule;

class ExampleRule extends EgalRule
{

    // ...

    public function validate($attribute, $value, $parameters = null): bool
    {
        // Твоя реализация метода
    }

    // ...

}
```

Для использования сформированного правила валидации используйте
[вызов через класс](https://laravel.com/docs/master/validation#using-rule-objects)
либо как строку.

При определении правила через строку берется атрибут `rule` у класса,
либо если он не указан берется название класса без постфикса `Rule` в
snake регистре (Класс - `App\Rules\UpperCaseRule`, строка определения
правила - `upper_case`).

Можно определить сообщение выводимое при провале валидации, для этого
переопределите метод `message`:

```php
namespace App\Rules;

use Egal\Validation\Rules\Rule as EgalRule;

class ExampleRule extends EgalRule
{

    // ...

    public function message()
    {
        // Твоя реализация метода
    }

    // ...

}
```

