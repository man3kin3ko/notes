В экосистему Android SDK входят:
- `platform-tools`, с утилитами `adb` и `fastboot` для взаимодействия с устройством и его загрузчиком
- `cmdline-tools`, предоставляющий `avdmanager` и `sdkmanager` (и его более актуальной альтернативой `android sdk`)
- `emulator`, с исполняемым файлом для запуска системных образов Android

Утилиты синхронизируются, используя переменные среды `ANDROID_HOME`(бывшая `ANDROID_SDK_HOME`) и `ANDROID_AVD_HOME` (как правило директория `$HOME/.android/avd/`)

```
$ANDROID_HOME/
├── cmdline-tools/
│   └── latest/
│       ├── bin/
│       │   ├── sdkmanager
│       │   └── avdmanager
│       └── lib/
├── system-images/android-36/google_apis/x86_64/
├── emulator/emulator
└── platform-tools/
    ├── adb
    └── fastboot
```

В `system-images` создаются директории платформ `android-x`, где `x` - номер версии API. Внутри этих папок создаются папки с версией системного образа - `google_apis` или `google_apis_playstore`. Версия с PlayStore требует рутования через [rootAVD](https://gitlab.com/newbit/rootAVD.git) с флагом `FASTBOOTIMG`. Далее идет папка с архитектурой, под которую распространяется образ. 

Посмотреть и установить доступные для загрузки системные образы можно через:
```
android sdk list --all
android sdk install "platforms;android-36" "system-images;android-36;google_apis;x86_64"
```

Чтобы создать устройство AVD, нужно выбрать устройство для эмуляции:
```
avdmanager list device | grep pixel
```

И затем запустить его с выбранным системным образом:
```
avdmanager -v create avd -k "system-images;android-36;google_apis;x86_64" -d "pixel_10" -n "AVD"
```

### Распаковка системного образа

Системные образы, используемые эмулятором, хранятся в директории `system-images/android-36/google_apis/x86_64/` в файле `system.img`.  Распаковать файл можно, примонтировав разделы образа утилитой `kpartx`. Образ мастер-раздела `super.img` находится на набольшем разделе:

```bash
❯ sudo kpartx -av system.img
add map loop0p1 (253:0): 0 2048 linear 7:0 2048
add map loop0p2 (253:1): 0 8617984 linear 7:0 4096
```

Если образ был в формате raw/unsparse, мы можем достать его утилитой `lunpack`:
```bash
sudo dd if=/dev/mapper/loop0p2 of=/tmp/super.img bs=1M
sudo kpartx -d system.img #отмонтировать system.img
lpunpack /tmp/super.img /tmp/extracted_partitions/
7z x /tmp/extracted_partitions/system.img -o/tmp/system_part
```

Образ в формате sparse необходимо сначала преобразовать в raw-формат:
```
simg2img system.img system.raw.img
```

Такая структура системного раздела появилась начиная с Android 10, так как некоторые OTA-обновления получались слишком большими из-за фиксированного размера разделов `system`, `vendor` и `product`. Подразделы могут храниться в формате EXT или EROFS, начиная с версии 13.

Если разделы хранятся в формате EXT, их можно просто примонтировать:
```
sudo mkdir -p /tmp/android_root/system/{"vendor","product","odm"}
cd /tmp/extracted_partitions
sudo mount -o loop,ro system.img /tmp/android_root
sudo mount -o loop,ro vendor.img  /tmp/android_root/vendor
sudo mount -o loop,ro product.img  /tmp/android_root/product
sudo mount -o loop,ro system_dlkm.img  /tmp/android_root/system_dlkm
sudo mount -o loop,ro system_ext.img  /tmp/android_root/system_ext
```

> Google использует функцию `shared_blocks` для экономии места в `system.img`. Ядро Linux физически не может смонтировать такую файловую систему в режиме `rw` и выдает ошибку суперблока. Разделы можно смонтировать на запись только через OverlayFS в определенном порядке

Размонтировать в обратном порядке:
```
sudo umount /tmp/android_root/system_ext
sudo umount /tmp/android_root/system_dlkm
sudo umount /tmp/android_root/product
sudo umount /tmp/android_root/vendor
sudo umount /tmp/android_root
```

### Файлы конфигураций

Директория `ANDROID_AVD_HOME` содержит файлы конфигураций, использующиеся при создании и запуске AVD устройств. Для каждого эмулятора создаются две основные сущности:

- `<AVD_name>.ini` - хранит строчку с путем к основной папке эмулятора
- `<AVD_name>.avd/` - папка с внутренним содержимым устройства, в которой ключевую роль играет файл `config.ini`

В `config.ini` использование `hw.keyboard=yes` позволяет передавать сигналы с клавиатуры хоста в эмулятор.

### Использование эмулятора

Эмулятор получает список доступных устройств из директории в `ANDROID_AVD_HOME`. 
Чтобы посмотреть их, доступна команда
```
emulator -list-avds
```

Пример запуска эмулятора:

```
emulator @<AVD_name> -use-system-libs -writable-system -http-proxy `ip -4 addr show $(ip route show default | awk '/default/ {print $5}') | awk '/inet / {split($2, a, "/"); print a[1] ":8080"}'` -selinux permissive -no-snapshot -qemu -enable-kvm
```

> ⚠️ Аргументы после `-qemu` передаются QEMU и не используются эмулятором, поэтому важен порядок

Флаг `-use-system-libs` решает проблему рендеринга видеокартой (серый экран), также можно использовать переменную `ANDROID_EMULATOR_USE_SYSTEM_LIBS=1`. 

`-selinux permissive` вместе с `-writable-system` необходим для рутования

`-no-snapshot` можно опускать, но он полезен при отладке для запуска без сохранения состояния

Однострочник после `-http-proxy` прокидывает весь трафик на 8080 порт адреса интерфейса по умолчанию хоста 

### Рутирование

Для использования rootAVD должны быть установлены переменные среды и запущено устройство, которое требуется рутовать. Пути системных образов должны передаваться относительно `ANDROID_HOME`, иначе скрипт будет возвращать ошибку со справкой об использовании.