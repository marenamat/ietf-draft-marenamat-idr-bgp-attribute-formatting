---
title: "Canonical textual representation of BGP Path Attributes"
category: std

docname: draft-marenamat-idr-bgp-attribute-formatting-latest
submissiontype: IETF  # also: "independent", "editorial", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Routing"
workgroup: "Inter-Domain Routing"
keyword:
 - text representation
 - BGP
 - attributes
 - communities
venue:
  group: "Inter-Domain Routing"
  type: "Working Group"
  mail: "idr@ietf.org"
  arch: "https://mailarchive.ietf.org/arch/browse/idr/"
  github: "marenamat/ietf-draft-marenamat-idr-bgp-attribute-formatting"
  latest: "https://marenamat.github.io/ietf-draft-marenamat-idr-bgp-attribute-formatting/draft-marenamat-idr-bgp-attribute-formatting.html"

author:
 -
    fullname: "Maria Matejka"
    organization: CZ.NIC
    email: "ietf.org@jmq.cz"

normative:
 RFC1997: bgp-communities
 RFC4271: bgp
 RFC4384: bgp-communities-for-data-collection
 RFC4360: bgp-extended-communities
 RFC4577: bgp-ospf-pe
 RFC4760: bgp-multiprotocol
 RFC4940: ospf-iana-considerations
 RFC5668: bgp-as4-ec
 RFC7153: bgp-ec-registry
 RFC7432: bgp-evpn
 RFC7543: cp-orf
 RFC7900: bgp-extranet-multicast
 RFC7902: bgp-additional-pmsi-tunnel-attribute-flags
 RFC8092: bgp-large-communities
 RFC9026: multicast-vpn-fast-upstream-failover
 RFC9753: bgp-evpn-tunnel-aggregation
 RFC9774: as-set-deprecation
 ISO.3166-1.2006: iso-country-codes

informative:
 RFC8642: bgp-policy-for-well-known-communities
 I-D.knoll-idr-cos-interconnect: knoll-cos
 I-D.knoll-idr-qos-attribute: knoll-qos

# Note: reference IDs fetched from https://bib.ietf.org/

...

--- abstract

Various implementations of the Border Gateway Protocol (BGP) use different
formats for displaying the Path Attributes. This document defines the preferred
textual formatting which is recommended for the implementations to use for
human interfaces.

To achieve consistent value formatting, this document formally updates RFC 9026
by canonicalizing the well-known community name formats.

This document updates RFC 4360, RFC 4577, RFC 7432, and ... by specifying the canonical
textual formatting of extended communities specified there.

This document updates RFC 4940 by adding a textual tag column to the OSPFv2 Link State Type registry.

This document updates RFC 7153 by adding a textual tag column to the extended
community registry.

(REMOVE THIS) This document is incomplete and needs completing the tables.

--- middle

# Introduction

Over 30 years of BGP existence have created a difference in router user interfaces.
While diversity is a good thing, displaying the same route attributes differently
leads to user confusion and elevated need for learning vendor specifics. With the
advent of APIs, often based on NETCONF and YANG, a need for canonical representation
has arised.

While most attributes are either a value, or a structure of values, which can be
easily modeled by YANG, with complex attributes like extended communities, there
is a lot of subvariants and semantics in their values. Users and implementations
usually format these values in a structured form which is hard to be modeled by YANG.

This document aims to summarize all of these in one place and specifies a
standard way of defining canonical formatting for new BGP path attributes.

It is expected that these formatted values would be used inside NETCONF and RESTCONF APIs,
wherever practical. Alternatives would be to send raw binary data, which would
need additional parsing on the client side, or to model full structures of the attributes
to the full depth, which would add significant amount of noise into both the
model and the data itself.

Deprecated and historic attributes are out of scope of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

**Raw attribute value**: The value of the BGP attribute as received or to be sent in a BGP message.

**Textual representation**: A human-readable string representing a value of a BGP attribute.

**Clear representation**: A textual representation which is unambiguosly convertible to a raw attribute value.

**Canonical representation**: A clear representation assigned to a specific raw attribute value.

**Formatter**: The part of software implementation which accepts a raw attribute value and outputs its textual representation.

**Canonical Formatter**: A formatter producing canonical representations.

**Parser**: The part of software implementation which accepts a textual representation and outputs a raw attribute value.

# General rules

Formatters MAY produce other textual representations if configured so.
Formatters SHOULD always produce clear representations, to avoid confusion.
Parsers SHOULD NOT assume a specific interpretation of a textual representation
which is not clear.

