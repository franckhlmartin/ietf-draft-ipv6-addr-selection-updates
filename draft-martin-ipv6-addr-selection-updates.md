%%%
title = "Updates to IPv6 Default Address Selection"
abbrev = "ipv6-addr-selection-updates"
ipr = "trust200902"
updates = [6724]
keyword = ["IPv6", "RFC6724", "address selection", "getaddrinfo", "DNS load balancing"]

date = 2026-09-08

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
This document updates RFC 6724 with three improvements to IPv6 destination
address selection. The updates allow recent IPv6 connection or service
failures to influence IPv6/IPv4 ordering, incorporate likely
source/destination address pairs when sorting candidates, and let ISP and
enterprise operators preserve DNS load-balancing order where Rule 9 would
otherwise override it. The updates are intended to be implementable inside
`getaddrinfo()` or an equivalent system mechanism, without requiring changes
to existing application-facing socket APIs.

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

[@!RFC6724] defines default source and destination address selection for IPv6
and dual-stack hosts. Operational experience has identified cases where
additional information available to a host or operator can improve destination
ordering. In particular, Section 10.3.1 of [@!RFC6724] notes that a host with
working IPv4 connectivity but broken IPv6 connectivity can experience unwanted
timeouts because the default policy normally prefers IPv6.

This document specifies three updates addressing recent IPv6 connection or
service failures, source/destination address-pair considerations, and DNS-based
load balancing. The implementations described in this document are believed to
benefit operators in their IPv6 deployments and IPv4 retirement.

## Requirements Language

The key words "**MUST**", "**MUST NOT**", "**REQUIRED**", "**SHALL**",
"**SHALL NOT**", "**SHOULD**", "**SHOULD NOT**", "**RECOMMENDED**",
"**NOT RECOMMENDED**", "**MAY**", and "**OPTIONAL**" in this document are to be
interpreted as described in BCP 14 [@!RFC2119] [@!RFC8174] when, and only when,
they appear in all capitals, as shown here.

## Motivation

Applications commonly use `getaddrinfo()` [@?POSIX] [@?RFC3493] to obtain an
ordered list of destination addresses. [@!RFC6724] defines the default
destination address selection algorithm that implementations apply when sorting
that list. The rules provide useful interoperability and policy control on the
global Internet, but three operational gaps have emerged:

* **Connectivity:** Static precedence can continue to prefer IPv6 after an IPv6
  connection or service-establishment attempt has recently failed. Happy
  Eyeballs [@?RFC8305] reduces the impact for applications that race IPv6 and
  IPv4 attempts, but other applications can repeatedly experience the same
  failure or delay.
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

The three enhancements are **OPTIONAL**, independent, and individually
configurable. Implementations **MUST NOT** enable any of them by default, and
enabling one enhancement **MUST NOT** implicitly enable another, so current
[@!RFC6724] behavior is preserved without deliberate administrative input.

The enhancements **SHOULD** be implementable inside `getaddrinfo()` or an
equivalent system resolver mechanism. This document does not require changes to
existing application-facing socket APIs or application source code. Experimental
evidence for address-pair-aware ordering appears in [@?GET-ADDR-PAIRS].

# Terminology

**Destination address selection:** The process by which a host orders
candidate destination addresses before returning them to an application,
as defined in [@!RFC6724].

**Address pair:** A combination of a selected or likely source address
and a candidate destination address used to evaluate connectivity or
ordering for a given communication attempt.

**Recent-failure state:** Host-local state recording a recent IPv6 connection
or service-establishment failure that can be consulted by destination address
selection.

**Network context:** Information identifying the network environment in which
a connection attempt is made, such as the outgoing interface, network
attachment, VPN, or equivalent implementation-specific context.

**Service:** The transport and port and, when available, higher-layer protocol
or service-binding information associated with an attempt. For example, the
service can be represented as `TCP/443`, `HTTPS/TCP/443`, or
`HTTPS/h3/QUIC/443`. Happy Eyeballs Version 3 [@?HAPPY-HEV3] similarly extends
connection establishment beyond transport-only information when such service
information is available.

# Updates to RFC6724

This section specifies updates to destination address selection in
[@!RFC6724].

## Connectivity-Informed Destination Selection {#connectivity-informed}

Connectivity-informed destination address selection is enabled by updating
Rule 6 of [@!RFC6724] from "Prefer higher precedence" to "Prefer higher
precedence unless recently failed".

The updated rule is:

