---
title: Лекция 5.4 - HTTP методы и REST API
---

# HTTP методы и REST API

## Введение

В предыдущих лабораторных мы использовали **GET-запросы** для получения данных. Но что, если нужно не получить, а **отправить** данные? Забронировать место, создать заказ, обновить профиль?

Для этого существуют другие HTTP-методы: **POST**, **PUT**, **PATCH**, **DELETE**.

---

## CRUD и HTTP

### Что такое CRUD

**CRUD** — четыре базовые операции с данными:

| Операция | Описание |
|----------|----------|
| **C**reate | Создать новую запись |
| **R**ead | Прочитать данные |
| **U**pdate | Обновить существующую запись |
| **D**elete | Удалить запись |

### Соответствие CRUD и HTTP

| CRUD | HTTP метод | Пример URL | Описание |
|------|------------|------------|----------|
| Create | **POST** | /api/bookings | Создать бронь |
| Read | **GET** | /api/movies | Получить список |
| Read | **GET** | /api/movies/1 | Получить один элемент |
| Update | **PUT** | /api/movies/1 | Заменить полностью |
| Update | **PATCH** | /api/movies/1 | Частичное обновление |
| Delete | **DELETE** | /api/movies/1 | Удалить |

---

## HTTP методы подробно

### GET — Получение данных

```http
GET /api/movies HTTP/1.1
Host: api.cinema.com
```

**Характеристики:**
- Только получение, без изменения
- **Идемпотентный** — повторный вызов даёт тот же результат
- **Безопасный** — не меняет состояние сервера
- Параметры в URL: `/api/movies?genre=action&year=2024`

### POST — Создание ресурса

```http
POST /api/bookings HTTP/1.1
Host: api.cinema.com
Content-Type: application/json

{
  "sessionId": 42,
  "seats": [
    {"row": 5, "number": 10},
    {"row": 5, "number": 11}
  ]
}
```

**Характеристики:**
- Создаёт новый ресурс
- **НЕ идемпотентный** — повторный вызов создаст ещё одну запись
- Данные в теле запроса (body)

### PUT — Полная замена ресурса

```http
PUT /api/users/123 HTTP/1.1
Content-Type: application/json

{
  "name": "Иван Иванов",
  "email": "ivan@example.com",
  "phone": "+7999..."
}
```

**Характеристики:**
- Заменяет ресурс целиком
- **Идемпотентный** — повторный вызов даёт тот же результат
- Если поле не указано — оно становится null/default

### PATCH — Частичное обновление

```http
PATCH /api/users/123 HTTP/1.1
Content-Type: application/json

{
  "email": "new-email@example.com"
}
```

**Характеристики:**
- Обновляет только указанные поля
- Остальные поля не меняются

### DELETE — Удаление ресурса

```http
DELETE /api/bookings/456 HTTP/1.1
```

**Характеристики:**
- Удаляет ресурс
- **Идемпотентный** — повторное удаление несуществующего = OK (или 404)

---

## Идемпотентность

### Определение

**Идемпотентность** — свойство операции, при котором многократное выполнение даёт тот же результат, что и однократное.

### Почему это важно

Представьте: вы нажали "Оплатить", запрос ушёл, но соединение прервалось до получения ответа. Что делать?

**Если POST не идемпотентен:**
- Повторить запрос? — Может создать дубль (две оплаты!)
- Не повторять? — Может оплата не прошла

**Решение: Idempotency Key**
```http
POST /api/payments HTTP/1.1
Idempotency-Key: abc123-unique-request-id

{ "amount": 1000 }
```

Сервер запоминает результат по ключу. Повторный запрос с тем же ключом вернёт сохранённый результат, а не создаст новый платёж.

### Таблица идемпотентности

| Метод | Идемпотентный | Безопасный |
|-------|---------------|------------|
| GET | Да | Да |
| HEAD | Да | Да |
| PUT | Да | Нет |
| DELETE | Да | Нет |
| POST | **Нет** | Нет |
| PATCH | **Нет** | Нет |

---

## Коды ответов HTTP

### Классы кодов

| Диапазон | Класс | Значение |
|----------|-------|----------|
| 1xx | Informational | Информационные |
| 2xx | Success | Успех |
| 3xx | Redirection | Перенаправление |
| 4xx | Client Error | Ошибка клиента |
| 5xx | Server Error | Ошибка сервера |

### Важные коды для разработчика

**2xx — Успех:**
| Код | Название | Когда использовать |
|-----|----------|-------------------|
| 200 | OK | Успешный GET, PUT, PATCH |
| 201 | Created | Успешный POST (создан ресурс) |
| 204 | No Content | Успешный DELETE (нечего возвращать) |

**4xx — Ошибка клиента:**
| Код | Название | Когда использовать |
|-----|----------|-------------------|
| 400 | Bad Request | Неверный формат запроса |
| 401 | Unauthorized | Требуется авторизация |
| 403 | Forbidden | Нет прав доступа |
| 404 | Not Found | Ресурс не найден |
| **409** | **Conflict** | Конфликт (место уже занято!) |
| 422 | Unprocessable | Валидация не прошла |
| 429 | Too Many Requests | Слишком много запросов |

**5xx — Ошибка сервера:**
| Код | Название | Когда использовать |
|-----|----------|-------------------|
| 500 | Internal Server Error | Необработанная ошибка |
| 502 | Bad Gateway | Проблема с upstream-сервером |
| 503 | Service Unavailable | Сервер перегружен |

### 409 Conflict — важный для бронирования

```http
POST /api/bookings HTTP/1.1

{ "sessionId": 42, "seats": [{"row": 5, "number": 10}] }
```

