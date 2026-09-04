# Changelog

Significant content changes to `draft-martin-ipv6-addr-selection-updates.md` are recorded here.
Formatting-only edits are omitted unless they affect published semantics.

## Unreleased

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
