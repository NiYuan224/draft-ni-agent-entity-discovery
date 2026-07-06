---
title: DNS-based Entity-Level Discovery and End-to-End Connection for AI Agents
abbrev: draft-ni-agent-entity-discovery
category: info

docname: draft-ni-agent-entity-discovery-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
# area: AREA
# workgroup: WG Working Group
keyword:
 - next generation
 - unicorn
 - sparkling distributed ledger
venue:
#  group: WG
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
  github: "NiYuan224/draft-ni-agent-entity-discovery"
  latest: "https://NiYuan224.github.io/draft-ni-agent-entity-discovery/draft-ni-agent-entity-discovery.html"

author:
 -
  name: Yuan Ni
  organization: Huawei
  email: niyuan1@huawei.com

 -
  name: Chunchi Peter Liu
  organization: Huawei
  email: liuchunchi@huawei.com

normative:
  RFC6698:
  RFC2119:
  RFC8174:
  RFC1034:
  RFC1035:
  RFC4033:
  RFC5280:
  RFC7250:
  RFC8446:
  RFC7517:
  RFC7519:
  RFC4279:

informative:

...

--- abstract

This document defines a new DNS resource record type, Agent Entity Discovery (AED), to publish agent-specific credential associations, which enables the cross-domain users or agents authenticate, and establish secure, end-to-end connections directly with a private-domain agent entity.



--- middle

# Introduction

AI agents are evolving from isolated, single-domain components into collaborative entities interacting across domain boundaries. This shift requires agent-specific credential associations, such as private-domain trust anchors, to be securely exchanged or published for cross-domain authentication. For instance, SPIFFE Federation allows disparate domains to securely exchange their trust bundles via the bundle endpoint, enabling agents to cryptographically verify other agents' credentials.  However, such a federation relies on out-of-band pre-configuration between participating organizations, and is therefore not suitable for dynamic, internet-scale discovery and authentication.

Conversely, existing internet-scale mechanisms such as DNSSEC {{RFC4033}} and DANE {{RFC6698}} only secure identities at the domain level. Consequently, a client cannot differentiate among multiple agents that are served from the same domain name, leading to risks of impersonation and lateral movement.

To bridge this gap, this document introduces a mechanism that re-anchors trust at the agent entity level rather than at the domain name level. By extending DNS Resource Records (RRs) to publish agent-specific credential associations, a cross-domain client can retrieve these records during the discovery phase and establish an end-to-end connection directly to the target AI agent, rather than the domain name that hosts it.

The credential association defined in this mechanism provides two functions, inheriting the definition of certificate associations of {{RFC6698}}: it functions either as a credential trust anchor used to cryptographically verify the signature of an agent's presented credential (either via certification path validation or JWT signature verification), or as a credential constraint used to perform a direct match against the agent's credential.


# Conventions and Terminology {#term}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.


# Workflow

This section outlines the workflow for establishing a secure, end-to-end connection directly with a private-domain agent entity (see Figure 1).

A private-domain administrator (e.g. a private-domain identity server) publishes agent-specific AED (Agent Entity Discovery) RRs to the DNS server. Then, a client can query and parse these records, and use the retrieved credential associations to verify the target agent during the connection.

~~~
+--------+      +----------+    +--------+      +--------+
| Client |      |DNS Server|    | Agent  |      | Admin  |
+--------+      +----------+    +--------+      +--------+
    |               |               |               |
    |               |1. Register AED                |
    |               |<------------------------------|
    |               |               |               |
    |2.Query AED    |               |               |
    |-------------->|               |               |
    |               |               |               |
    | Answer  AED   |               |               |
    |<--------------|               |               |
    |               |               |               |
    |3.TLS handshake|               |               |
    |------------------------------->               |
    |  (Validate X.509/RPK/PSK etc.)|               |
    |               |               |               |
    |4.Application credential validation (Optional) |
    |<------------------------------>               |
    |               |               |               |
    |               |               |               |
~~~
*Figure 1: Workflow of Agent Entity-Level Discovery and End-to-End Connection*

1. Registration: The private-domain administrator constructs a dedicated QNAME (as defined in {{QNAME}}) for the internal agent and publishes its AED RRs (as defined in {{RR}}) to the DNS server.

2. Discovery: A client sends a DNS query for the agent's specific QNAME with the AED QTYPE, and extracts the agent's credential associations from the recevied AED RRs. The client also obtains the agent's network location (IP address and port) via A/AAAA or SRV RRs. Subsequent TLS handshake messages SHOULD be sent to this obtained address.

3. Connection: The client initiates a direct TLS connection to the agent. During the handshake, the client validates the credential presented by the agent against the credential associations.

4. Application credential validation (Optional): If additional application-layer authentication is required inside the secure tunnel, the agent presents an application-layer token, then the client utilizes the credential associations from the AED RR to verify the token signature.

# Domain Names for AED RR {#QNAME}

