---
title: "Rich OAuth Error Responses"
abbrev: "Rich OAuth Error Responses"
category: info

docname: draft-watson-oauth-error-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Security"
workgroup: "Web Authorization Protocol"
keyword:
 - oauth
 - error
venue:
  group: "Web Authorization Protocol"
  type: "Working Group"
  mail: "oauth@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/oauth/"
  github: "njwatson32/oauth-error"
  latest: "https://njwatson32.github.io/oauth-error/draft-watson-oauth-error.html"

author:
 -
    fullname: Nicholas Watson
    organization: Google, LLC
    email: nwatson@google.com

normative:
  RFC6749:
  RFC6750:
  RFC7516:
  RFC8414:
  RFC9126:

informative:


--- abstract

Define an error-handling protocol extension for the OAuth 2.0 token endpoint
that allows the authorization server or resource server to specify an extra
parameter on error responses that should be passed through to followup
authorization requests.


--- middle

# Introduction

OAuth 2.0 [RFC6749] and [RFC6750] define several different error codes that
authorization servers and resource servers may return. On token endpoint calls,
most most runtime errors are lumped into invalid_grant, with the rest of the
errors indicating a coding or configuration error by the client. Similarly,
invalid_token covers most runtime errors on resource server calls.

Given the changing security, privacy, and regulatory landscapes since OAuth 2.0
was released, many authorization servers have new types of error conditions,
which may be too complex to adequately describe in a JSON response.

This specification extends OAuth to allow for authorization servers and resource
servers to return an extra parameter which can be passed back to the
authorization server when the user is present in order to provide a richer error
experience which may even allow for recovery by the user.

# Motivating Use Cases

Some errors which may require a richer experience include:

*   User account types which have reduced capabilities (e.g. underage or
    enterprise accounts)
*   Applications which can be disabled or restricted
*   Custom authentication strength requirements not covered by an existing spec
*   TODO more?

In the common case, errors requiring user attention are likely to appear at
authorization time, when the user is present to see and/or address them.
However, in circumstances where data used for access control is changed out of
band, these errors could result in failures at refresh token exchange or access
token usage. In this situation, the error codes defined by the original OAuth
2.0 spec are insufficiently expressive to address these different error
scenarios. The error_description parameter also cannot cover errors that:

*   Need to display richer detail (e.g. explain to user why account is blocked)
*   Contain sensitive data (e.g. account has been flagged as underage)

Because the user is not always be present when the client receives an error from
the authorization or resource server, there needs to be some way to preserve
error state until the user is present again. Simply returning invalid_grant so
that the client reattempts the authorization flow has a few downsides:

*   Some clients may not automatically restart authorization and instead render
    a button to do so. In this case the user has no explanation for why they're
    suddenly signed out of the client.
*   It may result in unnecessary work for the user if they need to complete some
    preliminary auth steps in order to reach the error state (e.g. sign in or
    account selection for authorization servers that support multiple login).
*   invalid_grant is semantically the wrong error code for many error
    conditions.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Error Responses

## Token endpoint

When returning an error response from the token endpoint, the authorization
server MAY return an additional parameter with any error code.

    error_state
        Indicates that the client SHOULD initiate a new authorization grant flow
        MUST add this parameter unmodified as the error_state parameter there.

In this case, `error_state` is a private contract wholly within the
authorization server, so its format requires no specification (though see
considerations below).

## Bearer authentication scheme

This specification defines the following `WWW-Authenticate` auth-param value,
which may be presented alongside any error code.

    error_state
        Indicates that the client SHOULD initiate a new authorization grant flow
        MUST add this parameter unmodified as the error_state parameter there.

If access tokens are validated by the authorization server, `error_state` is a
private contract wholly within the authorization server and requires no
specification (though see considerations below).

If access tokens are validated directly on the resource server (e.g. using an
authorization server public key), then the authorization server will need to
understand the `error_state` generated by the resource server. It is RECOMMENDED
to represent `error_state` as a JWT [RFC7519], and use JSON Web Encryption
[RFC7516] to encrypt it.

### Resource server `error_state` JWT

If using a JWT to represent `error_state`, the standard JWT claims MUST be set
as follows:

*   typ: "jwt+error"
*   iss: Endpoint URL of the resource server endpoint returning the error
*   aud: Authorization server issuer as defined by its metadata endpoint
    [RFC8414]
*   sub: Identifier of the user whose access token triggered the error
*   iat: Issue time
*   nbf: Not before
*   exp: Expiration time, but see considerations below

The JWT will contain other custom claims that can be understood by the
authorization server to display the error, but due to the diverse nature of
potential errors, it's not possible to enumerate them in a spec.

## `error_state` considerations

It is RECOMMENDED that `error_state` be opaque to the client, though clients
MUST NOT introspect the parameter regardless. If `error_state` contains any user
data, it MUST be encrypted. It is RECOMMENDED that `error_state` contains a
timestamp and the authorization server defines a reasonable timeframe in which
it must be used. Given that there may be some human-introduced delay before
`error_state` is sent to the authorization endpoint and the expected lack of
sensitivity of the `error_state` parameter, it is RECOMMENDED that the
expiration timestamp be at least 1 day.

The authorization server and resource server MAY use `error_state` to represent
both recoverable and non-recoverable errors. Examples of recoverable errors
could include step-up auth or re-acknowledgment of updated terms of service.
Examples of non-recoverable errors could include account or API disables where
the authorization server needs to communicate some information to the user.

It is NOT RECOMMENDED to distinguish between recoverable and non-recoverable
errors to the client, as it potentially leaks sensitive information to the
client. (Signals that should be shared for security and abuse reasons should use
the protocols defined in [OpenID SSF].)

## New error codes

This specification additionally defines the `access_denied` error code for
the token endpoint and resource servers, as an error code that may be more
appropriate in some cases where `error_state` is returned.

    access_denied
        Indicates that the request could not be completed, despite being
        well-formed and properly authenticated. This error code MUST be returned
        as an HTTP 403.

# Authorization Grant Request

If the client is not running in a context where it can initiate an authorization
grant flow (e.g. a workflow running on some cloud VM), it MAY transmit the
`error_state` parameter to the system that can.

Authorization servers that support `error_state` MUST support at least one of
the following to avoid sending long encrypted parameters in the URL:

*   HTTP POST requests to the authorization grant flow
*   Pushed authorization requests [RFC9126]

The authorization server MUST ignore an expired `error_state` parameter (though
it is entirely possible that the same error will be re-encountered during a
fresh authorization grant flow).

# Security Considerations

TODO Security
TODO Risk of attaching error_state to URL for different user.
TODO emails
TODO why no whole URL?


# IANA Considerations

TODO IANA


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
