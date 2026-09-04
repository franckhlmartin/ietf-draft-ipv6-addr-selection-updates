%%%
title = "Updates to IPv6 Default Address Selection"
abbrev = "ipv6-addr-selection-updates"
ipr = "trust200902"
updates = [6724]
keyword = ["IPv6", "RFC6724", "address selection", "getaddrinfo", "DNS load balancing"]

date = 2026-08-29

[seriesInfo]
name = "Internet-Draft"
value = "draft-martin-ipv6-addr-selection-updates-00"
status = "standard"

[[author]]
initials = "F."
surname = "Martin"
fullname = "Franck Martin"
organization = "Peachymango.org"
  [author.address]
  email = "franck@peachymango.org"

[[author]]
initials = "X."
surname = "Xiao"
fullname = "XiPeng Xiao"
organization = "Huawei Technologies Dusseldorf"
  [author.address]
  email = "xipengxiao@gmail.com"

[[author]]
initials = "B."
surname = "Carpenter"
fullname = "Brian E. Carpenter"
organization = "University of Auckland"
  [author.address]
  email = "brian.e.carpenter@gmail.com"
%%%

.# Abstract

This document updates RFC 6724 with three related improvements to
IPv6 destination address selection. The updates allow observed or recent
communication performance to influence ordering, incorporate likely
source/destination address pairs when sorting candidates, and let
ISP and enterprise operators preserve DNS load-balancing order where
Rule 9 would otherwise override it. All three enhancements are OPTIONAL, off by default, and
intended to be implementable internally by `getaddrinfo()` or an
equivalent system mechanism, without changing the existing API or
requiring application source-code changes.

.# About This Document

This note is to be removed before publishing as an RFC.

The latest revision of this draft can be found at
https://github.com/franckhlmartin/ietf-draft-ipv6-addr-selection-updates/.
Status information for this document may be found at
https://datatracker.ietf.org/doc/draft-martin-ipv6-addr-selection-updates/.

Discussion of this document takes place on the v6ops Working Group
mailing list (mailto:v6ops@ietf.org), which is archived at
https://mailarchive.ietf.org/arch/browse/v6ops/. Subscribe at
https://www.ietf.org/mailman/listinfo/v6ops/. Working group assignment
(v6ops versus 6man) is to be determined at adoption time.

Source for this draft and an issue tracker can be found at
https://github.com/franckhlmartin/ietf-draft-ipv6-addr-selection-updates.

{mainmatter}

# Introduction

[@!RFC6724] Section 6 notes that Rules 9 and 10 MAY be superseded when an
implementation has other means of sorting destination addresses---for
example, when it somehow knows which destination addresses will result in
the "best" communications performance. The purpose of this document is to
better qualify (based on operational experience) when an implementation MAY supersede those rules and how,
so that behavior is standardized rather than left to ad hoc local
heuristics. This document does not limit operators or implementations:
if other means of sorting produce better results, they remain permitted
and need not be confined to the mechanisms specified here. The
implementations described in this document are believed to benefit
operators in their IPv6 deployments and IPv4 retirement.

## Requirements Language

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be
interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when,
they appear in all capitals, as shown here.

## Motivation

Applications commonly use `getaddrinfo()` [@!RFC3493] to obtain an ordered
list of destination addresses. [@!RFC6724] defines the default destination
address selection algorithm that implementations such as glibc apply when
sorting that list. The rules provide useful interoperability and policy
control on the global Internet, but three operational gaps have emerged:

* **Performance:** Static rules do not reflect observed or recent
  communication performance on a particular host, network, destination,
  interface, or time period. Some of these gaps are already addressed at the
  application layer by connection racing, as specified in Happy Eyeballs
  [@?RFC8305], which tries multiple destination addresses concurrently
  rather than relying solely on resolver ordering.
* **Address pairs:** Sorting destinations without considering the likely
  source/destination pair can yield suboptimal or incorrect connectivity,
  especially when multiple source addresses are available.
