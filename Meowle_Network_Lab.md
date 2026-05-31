# Лабораторная работа: Meowle Network Lab
## Эрцеговац Данила Желькович, P4150
## Описание работы
### Установка приложения / Настройка прокси
- Установил приложение на Android Redmi 8
- Подключил телефон и ноутбук к одному вай фаю
- Узнал IPv4 адресс ноутбка
- Подключил прокси вручную в телефоне  ссуказание в хосте IP ноута и в порте 8888
- На ноуте пришёл запрос на разрешение подключения - разрешил
- В Чарлис вижу телефон и конкретно папку api
### Часть 1. Разведка API через Swagger
#### Таблица API-методов
| Method | Path                              | Назначение                                                            | Параметры                                                                                                             | Пример запроса                                                                                                   | Пример ответа                                                                                                                                       |
| ------ | --------------------------------- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| POST   | `/cats/add`                       | Добавление списка имён котиков через web API                          | Body JSON: `cats[]`, где каждый объект содержит `name`, `gender`, `description`. `gender`: `male`, `female`, `unisex` | `POST /cats/add`<br><br>`{"cats":[{"name":"Барсик","gender":"male","description":"Рыжий кот"}]}`                 | `{"cats":[{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}]}`                                    |
| GET    | `/cats/get-by-id`                 | Получение карточки котика по `id` через web API                       | Query: `id` — числовой идентификатор котика                                                                           | `GET /cats/get-by-id?id=1`                                                                                       | `{"cat":{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}}`                                       |
| POST   | `/cats/search`                    | Поиск котиков по имени и дополнительным характеристикам через web API | Body JSON: `name` — имя котика, `gender` — пол: `male`, `female`, `unisex`                                            | `POST /cats/search`<br><br>`{"name":"Бар","gender":"male"}`                                                      | `{"groups":[{"title":"Б","count":1,"cats":[{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}]}]}` |
| GET    | `/cats/search-pattern`            | Поиск котиков по началу имени                                         | Query: `name` — начало имени; `limit` — ограничение количества результатов                                            | `GET /cats/search-pattern?name=Бар&limit=10`                                                                     | `{"moreResults":false,"cats":[{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}]}`                |
| POST   | `/cats/save-description`          | Сохранение или изменение описания котика через web API                | Body JSON: `catId` — id котика; `catDescription` — новое описание                                                     | `POST /cats/save-description`<br><br>`{"catId":1,"catDescription":"Новое описание"}`                             | `{"id":1,"name":"Барсик","description":"Новое описание","tags":[],"gender":"male","likes":0,"dislikes":0}`                                          |
| GET    | `/cats/validation`                | Получение правил валидации                                            | Параметры отсутствуют                                                                                                 | `GET /cats/validation`                                                                                           | `[{"id":1,"description":"Правило валидации","regex":"^[A-Za-zА-Яа-я]+$"}]`                                                                          |
| GET    | `/cats/all`                       | Получение списка всех котиков                                         | Query: `order` — обязательный параметр сортировки: `asc` или `desc`; `gender` — необязательный фильтр по полу         | `GET /cats/all?order=asc&gender=male`                                                                            | `{"groups":[{"title":"Б","count":1,"cats":[{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}]}]}` |
| POST   | `/cats/{catId}/upload`            | Загрузка изображения котика через web API                             | Path: `catId` — id котика. Body: `multipart/form-data`, поле `file`                                                   | `POST /cats/1/upload`<br><br>`form-data: file=@cat.jpg`                                                          | `{"fileUrl":"/uploads/cat.jpg"}`                                                                                                                    |
| GET    | `/cats/{catId}/photos`            | Получение списка фотографий котика через web API                      | Path: `catId` — id котика                                                                                             | `GET /cats/1/photos`                                                                                             | `{"images":["/uploads/cat.jpg"]}`                                                                                                                   |
| GET    | `/version`                        | Получение версии проекта                                              | Параметры отсутствуют                                                                                                 | `GET /version`                                                                                                   | `{"build":1}`                                                                                                                                       |
| POST   | `/cats/{catId}/like`              | Добавление лайка котику через web API                                 | Path: `catId` — id котика                                                                                             | `POST /cats/1/like`                                                                                              | `OK`                                                                                                                                                |
| DELETE | `/cats/{catId}/like`              | Удаление лайка у котика через web API                                 | Path: `catId` — id котика                                                                                             | `DELETE /cats/1/like`                                                                                            | `OK`                                                                                                                                                |
| DELETE | `/cats/{catId}/remove`            | Удаление котика через web API                                         | Path: `catId` — id котика                                                                                             | `DELETE /cats/1/remove`                                                                                          | `OK`                                                                                                                                                |
| POST   | `/cats/{catId}/dislike`           | Добавление дизлайка котику через web API                              | Path: `catId` — id котика                                                                                             | `POST /cats/1/dislike`                                                                                           | `OK`                                                                                                                                                |
| DELETE | `/cats/{catId}/dislike`           | Удаление дизлайка у котика через web API                              | Path: `catId` — id котика                                                                                             | `DELETE /cats/1/dislike`                                                                                         | `OK`                                                                                                                                                |
| GET    | `/cats/likes-rating`              | Получение ТОП-10 котиков по лайкам через web API                      | Параметры отсутствуют                                                                                                 | `GET /cats/likes-rating`                                                                                         | `[{"name":"Барсик","likes":10}]`                                                                                                                    |
| GET    | `/cats/dislikes-rating`           | Получение ТОП-10 котиков по дизлайкам через web API                   | Параметры отсутствуют                                                                                                 | `GET /cats/dislikes-rating`                                                                                      | `[{"name":"Барсик","dislikes":3}]`                                                                                                                  |
| POST   | `/api/core/cats/search`           | Мобильный поиск котиков по имени, сортировке и полу                   | Body JSON: `name`, `order`, `gender`. `order`: `asc` или `desc`; `gender`: `male`, `female`, `unisex`                 | `POST /api/core/cats/search`<br><br>`{"name":"Бар","order":"asc","gender":"male"}`                               | `{"groups":[{"title":"Б","count":1,"cats":[{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}]}]}` |
| POST   | `/api/core/cats/add`              | Мобильное добавление одного или нескольких котиков                    | Body JSON: `cats[]`, где каждый объект содержит `name`, `gender`, `description`                                       | `POST /api/core/cats/add`<br><br>`{"cats":[{"name":"Мурка","gender":"female","description":"Серая кошка"}]}`     | `{"cats":[{"id":2,"name":"Мурка","description":"Серая кошка","tags":[],"gender":"female","likes":0,"dislikes":0}]}`                                 |
| GET    | `/api/core/cats/get-by-id`        | Мобильное получение карточки котика по `id`                           | Query: `id` — числовой идентификатор котика                                                                           | `GET /api/core/cats/get-by-id?id=1`                                                                              | `{"cat":{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":0,"dislikes":0}}`                                       |
| POST   | `/api/core/cats/save-description` | Мобильное сохранение описания котика                                  | Body JSON: `catId` — id котика; `catDescription` — новое описание                                                     | `POST /api/core/cats/save-description`<br><br>`{"catId":1,"catDescription":"Описание из мобильного приложения"}` | `{"id":1,"name":"Барсик","description":"Описание из мобильного приложения","tags":[],"gender":"male","likes":0,"dislikes":0}`                       |
| GET    | `/api/likes/cats/rating`          | Мобильный рейтинг по лайкам и дизлайкам                               | Параметры отсутствуют                                                                                                 | `GET /api/likes/cats/rating`                                                                                     | `{"likes":[{"name":"Барсик","likes":10}],"dislikes":[{"name":"Мурка","dislikes":4}]}`                                                               |
| POST   | `/api/likes/cats/{catId}/likes`   | Мобильное голосование за котика: лайк или дизлайк                     | Path: `catId` — id котика. Body JSON: `like` — boolean; `dislike` — boolean                                           | `POST /api/likes/cats/1/likes`<br><br>`{"like":true,"dislike":false}`                                            | `{"id":1,"name":"Барсик","description":"Рыжий кот","tags":[],"gender":"male","likes":1,"dislikes":0}`                                               |
| GET    | `/api/photos/cats/{catId}/photos` | Мобильное получение списка фотографий котика                          | Path: `catId` — id котика                                                                                             | `GET /api/photos/cats/1/photos`                                                                                  | `{"images":["/uploads/cat.jpg"]}`                                                                                                                   |
| POST   | `/api/photos/cats/{catId}/upload` | Мобильная загрузка фотографии котика                                  | Path: `catId` — id котика. Body: `multipart/form-data`, поле `file`                                                   | `POST /api/photos/cats/1/upload`<br><br>`form-data: file=@cat.jpg`                                               | `{"fileUrl":"/uploads/cat.jpg"}`                                                                                                                    |
#### Группировка методов по цели

