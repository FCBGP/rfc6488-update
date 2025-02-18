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
    RFC9582: # ROA

informative:
    RFC7908: # Problem Definition and Classification of BGP Route Leaks
    I-D.ietf-sidrops-aspa-profile:      # ASPA profile
    I-D.ietf-sidrops-aspa-verification: # ASPA verification
    I-D.geng-sidrops-asra-profile:      # ASRA profile


--- abstract

The Resource Public Key Infrastructure (RPKI) system facilitates the authorization of Internet Number Resources, which include a collection of Internet Protocol (IP) addresses and Autonomous System Numbers (ASNs). However, it is important to note that RPKI does not extend its authorization capabilities to the relationships between Autonomous Systems (ASes). This document endeavors to establish these relationships by introducing an option to the template of the RPKI-signed object.

--- middle

# Introduction {#Introduction}

The Border Gateway Protocol (BGP) is the mechanism that enables routers to build routing tables across the Internet. However, the original BGP protocol is deficient in several security features, such as the authorization of IP prefix announcements. This lack of security makes BGP susceptible to various attacks, including prefix hijacking and route leaks {{RFC7908}}.

The Resource Public Key Infrastructure (RPKI) system provides the authorization of Internet Number Resources (INRs). At its core, RPKI is a hierarchical Public Key Infrastructure (PKI) that associates INRs, such as Autonomous System Numbers (ASNs) and IP addresses, with public keys through the use of certificates. It is an opt-in service that provides security for Internet routing.

The Route Origin Authorization (ROA) is one of the most important RPKI-signed objects. It enables a certificate holder to authorize a specific ASN to announce designated IP prefixes. The ROA is signed using the private key associated with a certificate that encompasses the relevant IP space.

However, the inherent limitation of RPKI is that it can only establish the relationship between a single Autonomous System and specific prefixes; it cannot define or assert the relationships between two Autonomous Systems. Although the Autonomous System Provider Authorization (ASPA) has been proposed {{I-D.ietf-sidrops-aspa-profile}} {{I-D.ietf-sidrops-aspa-verification}}, it remains challenging for a third-party AS to validate the relationship between the issuer and its self-proclaimed providers. Suppose a third-party AS cannot verify the actual relationship between the issuer and its self-proclaimed providers. In that case, it may face deception and misjudgment when attempting to validate the AS_PATH using ASPA or similar methodologies, such as Autonomous System Relationship Authorization (ASRA).

This document proposes a modification involving multi-signers to the Signed Object Template for the Resource Public Key Infrastructure (RPKI) {{RFC6488}} to establish the relationships between Autonomous Systems better.

## Requirements Language

{::boilerplate bcp14-tagged}

# Signed Object Syntax {#signerInfos}

## signerInfos

In {{Section 2.1 of RFC6488}}, it states, "Additionally, the SignerInfos set MUST contain only a single SignerInfo object." To support multi-signers, this document proposes to relax this restriction to:

"Additionally, the SignerInfos set MUST contain **at least** one SignerInfo object. The first SignerInfo represents the original issuer, and the presence of any subsequent SignerInfos depends on the eContentType and eContent in the encapContentInfo field."

      SignedData ::= SEQUENCE {
              version CMSVersion,
              digestAlgorithms DigestAlgorithmIdentifiers,
              encapContentInfo EncapsulatedContentInfo,
              certificates [0] IMPLICIT CertificateSet OPTIONAL,
              crls [1] IMPLICIT RevocationInfoChoices OPTIONAL,
              signerInfos SignerInfos SIZE(1..MAX) }

### sid

For extra signers, they are not REQUIRED to follow the rule that "the sid MUST be the SubjectKeyIdentifier found in the EE certificate included in the CMS certificates field". However, they MUST ensure that the sid in their signerInfo is associated with the AS Number or AS Identifier present in the eContent field within the RPKI system. Failure to comply with this requirement will result in an invalid SignerInfo.

# Multi-Signers Signing Procedure {#MultiSignersSigningProcedure}

The extra signers MUST first validate the signed object following the procedure defined in {{Section 3 of RFC6488}} and the specific procedure of each signed object. If the signed objects are valid, it can perform a multi-signer signing procedure.

The signing procedure for an extra signer is similar to that of the issuer. The extra signer fills in a SignerInfo field, adds it to the signed object that requires its signature, and then resends it to the RPKI repository, awaiting synchronization.

# Signed Object Validation {#SignedObjectValidation}

This document also follows the principle of the signed object validation procedure defined in {{Section 3 of RFC6488}}. It introduces slight changes to these validation steps.

- In step 1.c of {{Section 3 of RFC6488}}, the SubjectKeyIdentifier field of the certificates field needs to match only the sid field of the first SignerInfo object.
- In step 2 of {{Section 3 of RFC6488}}, the public key of the EE certificate (contained within the CMS signed-data object) can be used to successfully verify the signature on the signed object of the first SignerInfo object. To verify the signatures of extra SignerInfo objects, it is necessary to query the CA certificates to obtain the appropriate public keys.

In this document, the Valid state resulting from the validation of the signed object is separated into three parts:

- **Valid**: This conforms to the original Valid state description. In this case, the signerInfos field contains only one SignerInfo object, indicating that this signed object is the raw signed object and has not been signed by other signers. This represents the common Valid state.

- **PartialValid**: This indicates that only the first SignerInfo object is valid while the other SignerInfo objects are invalid. It may represent an intermediate state of the signed object, which will transition to the TotallyValid State after being signed by all the multiple signers.

- **TotallyValid**: This means that all SignerInfo objects are valid. It is the expected final valid state.

If the above validation procedure and steps defined in {{Section 3 of RFC6488}} indicate that the signed object is Invalid, then the signed object MUST be discarded and treated as though no signed object were present. If all of the conditions above are true, then the signed object may be Valid.

When in use, the router can set different routing strategies to correspond to various verification results. This allows for tailored handling based on the state of the validation, including TotallyValid, PartialValid, Valid, Unknown, or Invalid results to appropriate processes or workflows.


# Definition of Specific Signed Objects {#DefinitionOfSpecificSignedObjects}

It is REQUIRED to specify whether this object supports multi-signers or not in defining a new signed object.

Typically, Route Origin Authorizations (ROAs) {{RFC9582}} do not need multi-signers, but Autonomous System Provider Authorization (ASPA) should support multi-signers.

<!--
# Operational Considerations

It is RECOMMENDED to have only one entity in the eContent field, which ideally means that the `signerInfos` should contain at most two SignerInfo objects.
-->

# Security Considerations

TODO Security


# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.
