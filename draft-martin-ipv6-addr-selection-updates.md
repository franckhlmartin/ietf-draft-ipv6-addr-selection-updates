%%%
title = "Updates to IPv6 Default Address Selection"
abbrev = "ipv6-addr-selection-updates"
ipr = "trust200902"
updates = [6724]
keyword = ["IPv6", "RFC6724", "address selection", "getaddrinfo", "data center"]

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
source/destination address pairs when sorting candidates, and mitigate
cases where Rule 9 defeats DNS-based load spreading in data-center
environments. All three enhancements are intended to be implementable
internally by `getaddrinfo()` or an equivalent system mechanism, without
changing the existing API or requiring application source-code changes.

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
  interface, or time period.
* **Address pairs:** Sorting destinations without considering the likely
  source/destination pair can yield suboptimal or incorrect connectivity,
  especially when multiple source addresses are available.
* **Data centers:** Rule 9 ("Use longest matching prefix") can collapse
  DNS round-robin or similar load-spreading techniques to a deterministic
  order, concentrating traffic on a single backend.

## Scope

This document is a focused update to destination address selection
(Section 6 of [@!RFC6724]). Source address selection (Section 5) is out
of scope except where needed to evaluate source/destination pairs for
destination ordering.

## Design Principle

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

## Data-Center Destination Selection {#data-center}

In environments where many servers are functionally equidistant and
operators rely on DNS or similar mechanisms to spread load, Rule 9
SHOULD NOT deterministically collapse an intentionally varied address
order to a single preferred destination when administrative policy
indicates a data-center or load-spreading context.

> TODO: Define how implementations detect or configure data-center
> behavior (for example, policy table, `/etc/gai.conf`, or scope
> heuristics) without requiring application changes.

# Implementation and Deployment Considerations

Implementations that apply these updates inside `getaddrinfo()` preserve
compatibility with existing applications that iterate the returned address
list. Operators MAY use existing policy mechanisms such as `/etc/gai.conf`
on glibc-based systems to influence precedence; however, such files alone
do not fully disable Rule 9 today.

Backward compatibility on the global Internet MUST be preserved: default
behavior outside configured or detected data-center contexts SHOULD
remain aligned with [@!RFC6724] unless administrative policy overrides it.

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

XiPeng Xiao proposed co-authoring this focused [@!RFC6724] update.
Brian Carpenter's getapr work informed the source/destination pair
consideration. Discussion on the v6ops mailing list, including threads
around Enhanced Dual Stack, helped shape the scope.

<reference anchor="EDS" target="https://datatracker.ietf.org/doc/html/draft-xiao-v6ops-eds-01">
  <front>
    <title>Enhanced Dual Stack: Automatic IPv6/IPv4 Selection Based on Performance</title>
    <author initials="X." surname="Xiao" fullname="XiPeng Xiao">
      <organization>Huawei Technologies Dusseldorf</organization>
    </author>
    <date year="2026" month="July" day="4"/>
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
