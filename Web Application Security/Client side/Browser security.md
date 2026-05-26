## Sandbox

Browser sandbox prevents JS from accessing files without the user's permissions, because an XSS attack would be able to escalate to Local File Read, what exactly happend in bugbounty report [Bug Bounty Guest Post: Local File Read via Stored XSS in The Opera Browser](https://blogs.opera.com/security/2021/09/bug-bounty-guest-post-local-file-read-via-stored-xss-in-the-opera-browser/).

It also restricts JS from direct access to system APIs, providing interfaces like Web Bluetooth API or MediaDevices API alongside with Permissions Policy instead.

To protect sites content from hardware attacks like Meltdown and Spectre, Chromium implemented [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation/) by changing its architecture. Since then, different web pages, regardless of how they are loaded (including images and iframes), are processed in separate "renderer" processes. Firefox and Safari utilize something similar:

- **Chromium**: Multiprocess with sandboxed renderers, communicate with browser kernel process via "broker" by Mojo IPC mechanisms
- **Firefox**: Electrolysis + sandboxing using OS features (such as seccomp on Linux or Job Objects on Windows)
- **Safari**: App Sandbox + WebContent process separation

## Single Origin Policy

The SOP is designed to restrict scripts running on one origin from _reading_ data from another origin. 

>Origin is defined as scheme, hostname and port combination in [RFC6454](https://datatracker.ietf.org/doc/html/rfc6454)

#### Inherited origin

Scripts executed from pages with an `about:blank` or `javascript:` URI inherit the origin of the document containing that URL, since these types of URLs do not contain information about an origin server. `data:` URIs get a new, empty, security context. Resources loaded via `file:///` scheme are often considered different origin regardless of their actual path on a filesystem. 

#### Cross-origin

When origin differs, a request is treated as *cross-origin*. While reading a cross-origin response is prohibited, embedding a cross-origin resource is permitted. These embeddings are `<img>`, `<form>`'s `action` attribute, `<frame>`, `<iframe>`, `<video>`,`<audio>`, `<link>` or `@import` in CSS context, which are an exact reason why CSRF attacks even exist.

Originally SOP was created by Netscape Navigator in 1995 to restrict cross-origin access to the DOM. If we open a website with different origin via `window.open` or an iframe, we can't change its `document.body.innerHTML` to insert our payload and therefore broke a security context. But a few exceptions exist:
- `location` object is writable, but not readable
- `window.length` is readable, but not writable
- `window.name` is readable and writable and sometimes is used as a workaroud for cross-origin communication
- we can execute `close`, `blur`, and `focus` (useful for triggering XSS in these attributes)
#### Same-origin

If we do share the same origin, for example, with a frame and its parent, we can access global variables: `window.parent.globalVar`.

While `LocalStorage` is shared between multiple windows of the same origin, `SessionStorage` is limited to the current window regardless of the origin.

#### Policy relaxation

- CORS protocol
- [JSONP](https://aszx87410.github.io/beyond-xss/en/ch2/csp-bypass/#bypassing-via-jsonp)
- WebSocket
- postMessage

### CORS

Preflight request is always sent with custom headers, HTTP methods other than GET and POST (CORS-safelisted method) with standard content types.

`CONNECT`, `TRACE`, or `TRACK` are forbidden.
## Cookie security

> Site is defined as scheme and TLD+1 (first level domain after TLD, like `.com` or `.net`)

When site is different, a requst from a browser is called *cross-site*.

## Forbidden headers

Forbidden headers are those that can't be either read (`Set-Cookie`) or set programmatically. Some of the headers, which cannot be set programmaticaly, are `Cookie`, `Origin`, `Sec-*`, `TE` and some others. While `Referer` is considered forbidden by the spec, it's still can be set with `fetch`:

```js
const response = await fetch(request, { body: JSON.stringify({ username: "example" }), referrer: "", })
```

#### Fetch Metadata

Fetch Metadata headers fall into `Sec-*` category. They are set automatically in modern browsers, so a server can decide, was it a legitimate request or a CSRF attack. This security feature is only available for HTTPS websites.

`Sec-Fetch-Dest: image` will be set in case of `<img src="https://website.com/email/change?email=pwned@evil-user.net">`, `empty` in case of fetch request, `iframe` in case of an iframe, etc. [See the full list of the header's values here](https://fetch.spec.whatwg.org/#concept-request-destination).

`Sec-Fetch-Site` - was a request `cross-site`, `same-site` or `same-origin`. The value is `none` when a user entered URL into the address bar.  

`Sec-Fetch-Mode` allows a server to distinguish between requests originating from a user navigating between HTML pages, and requests to load images and other resources. For example, this header would contain `navigate` for top level navigation requests, while `no-cors` is used for loading an image.
## Browsing context

^de86ca

**Browsing contexts** enclude windows, tabs, frames, iframes and popus. Each context is affected by it origin.

A browsing context may be part of a **browsing context group**, which is a set of **browsing contexts** that share common contexts, such as history, cookies, storage mechanisms and so on. The browsing contexts within a group retain references to each other, and therefore, can inspect each other's global objects and post messages to each other.

Contexts with the same origin can communicate via Brodcast Channel API's postMessage.
## Content Security Policy
## Mixed Content blocking

### Links
- https://aszx87410.github.io/beyond-xss/en/ch1/browser-security-model/
- https://web.archive.org/web/20250701172012/https://medium.com/@JIT_Shellcode/intro-to-sandbox-escapes-47720604a8ec
- https://developer.mozilla.org/en-US/docs/Glossary/Browsing_context
- https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Mixed_content
- https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_request_header
- https://developer.mozilla.org/en-US/docs/Glossary/Forbidden_response_header_name
- https://w3c.github.io/webappsec-fetch-metadata/
- https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Same-origin_policy
- https://ieftimov.com/posts/deep-dive-cors-history-how-it-works-best-practices/