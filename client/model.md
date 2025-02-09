## Model: существующие методы

Модель предназначена не только для вызова экшенов, но и для хранения полученной информации и включает в себя различные вспомогательные методы, позволяющие разработчику получать только нужную ему информацию (например, запрашивать только список релейшенов модели, список ее филдов, доступных ей экшенов, только нужные ему филды и т.д.).

Методы модели:
1. actionGetMetadata - описан под пунктом 6 в разделе “Actions: существующие экшены и их методы”

2. actionGetItems- описан под пунктом 1 в разделе “Actions: существующие экшены и их методы”

3. actionGetItem- описан под пунктом 2 в разделе “Actions: существующие экшены и их методы”

4. actionCreate- описан под пунктом 3 в разделе “Actions: существующие экшены и их методы”

5. actionUpdate- описан под пунктом 4 в разделе “Actions: существующие экшены и их методы”

6. actionDelete- описан под пунктом 5 в разделе “Actions: существующие экшены и их методы”

7. actionUpdateManyWithFilter- описан под пунктом 11 в разделе “Actions: существующие экшены и их методы”

8. actionDeleteManyWithFilter- описан под пунктом 12 в разделе “Actions: существующие экшены и их методы”

9. getModelMetadata - позволяет получить сохраненную метадату модели без дополнительного запроса к серверу  

10. getModelActionList - позволяет получить список всех экшенов, доступных модели

11. getModelValidationRules - позволяет получить список правил валидации для каждого филда

12. getModelActionsMetaData - позволяет получить полное описание всех экшенов, доступных модели

13. getModelFieldsWithTypes - позволяет получить fields with types модели

14. getSpecificFields:
    принимает параметры:
    - fields: массив с названиями нужных филдов 
    - filterType: ‘includes’ возвращает только указанные в массиве fields поля, ‘excludes’ возвращает все поля, кроме указанных в fields
    - dataToFilter: принимает массив айтемов для фильтрации