Parsers SHOULD be case-insensitive. There MUST NOT be any case where a
semantics was dependent on the letter case.

## Basic data types {#basic-data-types}

1. Integer values SHOULD be displayed as a single integer in decimal notation with
   no leading zeros.

2. Enumeration values SHOULD be displayed as the item identifier, as specified
   in their appropriate tables.  Many of these are to be (FIXME) updated by this document,
   {{iana-considerations}}, to include such identifiers suitable for canonical representation.

   Unknown enumeration values SHOULD be displayed as a single integer in decimal
   notation with no leading zeros.

3. Flags SHOULD be treated as separate values, which are either present (set to 1)
   or absent (set to 0). If appropriate, even an absent flag MAY be displayed.
   They SHOULD be represented as their flag identifiers, as specified in their
   appropriate tables. Many of these are to be (FIXME) updated by this document,
   {{iana-considerations}}, to include such identifiers suitable for canonical
   representation. Flags SHOULD be displayed in the order from the most
   significant to least significant bit.

   Unknown flags set to 1 SHOULD be displayed as a single integer in decimal
   notation with no leading zeros, representing the position of the set bit.

4. Enumeration and Flag Identifiers have a string representation, which is a sequence of
   lowercase or uppercase ASCII letters, numbers and hyphens. All identifiers
   for one set of flags or values MUST be mutually unique, and SHOULD be all
   either lowercase or uppercase, not both, to achieve a consistent canonical
   representation.

5. Values specified to be displayed in hexadecimal SHOULD be displayed padded by
   zeros to all significant bits.

6. Values specified as reserved SHOULD NOT be displayed (should be skipped).
   If these values are set to non-default values, the whole containing
   attribute structure SHOULD be displayed as a sequence of octets in hexadecimal,
   prefixed by `malformed`. The implementation MAY have configuration options
   to ignore non-default reserved values.

# Simple Path Attributes

The following attributes, are listed simply for the sake of completeness. Their
formatting is simple and easy to model coherently.

- Type 1 – `ORIGIN`
- Type 3 – `NEXT_HOP`
- Type 4 – `MULTI_EXIT_DISC` (MED)
- Type 5 – `LOCAL_PREF`
- Type 6 – `ATOMIC_AGGREGATE`
- Type 7 – `AGGREGATOR`
- Type 9 – `ORIGINATOR_ID`
- Type 10 – `CLUSTER_LIST`

All unknown attributes are considered to be a binary blob, and with that,
simple to format.

# `AS_PATH` Attribute

The `AS_PATH` attribute contains segments of ASN values. While this attribute is technically
possible to be modelled coherently by YANG, there are situations where the `AS_PATH` value
is expected to be rendered as a whole. In such cases, the contents of each segment
SHOULD be displayed as integers separated by single spaces (ASCII 32).

In addition to that, every `AS_CONFED_SEQUENCE` SHOULD be parenthesized (ASCII 40 and 41),

Boundary between two consecutive `AS_SEQUENCE` segments MAY be delimited by `|` (ASCII 124),
and while the segmentation bears no semantic value, it may be handy to see when debugging
corner-case router behavior.

Example: `(65501 65502) 65503 65504 | 65505 65506 65506 65506`

This would be an `AS_CONFED_SEQUENCE` of 65501 and 65502 followed by `AS_SEQUENCE` of 65503 and 65504,
and another `AS_SEQUENCE` containing 65505 and then 65506 three times.

A sequence of an unknown type, if needed to be displayed, SHOULD be surrounded by
square brackets (ASCII 91 and 93), and prefixed by `unknown:` followed by the
type value as an integer.

Example: `67 01 fb fb 02 02 fb f4 fb fe` in the semantics of RFC 4271 (two-octet ASNs)
shall be printed out as follows: `[ unknown:103 64507 ] 64500 64510`

Note: The `AS_SET` and `AS_CONFED_SET` segments have been deprecated by
{{-as-set-deprecation}}. If needed to be displayed, they SHOULD be displayed in
the same way as unknown types. The implementation MAY, if configured so, replace
the `unknown:1` and `unknown:4` by `set` and `confed-set`, respectively.

# `COMMUNITIES` Attribute

The `COMMUNITIES` attribute {{-bgp-communities}} is a set of uint32 values, which
are semantically a pair of an AS number and an arbitrary uint16 value. Following the
syntax used in {{-bgp-policy-for-well-known-communities}} and in vast majority
of current implementations, the value SHOULD be formatted as two integers
joined by a single colon (ASCII 58) with no whitespace.