| Требуемый сценарий             | Найденные методы                                                                                                                                    |
| ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Поиск котиков                  | `POST /cats/search`, `GET /cats/search-pattern`, `GET /cats/all`, `POST /api/core/cats/search`                                                      |
| Просмотр карточки по `id`      | `GET /cats/get-by-id`, `GET /api/core/cats/get-by-id`                                                                                               |
| Добавление имени               | `POST /cats/add`, `POST /api/core/cats/add`                                                                                                         |
| Изменение описания             | `POST /cats/save-description`, `POST /api/core/cats/save-description`                                                                               |
| Лайк                           | `POST /cats/{catId}/like`, `DELETE /cats/{catId}/like`, `POST /api/likes/cats/{catId}/likes`                                                        |
| Дизлайк                        | `POST /cats/{catId}/dislike`, `DELETE /cats/{catId}/dislike`, `POST /api/likes/cats/{catId}/likes`                                                  |
| Рейтинг                        | `GET /cats/likes-rating`, `GET /cats/dislikes-rating`, `GET /api/likes/cats/rating`                                                                 |
| Просмотр фото                  | `GET /cats/{catId}/photos`, `GET /api/photos/cats/{catId}/photos`                                                                                   |
| Загрузка фото                  | `POST /cats/{catId}/upload`, `POST /api/photos/cats/{catId}/upload`                                                                                 |
| Удаление данных через web API  | `DELETE /cats/{catId}/remove`                                                                                                                       |
| Изменение данных через web API | `POST /cats/save-description`, `POST /cats/{catId}/like`, `DELETE /cats/{catId}/like`, `POST /cats/{catId}/dislike`, `DELETE /cats/{catId}/dislike` |

