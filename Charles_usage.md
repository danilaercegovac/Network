# Практическая работа по использованию Charles
## Эрцеговац Данила P4150
### Пошаговое описание
- Установка Charles. Установил [v 5.1 Windows msi файл](https://www.charlesproxy.com/download/)
- Зашёл в Charles. Включил Windows proxy, Recording, SSL Proxy, Breakpoints
- Добавил брейкпоинт. Proxy -> Breakpoints settings -> +
- В первый раз добавил брейкопинт с настройками для https://jsonplaceholder.typicode.com/posts/1
```
Method - GET
Protocol - https
Host - jsonplaceholder.typicode.com
Port - (ничего)
Path - /posts/1
Query - *
Помечены галками запрос/ответ
```
Но ловился только запрос формата
```
URL https://jsonplaceholder.typicode.com
Method CONNECT
Status Sending request body…
Notes SSL Proxying not enabled for this host: enable in Proxy Settings, SSL locations
```
Это значит, что не установлен сертификат SSL -> Нельзя посмотреть метод, путь, ответ
- Установил сертификат SSL через Help -> SSL Proxying -> Install Charles Certificate. Вроде установил
- Добавил в Proxy -> SSL Proxying Settings нужную локацию
```
Host: jsonplaceholder.typicode.com
Port: 443
```
- Вновь отправил https://jsonplaceholder.typicode.com/posts/1 запрос - не вышло.
- Долго копался с сертификатом, в итоге решил для первого успешного выполнения сделать с http
- Добавил брейкпоинт
```
Protocol: http
Port: 80
Path: /posts/2
```
- Отправил http://jsonplaceholder.typicode.com/posts/2
- Чарлис остановил запрос. Я его заэкзекутил
- Чарлис остановил ответ. Я его поменял
<img width="769" height="502" alt="image" src="https://github.com/user-attachments/assets/8cc22f9f-6c72-4a27-98cc-c346124b1649" />
