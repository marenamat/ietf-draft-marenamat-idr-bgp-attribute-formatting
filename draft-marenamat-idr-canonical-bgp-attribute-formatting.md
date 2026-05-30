---
title: "Canonical textual representation of BGP Path Attributes"
category: bcp

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
 RFC4360: bgp-extended-communities
 RFC4577: bgp-ospf-pe
 RFC4760: bgp-multiprotocol
 RFC5668: bgp-as4-ec
 RFC7153: bgp-ec-registry
 RFC7432: bgp-evpn
 RFC8092: bgp-large-communities
 RFC9026: multicast-vpn-fast-upstream-failover
 RFC9774: as-set-deprecation

informative:
 RFC8642: bgp-policy-for-well-known-communities

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

Deprecated and historic attributes are out of scope of this document.

# Conventions and Definitions

{::boilerplate bcp14-tagged}

# Simple Path Attributes

The following attributes, are listed simply for the sake of completeness. Their
formatting is simple and easy to model coherently.

- Type 1 - ORIGIN
- Type 3 - NEXT_HOP
- Type 4 - MULTI_EXIT_DISC (MED)
- Type 5 - LOCAL_PREF
- Type 6 - ATOMIC_AGGREGATE
- Type 7 - AGGREGATOR
- Type 9 - ORIGINATOR_ID
- Type 10 - CLUSTER_LIST

# AS_PATH Attribute

The AS_PATH attribute contains segments of ASN values. While this attribute is technically
possible to be modelled coherently by YANG, there are situations where the AS_PATH value
is expected to be rendered as a whole. In such cases, the contents of each segment
SHOULD be displayed as decimal values separated by single spaces (ASCII 32).

In addition to that, boundary between two `AS_SEQUENCE` segments MAY be delimited by `|` (ASCII 124),
and `AS_CONFED_SEQUENCE` SHOULD be parenthesized (ASCII 40 and 41).

Example: `(65501 65502) 65503 65504 | 65505 65506 65506 65506`

This would be an `AS_CONFED_SEQUENCE` of 65501 and 65502 followed by `AS_SEQUENCE` of 65503 and 65504,
and another `AS_SEQUENCE` containing 65505 and then 65506 three times.

TODO: ABNF here?

# COMMUNITIES Attribute

The COMMUNITIES attribute {{-bgp-communities}} is a set of uint32 values, which
are semantically a pair of an AS number and an arbitrary uint16 value. Following the
syntax used in {{-bgp-policy-for-well-known-communities}} and in vast majority
of current implementations, the value SHOULD be formatted as two decimal values
with no leading zeros, joined by a single colon (ASCII 58) with no whitespace.

In addition to that, it is RECOMMENDED to format well-known communities as their
string name.

For the sake of formatting consistency, the "Standby PE" as defined
in {{-multicast-vpn-fast-upstream-failover}} is hereby renamed to STANDBY_PE.
The semantics stays the same.

TODO: ABNF here?

# EXTENDED_COMMUNITIES Attribute

The EXTENDED_COMMUNITIES attribute {{-bgp-extended-communities}} is a set of
uint64 values with a complicated structure. Copying a modified schema from
{{Section 2 of -bgp-extended-communities}}, the Type denotes how the value is
further split into sub-values, and the Sub-Type denotes the meaning.

       0                   1                   2                   3
       0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
      |     Type      |    Sub-Type   |                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+          Value                |
      |                                                               |
      +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

In general, the display format consists of the sub-type identifier followed
by a colon and then formatted values, separated by colons (ASCII 58).
Each sub-type has a string representation, which is a sequence of lowercase
ascii letters, numbers and hyphens. The sub-type identifiers MUST be unique to
the extent that the identifier together with the display format allows to
determine the Type and Sub-Type values.

Every new definition of a Extended Community Type or Sub-Type SHOULD include
a canonical textual representation of the value.

The following sections specify how the already specified Extended Community
variants are expected to be formatted.  The syntax is either reflecting the
current practice adopted by the majority of vendors, or trying to unify the
formatting where no majority exists.

## Display format for AS-Specific and IPv4-Specific Type values

Two-Octet AS-Specific Extended Community (Type 0x00 and 0x40) ({{-bgp-extended-communities}})
: Sub-Type identifier followed by the ASN (Global Administrator) and local value (Local Administrator)
: Example: `route-target:65499:42`

IPv4-Address-Specific Extended Community (Type 0x01 and 0x41) ({{-bgp-extended-communities}})
: Sub-Type identifier followed by the IPv4 address (Global Administrator) and local value (Local Administrator)
: Example: `route-origin:192.0.2.67:42`

Four-Octet AS-Specific Extended Community (Type 0x02 and 0x42) ({{-bgp-as4-ec}})
: Sub-Type identifier followed by the ASN (Global Administrator) with an `L` suffix,
  and local value (Local Administrator).
  While it's possible, in some cases, to distinguish between four-octet and two-octet ASN
  without the suffix, it SHOULD be used in all cases to avoid confusion.
: Example: `route-target:65544L:22`, `route-origin:65511L:12345`

## Display format for Opaque Extended Community (Type 0x03 and 0x43)

Specified in {{-bgp-extended-communities}}.

(TODO)

### Default Gateway Extended Community

Specified in {{Section 7.8 of -bgp-evpn}}. Formatted as `evpn-default-gateway`
with no colons.

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

# IANA Considerations

## BGP Well-known Communities

IANA is requested to rename the "Standby PE" BGP community value (`0xFFFF0009`)
to `STANDBY_PE`.

## Registries for BGP Extended Communities

IANA is requested to add a column "Identifier" to all the Sub-Type registries
as specified in {{Section 5.2 of -bgp-ec-registry}}. The identifiers MUST NOT
be reused in any other Sub-Type registries, unless explicitly specified.

The following data should be used to fill the newly added columns.

### EVPN Extended Community Sub-Types

| Sub-Type Value | Name                   | Identifier       |
|----------------|------------------------|------------------|
| 0x00           | MAC Mobility           | mac-mobility     |
| 0x01           | ESI Label              | esi-label        |
| 0x02           | ES-Import Route Target | es-import-target |

<!-- TODO: a lot of tabularization -->

### Transitive Two-Octet AS-Specific Extended Community Sub-Types

| Sub-Type Value | Name                   | Identifier        |
|----------------|------------------------|-------------------|
| 0x02           | Route Target           | route-target      |
| 0x03           | Route Origin           | route-origin      |
| 0x05           | OSPF Domain Identifier | ospf-domain-ident |

<!-- TODO: a lot of tabularization -->


--- back

# Acknowledgments
{:numbered="false"}

TODO acknowledge.

Jeff Haas for pushing me this direction.

Authors of BGP YANG.
