---
title: "Signed Object Template with Multi Signers for the Resource Public Key Infrastructure (RPKI)"
abbrev: "Multi Signers for RPKI Signed Objects"
category: std

docname: draft-guo-sidrops-multi-signers-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: ""
workgroup: "SIDR Operations"
keyword:
 - RPKI
# venue:
#  group: SIDROPS
#  type: Working Group
#  mail: WG@example.com
#  arch: https://example.com/WG
#  github: USER/REPO
#  latest: https://example.com/LATEST

author:
 -
    fullname: Yangfei Guo
    organization: ZGCLab
    email: guoyangfei@zgclab.edu.cn

normative:
    RFC6480:
    RFC6488:
    RFC6811: # BGP ROV

informative:
    RFC7908: # Problem Definition and Classification of BGP Route Leaks
    I-D.ietf-sidrops-aspa-profile:      # ASPA profile
    I-D.ietf-sidrops-aspa-verification: # ASPA verification
    I-D.geng-sidrops-asra-profile:      # ASRA profile


--- abstract

The Resource Public Key Infrastructure (RPKI) system facilitates the authorization of Internet Number Resources, which include a collection of Internet Protocol (IP) addresses and Autonomous System Numbers (ASNs). However, it is important to note that RPKI does not extend its authorization capabilities to the relationships between Autonomous Systems (ASes). This document endeavors to establish these relationships by introducing an option to the template of the RPKI-signed object.

--- middle

# Introduction

The Border Gateway Protocol (BGP) is the mechanism that enables routers to build routing tables across the Internet. However, the original BGP protocol is deficient in several security features, such as the authorization of IP prefix announcements. This lack of security makes BGP susceptible to various attacks, including prefix hijacking and route leaks {{RFC7908}}.

The Resource Public Key Infrastructure (RPKI) system provides the authorization of Internet Number Resources (INRs). At its core, RPKI is a hierarchical Public Key Infrastructure (PKI) that associates INRs, such as Autonomous System Numbers (ASNs) and IP addresses, with public keys through the use of certificates. It is an opt-in service that provides security for Internet routing. 

The Route Origin Authorization (ROA) is one of the most important RPKI-signed objects. ROAs enable a certificate holder to authorize a specific ASN to announce designated IP prefixes. These ROAs are signed using the private key associated with a certificate that encompasses the relevant IP space.

However, the inherent limitation of RPKI is that it can only establish the relationship between a single Autonomous System and specific prefixes; it cannot define or assert the relationships between two Autonomous Systems. Although the Autonomous System Provider Authorization (ASPA) has been proposed {{I-D.ietf-sidrops-aspa-profile}} {{I-D.ietf-sidrops-aspa-verification}}, it remains challenging for a third-party AS to validate the relationship between the issuer and its self-proclaimed providers. If a third-party AS cannot verify the actual relationship between the issuer and its self-proclaimed providers, it may face deception and misjudgment when attempting to validate the AS_PATH using ASPA or similar methodologies, such as Autonomous System Relationship Authorization (ASRA).

This document proposes a modification to the Signed Object Template for the Resource Public Key Infrastructure (RPKI) {{RFC6488}} to better establish the relationships between Autonomous Systems.


# Requirements Language

{::boilerplate bcp14-tagged}


# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
