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
Так как всем был дан один аккаунт, то логично что все коты принадлежат одному аккаунту => этот акк может изменять описание всех своих котов.
Также дополнительно проверил возможность изменить описание у несуществующего кота, показало 404 ошибку

**Как исправить:**
Проверять после запроса создал ли отправитель данного кота. И если нет, то возвращать `403 Forbidden`.

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

{"cats":[{"description":"","gender":"male","name":"имя"}]}
```

**Изменённый запрос:**

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"","gender":"МУЖ","name":"иимя"}]}
```

**Ответ сервера:**

```json
{"cats":[{"id":46,"name":"Иимя","description":"","tags":"","gender":"unisex","likes":0,"dislikes":0}]}
```

**Изменилось ли состояние данных:**
Кот добавлен, переменная gender приобрела значение unisex

**Результат:**
Кот добавлен, переменная gender приобрела значение unisex

**Является ли поведением багом или security-проблемой:**
Поведение является багом, так как пол важная личная информация и не может иметь по умолчанию значение унисекса. Значение "неизвестно" подошло бы

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
{"data":null,"isBoom":true,"isServer":false,"output":{"statusCode":404,"payload":{"statusCode":404,"error":"Not Found","message":"cat not found"},"headers":{}}}
```

**Изменилось ли состояние данных:**
Нет, запрос только читает данные.

**Результат:**
Кот (объект) не найден

**Является ли поведением багом или security-проблемой:**
Это ожидаемое поведение

**Как исправить:**
Всё ок
### Часть 4. Изменение ответов

### Эксперимент 1. Подмена имени или описания котика в ответе поиска

**Цель:** проверить, отображает ли приложение данные из ответа сервера без дополнительной проверки.

**Исходный запрос:**

```http
POST /api/core/cats/search HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"gender":null,"name":"великий","order":"asc"}
```

**Исходный ответ сервера:**

```json
{"groups":[{"title":"В","cats":[{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}],"count":1}]}
```

**Подменённый ответ:**

```json
{"groups":[{"title":"В","cats":[{"id":40,"name":"Великий","description": "Йоу","tags":"","gender":"male","likes":0,"dislikes":0}],"count":1}]}
```

**Что изменилось в приложении:**
Приложение отображает подменённый ответ

**Скриншот до:**

<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/ab17818e-61c4-4a45-a025-912b4169301b" />


**Скриншот после:**

<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/19a13a22-b047-400c-90da-f71f6d8fb6d9" />


**Сохранилось ли изменение на сервере:**
Нет. Подмена выполнялась только на уровне ответа, поэтому без отдельного запроса на сохранение данные на сервере измениться не должны.

### Эксперимент 2. Подмена количества лайков и дизлайков в ответе карточки

**Цель:** проверить, можно ли визуально изменить рейтинг котика в приложении через подмену ответа.

**Исходный запрос:**

```http
GET /api/core/cats/get-by-id?id=21 HTTP/1.1
Host: 144.31.255.0:3001
```

**Исходный ответ сервера:**

```json
{"cat":{"id":21,"name":"Турбопетров","description":"тихий, без суеты, черемша\n","tags":"","gender":"unisex","likes":42,"dislikes":40}}
```

**Подменённый ответ:**

```json
{"cat":{"id":21,"name":"Турбопетров","description":"тихий, без суеты, черемша\n","tags":"","gender":"unisex","likes":4202,"dislikes":4010}}
```

**Что изменилось в приложении:**
Изменилось

**Скриншот до:**
<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/873a34b7-c09d-4275-8e70-11830a380499" />


**Скриншот после:**

<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/08e441cc-2f19-4ecd-a170-86fab9e5b908" />

**Сохранилось ли изменение на сервере:**
Нет

### Эксперимент 3. Подмена списка результатов поиска

**Цель:** проверить поведение приложения при изменении массива результатов поиска.

**Исходный запрос:**

```http
POST /api/core/cats/search HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"gender":null,"name":"велик","order":"asc"}
```

**Исходный ответ сервера:**

```json
{"groups":[{"title":"В","cats":[{"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0},{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}],"count":2}]}
```

**Подменённый ответ:**

```json
{"groups":[{"title":"В","cats":[{"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0},{"id":40,"name":"Великий","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}, {"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}, {"id":42,"name":"Велик","description":"великий кот новгородский","tags":"","gender":"male","likes":0,"dislikes":0}],"count":4}]}
```

**Что изменилось в приложении:**
Появились добавленные коты

**Скриншот до:**
<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/a2258743-9a5d-4c8d-91d0-aaa6e36104d9" />


**Скриншот после:**
<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/988918ae-f9a8-43fd-a6b0-e5d30f4da979" />


**Сохранилось ли изменение на сервере:**
Нет

### Эксперимент 4. Подмена списка фото или URL изображения

**Цель:** проверить, можно ли изменить отображаемую фотографию котика через подмену ответа со списком изображений.

**Исходный запрос:**

```http
GET /api/photos/cats/42/photos HTTP/1.1
Host: 144.31.255.0:3001
```

**Исходный ответ сервера:**

```json
{"images":["/photos/image-1780233780068.jpg","/photos/image-1780233781011.jpg","/photos/image-1780235536129.jpg"]}
```

**Подменённый ответ:**

```json
{"images":["/photos/image-1780233780068.jpg","/photos/image-1780233781011.jpg","/photos/image-1780064483119.jpg"]}
```


**Что изменилось в приложении:**
третье фото изменилось

**Скриншот до:**
<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/959bb96b-389a-4f5a-9c26-27fcf87837a0" />


**Скриншот после:**
<img width="720" height="1520" alt="image" src="https://github.com/user-attachments/assets/adb55d09-0468-499e-bd61-48f81e1ce67a" />


**Сохранилось ли изменение на сервере:**
Нет

### Часть 5. Security-анализ

Особо проблем не нашёл, делюсь находками.

#### Finding 1. 

**Title:** Отсутствие защиты от повторного голосования

**Severity:** Medium

**Preconditions:**

* Пользователь авторизован в мобильном приложении или имеет возможность отправлять HTTP-запросы к API.
* Известен `catId` существующего котика.
* Есть доступ к Charles Replay / Postman / curl.

**Steps to reproduce:**

1. Перехватить запрос лайка в Charles Proxy:

```http
POST /api/likes/cats/18/likes HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"dislike":false,"like":true}
```

2. Повторить этот же запрос несколько раз через Replay / Postman / curl.
3. Сравнить значения `likes` в ответах сервера.

**Actual result:**

Повторная отправка запроса увеличивает количество лайков. В эксперименте после повторов значение `likes` увеличилось с `16` до `18`.

Пример ответа:

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

**Expected result:**

Один пользователь не должен иметь возможность бесконечно увеличивать рейтинг одного и того же котика повтором одинакового запроса. Повторное голосование должно быть идемпотентным: либо не менять счётчик, либо заменять предыдущее состояние голоса пользователя.

**Evidence:**

* Request/response из Charles Proxy.
* В Части 3 зафиксировано, что повтор запроса лайка увеличивает счётчик лайков.

**Recommendation:**

На сервере нужно хранить факт голосования пользователя за конкретный `catId`, например в таблице вида:

```text
user_id | cat_id | vote_type
```

Если пользователь уже голосовал, сервер должен обновлять существующий голос, а не добавлять новый. Также стоит добавить rate limit на endpoint голосования.

---

#### Finding 2. 

**Title:** Возможность изменения описания объекта по произвольному `catId`

**Severity:** Medium

**Preconditions:**

* Пользователь может отправлять запросы к API.
* Известен `catId` существующего котика.
* Endpoint `/api/core/cats/save-description` доступен из клиента.

**Steps to reproduce:**

1. Перехватить запрос изменения описания.
2. Изменить поле `catId` или отправить запрос напрямую:

```http
POST /api/core/cats/save-description HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"catDescription":"Изменил","catId":18}
```

3. Проверить ответ сервера и повторно открыть карточку котика.

**Actual result:**

Сервер изменил описание объекта с указанным `catId`.

Пример ответа:

```json
{
  "id": 18,
  "name": "Обормотик",
  "description": "Изменил",
  "tags": "",
  "gender": "unisex",
  "likes": 18,
  "dislikes": 13
}
```

**Expected result:**

Если в системе предполагаются разные пользователи или владельцы записей, сервер должен проверять, имеет ли текущий пользователь право изменять конкретный объект. Пользователь не должен иметь возможность изменять чужие записи только за счёт подстановки другого `catId`.

**Evidence:**

* Request/response из Charles Proxy.
* В Части 3 зафиксировано, что описание объекта изменилось после прямого запроса к `/api/core/cats/save-description`.

**Recommendation:**

Добавить серверную проверку прав доступа:

* определить владельца записи;
* связать запрос с текущим пользователем;
* разрешать изменение только владельцу или администратору;
* при отсутствии прав возвращать `403 Forbidden`.

Если по логике учебного стенда все записи общие и редактируемые, это следует явно зафиксировать как бизнес-правило. В реальном приложении такая модель была бы рискованной.

---

#### Finding 3.

**Title:** Невалидное значение `gender` принимается сервером и автоматически превращается в `unisex`

**Severity:** Low / Medium

**Preconditions:**

* Пользователь может отправить запрос добавления котика.
* Endpoint `/api/core/cats/add` доступен.
* В Swagger допустимые значения `gender`: `male`, `female`, `unisex`.

**Steps to reproduce:**

1. Отправить корректный запрос добавления:

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"","gender":"male","name":"имя"}]}
```

2. Изменить `gender` на невалидное значение:

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"","gender":"МУЖ","name":"иимя"}]}
```

**Actual result:**

Сервер создал объект и автоматически установил `gender` в `unisex`.

```json
{
  "cats": [
    {
      "id": 46,
      "name": "Иимя",
      "description": "",
      "tags": "",
      "gender": "unisex",
      "likes": 0,
      "dislikes": 0
    }
  ]
}
```

**Expected result:**

Сервер должен отклонять значения, которых нет в enum. Для невалидного `gender` ожидаемый ответ:

```http
HTTP/1.1 400 Bad Request
```

Пример тела ответа:

```json
{
  "error": "VALIDATION_ERROR",
  "message": "gender must be one of: male, female, unisex"
}
```

**Evidence:**

* Request/response из эксперимента с невалидным `gender`.
* Фактическое состояние данных изменилось: кот был создан, хотя поле содержало невалидное значение.

**Recommendation:**

Добавить строгую серверную валидацию enum-полей. Не заменять некорректное значение на значение по умолчанию без явного согласия пользователя. Это может приводить к некорректным данным и скрытым ошибкам в бизнес-логике.

---

#### Finding 4. 

**Title:** Бизнес-ошибки возвращаются с HTTP-статусом `200 OK`

**Severity:** Low

**Preconditions:**

* Пользователь может отправлять запросы к `/api/core/cats/add`.
* В системе уже существует котик с таким же именем.

**Steps to reproduce:**

1. Отправить запрос на добавление котика:

```http
POST /api/core/cats/add HTTP/1.1
Host: 144.31.255.0:3001
Content-Type: application/json

{"cats":[{"description":"великий кот новгородский","gender":"male","name":"великий"}]}
```

2. Повторить тот же запрос ещё раз.

**Actual result:**

Сервер вернул HTTP `200 OK`, но внутри body передал ошибку:

```json
{
  "cats": [
    {
      "errorDescription": "Такое значение уже существует"
    }
  ]
}
```

Похожее поведение наблюдалось и при слишком коротком имени:

```json
{
  "cats": [
    {
      "errorDescription": "Имя слишком короткое!"
    }
  ]
}
```

**Expected result:**

Для ошибки дубликата сервер должен возвращать статус `409 Conflict`. Для ошибки валидации — `400 Bad Request`.

Например:

```http
HTTP/1.1 409 Conflict
```

```json
{
  "error": "DUPLICATE_CAT_NAME",
  "message": "Такое значение уже существует"
}
```

**Evidence:**

* Request/response из Charles Proxy.
* В Части 2 и Части 3 зафиксировано, что при дубликате сервер возвращает ошибку внутри JSON, но HTTP-код остаётся `200`.

**Recommendation:**

Использовать корректные HTTP-статусы:

* `400 Bad Request` — ошибка валидации;
* `404 Not Found` — объект не найден;
* `409 Conflict` — конфликт состояния, например дубликат;
* `403 Forbidden` — нет прав на действие.

Это упростит обработку ошибок на клиенте и снизит риск неправильной интерпретации неуспешных операций как успешных.

---

#### Finding 5. 

**Title:** API работает по HTTP без транспортного шифрования

**Severity:** Medium

**Preconditions:**

* Мобильное устройство и атакующий находятся в одной сети или атакующий может контролировать сетевой маршрут.
* Приложение обращается к API по HTTP.

**Steps to reproduce:**

1. Подключить телефон и компьютер с Charles Proxy к одной Wi-Fi сети.
2. Настроить ручной HTTP proxy на телефоне.
3. Открыть приложение и выполнить действия: поиск, открытие карточки, лайк, изменение описания.
4. Убедиться, что запросы и ответы видны в Charles без установки сертификата.

**Actual result:**

Трафик приложения передаётся по HTTP. В Charles видны:

* URL запросов;
* request body;
* response body;
* данные карточек;
* значения лайков/дизлайков;
* запросы изменения описания;
* ответы API.

Также через Rewrite можно подменять ответы сервера, после чего приложение отображает изменённые данные.

**Expected result:**

В реальном приложении API должен использовать HTTPS. Клиент должен проверять TLS-сертификат сервера. Для чувствительных сценариев можно дополнительно использовать certificate pinning.

**Evidence:**

* Вся карта действий приложения была собрана через Charles Proxy.
* В Части 4 через Rewrite были успешно подменены имя/описание, лайки/дизлайки, список результатов поиска и список фото.
* Изменения отображались в приложении, хотя не сохранялись на сервере.

**Recommendation:**

Использовать HTTPS для всех API-запросов. Отключить передачу данных по HTTP. Для production-сборки рассмотреть:

* HSTS на backend/frontend;
* запрет cleartext traffic в Android Network Security Config;
* certificate pinning, если это соответствует модели угроз.

Для учебного стенда HTTP допустим как часть лабораторной работы, но в реальном приложении это была бы значимая проблема безопасности.

---

#### Finding 6.

**Title:**  Технически подробная структура ошибки раскрывается клиенту

**Severity:** Low

**Preconditions:**

* Пользователь может отправлять запросы к API.
* Запрашивается несуществующий объект.

**Steps to reproduce:**

1. Отправить запрос на получение несуществующего котика:

```http
GET /api/core/cats/get-by-id?id=999999 HTTP/1.1
Host: 144.31.255.0:3001
```

**Actual result:**

Сервер возвращает ошибку со служебными полями:

```json
{
  "data": null,
  "isBoom": true,
  "isServer": false,
  "output": {
    "statusCode": 404,
    "payload": {
      "statusCode": 404,
      "error": "Not Found",
      "message": "cat not found"
    },
    "headers": {}
  }
}
```

Сам факт `404 Not Found` является корректным, но тело ответа раскрывает внутреннюю структуру обработки ошибок: `isBoom`, `isServer`, `output`, `payload`, `headers`.

**Expected result:**

Клиенту должно возвращаться минимальное и стабильное тело ошибки, не раскрывающее внутренние детали используемых библиотек или middleware.

Например:

```json
{
  "error": "CAT_NOT_FOUND",
  "message": "Cat was not found"
}
```

**Evidence:**

* Request/response из эксперимента с несуществующим `catId`.
* Сервер корректно определил, что объект не найден, но вернул технически подробную структуру ошибки.

**Recommendation:**

Добавить единый error handler на backend-стороне и нормализовать ошибки перед отправкой клиенту. Не возвращать внутренние поля библиотек, stack trace, служебные флаги и технические структуры. Для клиента достаточно стабильного `error code`, человекочитаемого `message` и корректного HTTP-статуса.

---