In addition to that, it is RECOMMENDED to format well-known communities as their
string name.

For the sake of formatting consistency, the "Standby PE" as defined
in {{-multicast-vpn-fast-upstream-failover}} is hereby renamed to `STANDBY_PE`.
The semantics stays the same.

Example: `65544:6767`, `BLACKHOLE`

# `EXTENDED_COMMUNITIES` Attribute

The `EXTENDED_COMMUNITIES` attribute {{-bgp-extended-communities}} is a set of
uint64 values with a complicated structure. Copying a modified schema from
{{Section 2 of -bgp-extended-communities}}, the Type denotes the meaning
and formatting of the value. While the original specification allows for registering
regular single-octet types with 7-octet value, the only drafts ever seen specifying these
are {{-knoll-cos}} and {{-knoll-qos}} with last versions from 2019, and
therefore it is assumed that the Type is always extended to 2 octets.

     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |            Type               |                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+          Value                |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Commonly, the lower octet of the Type is called Sub-Type, as in {{-bgp-ec-registry}}.

The canonical formatting is the Type displayed as an enumeration value,
and then the Value split to its semantic parts, as specified in following
subsections. All the parts are displayed in order, beginning with the Type
identifier, and delimited by a single colon (ASCII 58).

If the Type is unknown, the whole extended community value is displayed
as `unknown-transitive` or `unknown-non-transitive` followed by all eight
octets of the extended community (including the type itself),
delimited by colons (ASCII 58).

Example: `67 89 ab cd ef 01 23 45` would be displayed as
`unknown-non-transitive:67:89:ab:cd:ef:01:23:45`

## Future Extended Community Type definitions

Every new definition of an Extended Community Type MUST include its enumeration
value identifier and a specification of canonical textual representation
of the value. Hereby, a paragraph shall be appended
to the {{Section 4 of -bgp-ec-registry}}:

> Every Sub-Type codepoint defined MUST specify a unique value of the display
> format type identifier, together with the appropriate canonical textual
> representation of the Value.
>
> For Sub-Type codepoints with similar semantics across multiple high octets,
> it is RECOMMENDED to assign similar type identifiers.
>
> Identifiers assigned by IANA MUST NOT begin with `unknown-` or `experimental-`.

For experimental types, an implementation MAY locally assign an identifier
beginning with `experimental-`, and format these appropriately.

## Display format for AS2-Specific Type Values

The structure of Two-octet AS-Specific Extended Community (Type high octet 0x00 and 0x40)
Value is first the two-octet AS number (Global Administrator) and then four-octet
local value (Local Administrator). Both values are integers.

### Display format for AS2-Specific BGP Data Collection Values {#ec-format-0x0008}

The actual structure of the AS2-Specific BGP Data Collection (Type 0x0008) Local Administrator value
is, by {{-bgp-communities-for-data-collection}}:

- 2 octets of zeros (reserved)
- 5 bits of region identifier ({{data-collection-registry}})
- 1 bit of satellite link flag (displayed as SAT if set to 1)
- 10 bits of country code (using two-character codes for formatting as defined in {{-iso-country-codes}})

There are also special values which are represented by a single identifier
from {{data-collection-registry}}.

The parser SHOULD also accept discontinued country codes, possibly with a warning.

Example:

- `00 08 fb f4 00 00 1a 0c` is formatted as `data-collection-as2:64500:AS:NP`
- `00 08 fb f6 00 00 00 06` is formatted as `data-collection-as2:64500:upstream`
- `00 08 fb f8 67 67 67 67` is formatted as `data-collection-as2:64500:malformed:67:67:67:67`

## Display format for IPv4-Specific Type Values

The structure of IPv4-Specific Extended Community (Type high octet 0x01 and 0x41)
Value is first the four-octet IPv4 address (Global Administrator) and then two-octet
local value (Local Administrator).

The Global Administrator value is formatted as an IPv4 address.
The Local Administrator value is formatted as a decimal integer with no leading zeros,
unless specified otherwise.

### Display format for IPv4-Specific BGP Data Collection Values

The actual structure of the BGP Data Collection (Type 0x0108) Local Administrator value
is, by {{-bgp-communities-for-data-collection}}:

- 5 bits of region identifier ({{data-collection-registry}})
- 1 bit of satellite link flag (displayed as SAT if set to 1)
- 10 bits of country code (using two-character codes for formatting as defined in {{-iso-country-codes}})

