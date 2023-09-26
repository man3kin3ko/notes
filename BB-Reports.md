# Platforms

- bugbounty.ru
- https://yandex.ru/bugbounty/index
- https://support.kaspersky.ru/vulnerability/overview/12428?el=12429#block2

# Schemes

>Attacking Redis db via CR symbols via `git://` link: https://hackerone.com/reports/441090

# Vulns

## CRLF Injection

[Session Fixation](https://hackerone.com/reports/15492) via [nginx misconfiguration](http://blog.volema.com/nginx-insecurities.html)

Escalation to XSS: 
```
?name=Bob%0d%0a%0d%0a<script>alert(document.domain)</script>
```

## Open Redirect

Can be escalated when app uses OAuth; redirection can be done using `javascript:` scheme (see https://rdnzx.medium.com/chaining-open-redirect-with-xss-to-account-takeover-36acf218a6d5 and some more payloads for fuzzing in hacktricks).

Redirection is found on reverse proxy and [can move you to internal servises (SSRF)](https://blog.detectify.com/2019/05/16/the-real-impact-of-an-open-redirect/).

## WebView

[RCE via @JavascriptInterface](https://dphoeniixx.medium.com/tiktok-for-android-1-click-rce-240266e78105)

## File upload

### Video

[SSRF-LFR in FFmpeg](https://hackerone.com/reports/1062888)

### XLSX

[XXE](https://hackerone.com/reports/105434)

## Serialization
### PHP

https://habr.com/ru/articles/708384/