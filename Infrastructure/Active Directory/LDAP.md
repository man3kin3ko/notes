Пример запуска анонимного бинда с самоподписанным сертификатом и без авторизации Kerberos:

```bash
LDAPTLS_REQCERT=never ldapsearch -H ldaps://DC-ADDR -b "dc=company,dc=com" -D '' -w '' -x
```

Шпаргалка:

```bash
LDAPTLS_REQCERT=never ldapsearch -H ldaps://DC-ADDR -b "dc=company,dc=com" -D "user@company.com" -W -s sub YOUR-QUERY-HERE
```

```
"(&(objectCategory=Computer)(|(operatingSystem=Windows XP*)(operatingSystem=Windows Vista*)(operatingSystem=Windows 7*)(operatingSystem=Windows Server 2008*)(operatingSystem=Windows Server 2000*)))" - EOL OSes

"(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" - S-PwdNotRequired (accounts with blank password)

"(&(objectcategory=user)(userAccountControl:1.2.840.113556.1.4.803:=65536))" - passwordNeverExpires

"(servicePrincipalName=*)" | grep servicePrincipalName: > spn-scan

"(&(objectCategory=person)(objectClass=user))" | grep "cn:\|description:" | awk -F '{print $2}' - Search credentials inside description

"(&(userAccountControl=4128)(logonCount=0))" | grep "sAMAccountName" # pre-2000 computers

"(objectclass=domain)" | grep "ms-ds-machineaccountquota" -i
```

Для каждого сервиса в LDAP будет существовать поле `servicePrincipalName`, из которого можно узнать доменное имя сервиса, [его тип](https://adsecurity.org/?p=1508) и иногда порт - это т.н. SPN scanning.

В случае, если со скомпрометированного хоста недоступен доменный DNS, но есть доступ до LDAP на ДК, можно сдампить домен через `ldapsearch` и [конвертировать его в формат BloodHound](https://github.com/SySS-Research/ldif2bloodhound). Дамп LDAP может быть шумным для SOC, т.к. [обычно пользователи используют протокол MS-SAMR](https://habr.com/ru/companies/pt/articles/423903/) для получения данных об объектах AD.