### Часть 2. Карта действий приложения
| Действие в приложении                            | HTTP method | URL                                                       | Request body                                                                             | Response code | Response body                                                                                                                                                                                                                                                                                        | Комментарий |
| ------------------------------------------------ | ----------- | --------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Поиск котика                                     | POST        | `http://144.31.255.0:3001/api/core/cats/search`           | `{"gender":null,"name":"барсик","order":"asc"}`                                          | `200`         | `{"groups":[]}`                                                                                                                                                                                                                                                                                      | |
| Открытие карточки котика                         | GET         | `http://144.31.255.0:3001/api/core/cats/get-by-id?id=21`  | —                                                                                        | `200`         | `{"cat":{"id":21,"name":"Турбопетров","description":"тихий, без суеты, черемша\n","tags":"","gender":"unisex","likes":42,"dislikes":40}}`                                                                                                                                                            |  |
| Получение фото при открытии карточки             | GET         | `http://144.31.255.0:3001/api/photos/cats/21/photos`      | —                                                                                        | `200`         | `{"images":["/photos/image-1780064483119.jpg"]}`                                                                                                                                                                                                                                                     |  |
| Загрузка изображения карточки                    | GET         | `http://144.31.255.0:3001/api/photos/cats/42/upload` | —                                                                                        | `200`         | `{"fileUrl": "/photos/image-1780235536129.jpg"}`                                                                                                                                                                                                                                                                     |  |
| Лайк                                             | POST        | `http://144.31.255.0:3001/api/likes/cats/5/likes`         | `{"dislike":false,"like":true}`                                                          | `200`         | `{"id":5,"name":"Буся","description":"Запись для ручного редактирования\"\"\"","tags":"","gender":"female","likes":3,"dislikes":2}`                                                                                                                                                                  |  |
| Дизлайк                                          | POST        | `http://144.31.255.0:3001/api/likes/cats/5/likes`         | `{"dislike":true,"like":false}`                                                          | `200`         | `{"id":5,"name":"Буся","description":"Запись для ручного редактирования\"\"\"","tags":"","gender":"female","likes":3,"dislikes":3}`                                                                                                                                                                  |   |
| Добавление нового котика                         | POST        | `http://144.31.255.0:3001/api/core/cats/add`              | `{"cats":[{"description":"великий кот новгородский","gender":"male","name":"великий"}]}` | `200`         | `{"cats":[{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}]}`                                                                                                                                                                    |   |
| Повторное добавление похожего/дублирующего имени | POST        | `http://144.31.255.0:3001/api/core/cats/add`              | `{"cats":[{"description":"великий кот новгородский","gender":"male","name":"великий"}]}` | `200`         | `{"cats":[{"errorDescription":"Такое значение уже существует"}]}`                                                                                                                                                                                                                                    | В сессии был повтор добавления. Сервер вернул бизнес-ошибку внутри body, но HTTP-код остался `200` |
| Изменение описания                               | POST        | `http://144.31.255.0:3001/api/core/cats/save-description` | `{"catDescription":"ууаа","catId":37}`                                                   | `200`         | `{"id":37,"name":"Бодя","description":"ууаа","tags":"","gender":"unisex","likes":1,"dislikes":0}`                                                                                                                                                                                                    |       |
| Открытие рейтинга                                | GET         | `http://144.31.255.0:3001/api/likes/cats/rating`          | —                                                                                        | `200`         | `{"likes":[{"id":21,"name":"Турбопетров","description":"тихий, без суеты, черемша\n","tags":"","gender":"unisex","likes":42,"dislikes":40},{"id":30,"name":"Леопольд","description":"Леопольд вернулся. Больше не удаляйте","tags":"","gender":"male","likes":25,"dislikes":0},{"id":18,"name":"Обормотик","description":"Разлохмаченный животик, мокрые лапки","tags":"","gender":"unisex","likes":15,"dislikes":13},{"id":20,"name":"Вадосик","description":"67","tags":"","gender":"male","likes":11,"dislikes":11},{"id":29,"name":"Новыйбогдан","description":"уу","tags":"","gender":"unisex","likes":8,"dislikes":6},{"id":5,"name":"Буся","description":"Запись для ручного редактирования\"\"\"","tags":"","gender":"female","likes":4,"dislikes":4},{"id":15,"name":"Лабакот","description":"Мокрые лапки, разлохмаченный животик, тыгыдямчатый, печатает на клавиатуре всякое..","tags":"","gender":"unisex","likes":4,"dislikes":0},{"id":37,"name":"Бодя","description":"ууаа","tags":"","gender":"unisex","likes":2,"dislikes":0},{"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0},{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}],"dislikes":[{"id":21,"name":"Турбопетров","description":"тихий, без суеты, черемша\n","tags":"","gender":"unisex","likes":42,"dislikes":40},{"id":18,"name":"Обормотик","description":"Разлохмаченный животик, мокрые лапки","tags":"","gender":"unisex","likes":15,"dislikes":13},{"id":20,"name":"Вадосик","description":"67","tags":"","gender":"male","likes":11,"dislikes":11},{"id":29,"name":"Новыйбогдан","description":"уу","tags":"","gender":"unisex","likes":8,"dislikes":6},{"id":5,"name":"Буся","description":"Запись для ручного редактирования\"\"\"","tags":"","gender":"female","likes":4,"dislikes":4},{"id":37,"name":"Бодя","description":"ууаа","tags":"","gender":"unisex","likes":2,"dislikes":0},{"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0},{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0},{"id":36,"name":"Длинное Описание","description":"ааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааааа","tags":"","gender":"unisex","likes":0,"dislikes":0},{"id":4,"name":"Кекс","description":"Имя для экспериментов с поиском","tags":"","gender":"unisex","likes":0,"dislikes":0}]}` |          |                                    
### Часть 3. Изменение запросов
#### Эксперимент 1. Изменение `catId` при получении карточки котика