The QNAME for an AED RR is constructed by prepending the agent identifier (agent_id) as the left-most label to the base domain name, as shown below:

~~~
<agent_id>.<base_domain>
~~~
*Figure 2: Domain Names for AED*

For example, to request AED RRs for an AI agent identified as "agent-007" hosted at "www.example.com", the QNAME "agent-007.www.example.com" is used.

To maintain flexibility, the internal structure and generation mechanism of the "agent_id" label are left open to deployment-specific choices or future specifications. However, any abstract identifier used MUST be mapped to a valid DNS label. Examples of such identifiers MAY include:

* a WIMSE workload identifier;

* a W3C DID;

* an encoded application-layer path component, etc.



# The AED RR {#RR}

This section defines a new DNS RR type: AED, which is used to associate a set of trust anchors or credential constraints with a specific AI agent entity, thus enabling a client to authenticate that AI agent.

## AED RDATA Wire Format

The RDATA for an AED RR consists of a one-octet Usage field, a one-octet Selector field, a one-octet Matching Type field, and a variable-length Credential Association Data field.

~~~
                        1 1 1 1 1 1 1 1 1 1 2 2 2 2 2 2 2 2 2 2 3 3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |    Usage      |   Selector    |  Matching Type|               /
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               /
   /                                                               /
   /         Credential Association Data (variable)                /
   /                                                               /
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~
*Figure 3: AED RDATA Wire Format*


### The Usage Field

A one-octet value, called "usage", specifies how the association data is to be used for agent authentication. The usages defined in this document are:

0 -- Certificate Trust Anchor: Usage 0 is used to specify a certificate or public key that MUST serve as a domain-specific trust anchor for TLS certification path validation of the presented agent certificate during the TLS handshake.

1 -- JWT Trust Anchor: Usage 1 is used to specify a JSON Web Key (JWK) that MUST serve as a domain-specific trust anchor to verify the signature of an application-layer JSON Web Token (JWT) presented by the agent inside the tunnel.

2 -- Certificate Constraint: Usage 2 is used to specify a certificate or the public key that MUST directly match the certificate presented by the agent during the TLS handshake.

3 -- JWT Constraint:  Usage 3 is used to specify a JWT that MUST directly match the JWT presented by the agent inside the tunnel.

4 -- RPK Constraint: Usage 4 is used to specify a raw public key (RPK) that MUST directly match the RPK presented by the agent during a TLS handshake {{RFC7250}} and be used to verify the agent's possession of the private key via the CertificateVerify signature.

5 -- PSK Constraint: Usage 5 is used to specify a pre-shared key (PSK) identity. The PSK MUST be pre-established out-of-band and stored securely by both the client and the agent so that the client can use the PSK identity to select the correct key during a TLS-PSK handshake {{RFC8446}}. The record MUST contain only the identity string, never the secret key material.



### The Selector Field

A one-octet value, called "selector", specifies which part of the trust anchor or credential is contained in the credential association data.  The selectors defined in this document are:

0 -- Full credential: The full binary or textual structure of the credential or trust anchor.

  * For X.509 certificates (usages 0 and 2): The DER-encoded binary structure of the full X.509 Certificate {{RFC5280}}.

  * For JWT trust anchors (usage 1): The UTF-8 encoded JSON string of the JWK {{RFC7517}}.

  * For JWT constraints (usage 3): The UTF-8 encoded string of the full JWT compact serialization {{RFC7519}}.

  * For PSK constraints (usage 5): The opaque string representing the PSK identity {{RFC8446}}.

1 -- SubjectPublicKeyInfo: DER-encoded binary structure of the public key.

  * For X.509 certificates (usages 0 and 2): The SubjectPublicKeyInfo field within the certificate, encoded in DER binary format {{RFC5280}}.

  * For RPK constraints (usage 4): The DER-encoded RPK structure {{RFC7250}}.


### The Matching Type Field

A one-octet value, called "matching type", specifies how the credential association is presented and matched. The types defined in this document are the same as Section 2.1.3 of {{RFC6698}}:

0 -- Exact match on selected content.

1 -- SHA-256 hash of selected content.

2 -- SHA-512 hash of selected content.


The following constraints apply to this document:

For JWT Trust Anchors (Usage 1): Matching type 1 or 2 is NOT RECOMMENDED unless the companion application protocol explicitly guarantees that the plain-text JWK is delivered alongside the token. Without the plain-text JWK, the client cannot obtain the public key material needed to verify the JWT signature.

For PSK Constraints (Usage 5): The matching type MUST be set to 0. The client requires the raw, unhashed PSK identity to construct the TLS ClientHello message.

### The Credential Association Data Field

This variable-length field contains the credential association data to be matched, constructed according to the preceding fields:

* The Usage field exemplifies the use of the credential association data.

* The Selector field determines whether the credential association data contains the full structure or the SubjectPublicKeyInfo of the credential.

* The Matching Type field determines the data's representation: the raw data, or its cryptographic hash.



# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