> **Rule 6: Prefer higher precedence unless recently failed.**
>
> When exactly one of DA and DB is an IPv6 destination and the other is an
> IPv4 destination, if applicable recent-failure state exists for the IPv6
> destination, prefer the IPv4 destination.
>
> Otherwise, if Precedence(DA) > Precedence(DB), then prefer DA. Similarly, if
> Precedence(DA) < Precedence(DB), then prefer DB.

When this enhancement is enabled, the component that determines that an IPv6
connection or service-establishment attempt has failed creates or refreshes a
recent-failure record. The logical record contains the following fields:

| Field | Example |
|---|---|
| Network context | Corp-WiFi-A |
| IPv6 destination | 2001:db8:1234:5678::42 |
| IPv6 source | 2001:db8:1111::123 |
| Service | TCP/443 |
| Failure type | timeout |
| Timestamp | 2026-09-07T14:32:18+02:00 |

The first four fields identify the scope to which the observation applies. An
entry is applicable when its network context, IPv6 destination, IPv6 source,
and service match the current candidate as far as those fields are known. The
`IPv6 source` and `Service` fields **SHOULD** be recorded when available. The
Service field can contain only transport-level information or can include
higher-layer protocol or service-binding information when available.

A failed attempt to establish a usable instance of the identified service,
such as a timeout, unreachable indication, connection refusal, or an
applicable higher-layer establishment failure, creates or refreshes
recent-failure state. Local API or resource errors unrelated to the attempted
IPv6 communication do not. No active probing or measurement of RTT, packet
loss, throughput, or other performance statistics is required.

Recent-failure state **SHOULD** expire 10 minutes after the most recent
applicable failure. This interval follows the stateful Happy Eyeballs guidance
in Section 4.2 of [@?RFC6555], which recommends retrying a failed preferred
address family every 10 minutes and notes that this can be implemented by
flushing state every 10 minutes. Implementations **MAY** make the interval
configurable. A subsequent successful establishment of the same service over
the same applicable IPv6 context **SHOULD** clear the state, and a relevant
network-context change **SHOULD** make the state inapplicable.

In the absence of applicable recent-failure state, Rule 6 behaves as specified
in [@!RFC6724], so IPv6 retains its normal default preference. This enhancement
changes destination ordering only; it does not remove IPv6 addresses from the
candidate list. Happy Eyeballs and other connection-racing mechanisms can
therefore continue to attempt both address families.

Implementations **MAY** expose recent-failure information to network-management
systems for aggregation and operational diagnosis, as discussed in
{{operational-diagnostics}}.

## Source/Destination Pair Consideration {#address-pairs}

When sorting destination addresses, implementations SHOULD consider the
likely source address that would be used for each candidate destination,
not only the destination in isolation. Ordering SHOULD prefer
source/destination pairs that are more likely to succeed or perform well.
This update aligns with the implementation architecture described in
[@!RFC6724], where `getaddrinfo()` may obtain source-address information when
sorting destinations. [@?GET-ADDR-PAIRS] demonstrates a prototype approach.

> TODO: Normative text for pair evaluation and interaction with existing
> Rules 2, 5, and 9.

## Preserving DNS Load-Balancing Order {#dns-load-balancing}

Operators who use DNS-based load balancing [@?RFC1794]---for example,
multiple A or AAAA records whose order is rotated by the authoritative
server---expect clients to try addresses in the order returned by DNS.
Rule 9 reorders those candidates by longest matching prefix, which can
concentrate traffic on one backend and undermine the operator's
load-spreading intent. DNS operators and routing-oriented guidance have
historically given conflicting advice on this point; this section
standardizes operator-controlled behavior.

Implementations MUST support administrative configuration of one or more
IPv4 and IPv6 prefix ranges for which Rule 9 does not apply. This update
does not itself change ordering across address families: Rules 1--8 continue
to apply before Rule 9. If the enhancement in {{connectivity-informed}} is
enabled, Rule 6 is applied as updated there; otherwise Rule 6 remains as
specified in [@!RFC6724]. When Rule 9 would otherwise reorder candidates of
the same address family, and a candidate destination address falls within a
configured range, the implementation MUST preserve the order received from
the name-resolution step (for example, the order of A or AAAA records in the
DNS response) among those same-family candidates, rather than reordering them
by longest matching prefix. This section does not introduce a new within-family
sort order; it only prevents Rule 9 from overriding DNS response
order for configured destinations. Configuration mechanisms MAY include a
policy table, `/etc/gai.conf`, or an equivalent system resolver setting; no
application changes are required.