There are also special values which are represented by a single identifier
from {{data-collection-registry}}.

The parser SHOULD also accept discontinued country codes, possibly with a warning.

Example:

- `01 08 c0 00 02 43 1e 0c` is formatted as `data-collection-ipv4:192.0.2.67:AS:SAT:NP`
- `01 08 c6 33 64 2a 00 06` is formatted as `data-collection-ipv4:198.51.100.42:upstream`
- `01 08 cb 00 71 67 24 43` is formatted as `data-collection-ipv4:203.0.113.103:AQ:SAT:67`

## Display format for AS4-Specific Type values

The structure of Four-octet AS-Specific Extended Community (Type high octet 0x02 and 0x42)
Value is first the four-octet AS number (Global Administrator) and then two-octet
local value (Local Administrator).

The Global Administrator value is formatted as a decimal integer with no leading zeros.
The Local Administrator value is formatted as a decimal integer with no leading zeros,
unless specified otherwise.

### Display format for IPv4-Specific BGP Data Collection Values

The actual structure of the BGP Data Collection (Type 0x0208) Local Administrator value
is, by {{-bgp-communities-for-data-collection}}:

- 5 bits of region identifier ({{data-collection-registry}})
- 1 bit of satellite link flag (displayed as SAT if set to 1)
- 10 bits of country code (using two-character codes for formatting as defined in {{-iso-country-codes}})

There are also special values which are represented by a single identifier
from {{data-collection-registry}}.

The parser SHOULD also accept discontinued country codes, possibly with a warning.

Example:

- `02 08 00 01 00 04 1a 0c` is formatted as `data-collection-as4:65540:AS:NP`
- `02 08 00 01 00 06 00 06` is formatted as `data-collection-as4:65542:upstream`
- `02 08 00 01 00 07 67 67` is formatted as `data-collection-as4:65543:26471`

## Display format for Opaque Type Values

The structure of Opaque Extended Community (Type high octet 0x03 and 0x43)
Value is specific for each Sub-Type, as specified in {{-bgp-extended-communities}}
and subsequent documents.

The value is often reserved and specified to be zero. Unless specified
otherwise, the value is therefore not displayed at all.

Example: `extranet-separation`

*TODO: There is no value specified in {{-cp-orf}} at all.*

### Display format for OSPF Route Type

The actual structure of the OSPF Route Type (Type 0x0306) Value is,
by {{-bgp-ospf-pe}}:

    +--------+--------+--------+--------+--------+--------+
    |             Area Number           |  Type  |0000000M|
    +--------+--------+--------+--------+--------+--------+

The Area Number is to be formatted as dotted-decimal.

Known values of Type are specified in {{Section 4.2.6 of -bgp-ospf-pe}}
as an explicit table, which is actually aligned with the IANA table later specified
by {{-ospf-iana-considerations}}. Hereby the {{Section 4.2.6 of -bgp-ospf-pe}}
is updated such that the OSPF Route Type encoding states:

    OSPF Route type: 1 byte, using values in the Registry for OSPFv2 Link State (LS) Type, as specified by {{Section 5.5 of -ospf-iana-considerations}}.

The Type is to be formatted as an identifier specified in {{ospfv2-link-state-type-registry}}.
An unknown value of Type is to be formatted as a decimal number with no leading zeros.

The Options shall be formatted as flags as specified in {{basic-data-types}}.
Currently, the only flag specified is TYPE-2-METRIC.

### Display format for Additional PMSI Tunnel Attribute Flags

The actual structure of the Additional PMSI Tunnel Attribute Flags Type (Type 0x0307) is,
by {{-bgp-additional-pmsi-tunnel-attribute-flags}}, a set of 48 flags. These
shall be displayed as flags as specified in {{basic-data-types}}.

This document hereby updates {{Section 3 of -bgp-additional-pmsi-tunnel-attribute-flags}}
by adding the following paragraph:

> Names of the flags in the "Additional PMSI Tunnel Attribute Flags" table SHOULD
> consist of uppercase letters, numbers and dashes, and MUST NOT contain any whitespace.

When feasible and practical, the implementation MAY also display this extended
community contents as an actual extension in the context of the PMSI Tunnel
path attribute. It SHOULD be displayed together with other extended communities
anyway though, to keep the notion of its actual location.

### Display format for Context-Specific Label Space ID

The actual structure of the Context-Specific Label Space ID Type (Type 0x0308) is,
by {{-bgp-evpn-tunnel-aggregation}}, a 2-octet ID-Type (enumeration, see {{context-specific-label-space-id-type}})
and a 4-octet ID-Value (integer).

