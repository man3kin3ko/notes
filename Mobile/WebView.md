Если соответствующее диплинку активити не имплеметировано в приложении, скорее всего, ссылка откроется в WebView. 

Если WebView был инициализирован с методом `setWebContentsDebuggingEnabled`, то он будет доступен для удаленного дебага при помощи chromium-браузеров. Если дебаг выключен, его можно включить при помощи [скрипта Frida](https://gist.github.com/1mm0rt41PC/cd492f24dc061019fb25222ba0b96d20/). Для доступа к открытой WebView странице нужно будет подключить устройство через adb и зайти по адресу `chrome://inspect/#devices`. 

## Уязвимости

### XSS

По умолчанию в WebView JS выключен и для его использования требуется вызвать метод `setJavaScriptEnabled` или изменить параметр `settings.javaScriptEnabled` для его добавления явно.

Если JS включен в WebView, то в случае обнаружения XSS на целевом домене злоумышленник может иметь:
- доступ к ФС через `file://path/file` если позволяет CSP и WebView был инициализирован с `setAllowFileAccess`
- доступ к данным других приложений на внешнем носителе, если приложению были даны права `_android.permission.READ_EXTERNAL_STORAGE _` и есть доступ к ФС
- доступ к интентам через схему `intent://`

### JavaScriptInterface

Если WebView был инициализирован с методом `AddJavaScriptInterface`, то у JS будет инициализирован в режиме **native bridge**. Это значит, что у него будет доступ ко всем методам приложения, объявленным с помощью `@JavascriptInterface`. В SDK ниже 17 это может привести к RCE.

### SSL

Иногда разработчики могут переопределять методы ванильного WebView, и нужно уделять внимание, не были ли перезаписаны методы, связанные с безопасностью:

```java
@Override public void onRecivedSslError(WebView view, SslErrorHandler handler, SslError error)
{
    handler.proceed(); // Ignore SSL certificate errors
}
```

Подобная конфигурация приведет к игнорированию всех ошибок, связанных с валидацией SSL-сертификатов и [позволит осуществлять атаки типа MiTM](https://stackoverflow.com/questions/36050741/webview-how-to-avoid-security-alert-from-google-play-upon-implementation-of-onr).

--- 
- https://book.hacktricks.xyz/mobile-pentesting/android-app-pentesting/webview-attacks