**Ответ 409:**
```http
HTTP/1.1 409 Conflict
Content-Type: application/json

{
  "error": "SeatAlreadyBooked",
  "message": "Место ряд 5, номер 10 уже забронировано",
  "bookedAt": "2024-01-15T14:30:00Z"
}
```

Это **race condition** — два человека одновременно попытались забронировать одно место.

---

## Оптимистичный vs Пессимистичный UI

### Пессимистичный UI (по умолчанию)

```
Пользователь нажал → Ждём ответ сервера → Показываем результат

[Нажал "Купить"]
        │
        ▼
   [Загрузка...]  ← Блокируем UI
        │
        ▼ (ответ сервера)
   [Показываем результат]
```

**Плюсы:** Надёжно, показываем реальное состояние
**Минусы:** Задержка ощущается пользователем

### Оптимистичный UI

```
Пользователь нажал → Сразу показываем успех → Подтверждаем (или откатываем)

[Нажал "Лайк"]
        │
        ▼
   [Сразу +1 лайк]  ← Оптимистично
        │
        ▼ (ответ сервера)
   [OK] → Оставляем
   [Ошибка] → Откатываем, показываем ошибку
```

**Плюсы:** Мгновенная реакция, приятный UX
**Минусы:** Сложнее реализовать, нужен откат

### Когда что использовать

| Сценарий | Подход | Почему |
|----------|--------|--------|
| Лайк/реакция | Оптимистичный | Низкий риск, высокая частота |
| Бронирование | Пессимистичный | Высокий риск, нужна точность |
| Добавить в корзину | Оптимистичный | Можно откатить |
| Оплата | Пессимистичный | Критически важно |

---

## POST-запрос в C#

### Базовый пример

```csharp
public async Task<BookingResponse> CreateBookingAsync(BookingRequest request)
{
    // Сериализуем объект в JSON
    var json = JsonConvert.SerializeObject(request);
    
    // Создаём контент запроса
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    
    // Отправляем POST-запрос
    var response = await _client.PostAsync($"{BaseUrl}/bookings", content);
    
    // Читаем ответ
    var responseJson = await response.Content.ReadAsStringAsync();
    
    // Обрабатываем статус
    if (response.IsSuccessStatusCode)
    {
        return JsonConvert.DeserializeObject<BookingResponse>(responseJson);
    }
    
    // Обработка ошибок
    throw new ApiException(response.StatusCode, responseJson);
}
```

### Обработка разных кодов

```csharp
public async Task<BookingResult> TryCreateBookingAsync(BookingRequest request)
{
    var json = JsonConvert.SerializeObject(request);
    var content = new StringContent(json, Encoding.UTF8, "application/json");
    
    var response = await _client.PostAsync($"{BaseUrl}/bookings", content);
    var responseJson = await response.Content.ReadAsStringAsync();
    
    switch (response.StatusCode)
    {
        case HttpStatusCode.OK:
        case HttpStatusCode.Created:
            var booking = JsonConvert.DeserializeObject<BookingResponse>(responseJson);
            return BookingResult.Success(booking);
            
        case HttpStatusCode.Conflict:
            return BookingResult.SeatAlreadyBooked();
            
        case HttpStatusCode.BadRequest:
            var error = JsonConvert.DeserializeObject<ErrorResponse>(responseJson);
            return BookingResult.ValidationError(error.Message);
            
        case HttpStatusCode.Unauthorized:
            return BookingResult.Unauthorized();
            
        default:
            return BookingResult.ServerError();
    }
}
```

---

## Тестирование API

### Postman / Insomnia

**Postman** — популярный инструмент для тестирования API:

1. Создайте новый запрос
2. Выберите метод (POST)
3. Введите URL
4. Добавьте Headers: `Content-Type: application/json`
5. Добавьте Body (JSON)
6. Отправьте и посмотрите ответ

### curl из командной строки

```bash
# GET
curl https://api.cinema.com/movies

# POST
curl -X POST https://api.cinema.com/bookings \
  -H "Content-Type: application/json" \
  -d '{"sessionId": 42, "seats": [{"row": 5, "number": 10}]}'
```

### Swagger UI

Если API документирован с помощью Swagger/OpenAPI, вы можете тестировать прямо в браузере:

```
https://api.cinema.com/swagger
```

---

## Связь с лабораторной работой

### Лаба 6: POST-запросы

В лабораторной работе вы:

1. **Создадите модели запроса/ответа:**
   ```csharp
   public class BookingRequest
   {
       public int SessionId { get; set; }
       public List<SeatInfo> Seats { get; set; }
   }
   ```

2. **Реализуете POST-запрос:**
   ```csharp
   public async Task<BookingResponse> CreateBookingAsync(BookingRequest request)
   ```

3. **Обработаете разные коды ответов:**
   - 200 OK → Показать билет
   - 409 Conflict → "Место занято", обновить карту
   - 400 Bad Request → Показать ошибку

---

## Заключение

**HTTP-методы** — это язык общения клиента и сервера:

- **GET** — получить (безопасно, идемпотентно)
- **POST** — создать (не идемпотентно!)
- **PUT/PATCH** — обновить
- **DELETE** — удалить

**Коды ответов** — способ сервера сообщить результат:

- 2xx — всё хорошо
- 4xx — клиент ошибся
- 5xx — сервер упал

Понимание этих концепций критически важно для работы с любым API.

---

## Полезные ресурсы

- [HTTP Methods - MDN](https://developer.mozilla.org/ru/docs/Web/HTTP/Methods)
- [HTTP Status Codes](https://httpstatuses.com/)
- [Postman](https://www.postman.com/)
- [REST API Tutorial](https://restfulapi.net/)
