
GraphQL query should optionally include the `query` operation type for reading operations or `mutation` for modifying operations and arbitrary query name. Mutations are always require input as an argument, meanwhile in queries it can be used to catch up a specific object instead of a group. 

Fingerprint: 
- Send `query{__typename}`  to receive `{"data": {"__typename": "query"}}`. `__typename` is a reserved query field which is used to get object type as a string.
- Notice `query not present` error in response.

Typical vulnerabilities are:
- Schema introspection
- Type suggestions
- Rate limiting bypass via nested queries
- CSRF if data are passed via GET and the server supports `application/x-www-form-urlencoded`
- Any authorization problems, such as IDOR

### See also

https://0xn3va.gitbook.io/cheat-sheets/web-application/graphql-vulnerabilities