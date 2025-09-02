
Burp Extensions: GraphQL Raider

GraphQL query should optionally include the `query` operation type for reading operations or `mutation` for modifying operations and arbitrary query name. Mutations are always require input as an argument, meanwhile in queries it can be used to catch up a specific object instead of a group. 

Fingerprint:
- Send `query{__typename}`  to receive `{"data": {"__typename": "query"}}`. `__typename` is a reserved query field which is used to get object type as a string.
- If JSON-based GraphQL engine such as Apollo is used, send `{"query":"query {__typename}"}` or `[{"query":"query {__typename}"}]` instead.
- Notice `query not present` error in response.

Typical vulnerabilities are:
- Schema introspection
- Type suggestions
- Rate limiting bypass via nested queries (query batching)
- CSRF if data is passed via GET request and the server supports `application/x-www-form-urlencoded`
- Any authorization problems, such as IDOR

#### Suggestions bruteforce

See https://github.com/Escape-Technologies/graphql-wordlist
#### Query batching

There are two possible ways to perform query batching.
The first one is simple and predictable for JSON-based endpoints:
```gql
[
	{
	"query":"mutation sendCode1($input1){result}",
	"variables": {
		"input1":"0001"
		}
	},
	{
	"query":"mutation sendCode2($input2){result}",
	"variables": {
		"input2":"0002"
		}
	}
]
```
The second one is based upon using aliases:
```gql
mutation BatchChangeObj($input1: InputObj!, $input2: InputObj!) {
    FOO: changeObj(input: $input1) {
        id
        }
    BAR: changeObj(input: $input2) {
        id
        }
    }
```
Pay attention to how input is provided. Such notation allows you to bruteforce object variables.

#### Language specials

`human(id: "1000")` <- this is called inline object literal. Passing object literals inline is not directly allowed for custom input types (i.e. InputObj above).

Passing variables to fragments:

```
query HeroComparison($first: Int = 3) { <— notice "="
  leftComparison: hero(episode: EMPIRE) {
    ...comparisonFields
  }
```
#### Other ways to obtain valid queries

Inspect JavaScript files (even the minified ones!) to find any valid GraphQL queries

### See also
- https://0xn3va.gitbook.io/cheat-sheets/web-application/graphql-vulnerabilities
- https://lab.wallarm.com/graphql-batching-attack/
- https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html
- https://www.vaadata.com/blog/graphql-api-vulnerabilities-common-attacks-and-security-tips/