**Цель:** проверить возможность прямого доступа к объектам по идентификатору.

**Исходный запрос:**

```http
GET /api/core/cats/get-by-id?id=21 HTTP/1.1
Host: 144.31.255.0:3001
```

**Изменённый запрос:**

```http
GET /api/core/cats/get-by-id?id=5 HTTP/1.1
Host: 144.31.255.0:3001
```

**Ответ сервера:**

```json
{
	"cat": {
		"id": 5,
		"name": "Буся",
		"description": "Запись для ручного редактирования\"\"\"",
		"tags": "",
		"gender": "female",
		"likes": 4,
		"dislikes": 4
	}
}
```

**Изменилось ли состояние данных:**
Нет, запрос только читает данные.

**Результат:**
Сервер вернул карточку другого котика по изменённому `id`.

**Является ли поведением багом или security-проблемой:**
Если подразумевается, что все коты всем доступны, то проблем нет. Но если требуется логика на ограничение просмотра тех или иных котов, то здесь проблема безопасности.

**Как исправить:**
Если второе условие, то следует реализовать логику авторизации. То есть проверять после запроса уровень доступа пользователя, например.

---

### Эксперимент 2. Повторение одного и того же запроса лайка несколько раз

**Цель:** проверить, можно ли накрутить лайки повторной отправкой одного и того же запроса.

