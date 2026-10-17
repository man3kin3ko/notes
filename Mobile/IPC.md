### IPC & drozer

```
adb forward tcp:31415 tcp:31415
drozer console connect
run app.package.attacksurface pkg-name
```

**Activity** - это наследник класса Activity. На экране может показываться только одно Activity одновременно. Те activity, которые могут вызываться другими приложениями, называются exported activities. В лучшем случае у приложения оно одно - точка входа в него. Если это не так, необходимо убедиться, что вызов exported activity не позволяет нарушать бизнес логику или обходить локальную аутентификацию.

> ❕ Если атрибут `exported` не указан, но прописан `<intent-filter>`, activity становится экспортируемым по умолчанию

```
run app.activity.info -a pkg-name
run app.activity.start --component pkg-name activity
```

Пример path traversal в exported activity, приводящее к исполнению кода: 

- https://hackerone.com/reports/1115864
- https://hackerone.com/reports/1377748

**Content providers** предоставляют другим приложениям доступ к ресурсами (SQLite базам и файлам) через схему `content://`, если они экспортированы.

```
run app.providers.info -a pkg-name
run scanner.provider.finduris pkg-name
run app.provider.query content://$founded_query
```

> Если targetSdk < 17, то content provider без указания атрибута `exported` будет экспортирован по умолчанию

**Service** - это фоновые процессы, обеспечивающие работу уведомлений или запуск интентов. Интенты - это компоненты, позволяющие запускать процессы асинхронной IPC коммуникации посредством Binder. Binder представляет собой виртуальное символьное устройство `/dev/binder`, которое позволяет посылать сообщения между изолированными процессами различных приложений. В Android Binder является надстройкой над OpenBinder. Интенты могут:
- запускать Activity
- запускать сервисы
- рассылать бродкаст-сообщения о событиях для других приложений

**Broadcast recivers** запускают интенты в зависимости от полученных бродкастовых сообщений.

Если у экспортируемого бродкаст ресивера, сервиса, провайдера или активити стоит кастомный permission, это можно эксплуатировать:

https://github.com/commonsguy/cwac-security/blob/master/PERMS.md

### Intents

Интенты - это объекты для передачи сообщений между процессами. Интенты могут быть явными (explicit, с указанием активити-получателя) и неявными (implicit, получателя выбирает ОС).

Обычно интенты создаются для того, чтобы из одного активити запустить другое активити, при этом опционально возможно передать вместе с вызовом какие-то параметры. Для того, чтобы определить, какие параметры будут использованы, у вызываемой активити создается фабричный метод:

```java
public static Intent newIntent(Context context, String token) {
	Intent Intent = new Intent(context, ClassName.class);
	intent.putExtra("token", token);
	return intent;
}
```

Интенты [могут вызываться сайтами посредством ссылок](https://developer.chrome.com/docs/multidevice/android/intents/#syntax) и создавать активити, если у активити задан атрибут `android.intent.category.BROWSABLE` 

Интенты могут создавать угрозу интент-инъекций и интент-редиректа.

Уязвимости intent redirect возникают тогда, когда exported activity передает интент в другое активити, которое злоумышленник не смог бы запустить иным образом (не является экспортируемым.) Пример уязвимого кода и его эксплуатации приведен ниже:
```kotlin
android.content.Intent deeperIntent = (android.content.Intent) getIntent().getParcelableExtra("url");
if (deeperIntent != null) {
    startActivity(deeperIntent);
    finish();
    return;
```

```java
Intent deeperIntent = new Intent();
deeperIntent.setClassName("com.victim", "com.victim.HiddenActivity");
deeperIntent.putExtra("url", "http://evil.com/");

Intent outerIntent = new Intent();
outerIntent.setClassName("com.victim", "com.victim.ExportedActivity");
outerIntent.putExtra("extra_intent", deeperIntent);
startActivity(outerIntent);
```

Следует уделять внимание exported activity, в котором интенты передаются далее в методы `startActivity()` и `sendBroadcast()` или создаются из строки методом `Intent.parseUri()`.

![](pics/intent-redirect.png)

- https://blog.oversecured.com/Interception-of-Android-implicit-intents/#typical-vulnerabilities-in-standard-android-actions
- https://blog.oversecured.com/Android-arbitrary-code-execution-via-third-party-package-contexts/

#### Taint flow analysis

Построить граф вызовов (taint flow) авторматически можно при помощи FlowDroid, он [поддерживает нативные библиотеки](https://github.com/secure-software-engineering/FlowDroid/issues/36).
