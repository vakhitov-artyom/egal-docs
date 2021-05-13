## Подключение авторизации:

1. Импортируются нужные классы:
```
import {AuthAction} from "@egal/model/compile/build/index.js";
import {EventObserver} from "@egal/model/compile/build/index"
```
- инициализируется новый объект, принимающий в себя параметры: 
    - имя модели
    - способ подключения (сейчас доступен только аксиос, позже можно будет использовать сокеты)
    - и устанавливается нужный url
```
const auth = new AuthAction('User', 'axios')
auth.setBaseURL(host.authUrl)
```

2. Метод авторизации выглядит следующим образом:
```
auth() {
   let userAuth = {email: this.email, password: this.password}
   auth.authUser(userAuth)
}
```
3. Метод регистрации следующим:
```
createAccount() {
   let userReg = {email: this.email, password: this.password}
   auth.registerNewUser(userReg)
}
```
4. Observer:
```
let observer = new EventObserver()
observer.subscribe('User', (data, actionName) => {
if (data !== 'Start Processing' && actionName !== 'error') {
switch (actionName) {
case 'registerByEmailAndPassword':
let userData = {email: this.email, password: this.password}
auth.authUser(userData);
break;
case 'loginByEmailAndPassword':
let userCred = {service_name: 'monolit', token: data[0]}
auth.loginToService(userCred)
break;
case 'loginToService':
GlobalVariables.tokenUST = data.toString();
}
} else if (actionName === 'error') {
this.errorAlert(data)
}})
```
5. Далее, уже в проекте нужно настроить axios interceptors таким образом, чтобы к каждому запросу добавлялся хэддер (UST токен).  Если  с сервера приходит ошибка о том, что юзер не авторизирован, нужно запросить новый UST токен, используя текущий UMT токен. Если UMT токен не актуален, нужно разлогинить пользователя.
