# Общение между сервисами

## Требования

Сервисы, которые будут общаться должны быть зарегистрированы в сервисе
авторизации.

> Регистрация сервисов пока не обязательна.

Сервис авторизации должен быть доступен.

Сервис авторизации должен иметь `SERVICE_NAME=auth`.

## Реализация

Пример:

```php
// Где-то в сервисе отправителе запроса
$request = new \Egal\Core\Communication\Request(
    'first', // Сервис назначения запроса
    'Model', // К какой модели обращение
    'test', // К какому действию обращение
    [] // Параметры действия
);
$request->call(); // Отправка и ожидание ответа
$response = $request->getResponse();
if ($response->hasError()) {
    $actionErrorMessage = $response->getActionErrorMessage(); // Получение сообщения ошибки
} else {
    $actionResultMessage = $response->getActionResultMessage(); // Получение сообщения результата выполнения действия
}
```

Пример с большим контролем процесса запроса и получения ответа:

```php
// Где-то в сервисе отправителе запроса
$request = new \Egal\Core\Communication\Request(
    'first', // Сервис назначения запроса
    'Model', // К какой модели обращение
    'test', // К какому действию обращение
    [] // Параметры действия
);
if (!$request->isConnectionOpened) {
    $request->openConnection(); // Открытие подключения
}
$request->publish(); // Публикация сообщения запроса
$request->waitReplyMessages(); // Ожидание ответа
$request->closeConnection(); // Закрытие подключения
$response = $request->getResponse();
if ($response->hasError()) {
    $actionErrorMessage = $response->getActionErrorMessage(); // Получение сообщения ошибки
} else {
    $actionResultMessage = $response->getActionResultMessage(); // Получение сообщения результата выполнения действия
}
```

