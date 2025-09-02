## Property pollution
by [Dana Epp](https://danaepp.com/attacking-apis-by-tainting-data-in-weird-places)

If an application sends to server partial updates like this
```http
PUT /api/user HTTP/1.1
Content-Type: application/json

{
 "userName":"dana",
 "pwd":"password"
}
```
And you know there is also account property which is called `accountType` and can be equal either to `user` or `admin`. But the developer strips that field. The thing you can do is
```http
PUT /api/user HTTP/1.1
Content-Type: application/json

{
 "accountType":"user",
 "userName":"dana",
 "pwd":"password",
 "accountType":"admin"
}
```
## Server side prototype pollution
by [Dana Epp](https://danaepp.com/structured-format-injection)

Imagine that you have found a web application with a user object containing id, name and role fields.

While observing your traffic, you noticed a following request:
```http
POST /myprofile HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Dana
```
Even if you still cannot perform a mass assignment, it is possible to inject JSON values which would be included in the server-side request:
````json
name=Dana","role":"admin
````
which would result in `PUT /api/users/1234{"name"="Dana","role":"admin"}`.

If your application API only supports the `application/json` content type, it is also worth trying:
```http
POST /myprofile HTTP/1.1
Content-Type: application/json

{"name"="Dana\",\"role\":\"admin"}
```

This is sometimes called **auto-binding**.
## Standard diversity prototype pollution
by [cybred](https://t.me/cybred)

Возьмем небольшое приложение с двумя микросервисами:
- Cart - реализует бизнес-логику корзины
- Payment - используется для обработки платежей

Cart написан на Python с Flask и принимает ID товаров с их количеством. Попробуем отправить в него запрос с двумя одинаковыми ключами:
```
"cart": [
    {
        "id": 0,
        "qty": 5
    },
    {
        "id": 1,
        "qty": -1,
        "qty": 1
    }
]
```
Сервис провалидирует JSON в соответствии со схемой `jsonschema.validate(instance=data, schema=schema`). Убедится, что `id: 0 <= x <= 10 and qty: >= 1`. На этом этапе не будет ошибки (не смотря на то, что один из отправленных `qty` не подходит под условие), поскольку Flask использует [стандартный JSON-парсер](https://www.json.org/json-en.html) из Python, а тот сериализует данные, отдавая приоритет последнего ключа (qty = 1).

Дальше провалидированный JSON отправляется в микросервис Payment.

А микросервис Payment написан уже на Go и использует другой парсер [buger/jsonparser](https://github.com/buger/jsonparser). Он уже не валидирует JSON (ведь валидация была на предыдущем шаге), но использует приоритет первого ключа (`qty = -1`). Считает итоговую сумму `total = total + productDB[id]["price"].(int64) * qty` и генерирует чек.

Мы смотрим в чек, который вернулся в ответе, и видим ошибку. Нам будет отправлено шесть товаров стоимостью 700 долларов, но с нас взяли только 300 долларов, из-за расчетов со вторым ключом.

Такие ошибки возникают из-за того, что существует много стандартов JSON:
1. [json.org](http://www.json.org/)
2. [IETF RFC 4627](https://tools.ietf.org/html/rfc4627)
3. [ECMAScript 262](http://www.ecma-international.org/ecma-262/5.1/#sec-15.12)
4. [ECMA 404](http://www.ecma-international.org/publications/standards/Ecma-404.htm)
5. [IETF RFC 7158](https://tools.ietf.org/html/rfc7158)
6. [IETF RFC 7159](https://tools.ietf.org/html/rfc7159)
7. [JSON5](https://json5.org/)
8. [HJSON](https://hjson.github.io/)

И в каждом из них свои правила парсинга JSON: о том, как обрабатывать дублирующие ключи, что делать с большими числами с плавающей точкой, что считать валидным, а что нет. И на каждом из этих этапов могут возникнуть коллизии, позволяющие обходить средства защиты или вызывающие баги в бизнес логике.

Полезные ссылки:
- https://seriot.ch/json/parsing.html — большая таблица-сравнение: как разные парсеры обрабатывают разные значения.
- https://bishopfox.com/blog/json-interoperability-vulnerabilities — я рассказал только об одном баге, но их гораздо больше: здесь можно почитать обо всех остальных.
- https://github.com/a1phaboy/JsonDetect — расширение для Burp для определения того, какой парсер используется.