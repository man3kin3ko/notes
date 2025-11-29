First of all, OIDC is built upon OAuth2 protocol. OAuth2 is described in [RFC6749](https://datatracker.ietf.org/doc/html/rfc6749), and OIDC specification is located [on its own site](https://openid.net/specs/openid-connect-core-1_0-final.html). 

OAuth2 introduces the concept of `scopes`, while OIDC *defines specific subset* which is used between an IAM, an application and a user:

- profile
- address
- phone
- email
- offline_access

`Scopes` [may have children attributes](https://openid.net/specs/openid-connect-core-1_0-final.html#Claims), which are called `claims`:
```
profile 
├─ name
├─ family_name
├─ picture
├─ website
├─ gender
├─ birthdate
```

OIDC providers such as Keycloak allows its admins to **define custom scopes and claims**, whcih could be found in `.well-known/openid-configuration`.
## Grant types

OAuth2 defines several authorization schemas which are called `grants`.

- Password grant
- Refresh grant

## Расширения

### OIDC

OIDC Client-Initiated Backchannel Authentication Flow (CIBA) - расширение над OIDC, дающее возможность пользователю авторизовывать ограниченнные сеансы сессий для сотрудников поддержки

### OAuth 2.0

- OAuth 2.0 for Native Apps (RFC 8252) - для МП
- OAuth 2.0 Device Authorization Grant (RFC 8628) - для IoT
- SAML 2.0 Bearer Assertion Profiles for OAuth 2.0 (RFC 7522)
- JSON Web Token (JWT) Profile for OAuth 2.0 Client Authentication and Authorization Grants (RFC 7523)
![](pics/oauth-ext2.png)

![](pics/oauth-ext.png)


## Further reading


There is another protocol which is built on top of OAuth2.0 - User Managed Access 2. OAuth's concepts are also used in [Workload Identity Practices ](https://datatracker.ietf.org/doc/draft-ietf-wimse-workload-identity-practices/) IETF draft as a basic concept in cloud environments. 

OAuth2.1 is a draft standard which removes unsafe practices from the original standard. There is also [GNAP protocol](https://oauth.xyz/) in process.

---

https://openid.net/specs/openid-connect-core-1_0-final.html#rfc.section.16