# События и реакции

## Публикация событий

Публикация локальных событий происходит средствами Lumen.

> Для генерации файлов событий используйте [генераторы](#Генерация-файла-события).

Для публикации стандартных событий модели как глобальные используйте `$dispatchesEvents` и `ModelGlobalEvents`:
```php
namespace App\Models;

use Egal\Model\Events\SavedModelGlobalEvent;
use Egal\Model\Events\SavingModelGlobalEvent;

class ExampleModel extends EgalModel {

    // ...

    protected $dispatchesEvents = [
        'saved' => SavedModelGlobalEvent::class,
        'saving' => SavingModelGlobalEvent::class
    ];

    // ...

}
```

## Определение обработчиков событий

Для определения обработчиков нам потребуется файл `app/Providers/EventServiceProvider`. Если данного файла не обнаружено - [сгенерируйте его](#Генерация-файла-eventserviceprovider).

В следующем блоке определяются локальные события и их обработчики:
```php
namespace App\Providers;

use Egal\Core\Events\EventServiceProvider as ServiceProvider;
use App\Listeners\ExampleListener;
use App\Event\ExampleEventr;

class EventServiceProvider extends ServiceProvider
{

    // ...

    protected $listen = [
        ExampleEvent::class => [
            ExampleListener::class
        ]
    ];

    // ...

}
```

В следующем блоке определяются паттерны глобальных событий и их обработчики:
```php
namespace App\Providers;

use Egal\Core\Events\EventServiceProvider as ServiceProvider;
use App\Listeners\ExampleGlobalListener;

class EventServiceProvider extends ServiceProvider
{

    // ...

    public array $globalListen = [
        'service' => [
            'Model' => [
                'event-message' => [
                    ExampleGlobalListener::class
                ]
            ]
        ]
    ];

    // ...

}
```

## Обработка событий

При возникновении события вызывается метод `handle()` у обработчика события. Всю логику обработки события нужно разместить в данном методе.
```php
namespace App\Listeners;

use Egal\Core\Listeners\GlobalEventListener;
use Illuminate\Support\Facades\Log;

class ExampleListener extends GlobalEventListener
{

    public function handle(array $data): void
    {
        // Твоя реализация метода
    }

}
```

> Для генерации файлов обработчиков событий используйте [генераторы](#Генерация-файла-обработчика-события).


## Генераторы

### Генерация файла EventServiceProvider

Консольная команда:
```bash
php artisan egal:make:event-service-provider
```

### Генерация файла события

Консольная команда:
```bash
php artisan egal:make:event Example
```
Или эта команда, для генерации файла глобольного события:
```bash
php artisan egal:make:event Example --global
```
После выполнения данной команды в директории `app/Events` сгенерируется файл события.

> Глобальные события публикуются на всё приложение (не только в рамках сервиса).
> А также обработка глобального события может происходить не в текущем клоне сервиса.

### Генерация файла обработчика события

Консольная команда:
```bash
php artisan egal:make:listener Example
```
Или эта команда, для генерации файла обработчика глобольного события:
```bash
php artisan egal:make:listener Example --global
```
После выполнения данной команды в директории `app/Events` сгенерируется файл события.
