# Validation

### Обфускация

- против регулярок - переносы строк, смена регистра
- внутри HTML-атрибутов и SVG-тегов можно использовать XML/HTML entity encoding
- названия функций можно кодировать в UTF  `\u00 + 'Щ'.charCodeAt(0).String(16)`
- в функцию можно передать hex-энкод:

```js
setTimeout`\x61\x6c\x65\x72\x74\x28\x31\x29\` 
```

# CSP

### JSONP

### Links

- https://dev.to/leapcell/a-deep-dive-into-javascript-sandboxing-97b
- https://aszx87410.github.io/beyond-xss/en/ch2/csp-bypass/

# SOP

SOP bypass can be considered a vulnerability in a brwoser itself, if it doesn't follow the inended behavior. An example is [Issue 1359122: Security: SOP bypass leaks navigation history of iframe from other subdomain if location changed to about:blank](https://bugs.chromium.org/p/chromium/issues/detail?id=1359122&q=subdomain%20host%20leak&can=1):

Suppose the web page is `a.example.com` and contains an iframe with the URL `b.example.com`. By redirecting the iframe to `about:blank` using `frames[0].location = 'about:blank'`, the iframe becomes same-origin with `a.example.com`. At this point, accessing the iframe's navigation history using `frames[0].navigation.entries()` allows retrieving the original URL of `b.example.com`.

This should not happen. When an iframe is redirected to another URL, `navigation.entries()` should be cleared. Therefore, this is a bug.

### Links

- https://aszx87410.github.io/beyond-xss/en/ch1/browser-security-model/