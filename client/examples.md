## Примеры запросов использования экшенов отдельно от модели:
Для использования любого из экшенов отдельно от модели, нужно инициализировать новый экземпляр класса и указать нужные параметры.

Запрос на получение метадаты всех моделей приложения:
```
let allMetadata = new GetAllMetaDataAction('auth', 'getAllModelsMetadata', 'ModelManager')
allMetadata.socketConnect()
```
Пример использования **getItems**:
 ```
let orders = [['name', 'desc'], ['id', 'asc']]

const filter = [
 {
   field: 'email',
   operator: 'eq',
   value: 'email@email.com'
 },
 {
   field: 'id,
   operator: 'eq',
   value: [1,2,3,4,5]
 },
 {
   left: {
     field: 'age',
     operator: 'gt',
     value: 20
   },
   type: 'or',
   right: {
     field: 'age',
     operator: 'lt',
     value: 30
   }
 },
 {
   left: {
     field: 'name',
     operator: 'gt',
     value: 'name1'
   },
   type: 'and',
   right: {
     field: 'age',
     operator: 'lt',
     value: 20
   }
 }
];
 
let messageTest = new GetItemsAction('auth', 'User', 'getItems')
messageTest.actionParameters.orders(orders)
messageTest.actionParameters.with(['user', 'roles'])
messageTest.actionParameters.filters(filter)
messageTest.socketConnect()
```
Что происходит в примере:

в переменных *orders* и *filter* хранятся параметры, которые нужно передать вместе с запросом. Вид этих параметров фиксированный и всегда должен быть таким, как указано.

Массив *orders* хранит массивы с указанием названия сортируемого поля и порядком сортировки (asc: от меньшего к большему, desc: от большего к меньшему).

Массив *filter* хранит объекты с указанием фильтруемого поля, типа фильтрации и значения для фильтрации.

Все типы фильтрации указаны здесь: (СПИСОК ФИЛЬТРОВ)

Когда все параметры определены, вызывается метод **socketConnect()** для соединения с сервером через сокеты или **axiosConnect()** для соединения посредством *http* запроса.

Возможность запрашивать только определенные поля есть только с использованием модели.

**getItem:**
запрос getItem осуществляется почти так же как getItems, но в дополнение к возможным параметрам нужно обязательно указать id нужной сущности с помощью метода setId()
 ```
messageTest.actionParameters.setId(12)

create:
 
let createParams = { email: 'test@createParams.com', password: '123456' }
let createAction = new CRUDAction('auth', 'User', 'create', createParams)
createAction.socketConnect()

update:
 
let updateParams = {id:'id обновляемой записи', email:'test@createParamsUpdated.com'}
 
let updateAction = new CRUDAction('auth', 'User', 'update', updateParams)
updateAction.socketConnect()

delete:
 
let deleteParams = { id: 'id удаляемой записи'}
let deleteAction = new CRUDAction('auth', 'User', 'delete', deleteParams)
deleteAction.socketConnect()
```


**createMany:**
```
let createParams = [{email:'test1@createParams1.com', password: '123456'}, {email:'test2@createManyParams2.com', password: '123456'}]
let createAction = new CRUDAction('auth', 'User', 'createMany', createParams)
createAction.socketConnect()
```

**updateMany:**
 ```
let updateParams = [{id:'', email:'yulya@createParams1Updated.com'}, {id:'', email:'yulya@createManyParams2Updated.com'}]
let updateAction = new CRUDAction('auth', 'User', 'updateMany', updateParams)
updateAction.socketConnect()
```

**deleteMany:*
 ```
let deleteParams = ['', '', '']
let deleteAction = new CRUDAction('auth', 'User', 'deleteMany', deleteParams)
deleteAction.socketConnect()
```

в параметрах указывается массив *id* удаляемых записей


**updateManyRaw:**
 ```
let actionFilters = [{
    field: 'name',
    type: 'eq',
    value: 'noname1'
},
    {
        field: 'name',
        type: 'eq',
        value: 'noname2'
    }]
 
let paramsUpdateRaw = { attributes: { name: 'yesname' }, filter: [...actionFilters] }
let updateRawAction = new CRUDAction('auth', 'User', 'updateManyRaw', paramsUpdateRaw)
updateRawAction.socketConnect()
```

**deleteManyRaw:**
 ```
let paramsDeleteRaw = { filter: [...actionFilters] }
let deleteRawAction = new CRUDAction('auth', 'User', 'deleteManyRaw', paramsDeleteRaw)
deleteRawAction.socketConnect()
```


## Пример инициализации модели и ее методов:

Для того чтобы инициализировать нужную модель нужно сделать следующее:
 ```
let newModel = new Model('User')
```


