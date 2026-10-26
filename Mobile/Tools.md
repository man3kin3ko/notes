# Статический анализ приложений

### FireBase

Проверить корректность настройки FireBase: в MobSF найти адрес проекта и перейти на `https://project.firebase.io/.json`. Если в возвращенном значении будет `Database have been deactivated` или `Permission denied`, то FB выключен или корректно настроен.

https://cloud.hacktricks.xyz/pentesting-cloud/gcp-pentesting/gcp-services/gcp-databases-enum/gcp-firebase-enum

Некорректная конфигурация FB может привести к рассылке фишинговых пушей: https://abss.me/posts/fcm-takeover/

### Hardocded credentials

Иногда API-ключи могут храниться разработчиками в виде массива байт, i.e. `bytes = [104, 106, ..., 107]`, что не позволяет найти их обычным грепом. 

# Динамический анализ приложений

https://github.com/Genymobile/scrcpy - двустороннее копирование в буфер обмена, запись экрана и др.

https://github.com/rom1v/autoadb - автоматизация действий (старт сервера фриды, включение scrcpy)

## Frida

```
adb root
adb shell 
>setenforce 0
>/data/local/tmp/frida-server &

#magiskhide disable # можно использовать если frida установлена как дополнение magisk, но нужно чтобы клиент на пк совпадал по версии
```

Для сложных случаев (например, когда приложение использует кастомное шифрование внутри TLS-сессии) существует аддон для Burp - Brida. Плагин позволяет перехватывать большую часть java-вызовов, связанных с криптографией, что позволяет, например, быстро создавать скрипты для подделки boolean-based биометрической аутентификации. В [этом видео авторы](https://www.youtube.com/watch?v=RawqXSslsQk) показывают, как его использовать.

> ⚠️ Brida не сможет создавать хуки для нативных криптографических функций

## Анализ логов

⚠️ Python2.7, в blackarch есть отдельный пакет, так что можно не использовать pyenv

https://github.com/JakeWharton/pidcat

## Activity Manager

Из коробки запускать бродкаст сообщения и интенты можно через `am`:

```
adb shell am start -W -n com.example.app/.ui.MainActivity --es user_name "Ivan" --ei user_id 123 --ez is_premium true -d 
```
где `ez` - extra булево, `ei` - число, `es` - строка, получаются через `intent.getStringExtra`; `d` - дата, получаемая через `intent.getData` 

```
adb shell am startservice com.example.package/.MyService
```

```
adb shell am broadcast -a com.example.ACTION_MY_BROADCAST -n com.example.package/.MyReceiver
```