Keycloak is an open-source Identity and Access Management (IAM) solution.

It supports OIDC authorization protocol as well as SAML.
## Realms

## Identity providers

Each realm can have various IDPs to login from:

```
gitlab
github
facebook
google
linkedin
instagram
microsoft
bitbucket
twitter
openshift-v4
openshift-v3
paypal
stackoverflow
saml
oidc
keycloack-oidc
```

They can be simply enumerated via the following endpoint - `/auth/realms/REALM/broker/endpoint`. Disabled IDPs returns 404, active ones return an error. 

## Misconfiguration checklist

- [ ] Enumerate existing realms
- [ ] Is self-registration is enabled
- [ ] Is it is possible to login via social network IDPs
- [ ] For OIDC IDPS: Is brute-force protection enabled
- [ ] For OIDC IDPS: enumerate client_ids
- [ ] For OIDC IDPS: see if you can claim offline_access in scopes 
### Resources 

- https://csacyber.com/blog/pentesting-keycloak-part-1-identifying-misconfiguration-using-risk-management-tools
- https://csacyber.com/blog/pentesting-keycloak-part-2