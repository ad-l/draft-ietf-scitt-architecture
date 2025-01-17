---
v: 3

title: An Architecture for Trustworthy and Transparent Digital Supply Chains
abbrev: SCITT Architecture
docname: draft-ietf-scitt-architecture-latest

area: Security
wg: SCITT
kw: Internet-Draft
cat: std
consensus: yes
submissiontype: IETF

kramdown_options:
  auto_id_prefix: sec-

venue:
  group: SCITT
  mail: scitt@ietf.org
  github: ietf-wg-scitt/draft-ietf-scitt-architecture

author:
- name: Henk Birkholz
  org: Fraunhofer SIT
  abbrev: Fraunhofer SIT
  email: henk.birkholz@sit.fraunhofer.de
  street: Rheinstrasse 75
  code: '64295'
  city: Darmstadt
  country: Germany
- name: Antoine Delignat-Lavaud
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: antdl@microsoft.com
  country: UK
- name: Cedric Fournet
  organization: Microsoft Research
  street: 21 Station Road
  code: 'CB1 2FB'
  city: Cambridge
  email: fournet@microsoft.com
  country: UK
- name: Yogesh Deshpande
  organization: ARM
  street: 110 Fulbourn Road
  code: 'CB1 9NJ'
  city: Cambridge
  email: yogesh.deshpande@arm.com
  country: UK
- ins: S. Lasker
  name: Steve Lasker
  org: DataTrails
  email: steve.lasker@datatrails.ai
  code: '98199'
  city: Seattle
  region: WA
  country: United States

contributor:
  - ins: O. Steele
    name: Orie Steele
    organization: Transmute
    email: orie@transmute.industries
    country: United States
    contribution: >
      Orie contributed to improving the generalization of COSE building blocks and document consistency.

normative:
  RFC2046:
  RFC6838:
  RFC9052: COSE
  RFC9360:
  COSWID: RFC9393

  CWT_CLAIMS_COSE: I-D.ietf-cose-cwt-claims-in-headers
  IANA.cwt:
  IANA.media-types:
  IANA.named-information:
  RFC6570: URITemplate
  RFC4648: Base64Url

informative:

  I-D.draft-ietf-cose-merkle-tree-proofs: COMETRE

  RFC2397: DataURLs
  RFC6024:
  RFC8141: URNs
  RFC9162: CT
  RFC9334: rats-arch
  CWT_CLAIMS:
    target: https://www.iana.org/assignments/cwt/cwt.xhtml
    title: CBOR Web Token (CWT) Claims

  CycloneDX:
    target: https://cyclonedx.org/specification/overview/
    title: CycloneDX

  EQUIVOCATION: DOI.10.1145/1323293.1294280

  in-toto:
    target: https://in-toto.io/
    title: in-toto

  MERKLE: DOI.10.1007/3-540-48184-2_32

  PBFT: DOI.10.1145/571637.571640

  SLSA:
    target: https://slsa.dev/
    title: SLSA

  SPDX-CBOR:
    target: https://spdx.dev/use/specifications/
    title: SPDX Specification

  SPDX-JSON:
    target: https://spdx.dev/use/specifications/
    title: SPDX Specification

  SWID:
    target: https://csrc.nist.gov/Projects/Software-Identification-SWID/guidelines
    title: SWID Specification

  URLs:
    target: https://url.spec.whatwg.org/
    title: URL Living Standard

  I-D.draft-ietf-core-href: CURIs

  KEY-MANAGEMENT: DOI.10.6028/NIST.SP.800-57pt2r1

--- abstract

Traceability of physical and digital Artifacts in supply chains is a long-standing, but increasingly serious security concern.
The rise in popularity of verifiable data structures as a mechanism to make actors more accountable for breaching their compliance promises has found some successful applications to specific use cases (such as the supply chain for digital certificates), but lacks a generic and scalable architecture that can address a wider range of use cases.

This document defines a generic, interoperable and scalable architecture to enable transparency across any supply chain with minimum adoption barriers.
It provides flexibility, enabling interoperability across different implementations of Transparency Services with various auditing and compliance requirements.
Issuers can register their Signed Statements on any Transparency Service, with the guarantee that all Auditors and Verifiers will be able to verify them.

--- middle

# Introduction

This document describes the scalable, flexible, and decentralized SCITT architecture.
Its goal is to enhance auditability and accountability across supply chains.

In supply chains, downstream artifacts are built upon upstream artifacts.
The complexity of traceability and quality control for these supply chains increases with the number of artifacts and parties contributing to them.
There are many parties who publish information about artifacts:
For example, the original manufacturer may provide information about the state of the artifact when it left the factory.
The shipping company may add information about the transport environment of the artifact.
Compliance auditors may provide information about their compliance assessment of the artifact.
Security companies may publish vulnerability information about an artifact.
Some of these parties may publish information about their analysis or use of an artifact.

SCITT provides a way for Relying Parties to obtain this information in a way that is "transparent", that is, parties cannot lie about the information that they publish without it being detected.
SCITT achieves this by having producers publish information in a Transparency Service, where Relying Parties can check the information.

## Requirements Notation

{::boilerplate bcp14-tagged}

# Terminology {#terminology}

The terms defined in this section have special meaning in the context of Supply Chain Integrity, Transparency, and Trust, which are used throughout this document.
When used in text, the corresponding terms are capitalized.
To ensure readability, only a core set of terms is included in this section.

**Editor's Note:**: *The label "394" is expected to be reserved by this document, in the COSE Header Parameters Registry.*

The terms "header", "payload", and "to-be-signed bytes" are defined in {{RFC9052}}.

Append-only Log (Ledger):

: the verifiable append-only data structure that stores Signed Statements in a Transparency Service, often referred to by the synonym, Ledger.
SCITT supports multiple Ledger and Receipt formats to accommodate different Transparency Service implementations, and the proof types associated with different types of Append-only Logs.

Artifact:

: a physical or non-physical item that is moving along a supply chain.

Auditor:

: an entity that checks the correctness and consistency of all Transparent Statements issued by a Transparency Service.

Client:

: an application making protected Transparency Service resource requests on behalf of the resource owner and with its authorization.
The term "Client" does not imply any particular implementation characteristics (e.g., whether the application executes on a server, a desktop, or other devices).

