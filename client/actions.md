## Actions: существующие экшены

Запросы выполняются посредством экшенов: определенных команд для сервера выполнить какое-либо действие.

Дефолтные экшены фреймворка включают в себя:

1. Action getItems: с его помощью можно получить все данные для нужной модели (с возможностью получить только определенные пользователем филды).
    Пример вызова getItems со всеми параметрами можно посмотреть в разделе “Примеры запросов”
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (getItems)
    - actionParams: настраиваемые параметры (фильтр, сортировка, with’ы)
2. Action getItem: с его помощью можно получить конкретную сущность в нужной модели (с возможностью получить только определенные пользователем филды).

    Пример вызова getItem со всеми параметрами можно посмотреть в разделе “Примеры запросов”
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (getItem)
    - actionParams: настраиваемые параметры (сортировка, with’ы, id нужной сущности)
3. Action create: с его помощью создается новая сущность.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (create)
    - actionParams: параметры запроса (вся нужная информация для создания сущности)
4. Action update: с его помощью обновляется выбранная сущность.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (update)
    - actionParams: параметры запроса (вся нужная информация для обновления сущности)
5. Action delete: с его помощью удаляется определенная сущность.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (delete)
    - actionParams: параметры запроса (id удаляемой сущности)
6. Action getMetadata: с его помощью получается метадата нужной модели.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (getMetadata)
7. Action getAllModelsMetadata: с его помощью получается метадата всех существующих моделей
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели (ModelManager)
    - actionName: название экшена (getAllModelsMetadata)
8. Action createMany: с его помощью можно создать несколько новых сущностей.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (createMany)
    - actionParams: параметры запроса (вся нужная информация для создания сущностей)
9. Action updateMany: с его помощью можно обновить сразу несколько сущностей.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (updateMany)
    - actionParams: параметры запроса (вся нужная информация для обновления сущностей)
10. Action deleteMany: с его помощью можно удалить сразу несколько сущностей.
    принимает параметры:
    - microserviceName: название микросервиса
    modelName: название модели
    - actionName: название экшена (deleteMany)
    - actionParams: параметры запроса (id удаляемых сущностей)
11. Action updateManyWithFilter: с его помощью можно обновить сразу несколько сущностей на основе указанных фильтров.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (updateManyRaw)
    - actionParams: параметры запроса (вся нужная информация для обновления сущностей)
12. Action deleteManyWithFilter: с его помощью можно удалить сразу несколько сущностей на основе выбранных фильтров.
    принимает параметры:
    - microserviceName: название микросервиса
    - modelName: название модели
    - actionName: название экшена (deleteManyRaw)
    - actionParams: параметры запроса (id удаляемых сущностей)