%%%
Title = "CDR+ Admission Control: Baseline"
area = "Internet"
workgroup = "cdrplus-parity"

[seriesInfo]
name = "Internet-Draft"
value = "draft-authors-cdrplus-admission-control-baseline-latest"
stream = "IETF"
status = "experimental"

date = 2022-06-27T00:00:00Z

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

Describes the Ecosystem Authority and mechanisms for controlling admission into a Consumer Data Right aligned ecosystem.

.# Notational Conventions

The keywords "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**", "**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",  "**MAY**", and "**OPTIONAL**" in this document are to be interpreted as described in [@!RFC2119].

{mainmatter}

# Scope

This document specifies methods for the following:

- Dynamic Client Registration using trust asserted by the Ecosystem Authority
- Participant management through a centralised API
- Transport level security managed by the Ecosystem Authority

# Terms & Definitions

CDR
:  Consumer Data Right

Data Holder
: A Data Holder is a party that holds consumer data for which sharing is to be conducted and for which their customer, the Consumer, participates in the authorisation process initiated by a Data Recipient. Please refer to the expanded description of Data Holder within this document.

Data Recipient
: A Data Recipient is a party that receives consumer data from a Data Holder. This occurs by way of a Software Product.

Ecosystem Authority
: The Ecosystem Authority represents the designated arbiter of trust between Data Holders, Data Recipients and the Consumer. Within the Australian Consumer Data Right this is the Australian Competition and Consumer Commission.

Software Product
: A Software Product represents the technology infrastructure, ostensibly a client registration with an authorisation server, operated by a Data Recipient for the purposes of receiving consumer data.

Software Product Identifier
: TODO

# Introduction

This specification is currently a placeholder. Please refer to the Register APIs and Security Profile sections of the [@!CDS] until this document evolves further.

## Data Recipient and Accreditation

Ecosystems in alignment with this specification, notably the CDR, perform external validation of participant capabilities, particularly of Data Recipients.

This occurs by way of a combination of Ecosystem Authority verification, external technology audit standards (notably ASAE3150) and legal assertions by Data Recipients carrying higher accreditation as to the suitability of new subordinate Data Recipients.

As a result of this validation Data Recipients are granted an accreditation status which in turn influences the authorisation scopes that are made available to their relevant Software Products. Further detail on this process in the context of the CDR can be found within the CDR Rules and within guidelines published on the CDR website.

The subject of accreditation is not intended to be covered by this specification, nor is the broader concept of a Data Recipient. As a consequence this document focuses primarily on the relationship between Data Holder and Software Product with the Ecosystem Authority providing third party assurance with respect to technical admission control.

## Ecosystem Topology

# Ecosystem Authority

The Ecosystem Authority is considered the primary arbiter of trust within the established ecosystem. In order to provide multiple layers of trust the Ecosystem Authority operates a combination of:

1. A TLS Certificate Authority
2. A set of APIs representing the ecosystem (effectively a "phone book")
3. A JWT signing authority for the purposes of asserting permitted authorisation scopes of Software Products

Within an Australian context the ecosystem is the Consumer Data Right and the Ecosystem Authority is the Register operated by the Australian Competition and Consumer Commission.

## Certificate Authority

The Ecosystem Authority SHALL operate a Certificate Authority (CA), in accordance with ??TLS-Spec??.

The Certificate Authority:

1. **MUST** issue certificates of at least 2048 bits
2. **MUST** issue certificates using the RSA encryption suite
3. **MUST** enforce the certificate profile as specified for each participant
4. **MUST NOT** issue certificates exceeding 365 days
5. **SHOULD** issue participant certificates via an Intermediate CA

<{{yolo.json}}

## Authentication

For endpoints requiring authentication the Ecosystem Authority **SHALL** authenticate the confidential client using `private_key_jwt` specified in section 9 of [@!OIDC-Core].

- Establishment of trust between two parties, notably Data Holder and Data Recipient
- Centralised Certificate Authority to ensure transport level enforcement of ecosystem trust

{backmatter}

<reference anchor="OIDC-Core" target="http://openid.net/specs/openid-connect-core-1_0.html"> <front> <title>OpenID Connect Core 1.0 incorporating errata set 1</title> <author initials="N." surname="Sakimura" fullname="Nat Sakimura"> <organization>NRI</organization> </author> <author initials="J." surname="Bradley" fullname="John Bradley"> <organization>Ping Identity</organization> </author> <author initials="M." surname="Jones" fullname="Mike Jones"> <organization>Microsoft</organization> </author> <author initials="B." surname="de Medeiros" fullname="Breno de Medeiros"> <organization>Google</organization> </author> <author initials="C." surname="Mortimore" fullname="Chuck Mortimore"> <organization>Salesforce</organization> </author> <date day="8" month="Nov" year="2014"/> </front> </reference>

<reference anchor="CDRPLUS-INFOSEC-BASELINE" target="https://cdrplus.github.io/cdrplus-infosec-baseline/draft-cdrplus-infosec-baseline.html"> <front><title>CDR+ Security Profile: Baseline</title><author initials="S." surname="Low" fullname="Stuart Low"><organization>Biza.io</organization></author></front> </reference>

<reference anchor="CDS" target="https://consumerdatastandardsaustralia.github.io/standards"><front><title>Consumer Data Standards (CDS)</title><author><organization>Data Standards Body (Treasury)</organization></author></front> </reference>

