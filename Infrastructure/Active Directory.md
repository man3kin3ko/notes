## LDAP

В случае, если со скомпрометированного хоста недоступен доменный DNS, но есть доступ до LDAP на ДК, можно сдампить домен через `ldapsearch` и [конвертировать его в формат BloodHound](https://github.com/SySS-Research/ldif2bloodhound). Дамп LDAP может быть шумным для SOC, т.к. [обычно пользователи используют протокол MS-SAMR](https://habr.com/ru/companies/pt/articles/423903/) для получения данных об объектах AD.

Для каждого сервиса в LDAP будет существовать поле `servicePrincipalName`, из которого можно узнать доменное имя сервиса, [его тип](https://adsecurity.org/?p=1508) и иногда порт - это т.н. SPN scanning.

## Certificate Templates

https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf

Сертификаты в AD используются для аутентификации пользователей, защиты соединения, подписи исполняемых файлов и Powershell-скриптов.