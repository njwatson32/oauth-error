---
###
# Internet-Draft Markdown Template
#
# Rename this file from draft-todo-yourname-protocol.md to get started.
# Draft name format is "draft-<yourname>-<workgroup>-<name>.md".
#
# For initial setup, you only need to edit the first block of fields.
# Only "title" needs to be changed; delete "abbrev" if your title is short.
# Any other content can be edited, but be careful not to introduce errors.
# Some fields will be set automatically during setup if they are unchanged.
#
# Don't include "-00" or "-latest" in the filename.
# Labels in the form draft-<yourname>-<workgroup>-<name>-latest are used by
# the tools to refer to the current version; see "docname" for example.
#
# This template uses kramdown-rfc: https://github.com/cabo/kramdown-rfc
# You can replace the entire file if you prefer a different format.
# Change the file extension to match the format (.xml for XML, etc...)
#
###
title: "Rich OAuth Error Responses"
abbrev: "Rich OAuth Error Responses"
category: info

docname: draft-watson-oauth-error-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: AREA
workgroup: WG Working Group
keyword:
 - oauth
 - error
venue:
  group: WG
  type: Working Group
  mail: WG@example.com
  arch: https://example.com/WG
  github: USER/REPO
  latest: https://example.com/LATEST

author:
 -
    fullname: Nicholas Watson
    organization: Google, LLC
    email: nwatson@google.com

normative:

informative:


--- abstract

Define an error-handling protocol extension for the OAuth 2.0 token endpoint
that allows the authorization server or resource server to specify an extra
parameter on error responses that should be passed through to followup
authorization requests.


--- middle

# Introduction

OAuth 2.0 [RFC 6749] and [RFC 6750] define several different error codes that
authorization servers and resource servers may return. On token endpoint calls,
most most runtime errors are lumped into invalid_grant, with the rest of the
errors indicating a coding or configuration error by the client. Similarly,
invalid_token covers most runtime errors on resource server calls.

Given the changing security, privacy, and regulatory landscapes since OAuth 2.0
was released, many authorization servers need more expressive error handling
mechanisms.

The error_description parameter is insufficient and cannot cover errors that:

* Require/allow user action (e.g. reauth required, request access from Enterprise admin)
* Need to display richer detail (e.g. explain to user why account is blocked)
* Contain sensitive data (e.g. account has been flagged as underage)

Furthermore, the user may not always be present when the client receives an error
from the authorization or resource server, so there needs to be some way to
preserve error state until the user is present again.

This specification extends OAuth to allow for authorization servers and resource
servers to return an extra parameter which can be passed back to the authorization
server when the user is present in order to provide a richer error experience
which may even allow for recovery by the user.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Error Responses

## Token endpoint

TODO: also add access_denied to token endpoint? IMO it should be there but it
also feels a little "tacked on" here.

When returning an error response from the token endpoint, the authorization
server MAY return an additional parameter with any error code.

    error_state
        Indicates that the client SHOULD initiate a new authorization grant flow
        and add this parameter unmodified as the error_state parameter there.

## Bearer authentication scheme

This specification defines the following WWW-Authenticate auth-param value,
which may be presented alongside any error code.

    error_state
        Indicates that the client SHOULD initiate a new authorization grant flow
        and add this parameter unmodified as the error_state parameter there.

## error_state considerations

If the client is not running in a context where it can initiate an authorization
grant flow (e.g. a workflow running on some cloud VM), it may transmit the
error_state parameter to the system that can.

It is RECOMMENDED that error_state be opaque to the client, though clients MUST
NOT introspect the parameter regardless. If error_state contains any user data,
it MUST be encrypted. It is RECOMMENDED that error_state contains a timestamp
and the authorization server defines a reasonable timeframe in
which it must be used. The authorization server MUST ignore an expired error_state
parameter (though it is entirely possible that the same error will be re-encountered
during a fresh authorization grant flow).

The authorization server and resource server may use error_state to represent both recoverable and
non-recoverable errors. Examples of recoverable errors could include step-up
auth or re-acknowledgment of updated terms of service. Examples of non-recoverable
errors could include account or API disables where the authorization server needs
to communicate some information to the user.

It is NOT RECOMMENDED to distinguish between recoverable and non-recoverable
errors to the client, as it potentially leaks sensitive information to the client.
(Signals that should be shared for security and abuse reasons should use the
protocols defined in [OpenID SSF].)

[The same normative language applies here too. Do I copy-paste?]

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