Далее нужно указать URL для связи с сервером с помощью метода модели **setBaseUrl**. Т.к в зависимости от типа связи (сокеты или *axios*) URL будет отличаться нужно также указывать тип связи:
```
/**
* инициализация базового URL для сокетов
* нельзя определить один url для обоих видов связи, они слишком отличаются и указывать их надо полностью обязательно

newModel.setBaseUrl('amqp://domain:5672/', 'socket');
/**
* инициализация базового URL для axios
*/
newModel.setBaseUrl('http://domain:8081', 'axios');
```

После того, как модель инициализирована можно использовать ее методы.


Экшены, вызываемые через модель, используются так (нужно обратить внимание, что передача параметров отличается от отдельного использования экшенов):


**getItems:**
 ```
let orders = [['name', 'desc'], ['id', 'asc']]
const filter = [
  {
    field: 'email',
    operator: 'eq',
    value: 'email@email.com'
  },
  {
    left: {
      field: 'age',
      operator: 'gt',
      value: 20
    },
    type: 'or',
    right: {
      field: 'age',
      operator: 'lt',
      value: 30
    }
  },
  {
    left: {
      field: 'name',
      operator: 'gt',
      value: 'name1'
    },
    type: 'and',
    right: {
      field: 'age',
      operator: 'lt',
      value: 20
    }
  }
];
 
 
let page = 2
let perPage = 25
let withs = ['user', 'roles']
 
let messageTest = newModel.actionGetItems('auth', 'getItems', 'socket', with, filter, orders, page, perPage)
```

Что происходит в примере:

в переменных *orders* и *filter* хранятся параметры, которые нужно передать вместе с запросом. Вид этих параметров фиксированный и всегда должен быть таким, как указано.

Массив *orders* хранит массивы с указанием названия сортируемого поля и порядком сортировки (asc: от меньшего к большему, desc: от большего к меньшему).

Массив *filter* хранит объекты с указанием фильтруемого поля, типа фильтрации и значения для фильтрации.

Все типы фильтрации указаны здесь: (ССЫЛКА НА СПИСОК ФИЛЬТРОВ).

Вместо вызова метода **socketConnect()** или **axiosConnect()**, в параметрах указывается тип подключения: ‘socket’ или ‘axios’.

После запроса, данные сохраняются в модели и их можно вызвать методом **newModel.getItems()**.

**getItem:**

запрос getItem осуществляется почти так же как getItems, но в дополнение к возможным параметрам нужно обязательно указать id перед типом соединения.
 ```
messageTest.actionParameters.setId(12)
```

**create:**
``` 
let createParams = {email:'test1@createParams1.com', password: '123456'}
let createAction = newModel.actionCreate('auth', 'create', 'socket', createParams)
```

**update:**
 ```
let updateParams = {id:'', email:'yulya@createParams1Updated.com'}
let updateAction = newModel.actionUpdate('auth', 'update', 'socket', updateParams)
```

**delete:**
 ```
let deleteParams = ['']
let actionDelete = newModel.actionDelete('auth', 'delete', 'socket', deleteParams)
```

**createMany:**
 ```
let createParams = [{email:'yulya@createParams1.com', password: '123456'}, {email:'yulya@createManyParams2.com', password: '123456'}]
let createManyAction = newModel.actionCreate('auth', 'createMany', 'socket', createParams)
```

**updateMany:**
 ```
let updateParams = [{id:'', email:'yulya@createParams1Updated.com'}, {id:'', email:'yulya@createManyParams2Updated.com'}]
 
let updateManyAction = newModel.actionUpdate('auth', 'updateMany', 'socket', updateParams)
```

**deleteMany:**
 ```
let deleteManyParams = ['', '', '']
let deleteManyAction = newModel.actionDelete('auth', 'deleteMany', 'socket', deleteManyParams)
```
в параметрах указывается массив *id* удаляемых записей.


**updateManyRaw:**
 ```
let actionFilters = [{
    field: 'name',
    type: 'eq',
    value: 'noname1'
},
    {
        field: 'name',
        type: 'eq',
        value: 'noname2'
    }]
let paramsDeleteRaw = { filter: [...actionFilters] }
let updateRaw = newModel.actionUpdateManyWithFilter('auth', 'updateManyRaw', 'socket', paramsUpdateRaw)
 ```

**deleteManyRaw:**
 ```
let paramsDeleteManyRaw = {filter:[...actionFilters]}
let deleteRaw = newModel.actionDeleteManyWithFilter('auth', 'deleteManyRaw', 'socket', paramsDeleteManyRaw)
```

Запрос на получение метадаты модели
 ```
let metadata = newModel.actionGetMetadata('auth', 'getMetadata', 'socket')
```