**Исходный запрос:**

```http
POST /api/likes/cats/5/likes HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"dislike":false,"like":true}
```

**Изменённый запрос:**
Запрос был повторён несколько раз без изменений.

**Ответ сервера:**

```json
{
	"id": 18,
	"name": "Обормотик",
	"description": "Разлохмаченный животик, мокрые лапки",
	"tags": "",
	"gender": "unisex",
	"likes": 16,
	"dislikes": 13
}
```
После двух повторов
```json
{
	"id": 18,
	"name": "Обормотик",
	"description": "Разлохмаченный животик, мокрые лапки",
	"tags": "",
	"gender": "unisex",
	"likes": 18,
	"dislikes": 13
}
```

**Изменилось ли состояние данных:**
Повторы увеличили количество лайков

**Результат:**
Повторы увеличили количество лайков

**Является ли поведением багом или security-проблемой:**
Если требуется как вконтакте или инстаграме, то сейчас баг. Если допустимо несколько лайков от одного человека, как например реакции в контуре или гугл мите, то окей

**Как исправить:**
Если требуется как вконтакте или инстаграме, то требуется привязывать лайки/дизлайки к пользователям и реализовать логику, что если идёт повтор, то кол-во лайков не увеличивается.

---

### Эксперимент 3. Изменение описания записи по чужому или произвольному `catId`

**Цель:** проверить, можно ли изменить описание котика, которого пользователь не создавал.

**Исходный запрос:**

```http
POST /api/core/cats/save-description HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"catDescription": "Разлохмаченный животик, мокрые лапки.","catId": 18}
```

**Изменённый запрос:**

```http
POST /api/core/cats/save-description HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"catDescription": "Изменил","catId": 18}
```

**Ответ сервера:**

```json
{"id":18,"name":"Обормотик","description":"Изменил","tags":"","gender":"unisex","likes":18,"dislikes":13}
```

**Изменилось ли состояние данных:**
Изменилось

**Результат:**
Изменилось

**Является ли поведением багом или security-проблемой:**
Так как всем был дан один аккаунт, то логично что все коты принадлежат одному аккаунту => этот акк может

**Как исправить:**
На сервере необходимо проверять, имеет ли текущий пользователь право изменять конкретную запись. Если пользователь не является владельцем записи или не имеет роли администратора, сервер должен возвращать `403 Forbidden`.

---

### Эксперимент 4. Попытка добавить дубликат имени

**Цель:** проверить обработку дубликатов и корректность HTTP-статуса при бизнес-ошибке.

**Исходный запрос:**

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"великий кот новгородский","gender":"male","name":"великий"}]}
```

**Изменённый запрос:**
Тот же запрос был отправлен повторно.

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"великий кот новгородский","gender":"male","name":"великий"}]}
```

**Ответ сервера:**

```json
{"cats":[{"errorDescription":"Такое значение уже существует"}]}
```

**Изменилось ли состояние данных:**
Нет, новая запись с дублирующим именем не была создана.

**Результат:**
Сервер обнаружил дубликат, но вернул HTTP-код `200 OK`, а описание ошибки передал внутри тела ответа.

