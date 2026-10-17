Deeplink это

Вызов deeplink через adb:

```
adb shell am start -W -a android.intent.action.VIEW -d "myapp://open?param="
```

До версии 18 в WebView Android и до версии 25 в Chrome посредством диплинкинга можно вызывать приложение в фрейме:

```html
<iframe src="paulsawesomeapp://page1"></iframe>
```

Для каких доменов и URI ссылки откроются в самом приложении будет зависеть от настроек в `AndroidManifest.xml` в тегах `<intent-filter>`. Для того, чтобы диплинки по `http://` и `https://` схемам открывались корректно, на соответствующем домене должен быть доступен файл Digital Asset Links (DAL).

Если при объявлении intent-filter был установлен атрибут `"autoVerify=true"`, при открытии зарегестрированной ссылки система не будет постоянно спрашивать, в каком приложении пользователь хотел бы ее открыть. Однако, для этого DAL должен быть корректно сформирован.

OWASP рекомендует передавать файл по HTTPS, чтобы избежать подмены ` "package_name": "com.example.maliciousapp"` и фишинга.