Envelope:

: metadata, created by the Issuer to produce a Signed Statement.
The Envelope contains the identity of the Issuer and information about the Artifact, enabling Transparency Service Registration Policies to validate the Signed Statement.
A Signed Statement is a COSE Envelope wrapped around a Statement, binding the metadata in the Envelope to the Statement.
In COSE, an Envelope consists of a protected header (included in the Issuer's signature) and an unprotected header (not included in the Issuer's signature).

Equivocation:

: a state where it is possible for a Transparency Service to provide different views of its Append-only log to Verifiers about the same Artifact {{EQUIVOCATION}}.

Feed:

: A collection of Receipts, as recorded by the Transparency Service, based on filtering of properties from the envelope including, but not limited to the `sub` field of the `CWT_Claims`.
Verifiers may use the Feed to ensure completeness and Non-equivocation in supply chain evidence by identifying all Transparent Statements linked to the Artifact they are evaluating.

Issuer:

: organizations, stakeholders, and users involved in creating or attesting to supply chain artifacts, releasing authentic Statements to a definable set of peers.
An Issuer may be the owner or author of Artifacts, or an independent third party such as an auditor, reviewer or an endorser.

Non-equivocation:

: a state where it is impossible for a Transparency Service to provide different views of its append-only log to Verifiers about the same Artifact.
Over time, an Issuer may register new Signed Statements about an Artifact in a Transparency Service with new information. However, the consistency of a collection of Signed Statements about the Artifact can be checked by all Verifiers.

Receipt:

