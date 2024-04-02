%%%
title = "DataRight+: Admission Control Baseline"
area = "Internet"
workgroup = "datarightplus"
submissionType = "independent"


[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-datarightplus-admission-control-baseline-latest"
stream = "independent"
status = "experimental"

date = 2023-04-03T00:00:00Z

[[author]]
initials="S."
surname="Low"
fullname="Stuart Low"
organization="Biza.io"
[author.address]
email = "stuart@biza.io"

[[author]]
initials="B."
surname="Kolera"
fullname="Ben Kolera"
organization="Biza.io"
[author.address]
email = "bkolera@biza.io"
%%%

.# Abstract

The establishment of a shared model of trust is critical to any functioning technology ecosystem, particularly when it relates to the sharing of data and the execution of Consumer specific actions. Traditional models of trust have typically revolved around implicit trust established through bi-lateral arrangements (i.e. legal contracts) between participants. The issue with this approach is that, at scale, it is not possible for all participants to efficiently establish communication with all other participants. This leads to the requirement for a mechanism to establish trust across participants in a way that the business layer of an organisation has confidence in ensuring participant interaction is validated.

.# Notational Conventions

The keywords  "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

This document specifies methods for the following:

- Participant management
- Dynamic Client Registration
- Transport level security

# Terms & Definitions

Certificate Practice Statement
: A statement of the practices that a certification authority (CA) employs in issuing, suspending, revoking, and renewing certificates and providing access to them, in accordance with specific requirements.

CDR
:  Consumer Data Right

Provider
: A Provider is a party that holds consumer data for which sharing is to be conducted and for which their customer, the Consumer, participates in the authorisation process initiated by a Initiator. Please refer to the expanded description of Provider within this document.

Initiator
: A Initiator is a party that receives consumer data from a Provider. This occurs by way of a Software Product.

Ecosystem Authority
: The Ecosystem Authority represents the designated arbiter of trust between Providers, Initiators and the Consumer. Within the Australian Consumer Data Right this is the Australian Competition and Consumer Commission.

Software Product
: A Software Product represents the technology infrastructure, ostensibly a client registration with an authorisation server, operated by a Initiator for the purposes of receiving consumer data.

Software Product Identifier
: TODO

Initiator Base URI
: TODO

Initiator Identifier
: TODO

Initiator Brand Identifier
: TODO

Revocation URI
: TODO

SSA
: Software Statement Assertions

Initiator Arrangement Revocation Endpoint (IARE)
: TODO

Initiator Base URI (IBU)
: TODO

Ecosystem Policy
: TODO

Provider Registration Scope
: A string value as defined by the relevant ecosystem profile.

Initiator Entity Identifier

# Introduction

Describes the operation of an ecosystem and other mechanisms for controlling admission of participants.

This specification describes a technical mechanism for a group of cooperating participants to establish a central source of truth of the permitted participants. In addition, it describes means and methods for participants to discover the existence of others, track the status of these participants and provide metadata of how to describe them to other participants.

*Note:* This specification is heavily influenced by the original definition in the [@!CDS] but avoids ecosystem specific statements in favour of relying on the respective ecosystem and [@!DATARIGHTPLUS-ROSETTA] to provide elaboration.

# Ecosystem Authority

The Ecosystem Authority is considered the primary arbiter of trust within the established ecosystem. In order to provide multiple layers of trust the Ecosystem Authority **SHALL** operate a combination of:

1. Ecosystem Certificate Authority ("Ecosystem CA"): An X.509 Public Key Infrastructure (PKI) compliant certificate authority
2. Ecosystem Directory ("Directory"): A set of APIs delivering metadata of participants
3. Ecosystem Signing Authority ("SSA Authority"): An X.509 JSON Web Signature ([@!JWS]) based signing authority

## General Topology

This specification outlines the requirements for the various components of a data sharing ecosystem.

                                 +-------------------------+
          +--------------------->|      Ecosystem CA       <---------------------------+
          |Verify                +-------------------------+                     Verify|
          |                      +-------------------------+                           |
          |                      |        Directory        |-----------------+         |
          |          +---------->+-------------------------<-------+         |         |
          |          |           +-------------------------+       |         |         |
          |          | +---------|      SSA Authority      |<---+  |         |         |
          |          | |         +-------------------------+    |  |         |         |
          | Provider | | Software                      SSA JWKS |  |Initiator|Trigger  |
          | Metadata | | Statement                     Retrieval|  |Metadata |Directory|
          |          | | Assertion (SSA)                        |  |         |Refresh  |
          |          | |                                        |  |         |         |
          |          | |                                        |  |         |         |
          |          | |                                        |  |         |         |
          |          | v                                        |  |         v         |
       +-----------------------+                               +------------------------+
       |                       |   O------------------------O  |                        |
       |       Initiator       |   |    Mutual TLS Channel  |  |        Provider        |
                               |   |  incl SSA registration |  |                        |
       |                       |<--O------------------------O->|                        |
       +-----------------------+                               +------------------------+

The components provided in this diagram are further stipulated in the sections that follow.

## Ecosystem CA

The Ecosystem Certificate Authority:

1. **SHALL** issue certificates of at least 2048 bits
2. **SHALL** issue certificates using the RSA encryption suite
3. **SHALL** enforce the certificate profile as specified for each participant
4. **SHALL NOT** issue certificates exceeding 365 days
5. **SHALL** issue participant certificates via an Intermediate CA
6. **SHALL** manage, maintain and publish a [@!RFC5280] Certificate Revocation List
7. **SHALL** support [@!RFC6960] OCSP endpoints
8. **SHALL** manage and maintain a suitable Certificate Practice Statement

## Directory

The Directory is a combination of protected and generally available endpoints.

### Authorisation Server

The Ecosystem Directory:
1. **SHALL** make available an OAuth 2.0 [@!RFC6749] authorisation server
1. **SHALL** authenticate the confidential client using `private_key_jwt` specified in section 9 of [@!OIDC-Core]
1. **SHALL** support discovery, as defined in [@!OIDC-Discovery];

The means by which participants acquires their relevant `client_id` is outside the scope of this document.

### Resource Endpoints

The Ecosystem Directory **SHALL** make available an authenticated endpoint secured with a `scope` value of `cdr:register` (or other value specified by the Ecosystem Authority) and accessed via MTLS.

As described further in [@!DATARIGHTPLUS-REDOCLY-ID1] the following authenticated endpoints **SHALL** be made available:

| Resource Server Endpoint                                                          | `x-v` |
|-----------------------------------------------------------------------------------|-------|
| `GET /cdr-register/v1/{industry}/data-holders/brands`                             | `2`   |


In addition, the Ecosystem Directory **SHALL** deliver the following unauthenticated and generally available endpoints, in accordance with [@!DATARIGHTPLUS-REDOCLY-ID1]:

| Resource Server Endpoint                                                          | `x-v` |
|-----------------------------------------------------------------------------------|-------|
| `GET /cdr-register/v1/{industry}/data-holders/brands/summary`                     | `1`   |
| `GET /cdr-register/v1/{industry}/data-holders/status`                             | `1`   |
| `GET /cdr-register/v1/{industry}/data-recipients/brands/software-products/status` | `2`   |
| `GET /cdr-register/v1/{industry}/data-recipients/status`                          | `2`   |
| `GET /cdr-register/v1/{industry}/data-recipients`                                 | `3`   |

## SSA Authority

The SSA Authority issues JSON documents, signed using JWS, containing verified metadata sourced from the Ecosystem Authority.

In addition to `scope` values defined within relevant resource sets the SSA Authority shall also include the Provider Registration Scope.

### SSA Attributes

The unsigned SSA is a JSON document containing the following attributes:

- `iss`: **REQUIRED** Contains the iss (issuer) value of the SSA Authority. Unless specified otherwise this is set to `cdr-register`
- `iat`: **REQUIRED** The time at which the request was issued by the SSA Authority, expressed as seconds since 1970-01-01T00:00:00Z
- `jti`: **REQUIRED** A unique identifier for the document
- `legal_entity_id`: **OPTIONAL** The Initiator Entity Identifier
- `legal_entity_name`: **OPTIONAL** Human-readable name of the Initiator Entity
- `org_id`: **REQUIRED** The Initiator Brand Identifier
- `org_name`: **REQUIRED** Human-readable name of the Initiator Brand
- `client_name`: **REQUIRED** Human-readable name of the Initiator
- `client_description`: **REQUIRED** Human-readable description of the Initiator
- `client_uri`: **REQUIRED** Website address of the Initiator
- `redirect_uris`: **REQUIRED** List of URI for use as `redirect_uri` values in [@!OIDC-Core] authorisation establishments
- `sector_identifier_uri`: **OPTIONAL** URI to be used as an input to the production of Pseudonymous Identifiers as described in section 8 of [@!OIDC-DCR]
- `logo_uri`: **REQUIRED** URI referencing a logo of the Initiator
- `tos_uri`: **OPTIONAL** URI referencing the Terms of Service of the Initiator
- `policy_uri`: **OPTIONAL** URI referencing the Ecosystem Policy of the Initiator
- `jwks_uri`: **REQUIRED** URI referencing the JSON Web Key Set [@!RFC7517] used by the Initiator for authentication purposes
- `revocation_uri`: **REQUIRED** URI referencing the ICARE endpoint as specified within [@!DATARIGHTPLUS-INFOSEC-SHARING-V1], if [@!DATARIGHTPLUS-INFOSEC-SHARING-V1] is supported by the Ecosystem
- `recipient_base_uri`: **REQUIRED** Base URI to use for Initiator provider endpoints
- `software_id`: **REQUIRED** The Initiator Identifier
- `software_roles`: **REQUIRED** Role identifier of the Ecosystem. The only permitted value is currently `data-recipient-software-product`
- `scope`: **REQUIRED** A space-separated list of scope values permitted to be accessed by the Initiator

#### Example

A non-normative example of an unsigned SSA:

```json
{
  "iss": "cdr-register",
  "iat": 1571808111,
  "exp": 2147483646,
  "jti": "3bc205a1ebc943fbb624b14fcb241196",
  "client_name": "Mock Software",
  "client_description": "A mock software product",
  "client_uri": "https://www.mockcompany.com.au",
  "legal_entity_id": "3B0B0A7B-3E7B-4A2C-9497-E357A71D07C7",
  "legal_entity_name": "Mock Company Pty Ltd.",
  "org_id": "3B0B0A7B-3E7B-4A2C-9497-E357A71D07C8",
  "org_name": "Mock Company Brand",
  "redirect_uris": [
    "https://www.mockcompany.com.au/redirects/redirect1",
    "https://www.mockcompany.com.au/redirects/redirect2"
  ],
  "sector_identifier_uri": "https://www.mockcompany.com.au/sector_identifier.json",
  "logo_uri": "https://www.mockcompany.com.au/logos/logo1.png",
  "tos_uri": "https://www.mockcompany.com.au/tos.html",
  "policy_uri": "https://www.mockcompany.com.au/policy.html",
  "jwks_uri": "https://www.mockcompany.com.au/jwks",
  "revocation_uri": "https://www.mockcompany.com.au/revocation",
  "recipient_base_uri": "https://www.mockcompany.com.au",
  "software_id": "740C368F-ECF9-4D29-A2EA-0514A66B0CDE",
  "software_roles": "data-recipient-software-product",
  "scope": "openid profile bank:accounts.basic:read bank:accounts.detail:read bank:transactions:read bank:payees:read bank:regular_payments:read datarightplus:registration"
}


```

### Authorisation Server

The SSA Authority:
1. **SHALL** make available an OAuth 2.0 [@!RFC6749] authorisation server
1. **SHALL** authenticate the confidential client using `private_key_jwt` specified in section 9 of [@!OIDC-Core]
1. **SHALL** support discovery, as defined in OpenID Connect Discovery 1.0 [@!OIDC-Discovery];

The means by which participants acquires their relevant `client_id` is outside the scope of this document.

### Resource Endpoints

#### SSA Retrieval Endpoint

The SSA Authority **SHALL** make available an endpoint for Initiators to download a compliant, time limited and cryptographically signed SSA which describes the Initiator themselves while asserting the authority of the SSA Authority.

This endpoint will be available through an authenticated `GET` request to the path `/cdr-register/v1/{industry}/data-recipients/brands/{initiatorBrandId}/software-products/{initiatorId}/ssa` and **SHALL** return a JWS signed string of the document. Once provided to any participant, the Initiator ID contained within the `software_id` attribute, **SHALL NOT** change for the lifetime of the Initiator.

#### SSA Authority JWKS

The SSA Authority **SHALL** make available the public signing keys, in JSON Web Key Set [@!RFC7517] format, used for signing the SSA at the unauthenticated and generally available endpoint of `GET /cdr-register/v1/jwks`.

# Provider

In order to provide streamlined registration of Initiators the Provider must make available a service to facilitate registration of new OAuth 2.0 clients.

## Authorisation Server

The Provider authorisation server is required to perform a number of prescribed functions.

### Dynamic Client Registration (DCR)

The Provider authorisation server **SHALL** support [@!OIDC-DCR].

In addition, the Provider authorisation server:
1. **SHALL** require SSA documents as part of the Client Registration Request outlined in Section 3.1 of [@!OIDC-DCR] and;
2. **SHALL** validate provided SSA documents as per [SSA Attributes] and;
3. **SHALL** verify the signature of the SSA using the [SSA Authority JWKS] endpoint;
4. **SHALL** support client management endpoints described in Section 2 of [@!RFC7592];
5. **SHALL** authenticate Initiators using `private_key_jwt` and an Ecosystem Authority defined `scope` value (typically `cdr:registration`);
6. **SHALL** require MTLS for all DCR related endpoints;
7. **SHALL NOT** permit multiple client registrations per Initiator Identifier;
8. **SHALL** supply `client_id_issued_at` in addition to the other **REQUIRED** fields outlined in section 3.2 [@!OIDC-DCR];
9. **SHALL** update Initiator statuses, at least every 5 minutes, by polling the `GET /cdr-register/v1/{industry}/data-recipients/brands/software-products/status` endpoint outlined in [Resource Endpoints]

Where there are overlapping attributes between the dynamic registration request and the provided SSA, the content of the SSA **SHALL** take precedence.

# Initiator

Initiators participating in the ecosystem:
1. **SHALL** support endpoint discovery as specified in [@!OIDC-Discovery]
2. **SHALL** support dynamic registration in accordance with [@!OIDC-DCR]
3. **SHALL** retrieve a time-limited SSA, from the [SSA Retrieval Endpoint] and use it as the `software_statement` attribute for dynamic registration requests
4. **SHALL** support the client management provisions contained in Section 2 [@!RFC7592]

# Implementation Considerations

The use of OCSP Stapling within the CDR ecosystem is **NOT RECOMMENDED**.

For MTLS endpoints, all participants **MUST** verify certificates used (client) and presented (server) are current and valid. This implicitly means all parties are required to utilise CRL or OCSP endpoints to maintain confidence in revocation status.

{backmatter}

<reference anchor="OIDC-Core" target="http://openid.net/specs/openid-connect-core-1_0.html"> <front> <title>OpenID Connect Core 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros"> <organization>Google</organization> </author> <author initials="C." surname="Mortimore" fullname="Chuck Mortimore"> <organization>Salesforce</organization> </author> <date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="OIDC-DCR" target="https://openid.net/specs/openid-connect-registration-1_0.html"> <front> <title>OpenID Connect Dynamic Client Registration 1.0 incorporating errata set 2
</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <date day="15" month="Dec" year="2023"/> </front> </reference>

<reference anchor="JWS" target="https://datatracker.ietf.org/doc/html/rfc7515"> <front> <title>JSON Web Token (JWT)</title> <author fullname="M. Jones"> <organization>Microsoft</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author><author fullname="N. Sakimura"> <organization>Nomura Research Institute</organization> </author> <date month="May" year="2015"/></front> </reference>

<reference anchor="OIDC-Discovery" target="https://openid.net/specs/openid-connect-discovery-1_0.html"> <front> <title>OpenID Connect Discovery 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="E." surname="Jay"> <organization>Illumila</organization> </author><date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="DATARIGHTPLUS-INFOSEC-BASELINE" target="https://datarightplus.github.io/datarightplus-infosec-baseline/draft-datarightplus-infosec-baseline.html"> <front><title>DataRight+ Security Profile: Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-INFOSEC-SHARING-V1" target="https://datarightplus.github.io/datarightplus-sharing-arrangement-v1/draft-authors-datarightplus-sharing-arrangement-v1-00"> <front><title>CDR: Sharing Arrangement V1 (00)</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-ROSETTA" target="https://datarightplus.github.io/datarightplus-rosetta/draft-authors-datarightplus-rosetta.html"> <front><title>DataRight+ Rosetta Stone</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="DATARIGHTPLUS-REDOCLY-ID1" target="https://datarightplus.github.io/datarightplus-redocly/?v=ID1"> <front><title>DataRight+: Redocly (ID1)</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author><author initials="B." surname="Kolera" fullname="Ben Kolera"><organization>Biza.io</organization></author>
<author initials="W." surname="Cai" fullname="Wei Cai"><organization>Biza.io</organization></author></front> </reference>