**Является ли поведением багом или security-проблемой:**
Это скорее баг API-дизайна и обработки ошибок, чем security-проблема. Клиенту сложнее отличать успешные операции от ошибок, так как HTTP-статус остаётся `200`.

**Как исправить:**
При попытке создать дубликат сервер должен возвращать более подходящий HTTP-статус, например `409 Conflict`, и структурированное тело ошибки:

```json
{
  "error": "DUPLICATE_CAT_NAME",
  "message": "Такое значение уже существует"
}
```

---

### Эксперимент 5. Отправка невалидного значения `gender`

**Цель:** проверить серверную валидацию enum-поля `gender`.

**Исходный запрос:**

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"Описание для проверки","gender":"male","name":"Тестовый"}]}
```

**Изменённый запрос:**

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"Описание для проверки","gender":"admin","name":"Тестовый"}]}
```

**Ответ сервера:**

```json
TODO: вставить фактический response body
```

**Изменилось ли состояние данных:**
TODO: указать, была ли создана запись с невалидным `gender`.

**Результат:**
TODO: например, сервер отклонил запрос / сервер создал запись с некорректным значением.

**Является ли поведением багом или security-проблемой:**
Если сервер принимает значения вне допустимого списка `male`, `female`, `unisex`, это баг серверной валидации. В зависимости от влияния на логику приложения может быть security-проблемой.

**Как исправить:**
Сервер должен валидировать `gender` на backend-уровне и принимать только значения из разрешённого enum. При ошибке необходимо возвращать `400 Bad Request` с понятным описанием ошибки.

---

### Эксперимент 6. Обращение к несуществующему `catId`

**Цель:** проверить обработку ошибок при обращении к объекту, которого не существует.

**Исходный запрос:**

```http
GET /api/core/cats/get-by-id?id=21 HTTP/1.1
Host: 144.31.255.0:3001
```

**Изменённый запрос:**

```http
GET /api/core/cats/get-by-id?id=999999 HTTP/1.1
Host: 144.31.255.0:3001
```

**Ответ сервера:**

```json
TODO: вставить фактический response body
```

**Изменилось ли состояние данных:**
Нет, запрос только читает данные.

**Результат:**
TODO: указать, какой статус и тело ответа вернул сервер.

**Является ли поведением багом или security-проблемой:**
Если сервер корректно возвращает `404 Not Found`, это ожидаемое поведение. Если сервер возвращает `200 OK` с ошибкой внутри body или необработанную ошибку/stack trace, это баг обработки ошибок.

**Как исправить:**
Для несуществующего объекта сервер должен возвращать `404 Not Found` и безопасное сообщение без технических деталей реализации:

```json
{
  "error": "CAT_NOT_FOUND",
  "message": "Cat was not found"
}
```

---

### Сводная таблица экспериментов

| № | Эксперимент                              | Endpoint                               | Ожидаемая проверка на сервере                        | Фактический результат        | Проблема                             |
| - | ---------------------------------------- | -------------------------------------- | ---------------------------------------------------- | ---------------------------- | ------------------------------------ |
| 1 | Изменение `catId` при получении карточки | `GET /api/core/cats/get-by-id?id=...`  | Проверка доступа к объекту, если данные не публичные | TODO                         | TODO                                 |
| 2 | Повторение лайка                         | `POST /api/likes/cats/{catId}/likes`   | Защита от повторного голосования                     | TODO                         | TODO                                 |
| 3 | Изменение описания чужой записи          | `POST /api/core/cats/save-description` | Проверка прав на изменение записи                    | TODO                         | TODO                                 |
| 4 | Добавление дубликата имени               | `POST /api/core/cats/add`              | Проверка уникальности и корректный HTTP-статус       | Ошибка в body при HTTP `200` | Некорректная обработка бизнес-ошибки |
| 5 | Невалидный `gender`                      | `POST /api/core/cats/add`              | Валидация enum `gender`                              | TODO                         | TODO                                 |
| 6 | Несуществующий `catId`                   | `GET /api/core/cats/get-by-id?id=...`  | Корректный `404 Not Found`                           | TODO                         | TODO                                 |


### Часть 4. Изменение ответов
### Часть 5. Security-анализ