# Implementation and Deployment Considerations

Implementations that apply these updates inside `getaddrinfo()` [@?POSIX]
[@?RFC3493] or an equivalent system mechanism preserve compatibility with
existing applications that use the returned address ordering. For connectivity-informed
selection, the implementation needs a mechanism to retain recent connection or
service-establishment failures for later use by destination selection; no new
application-facing API is required.

Operators MAY use existing policy mechanisms such as `/etc/gai.conf` on
glibc-based systems to influence precedence; however, such files alone do not
fully disable Rule 9 today. The connectivity-informed enhancement does not
require dynamic modification of the RFC 6724 policy table; the existing policy
table continues to express address preference, while recent-failure state
qualifies the Rule 6 preference when applicable.

Backward compatibility on the global Internet MUST be preserved: with no
operator configuration, implementations MUST behave as [@!RFC6724].

This document is related to, but distinct from, the Enhanced Dual Stack
(EDS) framework [@?EDS]. EDS describes a broader host-side deployment
model; this document normatively updates destination address selection
rules.

## Operational Diagnostics {#operational-diagnostics}

Implementations MAY emit recent-failure records defined in {{connectivity-informed}} to a system logging or telemetry facility (for example, syslog [RFC5424]) as an aid to operational diagnosis. Such records SHOULD preserve the per-event fields defined in {{connectivity-informed}}. Logging is independent of the recent-failure state used by Rule 6; expiration or clearing of that state does not require deletion of corresponding log messages.

Retention, forwarding, filtering, and aggregation of these messages are matters of local policy and are outside the scope of this document. Operators MAY use existing log-management or network-management systems to collect and analyze them across hosts and time.

# Security Considerations

Recent-failure state influences address ordering and could temporarily cause
IPv4 to be preferred over IPv6. Implementations SHOULD derive this state from
connection or service-establishment outcomes observed locally by the host or a
trusted host networking component, and SHOULD resist poisoning of ordering
decisions from unauthenticated off-path input. The bounded lifetime specified
in {{connectivity-informed}} prevents a transient failure from suppressing the
normal IPv6 preference indefinitely.

Recent-failure records and aggregated diagnostics can reveal destinations,
source addresses, services, network attachments, and traffic patterns. Access
to such information SHOULD be restricted to authorized entities, and exported
information SHOULD be minimized according to operational need and protected
according to local security and privacy policy.

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
<reference anchor="POSIX" target="https://pubs.opengroup.org/onlinepubs/9799919799/functions/getaddrinfo.html">
  <front>
    <title>IEEE/Open Group Standard for Information Technology -- Portable Operating System Interface (POSIX(TM)) Base Specifications, Issue 8</title>
    <author>
      <organization>IEEE and The Open Group</organization>
    </author>
    <date year="2024"/>
  </front>
  <seriesInfo name="IEEE Std" value="1003.1-2024"/>
</reference>
<reference anchor="RFC1794" target="https://www.rfc-editor.org/info/rfc1794">
  <front>
    <title>DNS Support for Load Balancing</title>
    <author initials="T." surname="Brisco" fullname="T. Brisco">
    </author>
    <date year="1995" month="April"/>
  </front>
</reference>
<reference anchor="RFC6555" target="https://www.rfc-editor.org/info/rfc6555">
  <front>
    <title>Happy Eyeballs: Success with Dual-Stack Hosts</title>
    <author initials="D." surname="Wing" fullname="D. Wing">
    </author>
    <author initials="A." surname="Yourtchenko" fullname="A. Yourtchenko">
    </author>
    <date year="2012" month="April"/>
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
<reference anchor="HAPPY-HEV3" target="https://datatracker.ietf.org/doc/draft-ietf-happy-happyeyeballs-v3/">
  <front>
    <title>Happy Eyeballs Version 3: Better Connectivity Using Concurrency</title>
    <author initials="T." surname="Pauly" fullname="Tommy Pauly">
    </author>
    <author initials="D." surname="Schinazi" fullname="David Schinazi">
    </author>
    <author initials="N." surname="Jaju" fullname="Nidhi Jaju">
    </author>
    <author initials="K." surname="Ishibashi" fullname="Kenichi Ishibashi">
    </author>
    <date year="2026" month="July" day="2"/>
  </front>
  <seriesInfo name="Internet-Draft" value="draft-ietf-happy-happyeyeballs-v3-04"/>
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

