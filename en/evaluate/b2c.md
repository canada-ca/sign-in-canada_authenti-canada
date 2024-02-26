---
layout: default
ref: integrate-b2c
title: Integrating Azure Active Directory B2C
lang: en
---
# Integrating Azure Active Directory B2C

Government of Canada digital services that leverage [Azure Active Directory
B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview) to
centrally manage user identity data, and/or to authorize access to their
application programming interfaces can configure their B2C tenant to use
Sign In Canada as its authentication service. This allows your users to benefit from
being able to use the same trusted credential to access multiple government services,
while your applications benefit from all of the features provided by Azure Active
Directory B2C.

## Requesting Connectivity to CATE for your B2C tenant

In order to integrate your non-production B2C tenant with the Sign In Canada
[Client Acceptance Test Environment](../discover/cate.html) (CATE), please send
an email to <Signin-AuthentiCanada@tbs-sct.gc.ca> with the following information:

1. The domain name of your B2C tenant. This can either be a default domain name
   such as `https://<tenant-name>.b2clogin.com` or a [custom
   domain](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-domain?pivots=b2c-user-flow).
2. Your B2C [Tenant ID](https://docs.microsoft.com/en-us/azure/active-directory-b2c/tenant-management#get-your-tenant-id)
3. The name of your [user flow or custom policy](https://docs.microsoft.com/en-us/azure/active-directory-b2c/add-sign-up-and-sign-in-policy?pivots=b2c-user-flow)

The Sign in Canada team will use this information to configure your B2C tenant
as a client, and provide you with a client ID and client secret for your tenant.

<div class="alert alert-info">
<p>
You must have a valid myKey credential so that the Sign in Canada team can send you these client credentials via encrypted email.
</p>
</div>

## Configuring Sign In Canada as an Identity Provider

To configure the Sign In Canada CATE environment as your B2C tenant's external
identity provider you can follow [these instructions](https://docs.microsoft.com/en-us/azure/active-directory-b2c/identity-provider-generic-openid-connect?pivots=b2c-user-flow#add-the-identity-provider)
and use the following settings:

|Setting           | Value                                                                   |
|------------------|-------------------------------------------------------------------------|
|Name              |Sign In Canada CATE                                                      |
|Metadata URL      |https://te-auth.id.tbs-sct.gc.ca/oxauth/.well-known/openid-configuration |
|Client ID         |[*as provided by the Sign In Canada team*]                               |
|Client secret     |[*as provided by the Sign In Canada team*]                               |
|Scope             |openid                                                                   |
|Response type     |code                                                                     |
|Response mode     |query                                                                    |
|Domain hint       |[*leave empty*]                                                          |
{: .table .table-bordered .table-condensed}

##### Identity provider claims mappings

|Name           | Claim to map     |
|---------------|------------------|
|User ID        | sub              |
|Display name   | sub              |
|Given name     | [*leave empty*]  |
|Surname        | [*leave empty*]  |
|Email          | [*leave empty*]  |
{: .table .table-bordered .table-condensed}

## Known issues

This is a current list of known interoperability issues between Azure Active
Directory and Sign In Canada, along with workarounds to address them.

### Issue 1: B2C does not include an `id_token_hint` in logout requests sent to an external identity provider

[Documented here](https://docs.microsoft.com/en-us/azure/active-directory-b2c/session-behavior?pivots=b2c-custom-policy#sign-out)

This forces Sign In Canada to rely on its session cookie, however that session
cookie will be blocked by most browsers as a third party cookie if the the B2C
tenant is hosted under a different top-level domain than Sign In Canada.

**Workaround**

The B2C tenant must be configured to use a custom domain that is in the same
top-level domain as the external IDP. In the case of Sign In Canada production
environment, that top level domain is canada.ca, so any production B2C tenant
**must** use a custom domain that is a subdomain under canada.ca.

### Issue 2: B2C does not provide a an OpenID logout endpoint that accepts single logout requests coming from an external OpenID provider

**Workaround**

To log out the B2C session, The Sign In Canada platform can use the
front-channel logout endpoint that B2C provides to its own clients. The B2C
policy/user flow must be configured to not require the id_token_hint, since the
Sign In Canada platform has no way to obtain a B2C client’s ID token.

To log out the session at B2C’s clients, the Sign In Canada platform can send
each of them a logout request directly.

### Issue 3: B2C’s logout endpoint sets an X-Frame-Options header to “deny” which blocks logout requests from Sign In Canada

**Workaround**

A "Modify Response Header" rule can be configured on Azure Front Door to remove
the offending header.

