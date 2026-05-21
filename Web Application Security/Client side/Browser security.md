## Sandbox

Browser sandbox prevents JS from accessing files without the user's permissions, because an XSS attack would be able to escalate to Local File Read, what exactly happend in bugbounty report [Bug Bounty Guest Post: Local File Read via Stored XSS in The Opera Browser](https://blogs.opera.com/security/2021/09/bug-bounty-guest-post-local-file-read-via-stored-xss-in-the-opera-browser/).

It also restricts JS from direct access to system APIs, providing interfaces like Web Bluetooth API or MediaDevices API alongside with Permissions Policy instead.

To protect sites content from hardware attacks like Meltdown and Spectre, Chromium implemented [Site Isolation](https://www.chromium.org/Home/chromium-security/site-isolation/) by changing its architecture. Since then, different web pages, regardless of how they are loaded (including images and iframes), are processed in separate "renderer" processes. Firefox and Safari utilize something similar:

- **Chromium**: Multiprocess with sandboxed renderers, communicate with browser kernel process via "broker" by Mojo IPC mechanisms
- **Firefox**: Electrolysis + sandboxing using OS features (such as seccomp on Linux or Job Objects on Windows)
- **Safari**: App Sandbox + WebContent process separation

## Single Origin Policy

SOP prohibits JS access from different web-pages, based upon its origin.

Origin is defined as ....

Preflight request
## Browsing context

^de86ca

**Browsing contexts** enclude windows, tabs, frames, iframes and popus. Each context is affected by it origin.

A browsing context may be part of a **browsing context group**, which is a set of **browsing contexts** that share common contexts, such as history, cookies, storage mechanisms and so on. The browsing contexts within a group retain references to each other, and therefore, can inspect each other's global objects and post messages to each other.

Contexts with the same origin can communicate via Brodcast Channel API's postMessage.

## Content Security Policy


### Links
- https://aszx87410.github.io/beyond-xss/en/ch1/browser-security-model/
- https://web.archive.org/web/20250701172012/https://medium.com/@JIT_Shellcode/intro-to-sandbox-escapes-47720604a8ec
- https://developer.mozilla.org/en-US/docs/Glossary/Browsing_context