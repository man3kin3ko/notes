# DOM-based vulnerabilities

```
Taint flow = source -> sink
```

An example of a source is the `location.search`, `document.referrer` and `document.cookie` properties since they are controllable by an attacker.

## DOM Clobbering

When you can inject HTML tags into page, but XSS is not possible, DOM clobbering technique can be used. By setting `id` or `name` attributes of some elements (embed, form, iframe, image, img, object) we can create objects as global variables in the DOM.
Then, if JavaScript on the page uses a property of a global object, you can just overwrite this object and pass your payload into the previously uncontrollable part of the code.

Some in-the-wild examples:
- [DoS via overwriting methods](https://hackerone.com/reports/1077136)
- [XSS in Gmail AMP4Email](https://research.securitum.com/xss-in-amp4email-dom-clobbering/)

And a [cheatsheet](https://tib3rius.com/dom/).