Note: For ID-Type zero (MPLS Label), {{Section 4.1 of -bgp-evpn-tunnel-aggregation}}
does not specify the value of the bottom 12 bits of the ID-Value. This document
hereby updates {{Section 4.1 of -bgp-evpn-tunnel-aggregation}} by adding a sentence
to its third paragraph (which specifies the ID-Value):

> The remaining 12 bits are reserved and MUST be set to zero.

*TODO: more EC to inspect*

## Display format for EVPN Extended Communities (Type 0x06)

### ESI Label Extended Community

Specified in {{Section 7.5 of -bgp-evpn}}. Formatted as `esi-label` followed by
flags and label value.

Flags: `S` for Single-Active, `A` for All-Active

Example: `esi-label:A:67`

### ES-Import Route Target Extended Community

Specified in {{Section 7.6 of -bgp-evpn}}. Formatted as `es-import-target`
followed by the ES-Import value formatted as single bytes in hexadecimal notation.

Example: `es-import-target:fe:ed:0d:b8:1e:a9`

### MAC Mobility Extended Community

Specified in {{Section 7.7 of -bgp-evpn}}. Formatted as `mac-mobility`
followed by flags and sequence number.

Flags: `S` for sticky/static, nothing otherwise

Example: `mac-mobility:S:67`, `mac-mobility::42`


<!-- TODO:
Transport Class Extended Community (Type 0x0a and 0x4a)
SFC Extended Community (Type 0x0b)
Generic Transitive Extended Community Part 1 (Type 0x80)
Generic Transitive Extended Community Part 2 (Type 0x81)
Generic Transitive Extended Community Part 3 (Type 0x82)
-->


# Security Considerations

There are no security considerations in formatting the path attributes.

# IANA Considerations {#iana-considerations}

## BGP Well-known Communities

IANA is requested to rename the "Standby PE" BGP community value (`0xFFFF0009`)
to `STANDBY_PE`.

## Registries for BGP Extended Communities

IANA is requested to add a column "Identifier" to all the Sub-Type registries
as specified in {{Section 5.2 of -bgp-ec-registry}}. The identifiers MUST NOT
be reused in any other Sub-Type registries, unless explicitly specified.

The following data should be used to fill the newly added columns.

### Transitive Two-Octet AS-Specific Extended Community Sub-Types {#ec-registry-0x00}

| Sub-Type Value | Name                   | Identifier                |
|----------------|------------------------|---------------------------|
| 0x02           | Route Target           | route-target-as2          |
| 0x03           | Route Origin           | route-origin-as2          |
| 0x05           | OSPF Domain Identifier | ospf-domain-id-as2        |
| 0x08           | BGP Data Collection    | data-collection-as2       |
| 0x09           | Source AS              | source-as2                |
| 0x0A           | L2VPN Identifier       | i2vpn-id-as2              |

### Transitive IPv4-Address-Specific Extended Community Sub-Types {#ec-registry-0x01}

| Sub-Type Value | Name                                | Identifier                         |
|----------------|-------------------------------------|------------------------------------|
| 0x02           | Route Target                        | route-target-ipv4                  |
| 0x03           | Route Origin                        | route-origin-ipv4                  |
| 0x05           | OSPF Domain Identifier              | ospf-domain-id-ipv4                |
| 0x07           | OSPF Router ID                      | ospf-router-id-ipv4                |
| 0x0A           | L2VPN Identifier                    | i2vpn-id-ipv4                      |
| 0x0B           | VRF Route Import                    | vrf-route-import-ipv4              |
| 0x12           | Inter-Area P2MP Segmented Next-Hop  | p2mp-inter-area-segmented-nh-ipv4  |
| 0x20           | MVPN SA RP-address                  | mvpn-sa-rp-address-ipv4            |


The IANA is hereby requested to fix the typo at 0x07 Name.

### Transitive Four-Octet AS-Specific Extended Community Sub-Types {#ec-registry-0x02}

| Sub-Type Value | Name                   | Identifier                |
|----------------|------------------------|---------------------------|
| 0x02           | Route Target           | route-target-as4          |
| 0x03           | Route Origin           | route-origin-as4          |
| 0x05           | OSPF Domain Identifier | ospf-domain-ident-as4     |
| 0x08           | BGP Data Collection    | data-collection-as4       |
| 0x09           | Source AS              | source-as4                |

### Transitive Opaque Extended Community Sub-Types {#ec-registry-0x03}

