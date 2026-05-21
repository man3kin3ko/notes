
```
Taint flow = source -> sink
```

An example of a source is the `location.search`, `document.referrer` and `document.cookie` properties since they are controllable by an attacker.

## DOM Clobbering

When you can inject HTML tags into page, but XSS is not possible, DOM clobbering technique can be used. By setting `id` or `name` attributes of some elements (embed, form, iframe, image, img, object) we can create objects as global variables in the DOM.
Then, if JavaScript on the page uses a property of a global object, you can just overwrite this object and pass your payload into the previously uncontrollable part of the code.

Some in-the-wild examples:
- [DoS via overwriting methods](https://hackerone.com/reports/1077136)
- [XSS in Gmail AMP4Email](https://web.archive.org/web/20230605031333/https://research.securitum.com/xss-in-amp4email-dom-clobbering/)

And a [cheatsheet](https://tib3rius.com/dom/).

## Cross Document Messaging vulnerabilities

`postMessage` was introduced as part of the HTML5 Web Messaging protocol which loosen SOP policy by allowing communication from different browsing contexts [[Browser security#^de86ca]].

It increases a web application's attack surface and can lead to data leakage if configured to send messages to any sources (wildcard):

```
window.postMessage(sensitiveData, '*');  // Any window can receive
```

Examples:
- [API token leakage in Kaspersky Internet Security via wildcard targetOrigin](https://palant.info/2019/11/25/kaspersky-the-art-of-keeping-your-keys-under-the-door-mat/#from-chrome-and-firefox-extensions)
- [OAuth token is exposed due to attacker-controllable `targetOrigin`](https://hackerone.com/reports/821896)
- [OAuth token is exposed due to attacker-controllable `window.opener`](https://hackerone.com/reports/314814)
- [OAuth token is exposed due to attacker-controllable `targetOrigin` and `window.opener`](https://hackerone.com/reports/826394)

An app can also recieve data from any sender and process it in unsafe way if it doesn't validate `event.origin` in `addEventListener` by its own:

```
//The ‘.’ character within a regular expression means “any character”, therefore we can 
//replace the character in each corresponding position with any other character.For 
//example, messages originating from http://mailXgoogle.com would be permitted.

function receiveMessage(event){

	// Match on mail.google.com and www.google.com
	var regex = /^https*:\/\/(mail|www).google.com$/i;
	
	if (!regex.test(event.origin)){
		return;
	}
	
}
```

Because `Event.data` is transmitted raw, `postMessage`  often serves as a taint source for DOM XSS attacks:

![](https://www.yeswehack.com/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fd51e1jt0%2Fproduction%2Fbbfd0be7c0c01f8b877ff0272a2054e87d929689-700x600.webp%3Fw%3D1200%26q%3D85%26auto%3Dformat&w=1200&q=85)
![](https://www.yeswehack.com/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fd51e1jt0%2Fproduction%2Fe765a0289409308b68e70c65754e550693fc3386-700x280.webp%3Fw%3D1200%26q%3D85%26auto%3Dformat&w=1200&q=85)

`postMessage` can be observed by DevTools under Sources tab, Global Listeners section
![](https://www.sjoerdlangkemper.nl/images/chrome-message-event-listener.png)
Or by using specialized extentions like https://github.com/Sjord/messpostage or https://github.com/benso-io/posta.
### Links 
- https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/11-Client-side_Testing/11-Testing_Web_Messaging
- https://www.yeswehack.com/learn-bug-bounty/introduction-postmessage-vulnerabilities

## DOM XSS

https://hackerone.com/reports/398054
https://hackerone.com/reports/900619
https://hackerone.com/reports/231053
https://hackerone.com/reports/2371019
https://hackerone.com/reports/499030
https://hackerone.com/reports/219957