* **DNS load balancing:** Operators commonly spread load by returning
  multiple address records and varying their order---DNS-based load
  balancing as described in [@?RFC1794]. Rule 9 ("Use longest matching
  prefix") can collapse that intentionally varied order to a single
  preferred destination, defeating the operator's intent. DNS and routing
  communities have offered conflicting advice on whether resolver ordering
  or DNS response order should prevail; this document closes that gap for
  configured deployments.

## Scope

This document is a focused update to destination address selection
(Section 6 of [@!RFC6724]). Source address selection (Section 5) is out
of scope except where needed to evaluate source/destination pairs for
destination ordering.

## Design Principle

All three enhancements are OPTIONAL. Implementations MUST NOT enable them
by default; each enhancement MUST remain disabled until an operator
explicitly configures it, so current [@!RFC6724] behavior is preserved
without deliberate administrative input.

All three enhancements SHOULD be implementable internally by `getaddrinfo()`
or the equivalent system resolver mechanism. Applications SHOULD NOT need
API changes or source-code modifications to benefit. Experimental evidence
for address-pair-aware ordering appears in [@?GET-ADDR-PAIRS].

# Terminology

**Destination address selection:** The process by which a host orders
candidate destination addresses before returning them to an application,
as defined in [@!RFC6724].

**Address pair:** A combination of a selected or likely source address
and a candidate destination address used to evaluate connectivity or
ordering for a given communication attempt.

# Updates to RFC6724

This section specifies updates to destination address selection in
[@!RFC6724]. Exact rule numbering and ordering relative to existing
Rules 1--10 are for further editor and working group discussion.

## Performance-Aware Destination Selection {#performance-aware}

Implementations MAY use observed or recent communication performance
when ordering destination addresses that are otherwise equal under
Rules 1--8. When applied, this behavior SHOULD integrate with or
replace the tie-breaking role of Rules 9 and 10 for affected candidates.

> TODO: Specify ordering relative to Rules 9 and 10; define what
> "observed or recent performance" means (latency, loss, success rate,
> scope, and retention period).

## Source/Destination Pair Consideration {#address-pairs}

When sorting destination addresses, implementations SHOULD consider the
likely source address that would be used for each candidate destination,
not only the destination in isolation. Ordering SHOULD prefer
source/destination pairs that are more likely to succeed or perform well.

This update aligns with the implementation architecture described in
[@!RFC6724], where `getaddrinfo()` may obtain source-address information
when sorting destinations. [@?GET-ADDR-PAIRS] demonstrates a prototype
approach.

> TODO: Normative text for pair evaluation and interaction with existing
> Rules 2, 5, and 9.

## Preserving DNS Load-Balancing Order {#dns-load-balancing}

Operators who use DNS-based load balancing [@?RFC1794]---for example,
multiple A or AAAA records whose order is rotated by the authoritative
server---expect clients to try addresses in the order returned by DNS.
Rule 9 reorders those candidates by longest matching prefix, which can
concentrate traffic on one backend and undermine the operator's load-spreading intent. DNS operators and routing-oriented guidance have
historically given conflicting advice on this point; this section
standardizes operator-controlled behavior.

Implementations MUST support administrative configuration of one or more
IPv4 and IPv6 prefix ranges for which Rule 9 does not apply. This update
does not change ordering across address families: Rules 1--8, including
default IPv6-over-IPv4 preference, continue to apply unchanged. When Rule
9 would otherwise reorder candidates of the same address family, and a
candidate destination address falls within a configured range, the
implementation MUST preserve the order received from the name-resolution
step (for example, the order of A or AAAA records in the DNS response)
among those same-family candidates, rather than reordering them by
longest matching prefix. This section does not introduce a new within-family sort order; it only prevents Rule 9 from overriding DNS response
order for configured destinations. Configuration mechanisms MAY include a
policy table, `/etc/gai.conf`, or an equivalent system resolver setting; no
application changes are required.

# Implementation and Deployment Considerations

Implementations that apply these updates inside `getaddrinfo()` preserve
compatibility with existing applications that iterate the returned address
list. Operators MAY use existing policy mechanisms such as `/etc/gai.conf`
on glibc-based systems to influence precedence; however, such files alone
do not fully disable Rule 9 today.

Backward compatibility on the global Internet MUST be preserved: with no
operator configuration, implementations MUST behave as [@!RFC6724].
None of the enhancements in this document take effect until explicitly
enabled.

This document is related to, but distinct from, the Enhanced Dual Stack
(EDS) framework [@?EDS]. EDS describes a broader host-side deployment
model; this document normatively updates destination address selection
rules.

# Security Considerations

Performance caches or probes used for address ordering MUST NOT expose
confidential traffic patterns beyond what the host already observes locally.
Implementations SHOULD bound retention of performance state and resist
poisoning of ordering decisions from unauthenticated off-path input.

> TODO: Expand (cache poisoning, timing side channels, probe amplification).

# IANA Considerations

This document has no IANA actions.

# Acknowledgements

Discussion on the v6ops mailing list, including threads around Enhanced
Dual Stack, helped shape the scope.

<reference anchor="EDS" target="https://datatracker.ietf.org/doc/html/draft-xiao-v6ops-eds-01">
  <front>
    <title>Enhanced Dual Stack: Automatic IPv6/IPv4 Selection Based on Performance</title>
    <author initials="X." surname="Xiao" fullname="XiPeng Xiao">
      <organization>Huawei Technologies Dusseldorf</organization>
    </author>
    <date year="2026" month="July" day="4"/>
  </front>
</reference>

<reference anchor="RFC1794" target="https://www.rfc-editor.org/info/rfc1794">
  <front>
    <title>DNS Support for Load Balancing</title>
    <author initials="T." surname="Brisco" fullname="T. Brisco">
    </author>
    <date year="1995" month="April"/>
  </front>
</reference>

<reference anchor="RFC8305" target="https://www.rfc-editor.org/info/rfc8305">
  <front>
    <title>Happy Eyeballs Version 2: Better Connectivity Using Concurrency</title>
    <author initials="D." surname="Schinazi" fullname="D. Schinazi">
    </author>
    <author initials="T." surname="Pauly" fullname="T. Pauly">
    </author>
    <date year="2017" month="December"/>
  </front>
</reference>

<reference anchor="GET-ADDR-PAIRS" target="https://github.com/becarpenter/getapr">
  <front>
    <title>Get Address Pairs for Socket Programming in Python</title>
    <author initials="B." surname="Carpenter" fullname="Brian E. Carpenter">
      <organization>University of Auckland</organization>
    </author>
  </front>
</reference>

{backmatter}
