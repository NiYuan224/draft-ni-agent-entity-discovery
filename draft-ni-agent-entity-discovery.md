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

AI agents are evolving from isolated, single-domain components into collaborative entities interacting across domain boundaries. This shift requires agent-specific credential associations, such as private-domain trust anchors, to be securely exchanged or published for cross-domain authentication. For instance, SPIFFE Federation allows disparate domains to securely exchange their trust bundles via the SPIFFE Federation API, enabling agents to cryptographically verify other agents' credentials.  However, such a federation relies on out-of-band pre-configuration between participating organizations, and is therefore not suitable for dynamic, internet-scale discovery and authentication.

Conversely, existing internet-scale mechanisms such as DNSSEC {{RFC 4033}} and DANE {{RFC 6698}} only secure identities at the domain level. Consequently, a client cannot differentiate among multiple agents that are served from the same domain name, leading to risks of impersonation and lateral movement.

To bridge this gap, this document introduces a mechanism that re-anchors trust at the agent entity level rather than at the domain name level. By extending DNS Resource Records (RRs) to publish agent-specific credential associations, a cross-domain client can retrieve these records during the discovery phase and establish an end-to-end connection directly to the target AI agent, rather than the domain name that hosts it.

The credential association defined in this mechanism provides two functions, inheriting the definition of certificate associations of {{RFC 6698}}: it functions either as a credential trust anchor used to cryptographically verify the signature of an agent's presented credentials (either via certification path validation or JWT signature verification), or as a credential constraint used to perform a direct match against the agent's credential.



# Conventions and Definitions

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
