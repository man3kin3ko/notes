Burp Extension: WSDLer
#### A simple SOAP message contains:

- **Envelope:** Identifies the XML documents, has a name space and encoding details.
- **Header:** Has header information like content type and character set etc.
- **Body:** Contains the request and response information.
- **Fault:** Errors and status information.

Each HTTP request can contain a header called `SOAP-Action`, which is used to perform an operation defined in its content. It is another entry point for an attacker.
![](https://blog.securelayer7.net/wp-content/uploads/2020/06/soapaction1.jpg)