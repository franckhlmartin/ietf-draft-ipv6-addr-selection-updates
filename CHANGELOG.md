# Changelog

Significant content changes to `draft-martin-ipv6-addr-selection-updates.md` are recorded here.
Formatting-only edits are omitted unless they affect published semantics.

## Unreleased

- **Build/nits:** Replace leftover `{{anchor}}` xrefs with mmark `(#anchor)` links;
  cite syslog as informative [RFC5424]; set document date to 12 September 2026;
  bold remaining RFC 2119 keywords (`MUST`/`SHOULD`/`MAY`) so they render as
  `<bcp14>`.

- **getaddrinfo() / POSIX:** Restore `getaddrinfo()` in the Abstract, Motivation,
  Design Principle, address-pair section, and Implementation Considerations as the
  standard host API (with "or equivalent"), and add informative [POSIX]
  (IEEE Std 1003.1-2024) alongside [RFC3493].

- **Connectivity-informed selection (§3.1):** Replace performance-aware placeholder with
  normative update to Rule 6 ("Prefer higher precedence unless recently failed"); define
  recent-failure records (network context, destination, source, service), 10-minute
  expiry aligned with [RFC6555], and clearance on success or network-context change.
- **Terminology:** Add recent-failure state, network context, and service; cite Happy
  Eyeballs Version 3 [HAPPY-HEV3] for service-binding examples.
- **Abstract / Introduction / Motivation:** Reframe around recent IPv6 connection or
  service failures and [RFC6724] Section 10.3.1 timeouts rather than generic performance
  sorting or Rules 9/10 supersession.
- **Design principle:** Enhancements are independent and individually configurable;
  implementable within existing host networking mechanisms without API changes.
- **DNS load balancing (§3.3):** Clarify interaction with updated Rule 6 when
  connectivity-informed selection is enabled.
- **Operational diagnostics:** New subsection for aggregating recent-failure records for
  network management.
- **Security Considerations:** Rewrite for recent-failure state poisoning, lifetime, and
  privacy of exported diagnostics.
- **References:** Add [RFC6555], [HAPPY-HEV3], and proper citation for [RFC3493].

- **Abstract:** Clarify that the DNS load-balancing update is for ISP and enterprise
  operators, not only ISPs.
- **Acknowledgements:** Remove co-authors XiPeng Xiao and Brian Carpenter; keep thanks
  to the v6ops mailing list.

- **Introduction:** Frame the draft against [RFC6724] Section 6 (Rules 9 and 10 MAY be
  superseded); qualify scope based on operational experience; clarify that operators are
  not limited to the mechanisms defined here; note expected benefit for IPv6 deployment
  and IPv4 retirement.
- **Performance motivation:** Cite Happy Eyeballs [RFC8305] as application-layer
  mitigation for performance-based ordering gaps.
- **Design principle:** All three enhancements are OPTIONAL and off by default until
  explicitly configured by an operator.
- **DNS load balancing (§3.3):** Rename from data-center framing; cite [RFC1794]; describe
  DNS versus routing guidance conflict; require configurable IPv4/IPv6 prefix ranges where
  Rule 9 is skipped; preserve same-family DNS response order without changing Rules 1--8
  (including IPv6-over-IPv4 preference).
- **References:** Add informative references for [RFC1794] and [RFC8305].
- **Repository:** Add Cursor `/commit` workflow (`.cursor/commands/commit.md`) and
  `scripts/commit-check-changes.sh`.

- **Initial skeleton (-00):** Repository scaffold, mmark/xml2rfc toolchain, and
  placeholder sections for performance-aware selection, source/destination pair
  consideration, and data-center Rule 9 behavior.
- **Authors:** Franck Martin, XiPeng Xiao, Brian E. Carpenter.
- **Stream/WG:** Individual submission; working group assignment TBD.
