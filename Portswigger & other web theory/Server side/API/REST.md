## [Спецификация OpenAPI](https://swagger.io/specification/)

Конвертировать из коллекции Postman:

https://www.postman.com/postman/workspace/postman-open-technologies-convert-postman-collections-into-openapi/overview

Спецификацию OpenAPI используют многие автоматические фаззеры, такие, как RESTler.

Чтобы найти спецификацию, нужно перебрать [директории](https://github.com/hAPI-hacker/Hacking-APIs/blob/main/api_docs_path) и [поддомены](https://github.com/hAPI-hacker/Hacking-APIs/blob/main/docs_subdomain) сервиса.

## Аутентификация

`Authentication Token Obtain and Replace` плагин в бурпе может за вас сходить на другой сайт с куками, получить JWT и подставить его к Repeater/Scanner/Extension-запросам

```
Authorization: {JWT, token, Bearer, none} ...
X-Access-Token: ...
X-Auth-Token: ...
X-My-Header: - проверить использование кастомных заголовков в Access-Allow-Headers после OPTIONS-запроса
```

[ ⌛ Как проверять криптографическую надежность токенов ]

### Уязвимости

### JWT

https://token.dev/

Иногда поля токена обрабатываются другими сервисами, и туда можно осуществить инъекцию

https://github.com/ticarpi/jwt_tool

Для бека на node.js: https://github.com/aalex954/CVE-2022-23529-Exploration

#### Scan

```bash
python3 jwt_tool.py -t $URL -rh "Authorization: $TOKEN" -M at -np
```

#### Key Confusion Attack

https://portswigger.net/web-security/jwt/algorithm-confusion

Если используется ассиметричный алгоритм, например, RS256, сменить его на HS256 и попытаться использовать публичный ключ как симметричный ключ:

```bash
python3 jwt_tool.py $TOKEN -X k -pk public-key.pem
```

#### Cracking

```
hashcat -a 0 -m 16500 jwt.txt passlist.txt
```

https://github.com/wallarm/jwt-secrets/blob/master/jwt.secrets.list

#### kid / jku header

https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/JSON%20Web%20Token/README.md#jwt-claims

#### Постэксплуатация

Внедрение самоподписанных cертификатов через заголовок `"x5c"`.

Ошибки сериализации при обработке внедренного заголовка Content-Type `"cty"`

### Buisness logiс bypass
Если дана последовательность ручек:

```
POST /v1/user/register (phone, birthdate, pan) -> registerToken
POST /v1/user/register/check (registerToken, otp)
POST /v1/user/register/confirm (registerToken, login, password)
```
Попробовать просто пропустить те, что требуют предыдущий этап.

Если в документации сказано, что какие-то вещи строго нельзя делать, их нужно делать. Если даны ограничения по объему, формату или кодировке данных, их необходимо нарушать.

## See also

- https://cheatsheetseries.owasp.org/cheatsheets/REST_Security_Cheat_Sheet.html
- https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/web-api-pentesting
