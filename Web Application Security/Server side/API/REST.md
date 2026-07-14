## Разведка

Многие современные приложения используют Swagger для автоматической генерации документации. В зависимости от стэка, он может быть развернут по разным путям:

- Spring (Java) `/swagger-ui/index.html` или `/swagger-ui.html`
- ASP.NET `/swagger/index.html` или `/swagger`
- FastAPI (Python) `/docs`
- Node.js `/api`
- Go `/swagger/index.html` 
- 1С `/hs/swagger/ui`

Доступ к веб-интерфейсу может быть заблокирован, но используемая спецификация OpenAPI еще может быть доступна по путям:

- .NET `/swagger/v1/swagger.json`
- Spring `/v3/api-docs`, `/v2/api-docs`, `/v1/api-docs`
- Node.js `swagger_output.json` в корне проекта
- FastAPI `openapi.json`

Другие возможнык [директории](https://github.com/hAPI-hacker/Hacking-APIs/blob/main/api_docs_path) и [поддомены](https://github.com/hAPI-hacker/Hacking-APIs/blob/main/docs_subdomain) сервиса.

OpenAPI также позволяет генерировать YAML-спецификацию, при этом в Java приложении она была разбита на несколько файлов со следующей конвенцией имен:
```
microservice-slug.yaml
microservice-slug-dev.yaml
microservice-slug-description.yaml
microservice-slug-dto.yaml
```

Спецификацию OpenAPI используют многие автоматические фаззеры, такие, как RESTler. Если заказчик предоставил список запросов как коллекцию Postman: https://www.postman.com/postman/workspace/postman-open-technologies-convert-postman-collections-into-openapi/overview

Если в пути есть версия, необходимо проверить попытку даунгрейда (для поиска легаси) и апгрейда (функциональность в разработке).

Использовать метод `OPTIONS` для проверки доступных методов **на каждой ручке**. Проверить поведение при смене `Content-Type`.

Перебрать параметры на известных ручках при помощи Param Miner. 

При отсутствии спецификации фаззить методы для обнаружения новой поверхности для атаки.

## Аутентификация

`Authentication Token Obtain and Replace` плагин в бурпе может за вас сходить на другой сайт с куками, получить JWT и подставить его к Repeater/Scanner/Extension-запросам

```
Authorization: {JWT, token, Bearer, none} ...
X-Access-Token: ...
X-Auth-Token: ...
X-My-Header: - проверить использование кастомных заголовков в Access-Allow-Headers после OPTIONS-запроса
```

## Mass assignment

Уязвимость возникает, когда ручка способна изменять незаявленные поля из запроса, что нарушеает логику приложения или существующий контроль доступа.

```python
from flask import Flask, request

class User:
    def __init__(self, **kwargs):
	    # (2) стремясь к универсальности, конструктор использует все поля 
        self.__dict__.update(kwargs)
        
app = Flask(__name__)
        
@app.route('/user/', methods=['PUT'])
def update_user():
	# (1) в конструктор передается весь словарь без фильтрации
	data = User(**request.get_json())
```

Пример с ORM:
```python
from flask import Flask, request, jsonify
from sqlalchemy import create_engine, Column, Integer, String, Boolean
from sqlalchemy.orm import sessionmaker, declarative_base

app = Flask(__name__)
engine = create_engine('sqlite:///example.db')
Session = sessionmaker(bind=engine)
Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)
    is_admin = Column(Boolean, default=False)

@app.route('/user/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    session = Session()
    session.query(User).filter_by(id=user_id).update(request.get_json())
    session.commit()
```

### Рекомендации

- Использование белых списков полей
- Использование схемы API (Pydantic), паттерна DTO

### Обход фильтрации через parameter pollution

Дана ручка для изменения пользователя, при этом известно, что объект пользователя имеет поле `accountType`, которое может быть равно `user` или `admin`. При указании этого атрибута в запросе, он фильтруется.
```http
PUT /api/user HTTP/1.1
Content-Type: application/json

{
 "userName":"dana",
 "pwd":"password"
}
```

Для обхода защиты возможно использовать parameter pollution
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

## Server-side parameter pollution

Часто API выполняют роль посредника между другими внутренними сервисами, включая пользовательский ввод в другой запрос на стороне сервера. В таком случае атакующий может составить полезную нагрузку, которая атакует вторичный контекст.

```http
POST /myprofile HTTP/1.1
Content-Type: application/x-www-form-urlencoded

name=Dana
```

Предполагая, что введенные данные используются для формирования JSON к внутреннему API-сервису, попробуем внедрить дополнительные поля:

````json
name=Dana","role":"admin
````
Чтобы запрос на стороне сервера имел вид:
```http
PUT /api/users/1234

{"name"="Dana","role":"admin"}
```
Аналогичная инъекция для JSON
```http
POST /myprofile HTTP/1.1
Content-Type: application/json

{"name"="Dana\",\"role\":\"admin"}
```

Если уязвимый параметр находится в GET-запросе, поведение можно изменить при помощи символов `#`, `&` и `=`. Например, запрос к GET `/userSearch?name=peter&back=/home` сервер использует для обращения к `GET /users/search?name=peter&publicProfile=true`, тогда при внедрении полезной нагрузки `GET /userSearch?name=peter%23&back=/home` часть запроса после `#` будет игнорироваться. Использование `&` и `=` аналогично предыдущим примерам позволит производить mass assignment, parameter pollution и может открыть новую поверхность для атаки.

Инъекция может попасть в путь внутреннего API вместо GET или POST запроса, в таком случае стоит попробовать path-traversal последовательности.
## See also

- https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/web-api-pentesting
- https://danaepp.com/attacking-apis-by-tainting-data-in-weird-places
- https://danaepp.com/structured-format-injection
- https://portswigger.net/web-security/api-testing/server-side-parameter-pollution