| Sub-Type Value | Name                                   | Identifier                |
|----------------|----------------------------------------|---------------------------|
| 0x03           | CP-ORF                                 | cp-orf                    |
| 0x04           | Extranet Source                        | extranet-source           |
| 0x05           | Extranet Separation                    | extranet-separation       |
| 0x06           | OSPF Route Type                        | ospf-route-type           |
| 0x07           | Additional PMSI Tunnel Attribute Flags | pmsi-tunnel-flags         |
| 0x08           | Context-Specific Label Space ID        | label-space-id            |
| 0x0d           | Default Gateway                        | evpn-default-gateway      |

IANA is hereby requested to fix the reference in "Transitive Opaque Extended
Community Sub-Types" table from Yakov Rekhter to {{-bgp-evpn}}.

(TODO)

### EVPN Extended Community Sub-Types {#ec-registry-0x06}

| Sub-Type Value | Name                    | Identifier        |
|----------------|-------------------------|-------------------|
| 0x00           | MAC Mobility            | mac-mobility      |
| 0x01           | ESI Label               | esi-label         |
| 0x02           | ES-Import Route Target  | es-import-target  |
| 0x03           | EVPN Router's MAC       | evpn-router-mac   |
| 0x04           | EVPN Layer 2 Attributes | evpn-layer2-attrs |
| 0x05           | E-Tree                  | evpn-etree        |
| 0x06           | DF Election             | evpn-df-election  |
| 0x08           | ARP/ND                  | evpn-arp-nd       |
| 0x09           | Multicast Flags         | evpn-mc-flags     |
| 0x0A           | EVI-RT Type 0           | evi-rt0           |
| 0x0B           | EVI-RT Type 1           | evi-rt1           |
| 0x0C           | EVI-RT Type 2           | evi-rt2           |
| 0x0D           | EVI-RT Type 3           | evi-rt3           |
| 0x0F           | Service Carving Time    | evpn-sct          |

(TODO)

## Registry for BGP Data Collection Communities {#data-collection-registry}

IANA is requested to add a column "Identifier" to the BGP Data Collection
Standard Communities Registry, which is populated as follows:

| Value            | Category                      | Identifier             |
|------------------|-------------------------------|------------------------|
| 0000000000000001 | Customer Routes               | customer               |
| 0000000000000010 | Peer Routes                   | peer                   |
| 0000000000000011 | Internal Routes               | internal               |
| 0000000000000100 | Internal More Specific Routes | internal-more-specific |
| 0000000000000101 | Special Purpose Routes        | special                |
| 0000000000000110 | Upstream Routes               | upstream               |

IANA is requested to add a column "Identifier" to the Region Identifiers Registry,
which is populated as follows:

| Value | Category                         | Identifier |
|-------|----------------------------------|------------|
| 00001 | Africa                           | AF         |
| 00010 | Oceania                          | OC         |
| 00011 | Asia                             | AS         |
| 00100 | Antarctica                       | AQ         |
| 00101 | Europe                           | EU         |
| 00110 | Latin America/Caribbean Islands  | LAC        |
| 00111 | North America                    | NA         |

## Registry for Context-Specific Label Space ID Type {#context-specific-label-space-id-type}

IANA is requested to add a column "Identifier" to the
Context-Specific Label Space ID Type Registry,
which is populated as follows:

| Value | Name       | Identifier |
|-------|------------|------------|
| 0     | MPLS Label | mpls-label |

## Registry for OSPFv2 Link State (LS) Type {#ospfv2-link-state-type-registry}

IANA is requested to add a column "Identifier" to the OSPFv2 Link State (LS)
Type, as specified in {{-ospf-iana-considerations}}, which is populated as follows:

| Value | Description               | Identifier       |
|-------|---------------------------|------------------|
| 1     | Router-LSA                | router           |
| 2     | Network-LSA               | network          |
| 3     | Summary-LSA (IP network)  | summary          |
| 4     | Summary-LSA (ASBR)        | summary-asbr     |
| 5     | AS-external-LSA           | as-external      |
| 6     | Group-membership-LSA      | group-membership |
| 7     | NSSA AS-external LSA      | nssa-as-external |
| 9     | Link-scoped Opaque LSA    | link-opaque      |
| 10    | Area-scoped Opaque LSA    | area-opaque      |
| 11    | AS-scoped Opaque LSA      | as-opaque        |


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

Jeff Haas, Ondřej Zajíček for major comments and suggestions.

Authors of BGP YANG.