: a Receipt is a cryptographic proof that a Signed Statement is recorded in the Append-only Log.
Receipts are based on Signed Inclusion Proofs as described in COSE Signed Merkle Tree Proofs {{-COMETRE}}.
Receipts can be built on different verifiable data structures, not just binary merkle trees.
Receipts consist of Transparency Service-specific inclusion proofs, a signature by the Transparency Service of the state of the Append-only Log, and additional metadata (contained in the signature's protected headers) to assist in auditing.

Registration:

: the process of submitting a Signed Statement to a Transparency Service, applying the Transparency Service's Registration Policy, adding to the Append-only Log, and producing a Receipt.

Registration Policy:

: the pre-condition enforced by the Transparency Service before registering a Signed Statement, based on information in the non-opaque header and metadata contained in its COSE Envelope.

Signed Statement:

: an identifiable and non-repudiable Statement about an Artifact signed by an Issuer.
In SCITT, Signed Statements are encoded as COSE signed objects; the `payload` of the COSE structure contains the issued Statement.

Statement:

: any serializable information about an Artifact.
To help interpretation of Statements, they must be tagged with a media type (as specified in {{RFC6838}}).
A Statement may represent a Software Bill Of Materials (SBOM) that lists the ingredients of a software Artifact, an endorsement or attestation about an Artifact, indicate the End of Life (EOL), redirection to a newer version,  or any content an Issuer wishes to publish about an Artifact.
The additional Statements about an Artifact are correlated by the Subject defined in the {{CWT_CLAIMS}} protected header.
The Statement is considered opaque to Transparency Service, and MAY be encrypted.

Subject:

: This term has the same definition as in RFC8392, which relies on the definition in RFC7519.
The `sub` (subject) claim identifies the principal that is the subject of the CWT.
The claims in a CWT are normally statements about the subject.
In SCITT, `sub` identifies the entity about which statements, and receipts are made.
The subject value MUST either be scoped to be locally unique in the context of the Issuer or be globally unique.
The processing of this claim is generally application specific.
The `sub` value is a case-sensitive string containing a StringOrURI value.
Issuer's use `sub` to identify the entity about which they are making Signed Statements.
Transparency Services use `sub` to identify the entity about which they are issuing a Receipt.

Transparency Service:

: an entity that maintains and extends the Append-only Log, and endorses its state.
A Transparency Service can be a complex distributed system, and SCITT requires the Transparency Service to provide many security guarantees about its Append-only Log.
The identity of a Transparency Service is captured by a public key that must be known by Verifiers in order to validate Receipts.

Transparent Statement:

: a Signed Statement that is augmented with a Receipt created via Registration in a Transparency Service.
The receipt is stored in the unprotected header of COSE Envelope of the Signed Statement.
A Transparent Statement remains a valid Signed Statement, and may be registered again in a different Transparency Service.

Verifier:

: organizations, stakeholders, and users involved in validating supply chain Artifacts.
Verifiers analyze Transparent Statements, verifying their proofs and inspecting the Statement payload, either before using corresponding Artifacts, or later to audit an Artifact's provenance on the supply chain.

{: #mybody}

# Definition of Transparency

In this document, the definition of transparency is intended to build over abstract notions of Append-only Logs and Receipts.
Existing transparency systems such as Certificate Transparency are instances of this definition.

A Signed Statement is an identifiable and non-repudiable Statement made by an Issuer.
The Issuer selects additional metadata and attaches a proof of endorsement (in most cases, a signature) using the identity key of the Issuer that binds the Statement and its metadata.
Signed Statements can be made transparent by attaching a proof of Registration by a Transparency Service, in the form of a Receipt that countersigns the Signed Statement and witnesses its inclusion in the Append-only Log of a Transparency Service.
By extension, the document may say an Artifact (a firmware binary) is transparent if it comes with one or more Transparent Statements from its author or owner, though the context should make it clear what type of Signed Statements is expected for a given Artifact.

Transparency does not prevent dishonest or compromised Issuers, but it holds them accountable.
Any Artifact that may be verified, is subject to scrutiny and auditing by other parties.
The Transparency Service provides a history of Statements, which may be made by multiple Issuers, enabling Verifiers to make informed decisions.

Transparency is implemented by providing a consistent, append-only, cryptographically verifiable, publicly available record of entries.
A SCITT instance is referred to as a Transparency Service.
Implementations of Transparency Services may protect their Append-only Log using a combination of trusted hardware, replication and consensus protocols, and cryptographic evidence.
A Receipt is an offline, universally-verifiable proof that an entry is recorded in the Append-only Log.
Receipts do not expire, but it is possible to append new entries (more recent Signed Statements) that subsume older entries (less recent Signed Statements).

Anyone with access to the Transparency Service can independently verify its consistency and review the complete list of Transparent Statements registered by each Issuer.
However, the Registrations on a separate Transparency Service is generally disjoint, though it is possible to take a Transparent Statement (i.e. a Signed Statement with a Receipt in its unprotected header, from a from the first Transparency Service ) and register it on another Transparency Service, where the second receipt will be over the first Receipt in the unprotected header.

Reputable Issuers are thus incentivized to carefully review their Statements before signing them to produce Signed Statements.
Similarly, reputable Transparency Services are incentivized to secure their Append-only Log, as any inconsistency can easily be pinpointed by any Auditor with read access to the Transparency Service.

The building blocks defined in SCITT are intended to support applications in any supply chain that produces or relies upon digital artifacts, from the build and supply of software and IoT devices to advanced manufacturing and food supply.

SCITT is a generalization of Certificate Transparency {{-CT}}, which can be interpreted as a transparency architecture for the supply chain of X.509 certificates.
Considering CT in terms of SCITT:

- CAs (Issuers) sign X.509 TBSCertificates (Artifacts) to produce X.509 certificates (Signed Statements)
- CAs submit the certificates to one or more CT logs (Transparency Services)
- CT logs produce Signed Certificate Timestamps (Transparent Statements)
- Signed Certificate Timestamps are checked by Verifiers
- The Append-only Log can be checked by Auditors

# Architecture Overview

The SCITT architecture consists of a very loose federation of Transparency Services, and a set of common formats and protocols for issuing and registering Signed Statements, and auditing Transparent Statements.

In order to accommodate as many Transparency Service implementations as possible, this document only specifies the format of Signed Statements (which must be used by all Issuers) and a very thin wrapper format for Receipts, which specifies the Transparency Service identity and the agility parameters for the Signed Inclusion Proofs.
Most of the details of the Receipt's contents are specified in the COSE Signed Merkle Tree Proof document {{-COMETRE}}.

~~~aasvg
  .----------.
 |  Artifact  |
  '----+-----'
       v
  .----+----.  .----------.   Identifiers
 | Statement ||  Envelope  +<-------------.
  '----+----'  '-----+----'                |
       |             |            .--------+--.
        '----. .----'            |  Identity   |
              |                  |  Documents  +---.
              v                   '------+----'     |
         .----+----.                     |          |
        |  Signed   |    COSE Signing    |          |
        | Statement +<-------------------+          |
         '----+----'                     |          |
              |                 +--------+------+   |
           .-' '--------------->+ Transparency  |   |
          |   .--------.        |               |   |
          |  | Receipt  +<------+  Service      +-+ |
          |  |          +.      +--+------------+ | |
          |   '-+------'  |        | Transparency | |
          |     | Receipt +<-------+              | |
          |      '------+'         | Service      | |
           '-------. .-'           +------------+-+ |
                    |                           |   |
                    v                           |   |
              .-----+-----.                     |   |
             | Transparent |                    |   |
             |  Statement  |                    |   |
              '-----+-----'                     |   |
                    |                           |   |
                    |'-------.     .------------)--'
                    |         |   |             |
                    |         v   v             |
                    |    .----+---+-----------. |
                    |   / Verify Transparent /  |
                    |  /      Statement     /   |
                    | '--------------------'    |
                    v                           v
           .--------+---------.      .----------+-----.
          / Collect Receipts /      /   Replay Log   /
         '------------------'      '----------------'
~~~

This section describes at a high level, the three main roles and associated processes in SCITT: Issuers and Signed Statements, Transparency Service and the Signed Statement Registration process, as well as Verifiers of the Transparent Statements and the Receipt validation process.

## Transparency Service

Transparency Services MUST feature an Append-only Log.
The Append-only Log is the verifiable data structure that registers Signed Statements and supports the production of Receipts.

All Transparency Services MUST expose APIs for the registration of Signed Statements and issuance of Receipts.

Transparency Services MAY support additional APIs for auditing, for instance, to query the history of Signed Statements.

Typically a Transparency Service has a single Issuer identity which is present in the `iss` claim of Receipts for that service.

Multi-tenant support can be enabled through the use of identifiers in the `iss` claim, for example, `ts.example` may have a distinct Issuer identity for each sub domain, such as `customer1.ts.example` and `customer2.ts.example`.

### Registration Policies

Registration Policies refer to additional checks over and above the Mandatory Registration Checks that are performed before a Signed Statement is accepted to be registered to the Append-only Log.

Transparency Services MUST maintain Registration Policies which govern whether or not a given Signed Statement is eligible for registration.

Registration Policies MUST be made transparent and available to all clients of the Transparency Service by registering them as Signed Statements on the Append-only Log.



This specification leaves implementation, encoding and documentation of Registration Policies to the operator of the Transparency Service.

#### Mandatory Registration Checks

Transparency Services MUST, at a minimum, perform the following checks before registering a Signed Statement:

- Authenticate the Issuer of the Signed Statement

The Transparency Service MUST authenticate the Issuer of Signed Statements by validating the COSE signature and checking the identity of the issuer through one of its configured trust anchors, using the `x5t` and `kid` headers in the protected header as hints. For instance, for X.509 signed claims the Transparency Service must validate a complete certificate chain from the certificate identified by `x5t` to one of the trusted root authority certificate of the Transparency Service.
The public key is used to verify digital signatures, and the associated data is used to constrain the types of information for which the trust anchor is authoritative."


Before a Signed Statement is registered, the trust anchor used to verify its Issuer MUST be registered with the Transparency Service.

### Initialization and bootstrapping {#ts-initialization}

Since a Registration Policy is required prior to the registration of any Signed Statements, a means is required to configure the first Registration Policy that is not the standard issuance of a Signed Statement.
Transparency Services MUST support at least one of these methods:

- A built-in default Registration Policy
- Acceptance of a first Signed Statement whose payload is a valid Registration Policy, without performing registration checks
- An out-of-band authenticated management interface

### Append-only Log

The security properties of the Append-only Log are determined by the choice of the verifiable data structure used to produce Receipts.

In addition to Receipts, some verifiable data structures might support additional proof types, such as proofs of consistency, or proofs of non inclusion.

Specific verifiable data structures, such those describes in {{-CT}} and {{-COMETRE}} are out of scope for this document.

### Adjacent Services

Transparency Services can be deployed along side other database or object storage technologies.
For example, a Transparency Service that is supporting a software package management system, might be referenced from the APIs exposed for package management.
Providing an ability to request a fresh receipt for a given software package, or to request a list of Signed Statements and Artifacts associated with a software package.

## Signed Statements

This specification prioritizes conformance to {{RFC9052}} and its required and optional properties.
Profiles and implementation specific choices should be used to determine admissability of conforming messages.
This specification is left intentionally open to allow implementations to make the restrictions that make the most sense for their operational use cases.

At least one identifier for an identity document MUST be included in the protected header of the COSE envelope, either `x5t` or `kid`.

- Support for `x5t` is mandatory to implement.
- Support for `kid` is optional.

When `x5t` is present, `iss` MUST be a string with a value between 1 and 8192 characters in length that fits the regular expression of a distinguished name.

The mechanisms for how Transparency Services obtain identity documents is out-of-scope of this document.

The `kid` header parameter MUST be present when the `x5t` header parameter is not present.
Key discovery protocols are out-of-scope of this document.

The protected header of a Signed Statement and a Receipt MUST include the `CWT Claims` header parameter as specified in {{Section 2 of CWT_CLAIMS_COSE}}.
The `CWT Claims` value MUST include the `Issuer Claim` (Claim label 1) and the `Subject Claim` (Claim label 2) {{IANA.cwt}}.

A Receipt is a Signed Statement, (cose-sign1), with addition claims in its protected header related to verifying the inclusion proof in its unprotected header. See {{-COMETRE}}.

{{fig-signed-statement-cddl}} illustrates a normative CDDL definition for of the protected header for Signed Statements and Receipts.

Everything that is optional in the following CDDL can potentially be discovered out of band and Registration Policies are not assured on the presence of these optional fields.
A Registration Policy that requires an optional field to be present MUST reject any Signed Statements or Receipts that are invalid according to the policy.

~~~ cddl

Signed_Statement = #6.18(COSE_Sign1)
Receipt = #6.18(COSE_Sign1)

COSE_Sign1 = [
  protected   : bstr .cbor Protected_Header,
  unprotected : Unprotected_Header,
  payload     : bstr / nil,
  signature   : bstr
]

Protected_Header = {
  &(CWT_Claims: 15) => CWT_Claims
  ? &(alg: 1) => int
  ? &(content_type: 3) => tstr / uint
  ? &(kid: 4) => bstr
  ? &(x5t: 34) => COSE_CertHash
  * int => any
}

CWT_Claims = {
  &(iss: 1) => tstr
  &(sub: 2) => tstr
  * int => any
}

Unprotected_Header = {
  ? &(receipts: 394)  => [+ Receipt]
}

~~~
{: #fig-signed-statement-cddl title="CDDL definition for Signed Statements and Receipts"}

~~~~ cbor-diag
{                                   / Protected                     /
  1: -7,                            / Algorithm                     /
  3: application/example+json,      / Content type                  /
  4: h'50685f55...50523255',        / Key identifier                /
  15: {                             / CWT Claims                    /
    1: software.vendor.example,     / Issuer                        /
    2: vendor.product.example,      / Subject                       /
  }
}
~~~~
{: #fig-signed-statement-protected-header-edn title="CBOR Extended Diagnostic Notation example of a Signed Statement's Protected Header"}

~~~~ cbor-diag
18(                                 / COSE Sign 1                   /
    [
      h'a4012603...6d706c65',       / Protected                     /
      {},                           / Unprotected                   /
      nil,                          / Detached payload              /
      h'79ada558...3a28bae4'        / Signature                     /
    ]
)
~~~~
{: #fig-signed-statement-edn title="CBOR Extended Diagnostic Notation example of a Signed Statement"}

{{fig-signed-statement-edn}} illustrates a payload that is detached.
This is to support very large supply chain artifacts, and to ensure that Transparent Statements can integrate with existing file systems.

There are many types of Statements (such as SBOMs, malware scans, audit reports, policy definitions) that Issuers may want to turn into Signed Statements.
An Issuer must first decide on a suitable format (`3`: payload type) to serialize the Statement payload.
For a software supply chain, payloads describing the software artifacts may include:

- {{COSWID}}
- {{CycloneDX}}
- {{in-toto}}
- {{SPDX-CBOR}}
- {{SPDX-JSON}}
- {{SLSA}}
- {{SWID}}

Once all the Envelope headers are set, an Issuer MUST use a standard COSE implementation to produce an appropriately serialized Signed Statement (the SCITT tag of `COSE_Sign1_Tagged` is outside the scope of COSE, and used to indicate that a signed object is a Signed Statement).

Issuers may produce Signed Statements about different Artifacts under the same Identity.
Issuers and Verifiers must be able to recognize the Artifact to which the statements pertain by looking at the Signed Statement.
The `iss` and `sub` claims, within the CWT_Claims protected header, are used to identify the Artifact the statement pertains to.

See Subject under {{terminology}} Terminology.

Issuers MAY use different signing keys (identified by `kid` in the resolved key manifest) for different Artifacts, or sign all Signed Statements under the same key.

An Issuer can make multiple Statements about the same Artifact.
For example, an Issuer can make amended Statements about the same Artifact as their view changes over time.

Multiple Issuers can make different, even conflicting Statements, about the same Artifact.
Verifiers can choose which Issuers they trust.

Multiple Issuers can make the same Statement about a single Artifact, affirming multiple Issuers agree.

### Registration

To register a Signed Statement, the Transparency Service performs the following steps:

1. **Client authentication:** A Client authenticates with the Transparency Service, to Register Signed Statements.
Authentication and authorization is implementation-specific, and out of scope of the SCITT Architecture.
1. **Issuer Verification:** The Transparency Service MUST perform resolution of the Issuer's identity, which may be different than the Client identity.
  This step may require that the service retrieves the Issuer ID in real-time, or rely on a cache of recent resolutions.
  For auditing, during Registration, the Transparency Service MUST store evidence of the lookup, including if it was resolved from a cache.
1. **Signature verification:** The Transparency Service MUST verify the signature of the Signed Statement, as described in {{RFC9360}}, using the signature algorithm and verification key of the Issuer.
1. **Signed Statement validation:** The Transparency Service MUST check that the Signed Statement includes the required protected headers listed above.
The Transparency Service MAY verify the Statement payload format, content and other optional properties.
1. **Apply Registration Policy:** The Transparency Service MUST check the attributes required by a policy are present in the protected headers.
  Custom Signed Statements are evaluated given the current Transparency Service state and the entire Envelope, and may use information contained in the attributes of named policies.
1. **Register the Signed Statement** to the append-only log.
1. **Return the Receipt**, which MAY be asynchronous from registration.
The Transparency Service MUST be able to provide a receipt for all registered Statements.
A receipt for a Signed Statement MAY be provided asynchronously.
Details about generating Receipts are described in {{Receipt}}.

The last two steps may be shared between a batch of Signed Statements recorded in the Append-only Log.

A Transparency Service MUST ensure that a Signed Statement is registered before releasing its Receipt.

The same Signed Statement may be independently registered in multiple Transparency Services, producing multiple, independent Receipts.
The multiple receipts may be attached to the unprotected header of the Signed Statement, creating a Transparent Statement.

## Transparent Statements {#Receipt}

The client (which is not necessarily the Issuer) that registers a Signed Statement and receives a Receipt can produce a Transparent Statement by adding the Receipt to the Unprotected Header of the Signed Statement.

When a Signed Statement is registered by a Transparency Service a Receipt becomes available.
When a Receipt is included in a Signed Statement a Transparent Statement is produced.

Receipts are based on Signed Inclusion Proofs as described in COSE Signed Merkle Tree Proofs ({{-COMETRE}}).

The registration time is defined as the timestamp at which the Transparency Service has added this Signed Statement to its Append-only Log.

**Editor's Note:** The WG is discussing if existing CWT claims might better support these design principles.

{{fig-transparent-statement-cddl}} illustrates a normative CDDL definition of Transparent Statements.

~~~ cddl
Transparent_Statement = #6.18(COSE_Sign1)

Unprotected_Header = {
  &(receipts: 394)  => [+ Receipt]
}
~~~
{: #fig-transparent-statement-cddl title="CDDL definition for a Transparent Statement"}

~~~~ cbor-diag
18(                                 / COSE Sign 1                   /
    [
      h'a4012603...6d706c65',       / Protected                     /
      {                             / Unprotected                   /
        394: [                      / Receipts (1)                  /
          h'd284586c...4191f9d2'    / Receipt 1                     /
        ]
      },
      nil,                          / Detached payload              /
      h'79ada558...3a28bae4'        / Signature                     /
    ]
)
~~~~
{: #fig-transparent-statement-edn title="CBOR Extended Diagnostic Notation example of a Transparent Statement"}

{{fig-transparent-statement-edn}} illustrates a payload that is detached.

The unprotected header can contain multiple receipts.

~~~~ cbor-diag
{                                   / Protected                     /
  1: -7,                            / Algorithm                     /
  4: h'50685f55...50523255',        / Key identifier                /
  -111: 1,                          / Verifiable Data Structure     /
  15: {                             / CWT Claims                    /
    1: transparency.vendor.example, / Issuer                        /
    2: vendor.product.example,      / Subject                       /
  }
}
~~~~
{: #fig-receipt-protected-header-edn title="CBOR Extended Diagnostic Notation example of a Receipt's Protected Header"}

Notice the verifiable data structure used is RFC9162_SHA256 in this case.
We know from the COSE Verifiable Data Structure Registry that RFC9162_SHA256 is value 1, and that it supports -1 (inclusion proofs) and -2 (consistency proofs).

~~~~ cbor-diag
18(                                 / COSE Sign 1                   /
    [
      h'a4012604...6d706c65',       / Protected                     /
      {                             / Unprotected                   /
        -222: {                     / Proofs                        /
          -1: [                     / Inclusion proofs (1)          /
            h'83080783...32568964', / Inclusion proof 1             /
          ]
        },
      },
      nil,                          / Detached payload              /
      h'10f6b12a...4191f9d2'        / Signature                     /
    ]
)
~~~~
{: #fig-receipt-edn title="CBOR Extended Diagnostic Notation example of a Receipt"}

Notice the unprotected header contains verifiable data structure proofs, see the protected header for details regarding the specific verifiable data structure used.

~~~~ cbor-diag
[                                   / Inclusion proof 1             /
  8,                                / Tree size                     /
  7,                                / Leaf index                    /
  [                                 / Inclusion hashes (3)          /
     h'c561d333...f9850597'         / Intermediate hash 1           /
     h'75f177fd...2e73a8ab'         / Intermediate hash 2           /
     h'0bdaaed3...32568964'         / Intermediate hash 3           /
  ]
]
~~~~
{: #fig-receipt-inclusion-proof-edn title="CBOR Extended Diagnostic Notation example of a Receipt's Inclusion Proof"}

This is a decoded inclusion proof for RFC9162_SHA256, other verifiable data structures might encode inclusion proofs differently.

### Validation {#validation}

Verifiers MUST apply the verification process as described in Section 4.4 of {{RFC9052}}.

APIs exposing verification logic for Transparent Statements may provide more details than a single boolean result.
For example, an API may indicate if the signature on the Receipt or Signed Statement is valid, if claims related to the validity period are valid, or if the inclusion proof in the Receipt is valid.

The algorithm-specific details of checking inclusion proofs are covered in {{-COMETRE}}.
The pseudo-code for validation of a transparent statement is as follows:

~~~python
let verify_transparent_statement(t) =
  let receipt = t.unprotected.scitt-receipt
  let version = receipt.protected.scitt-version or fail "Missing SCITT Receipt version"
  assert(version == 1)

  let leaf = COSE.serialize(t with .unprotected = {
    334 => receipt.unprotected.statement-registration-info
  })

  let vds = receipt.protected.verifiable-data-structure of fail "Missing verifiable data structure"
  let root = verify_inclusion_proof(vds, receipt.unprotected.scitt-inclusion-proof, leaf)
    or fail "Failed to verify inclusion proof"

  // Statement registration info has been authenticated by the inclusion proof
  receipt.protected.statement-registration-info = receipt.unprotected.statement-registration-info
  return COSE.verify(receipt, detached_payload=root)
~~~

Before checking a Transparent Statement, the Verifier must be configured with one or more identities of trusted Transparency Services.

Verifiers MAY be configured to re-verify the Issuer's Signed Statement locally, but this requires a fresh resolution of the Issuer's verification keys, which MAY fail if the key has been revoked.

Some Verifiers MAY decide to locally re-apply some or all of the Registration Policies, if they have limited trust in the Transparency Services.
In addition, Verifiers MAY apply arbitrary validation policies after the Transparent Statement has been verified and validated.
Such policies may use as input all information in the Envelope, the Receipt, and the Statement payload, as well as any local state.

Verifiers MAY offer options to store or share the Receipt of the Transparent Statement for auditing the Transparency Services in case a dispute arises.

# Privacy Considerations

Transparency Services are often publicly accessible.
Issuers should treat Signed Statements (rendering them as Transparent Statements) as publicly accessible.
In particular, a Signed Statement Envelope and Statement payload should not carry any private information in plaintext.

Transparency Services can have an authorization policy controlling who can access the Append-only Log.
While this can be used to limit who can read the Log, it may also limit the usefulness of the system.

Some jurisdictions have a Right to be Forgotten.
However, once a Signed Statement is inserted into the Append-only Log maintained by a Transparency Service, it cannot be removed from the Log.

# Security Considerations

On its own, verifying a Transparent Statement does not guarantee that its Envelope or contents are trustworthy.
Just that they have been signed by the apparent Issuer and counter-signed by the Transparency Service.
If the Verifier trusts the Issuer, it can infer that an Issuer's Signed Statement was issued with this Envelope and contents, which may be interpreted as the Issuer saying the Artifact is fit for its intended purpose.
If the Verifier trusts the Transparency Service, it can independently infer that the Signed Statement passed the Transparency Service Registration Policy and that has been persisted in the Append-only Log.
Unless advertised in the Transparency Service Registration Policy, the Verifier cannot assume that the ordering of Signed Statements in the Append-only Log matches the ordering of their issuance.

Similarly, the fact that an Issuer can be held accountable for its Transparent Statements does not on its own provide any mitigation or remediation mechanism in case one of these Transparent Statements turned out to be misleading or malicious.
Just that signed evidence will be available to support them.

An Issuer that knows of a changed state of quality for an Artifact, SHOULD Register a new Signed Statement, using the same `15` CWT `iss` and `sub` claims.

Issuers MUST ensure that the Statement payloads in their Signed Statements are correct and unambiguous, for example by avoiding ill-defined or ambiguous formats that may cause Verifiers to interpret the Signed Statement as valid for some other purpose.

Issuers and Transparency Services MUST carefully protect their private signing keys and avoid these keys being used for any purpose not described in this architecture document.
In cases where key re-use is unavoidable, keys MUST NOT sign any other message that may be verified as an Envelope as part of a Signed Statement.

Each of these functions MUST be carefully protected against both external attacks and internal misbehavior by some or all of the operators of the Transparency Service.

For instance, the code for the Registration Policy evaluation and endorsement may be protected by running in a Trusted Execution Environment (TEE).

The Transparency Service may be replicated with a consensus algorithm, such as Practical Byzantine Fault Tolerance {{PBFT}} and may be used to protect against malicious or vulnerable replicas.
Threshold signatures may be use to protect the service key, etc.

Issuers and Transparency Services MUST rotate verification keys for signature checking in well-defined cryptoperiods (see {{KEY-MANAGEMENT}}).

A Transparency Service MAY provide additional authenticity assurances about its secure implementation and operation, enabling remote attestation of the hardware platforms and/or software Trusted Computing Bases (TCB) that run the Transparency Service.
If present, these additional authenticity assurances MUST be registered in the Append-only Log and MUST always be exposed by the Transparency Services' APIs.
An example of Signed Statement's payloads that can improve authenticity assurances are trustworthiness assessments that are RATS Conceptual Messages, such as Evidence, Endorsements, or corresponding Attestation Results (see {{-rats-arch}}).

For example, if a Transparency Service is implemented using a set of redundant replicas, each running within its own hardware-protected trusted execution environments (TEEs), then each replica can provide fresh Evidence or fresh Attestation Results about its TEEs. The respective Evidence can show, for example, the binding of the hardware platform to the software that runs the Transparency Service, the long-term public key of the service, or the key used by the replica for signing Receipts. The respective Attestation Result, for example, can show that the remote attestation Evidence was appraised by a trusted Verifier and complies with well-known Reference Values and Endorsements.

## Security Guarantees

SCITT provides the following security guarantees:

1. Statements made by Issuers about supply chain Artifacts are identifiable, authentic, and non-repudiable
1. Statement provenance and history can be independently and consistently audited
1. Issuers can efficiently prove that their Statement is logged by a Transparency Service

The first guarantee is achieved by requiring Issuers to sign their Statements and associated metadata using a distributed public key infrastructure.
The second guarantee is achieved by storing the Signed Statement on an Append-only Log.
The third guarantee is achieved by implementing the Append-only Log using a verifiable data structure (such as a Merkle Tree {{MERKLE}}).

## Threat Model

The document provides a generic threat model for SCITT, describing its residual security properties when some of its actors (identity providers, Issuers, Transparency Services, and Auditors) are corrupt or compromised.

This model may need to be refined to account for specific supply chains and use cases.

SCITT primarily supports checking of Signed Statement authenticity, both from the Issuer (authentication) and from the Transparency Service (transparency).
These guarantees are meant to hold for extensive periods of time, possibly decades.

It can never be assumed that some Issuers and some Transparency Services will not be corrupt.

SCITT entities explicitly trust one another on the basis of their long-term identity, which maps to shorter-lived cryptographic credentials.
A Verifier SHOULD validate a Transparent Statement originating from a given Issuer, registered at a given Transparency Service (both identified in the Verifier's local authorization policy) and would not depend on any other Issuer or Transparency Services.

Authorized supply chain actors (Issuers) cannot be stopped from producing Signed Statements including false assertions in their Statement payload (either by mistake or by corruption), but these Issuers can made accountable by ensuring their Signed Statements are systematically registered at a trustworthy Transparency Service.

Similarly, providing strong residual guarantees against faulty/corrupt Transparency Services is a SCITT design goal.
Preventing a Transparency Service from registering Signed Statements that do not meet its stated Registration Policy, or to issue Receipts that are not consistent with their Append-only Log is not possible.
In contrast Transparency Services can be held accountable and they can be called out by any Auditor that replays their Append-only Log against any contested Receipt.
Note that the SCITT Architecture does not require trust in a single centralized Transparency Service.
Different actors may rely on different Transparency Services, each registering a subset of Signed Statements subject to their own policy.

In both cases, the SCITT Architecture provides generic, universally-verifiable cryptographic proof to individually blame Issuers or the Transparency Service.
On one hand, this enables valid actors to detect and disambiguate malicious actors who employ Equivocation with Signed Statements to different entities.
On the other hand, their liability and the resulting damage to their reputation are application specific, and out of scope of the SCITT Architecture.

Verifiers and Auditors need not be trusted by other actors.
In particular, so long as actors maintain proper control of their signing keys and identity infrastructure they cannot "frame" an Issuer or a Transparency Service for Signed Statements they did not issue or register.

### Append-only Log

If a Transparency Service is honest, then a Transparent Statement including a correct Receipt ensures that the associated Signed Statement passed its Registration Policy and was recorded appropriately.

Conversely, a corrupt Transparency Service may:

1. refuse or delay the Registration of Signed Statements
1. register Signed Statements that do not pass its Registration Policy (e.g., Signed Statement with Issuer identities and signatures that do not verify)
1. issue verifiable Receipts for Signed Statements that do not match its Append-only Log
1. refuse access to its Transparency Service (e.g., to Auditors, possibly after storage loss)

An Auditor granted (partial) access to a Transparency Service and to a collection of disputed Receipts will be able to replay it, detect any invalid Registration (2) or incorrect Receipt in this collection (3), and blame the Transparency Service for them.
This ensures any Verifier that trusts at least one such Auditor that (2, 3) will be blamed to the Transparency Service.

Due to the operational challenge of maintaining a globally consistent Append-only Log, some Transparency Services may provide limited support for historical queries on the Signed Statements they have registered, and accept the risk of being blamed for inconsistent Registration or Issuer Equivocation.

Verifiers and Auditors may also witness (1, 4) but may not be able to collect verifiable evidence for it.

### Availability of Receipts

Networking and Storage are trusted only for availability.

Auditing may involve access to data beyond what is persisted in the Transparency Services.
For example, the registered Transparency Service may include only the hash of a detailed SBOM, which may limit the scope of auditing.

Resistance to denial-of-service is implementation specific.

Actors may want to independently keep their own record of the Signed Statements they issue, endorse, verify, or audit.

### Confidentiality and Privacy

According to Zero Trust Principles any location in a network is never trusted.
All contents exchanged between actors is protected using secure authenticated channels (e.g., TLS) but may not exclude network traffic analysis.

The Transparency Service is trusted with the confidentiality of the Signed Statements presented for Registration.
Some Transparency Services may publish every Signed Statement in their logs, to facilitate their dissemination and auditing.
Others may just return Receipts to clients that present Signed Statements for Registration, and disclose the Append-only Log only to Auditors trusted with the confidentiality of its contents.

A collection of Signed Statements must not leak information about the contents of other Signed Statements registered on the Transparency Service.

Issuers must carefully review the inclusion of private/confidential materials in their Statements.
For example, Issuers must remove Personally Identifiable Information (PII) as clear text in the statement.
Alternatively, Issuers may include opaque cryptographic statements, such as hashes.

The confidentiality of queries is implementation-specific, and generally not guaranteed.
For example, while offline Envelope validation of Signed Statements is private, a Transparency Service may monitor which of its Transparent Statements are being verified from lookups to ensure their freshness.

### Cryptographic Agility

The SCITT Architecture supports cryptographic agility.
The actors depend only on the subset of signing and Receipt schemes they trust.
This enables the gradual transition to stronger algorithms, including e.g. post-quantum signature algorithms.

### Transparency Service Clients

Trust in clients that submit Signed Statements for Registration is implementation-specific.
An attacker may attempt to register any Signed Statement it has obtained, at any Transparency Service that accepts them, possibly multiple times and out of order.
This may be mitigated by a Transparency Service that enforces restrictive access control and Registration Policies.

### Impersonation

The identity resolution mechanism is trusted to associate long-term identifiers with their public signature-verification keys.
Transparency Services and other parties may record identity-resolution evidence to facilitate its auditing.

If one of the credentials of an Issuer gets compromised, the SCITT Architecture still guarantees the authenticity of all Signed Statements signed with this credential that have been registered on a Transparency Service before the compromise.
It is up to the Issuer to notify Transparency Services of credential revocation to stop Verifiers from accepting Signed Statements signed with compromised credentials.

# IANA Considerations

TBD; {{mybody}}.

## Media Type Registration

This section requests registration of the following media types {{RFC2046}} in
the "Media Types" registry {{IANA.media-types}} in the manner described
in {{RFC6838}}.

To indicate that the content is an scitt configuration represented as JSON:

- Type name: application
- Subtype name: scitt-configuration+json
- Required parameters: n/a
- Optional parameters: n/a
- Encoding considerations: binary; application/scitt-configuration+json values are represented as a JSON Object; UTF-8 encoding SHOULD be employed for the JSON object.
- Security considerations: See the Security Considerations section of TBD.
- Interoperability considerations: n/a
- Published specification: TBD
- Applications that use this media type: TBD
- Fragment identifier considerations: n/a
- Additional information:
  - Magic number(s): n/a
  - File extension(s): n/a
  - Macintosh file type code(s): n/a
- Person & email address to contact for further information: TBD
- Intended usage: COMMON
- Restrictions on usage: none
- Author: TBD
- Change Controller: IETF
- Provisional registration?  No
--- back

# Identifiers

This section provides informative examples of identifiers for statements, signed statements, and receipts.

SCITT Identifiers are primarily meant to be understood by humans and secondarily meant to be understood by machines, as such we define text encodings for message identifiers first, and then provide binary translations according to standard transformations for URLs and URNs to binary formats.

SCITT Identifiers for URLs and URNs that are not Data URLs MUST be represented in binary using {{-CURIs}}.

For each SCITT conceptual message, we define a Data URL format according to {{-DataURLs}}, a URN format according to {{-URNs}} and a URL format according to {{URLs}}.

Note that Data URLs require base64 encoding, but the URN definitions require base64url encoding.

Resolution and dereferencing of these identifiers is out of scope for this document, and can be implemented by any concrete api implementing the abstract interface defined as follows:

~~~
resource: content-type = dereference(identifier: identifier-type)
~~~

These identifiers MAY be present in a `tstr` field that does not otherwise restrict the string in ways that prevent a URN or URL from being present.

This includes `iss`, and `sub` which are used to express the Issuer and subject of a signed statement or receipt.

This also includes `kid` which is used to express a hint for which public key should be used to verify a signature.

All SCITT identifiers share common parameters to promote interoperability:

Let hash-name be an algorithm name registered in {{IANA.named-information}}.

To promote interoperability, the hash-name MUST be "sha-256".

Let base-encoding, be a base encoding defined in {{-Base64Url}}.

To promote interoperability, the base encoding MUST be "base64url".

In the blocks and examples that follow, note '\' line wrapping per RFC 8792.

## For Binary Content

Identifiers for binary content, such as Statements, or even Artifacts themselves are computed as follows:

Let the `base64url-encoded-bytes-digest` for the message be the base64url encoded digest with the chosen hash algorithm of bytes / octets.

Let the SCITT name for the message be the URN constructed from the following URI template, according to {{-URITemplate}}:

Let the `message-type`, be "statement" for Statements and Artifacts.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-bytes-digest}
~~~

## For SCITT Messages

Identifiers for COSE Sign 1 based messages, such as identifiers for Signed Statements and Receipts are computed as follows:

Let the `base64url-encoded-to-be-signed-bytes-digest` for the message be the base64url encoded digest with the chosen hash algorithm of the "to-be-signed bytes", according to {{Section 8.1 of RFC9052}}.

Let the SCITT name for the message be the URN constructed from the following URI template, according to {{-URITemplate}}:

Let the `message-type`, be "signed-statement" for Signed Statements, and "receipt" for Receipts.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-to-be-signed-bytes-digest}
~~~

Note that this means the content of the signature is not included in the identifier, even though signature related claims, such as activation or expiration information in protected headers are included.

As a result, an attacker may construct a new signed statement that has the same identifier as a previous signed statement, but has a different signature.

## For Transparent Statements

Identifiers for Transparent Statements are defined as identifiers for binary content, but with "transparent-statement" as the `message-type`.

~~~
urn:ietf:params:scitt:\
{message-type}:\
{hash-name}:{base-encoding}:\
{base64url-encoded-bytes-digest}
~~~

Note that because this identifier is computed over the unprotected header of the Signed Statement, any changes to the unprotected header, such as changing the order of the unprotected header map key value pairs, adding additional receipts, or adding additional proofs to a receipt, will change the identifier of a transparent statement.

Note that because this identifier is computed over the signatures of the signed statement and signatures in each receipt, any canonicalization of the signatures after the fact will produce a distinct identifier.

## Statements

### Statement URN

~~~
urn:ietf:params:scitt:statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-statement-urn align="left" title="Example Statement URN"}

### Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-statement-url align="left" title="Example Statement URL"}

### Statement Data URL

~~~
data:application/json;base64,SGVsb...xkIQ==
~~~
{: #example-statement-data-url align="left" title="Example Statement Data URL"}

## Signed Statements

### Signed Statement URN

~~~
urn:ietf:params:scitt:\
signed-statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-signed-statement-urn align="left" title="Example Signed Statement URN"}

### Signed Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:\
signed-statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-signed-statement-url align="left" title="Example Signed Statement URL"}

### Signed Statement Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-signed-statement-data-url align="left" title="Example Signed Statement Data URL"}

## Receipts

### Receipt URN

~~~
urn:ietf:params:scitt:receipt:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-receipt-urn align="left" title="Example Receipt URN"}

### Receipt URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:receipt:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-receipt-url align="left" title="Example Receipt URL"}

### Receipt Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-receipt-data-url align="left" title="Example Receipt Data URL"}

## Transparent Statements

### Transparent Statement URN

~~~
urn:ietf:params:scitt:\
transparent-statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-transparent-statement-urn align="left" title="Example Transparent Statement URN"}

### Transparent Statement URL

~~~
https://transparency.example/api/identifiers\
/urn:ietf:params:scitt:\
transparent-statement:sha-256:base64url:5i6UeRzg1...qnGmr1o
~~~
{: #example-transparent-statement-url align="left" title="Example Transparent Statement URL"}

### Transparent Statement Data URL

~~~
data:application/cose;base64,SGVsb...xkIQ==
~~~
{: #example-transparent-statement-data-url align="left" title="Example Transparent Statement Data URL"}

# Signing Statements Remotely

Statements, such as digital artifacts or structured data regarding artifacts, can be too large or too sensitive to be send to a remote Transparency Services over the Internet.
In these cases a statement can also be hash, which becomes the payload included in COSE to-be-signed bytes.
A Signed Statement (cose-sign1) MUST be produced from the to-be-signed bytes according to {{Section 4.4 of RFC9052}}.

~~~aasvg
   .----+-----.
  |  Artifact  |
   '+-+-------'
    | |
 .-'  v
|  .--+-------.
| |  Hash      +-+
|  '----------'  |     /\
 '-.             |    /  \     .----------.
    |            +-->+ OR +-->+  Payload   |
    v            |    \  /     '--------+-'
   .+--------.   |     \/               |
  | Statement +--+                      |
   '---------'                          |
                                        |
                                        |
           ...  Producer Network ...    |

                      ...

           ...   Issuer Network ...     |
                                        |
                                        |
 .---------.                            |
| Identity  |     (iss, x5t)            |
| Document  +--------------------+      |
 `----+----`                     |      |
      ^                          |      |
 .----+-------.                  |      |
| Private Key  |                 |      |
 '----+-------'                  v      |
      |                     .----+---.  |
      |                    |  Header  | |
      |                     '----+---'  |
      v                          v      v
    .-+-----------.       .------+------+--.
   /             /       /                  \
  /    Sign     +<------+ To Be Signed Bytes |
 /             /         \                  /
'-----+-------'           '----------------'
      v
 .----+-------.
| COSE Sign 1  |
 '------------'
~~~
