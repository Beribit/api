# Руководство подключению к API 


## Навигация
1. [Введение](#introduction)
2. [Тестирование](#testing)
3. [HTTP-коды возврата](#httpcodes)
4. [Авторизация](#auth)
5. [Аккаунт](#account)<br/>
5.1 [Получение баланса по всем валютам](#balance_all)<br/>
5.2 [Функция получения баланса по конкретной валюте](#balance_token)

---

## <a id="introduction">Введение</a>
### Для получение API-ключей необходимо:
1. Быть зарегистрированным на <a href="https://beribit.com/register" target="_blank">beribit.com</a>
2. В <a href="https://beribit.com/profile">настройках профиля</a> установить Google Authenticator
3. Cвязаться с <a href="https://t.me/beribitbot" target="_blank">технической поддержкой</a> и попросить выдать доступ

### Основная информация
- Базовый адрес: https://api.beribit.com
- API-ключи **чувствительны к регистру**.
- Все поля связанные со временем, указываются с нулевым смещением (UTC+0) в формате _YYYY-MM-DD'T'hh:mm:ss_ <br>
>**Если время сейчас 2023-09-14 15:44:31 (UTC+3), то в запросе оно передается в следующем формате:**
>```
> 2023-09-14T12:44:31
>```

---

## <a id="testing">Тестирование</a>

> Версия API на тестовом сервере может быть более новой и в некоторых моментах отличаться от описанной в документации.
> 
Для тестирования работы с API можно использовать тестовый сервер, https://test.beribit.com. Тестовый сервер во всём аналогичен рабочему, кроме исполнения депозитов/выводов, и использует 
копию базы с продакшна. Если для тестов необходимы дополнительные лимиты на тестовом сервера, они могут быть получены через техническую поддержку.

---

## <a id="httpcodes">HTTP-коды возврата</a>
|Код возврата| Описание |
|--|--|
| 200 | Операция выполнена успешно, возвращается объект JSON |
| 4XX | Используются для некорректно сформированных запросов; проблема на стороне отправителя |
| 401 | Используется при ошибке авторизации |
| 408 | Используется если превышен интервал запроса |
| 5XX | Используются для внутренних ошибок, проблема на стороне BERIBIT. **Важно НЕ рассматривать это как неудачную операцию; состояние выполнения НЕИЗВЕСТНО и могло быть выполнено успешно** |


## <a id="auth">Авторизация</a>
<b>**Обязательно:** Для каждого запроса ВНЕ ЗАВИСИМОСТИ ОТ ТИПА (GET/POST/DELETE) добавляется штамп времени (текущее время в формате UTC), передается в параметре запроса timestamp.</b>
>```
> GET https://api.beribit.com/accounts?timestamp=2023-08-20T13:51:00
>```
>```
> POST https://api.beribit.com/deposit/generate_address?timestamp=2023-08-20T13:51:00
>```

### Для авторизации необходимо поместить в следущие заголовоки запроса:
|Заголовок| Содержание |
|--|--|
| UID | **Персональный ключ (находится в API-ключах)** |
| SIGNATURE | **HMACSHA256-хеш (поле расчитываемое на каждый запрос)** |

---

### Примеры получения SIGNATURE
На ключах показаных в данном разделе можно проверить правильность генерации подписи в вашем приложении.

---
#### GET ЗАПРОС 
```
GET https://api.beribit.com/deposit/history?Timestamp=2023-08-20T13:51:00&Limit=10
```
|Параметр| Описание | Пример|
|--|--|--|
|PRIVATE_KEY|Приватный ключ HMACSHA256|ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==|
|QUERYSTRING|Все Query-параметры из HTTP запроса |?Timestamp=2023-08-20T13:51:00&Limit=10|

> Подписью является HMACSHA256(QUERYSTRING), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для GET-запроса:

    ?Timestamp=2023-08-20T13:51:00&Limit=10

Полученая подпись которую необходимо поместить в заголовок SIGNATURE

    45d8011a090e13502bcc1397650119ea4f37d369b3c9cdd64af2e92dbd493ad7

---
#### POST ЗАПРОС 
```
POST https://api.beribit.com/orders?Timestamp=2023-08-20T13:51:00
{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }
```
|Параметр| Описание | Пример|
|--|--|--|
|PRIVATE_KEY|Приватный ключ|ma8cy8DLE5SdlrB745b3MvfZbJyOoBTkUEc3YFvgMLc8eVgJjtjt/cp0PWR6ts357z5FOFUeuqTyHM0O7xn0Vw==|
|PARAMS|Все Query-параметры из HTTP запроса |?Timestamp=2023-08-20T13:51:00|
|PAYLOAD|Тело запроса|{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }|

Подписью является HMACSHA256(PARAMS + ":" + PAYLOAD), где SecretKey HMACSHA256 является PRIVATE_KEY

Пример тела подписи для POST-запроса:

    ?Timestamp=2023-08-20T13:51:00:{ "Market": "USDT_RUB", "Volume": 100.0, "Price": 97.0, "OrderSide": "buy", "OrderType": "limit" }

Полученая подпись которую необходимо поместить в заголовок SIGNATURE

    15786f9f487c2ed8bcc6ddbe4f107f9d8dde0b26179e35de94b21665706637ed

---

## <a id="account">Аккаунт</a>

### <a id="balance_all">Получение баланса по всем валютам</a>

Тип запроса: **GET**

Путь: **/accounts**

Описание: **данная функция отвечает за возвращение списка криптовалютных балансов.**

Результат успешного выполнения: **возвращается название криптовалюты, сумма на открытом счету, сумма на закрытом счету и таймштамп.**

Пример ответа:
```<json>
{
    "Success": true,
    "Result": [
        {
            "Currency": "RUB",
            "Balance": 10000.00,
            "Locked": 2000.00,
            "Time": "2023-09-15T09:48:40.8485648Z"
        },
        {
            "Currency": "ETH",
            "Balance": 300.053021,
            "Locked": 50.00,
            "Time": "2023-09-15T09:48:40.848655Z"
        },
        {
            "Currency": "USDT",
            "Balance": 300.04,
            "Locked": 2560.73,
            "Time": "2023-09-15T09:48:40.8486553Z"
        }
    ]
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```

---

### <a id="balance_token">Функция получения баланса по конкретной валюте</a>

Тип запроса: **GET**

Путь: **/account/{currency}**

Описание: **данная функция отвечает за возвращение баланса заданной валюты.**

Результат успешного выполнения: **возвращается сумма на открытом счету, сумма на закрытом счету и таймштамп.**

При ошибке: **возвращается код ответа 400 и текст ошибки, если:**
- **валидация модели прошла неуспешно.**

| Параметр | Тип | Обязательный | Примечание |
| -- | -- | -- | -- |
| Currency | string | обязательный | валюта, баланс которой требуется получить |

Пример запроса:
```<json>
{
  "Currency": "USDT",
}
```

Пример ответа:
```<json>
{
    "Success": true,
    "Result": {
        "Balance": 10000.00,
        "Locked": 3500.05,
        "Time": "2023-09-15T09:47:29.2933083Z"
    }
}
```

Пример ошибки:
```<json>
{
  "Success": false,
  "Error": {
    "Message": "Unauthorized"
    "Time": "2023-09-05T10:25:06.6590684Z"
  }
}
```
