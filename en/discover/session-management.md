---
layout: default
ref: discover-session
title: Session Management
lang: en
---
# Session Management

The Sign in Canada Platform functions as a session management authority that
centrally coordinates users’ sessions across multiple Relying Party (RP)
applications. There are two key features of the platform that support this:
single sign-on, and single logout.

## Single Sign-On

Once a user has successfully used a given credential or trusted digital identity
service to authenticate to one RP, Sign In Canada can allow them to silently
authenticate to other RPs subject to a single sign-on timeout period. The
platform accomplishes this by creating its own session with the user when they
first sign in, and then simply not prompting them to re-authenticate when they
sign in to other RPs within a specified timeout window.

The timeout window for single sign-on starts whenever a user enters their
correct password at a credential service provider (CSP), and ends after a
specified period of time has elapsed. By default this is 20 minutes, but Sign
In Canada relying parties can change this if desired. It is important to note
however that the single sign-on window of the legacy GCKey and Credential Broker
services, which is also 20 minutes, cannot be changed, it can only be disabled
(called forced authentication). In any situation where the "hard-coded" legacy
timeout might come into conflict with a relying party's preferred timeout, the
Sign In Canada platform will automatically synchronize the two, and resolve the
conflict.

_Note that in order for single sign-on to work, a user must use the same
credential or trusted digital identity to access all digital government
services. At present, this not possible since not all Government of Canada
services accept the same credentials. For example, accessing online tax services
provided by CRA is currently not possible using a CKey credential, therefore
users who chose to use a GCKey credential cannot single sign-on to CRA's [My
Account for
Individuals](https://www.canada.ca/en/revenue-agency/services/e-services/e-services-individuals/account-individuals.html)._

## Single Logout

When a user has visited multiple web sites (Relying Party applications) during
the same session, the Sign In Canada Platform has the ability to centrally
coordinate single logout. With single logout, the user only has to click one
logout button on the site they are currently visiting, and Sign In Canada will
then log them out of all the other sites they have visited during their session.

The propagation of logout across all of these sites can use the [OpenID Connect
Front-Channel
Logout](https://openid.net/specs/openid-connect-frontchannel-1_0.html)
mechanism, the [OpenID Connect Back-Channel
Logout](https://openid.net/specs/openid-connect-backchannel-1_0.html) mechanism,
or the [SAML Single Logout
Profile](https://www.oasis-open.org/committees/download.php/56782/sstc-saml-profiles-errata-2.0-wd-07.html)
using front-channel (HTTP-Redirect) and/or back-channel (SOAP) [bindings](https://www.oasis-open.org/committees/download.php/56779/sstc-saml-bindings-errata-2.0-wd-06.pdf).
The Sign In Canada Platform supports all of these.

Both the OpenID Connect (OIDC) and SAML protocols follow the same basic sequence
of events:

1. As a user visits multiple web sites (applications) within the same session,
   the Sign In Canada Platform keeps track of which sites they have visited.
2. When the user clicks a logout button on one of the the sites they are
   visiting, that site's web application sends a logout request to the
   Sign In Canada Platform.
3. Sign In Canada then sends a logout request to all of the other sites
   the user has visited, so they can log the user out.

## Transition and Coexistence

There will be an extended transition period as RP applications connect to Sign
In Canada and disconnect from the legacy Government of Canada Credential
Federation (GCCF) GCKey and Credential Broker services one at a time. During
this transition period, the Sign In Canada Platform will serve as the session
management authority for applications connected to it, while the GCKey and
Credential Broker services serve as the session management authority for
applications connected to them.

This creates a requirement for Sign In Canada to coordinate single logout with
GCKey and CBS, as well as with its own RPs. The platform accomplishes this by
supporting the SAML Single Logout profile as part of its integration, as a SAML
proxying identity provider, with GCKey and CBS. The Sign In Canada platform is
able to coordinate single logout with the legacy GCCF in both directions. That
is, regardless of whether the user logs out of Sign In Canada first or whether they
log out of the GCCF first.

### Scenario 1: A user logs out of Sign In Canada first

The following example scenario illustrates how single logout is achieved in the
case where the user clicks a logout button on a relying party site that is connected to
Sign In Canada.

```plantuml!
skinparam sequenceMessageAlign direction
skinparam responseMessageBelowArrow true
skinparam titleBorderRoundCorner 15
skinparam titleBorderThickness 2
skinparam titleBorderColor red
skinparam titleBackgroundColor Aqua-CadetBlue
title Logout initiated by a Sign In Canada Relying Party
Actor Browser
Participant "Relying Party" as RP
box "Sign In Canada"
Participant "OpenID Provider" as OP
Participant "Acceptance Framework" as Passport
end box
Participant "SAML IDP (CSP)" as IDP
Participant "Other SIC RP(s)" as Other_SIC
Participant "Other GCCF RP(s)" as Other_GCCF
Browser -> RP : logout
RP -> RP : Local logout
RP --> Browser : redirect to end_session endpoint
Browser -> OP : HTTP request to end_session endpoint
group Par
   OP ->> Other_SIC : OIDC back-channel logout request(s)
   Other_SIC -> Other_SIC : local logout
   OP <<-- Other_SIC : logout responses
end
Browser <-- OP : Logout propagation page with iframes
group Par
   Browser ->> Other_SIC : OIDC front-channel logout request(s)
   Other_SIC -> Other_SIC : local logout
   Browser <<-- Other_SIC : HTTP response(s)
   Browser ->> Passport : Logout request
   Browser <<-- Passport : SAML Logout Request
   Browser ->> IDP : SAML Logout Request
   loop
      IDP -> Other_GCCF : SAML Logout Request
      Other_GCCF -> Other_GCCF : local logout
      Other_GCCF --> IDP : SAML Logout Response
   end
   note right of IDP : GCCF logout complete
   Browser <<-- IDP : SAML Logout Response
end
alt an error occurred
   Browser -> OP : Load logout warning page
   Browser <-- RP : Logout warning page
else no errors
   Browser -> RP : Load logout page
   Browser <-- RP : Logout complete page
end
note right of Browser
   Sign In Canada
   Logout Complete
end note
```

The scenario begins when a user clicks a logout button on the relying party site
they have been using. When this happens:

1. The relying party application logs the user out locally first, then it
   redirects the browser to either the Acceptance Platforms's OpenID Connect end_session endpoint.
2. If any relying parties participating in the session support OpenID Connect
   back-channel logout, the Acceptance Platform first sends a logout token to
   all of them, via an HTTP POST to their registered back-channel logout URI(s)
   to trigger the logout actions by those RPs. If there are multiple relying
   parties then these requests are all sent simultaneously, in parallel.
3. Once back-channel logout has been completed, The Acceptance Platform then
   returns an [HTML](https://html.spec.whatwg.org/multipage/) logout propagation
   page to the browser. This page contains a number of
   [iframe](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element)
   elements. The `src` (source) attribute of all but one of the _iframe_
   elements points to the [front-channel logout
   URI](https://openid.net/specs/openid-connect-frontchannel-1_0.html#RPLogout)
   of one of the Sign In Canada relying party sites where the user has logged in
   during their current session. In the diagram, these other sites are
   identified as "Other OIDC RPs". The `src` attribute of the final _iframe_
   points to a URI endpoint of the Acceptance Platform's Acceptance Framework.
4. Upon receiving the propagation page, the browser begins to load all of the of
   the _iframe_s in parallel, by asynchronously sending an HTTP GET request to
   the `src` URI of each _iframe_.
5. As the other relying parties receive these requests, they log the user out
   locally, and then return an HTTP response to the browser.
6. When the Acceptance Platform's Acceptance Framework receives its request, it
   creates a SAML `LogoutRequest` message and then redirects the browser with
   that message (within the _iframe_) to the legacy CSP (GCKey or CBS) that the
   user chose to log in with.
7. The legacy CSP then logs the user out of their credential. If the user has visited
   any sites that are still connected directly to the CSP (instead of to Sign in
   Canada) it will also propagate the logout to all of those sites, one at a time.
8. The CSP then redirects the browser (within the _iframe_) back to the
   Acceptance Platform's Acceptance Framework with a SAML
   `LogoutResponse` message. The Acceptance Framework processes this
   message and then sends an HTTP response back to the browser.
9. The logout propagation page has a global `onerror` event handler that will be
   triggered if any kind of error occurs while all of the _iframe_s are
   propagating the logout to various sites asynchronously.
10. If any of the `iframes` fail or time-out, of if the SAML logout response
   from the legacy CSP indicates an error, then the `onerror`event handler will
   redirect the browser to an [error
   page](https://te-auth.id.tbs-sct.gc.ca/oxauth/partial.htm){:target="_blank"},
   warning the user that they may not be completely signed out and recommending
   that they close their browser.
11. If all of the `iframes` successfully propagate logout to their target site,
    then the browser is redirected to the logout landing page of the site where
    the user initially clicked "Logout" (the relying party's
    `post_logout_redirect_uri`).

### Scenario 2: A user logs out of the GCCF first

The next example scenario illustrates how single logout is achieved in the case
where the user clicks a logout button on a relying party site that is still
connected one of the legacy GCCF credential services (GCKey or CBS).

```plantuml!
skinparam sequenceMessageAlign direction
skinparam responseMessageBelowArrow true
skinparam titleBorderRoundCorner 15
skinparam titleBorderThickness 2
skinparam titleBorderColor red
skinparam titleBackgroundColor Aqua-CadetBlue
title Logout initiated by a GCCF Relying Party
Actor Browser
Participant "SAML Relying Party" as RP
Participant "SAML IDP (CSP)" as IDP
Participant "Other GCCF RP(s)" as Other_GCCF
box "Sign In Canada"
Participant "Acceptance Framework" as Passport
Participant "OpenID Provider" as OP
end box
Participant "Other SIC RP(s)" as Other_SIC
Browser -> RP : logout
RP -> RP : Local logout
Browser <-- RP : HTTP redirect with SAML Logout Request
Browser -> IDP : SAML Logout Request
loop
   IDP -> Other_GCCF : SAML Logout Request
   Other_GCCF -> Other_GCCF : local logout
   Other_GCCF --> IDP : SAML Logout Response
end
note right of IDP : GCCF logout complete
Browser <-- IDP  : HTTP redirect to Sign In Canada
Browser -> Passport : SAML Logout Request
Browser <-- Passport : HTTP Redirect to end_session endpoint
Browser -> OP : OIDC Logout request to end_session endpoint
group Par
   OP ->> Other_SIC : logout request(s)
   Other_SIC -> Other_SIC : local logout
   OP <<-- Other_SIC : logout response(s)
end
Browser <-- OP : Logout propagation page with iframes
group Par
   Browser ->> Other_SIC : OIDC front-channel logout request(s)
   Other_SIC -> Other_SIC : local logout
   Browser <<-- Other_SIC : HTTP response(s)
end
Browser -> Passport : Logout result (Success / Fail)
Browser <-- Passport : SAML Logout Response
note right of Browser
   Sign In Canada
   Logout Complete
end note
Browser -> IDP : SAML Logout Response
alt logout failure
   Browser <-- IDP : SAML Logout Response with "Partial Logout" code
   Browser -> RP : SAML Logout Response
   Browser <-- RP : Logout warning page
else logout sucessful
   Browser <-- IDP : SAML Logout Response with "Success" code
   Browser -> RP : SAML Logout Response
   Browser <-- RP : Logout complete page
end
```

The scenario begins when a user clicks a logout button on the relying party site
they have been using. When this happens:

1. The relying party application logs the user out locally first, then it
   redirects the browser to the legacy CSP's SAML Single Logout endpoint with a
   SAML `LogoutRequest` message.
2. If the user has visited additional sites that are still connected directly to
   the CSP (instead of to Sign In Canada) it will first propagate the logout to
   all of those sites using SAML back-channel (SOAP) logout. This is done one at
   a time, while the user waits, after which the user is now **completely logged
   out of the GCCF**.
3. To log the user out of Sign In Canada, the GCCF CSP then redirects the
   browser to Sign In Canada's Acceptance Framework with a SAML `LogoutRequest`
   message.
4. The Acceptance framework then creates an OpenID Connect logout request and
   redirects the browser to the Acceptance Platform's OpenID Provider (OP)'s
   end_session endpoint.
5. If any relying parties participating in the Acceptance Platform's session
   support back-channel logout, the Acceptance Platform sends a logout token to
   all of them first, via an HTTP POST to their registered back-channel logout
   URI(s). This triggers the logout actions by each of those RPs. If there are
   multiple relying parties then these requests are all sent simultaneously, in
   parallel.
6. Once back-channel logout has been completed, the Acceptance Platform returns
   an [HTML](https://html.spec.whatwg.org/multipage/) logout
   propagation page. This page contains a number of
   [iframe](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#the-iframe-element)
   elements. The `src` (source) attribute of each _iframe_ element points to the
   front-channel logout
   [URI](https://openid.net/specs/openid-connect-frontchannel-1_0.html#RPLogout)
   of one of the relying party sites where the user has logged in during their
   current session. In the diagram, these other sites are identified as "Other
   SIC RPs". The `src` attribute of the final _iframe_ points to a URI endpoint
   of the Acceptance Platform's Acceptance Framework.
7. Upon receiving the propagation page, the browser begins to load all of the of
   the _iframe_s in parallel, by asynchronously sending an HTTP GET request to
   the `src` URI of each _iframe_.
8. As the other relying parties receive these requests, they log the user out
   locally, and then return an HTTP response to the browser.
9. The logout propagation page has a global `onerror` event handler that will be
   triggered if any kind of error occurs while all of the _iframe_s are
   propagating the logout to various sites asynchronously.
10. If any of the `iframes` fail or time-out, the `onerror`event handler will
   redirect the browser to the Acceptance Framework with an HTTP request that
   indicates a logout problem.
    1. The Acceptance Framework will then redirect the browser back
   to the CSP with a SAML `LogoutResponse` message bearing a status code of
   `urn:oasis:names:tc:SAML:2.0:status:Responder`.
    2. The CSP will then in turn redirect the browser back to it's own RP with a
      `LogoutResponse` message bearing a top level status code of
      `urn:oasis:names:tc:SAML:2.0:status:Success` and a second-level status
      code of `urn:oasis:names:tc:SAML:2.0:status:PartialLogout`.
    3. This will cause the SAML RP to display error page, warning the user that
       they may not be completely signed out and recommending that they close
       their browser.
11. If all of the `iframes` successfully propagate logout to their target site,
    the JavaScript code will redirect the browser to the Acceptance Framework
    with an HTTP request that indicates a successful logout.
    1. The Acceptance Framework will then redirect the browser back to the CSP
       with a SAML `LogoutResponse` message bearing a status code of
       `urn:oasis:names:tc:SAML:2.0:status:Success`.
    2. The CSP will then in turn redirect the browser back to it's own RP with a
       `LogoutResponse` message bearing a top level status code of
       `urn:oasis:names:tc:SAML:2.0:status:Success` and no  second-level status
       code.
    3. This will cause the SAML RP to display their usual logout or landing page.
