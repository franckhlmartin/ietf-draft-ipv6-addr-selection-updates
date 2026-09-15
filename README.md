# ietf-draft-ipv6-addr-selection-updates

This repository contains the source for an individual IETF Internet-Draft:
**"Updates to IPv6 Default Address Selection"** (`draft-martin-ipv6-addr-selection-updates`).

Co-authors: Franck Martin, XiPeng Xiao, Brian E. Carpenter.

**Working group:** TBD (v6ops versus 6man to be decided at adoption). Initial
discussion is on the [v6ops mailing list](https://www.ietf.org/mailman/listinfo/v6ops).

**IETF Datatracker:** [draft-martin-ipv6-addr-selection-updates](https://datatracker.ietf.org/doc/draft-martin-ipv6-addr-selection-updates/)

**Editor's copy (HTML):** [franckhlmartin.github.io/ietf-draft-ipv6-addr-selection-updates](https://franckhlmartin.github.io/ietf-draft-ipv6-addr-selection-updates/) ([HTML](https://franckhlmartin.github.io/ietf-draft-ipv6-addr-selection-updates/draft-martin-ipv6-addr-selection-updates.html), [TXT](https://franckhlmartin.github.io/ietf-draft-ipv6-addr-selection-updates/draft-martin-ipv6-addr-selection-updates.txt), [XML](https://franckhlmartin.github.io/ietf-draft-ipv6-addr-selection-updates/draft-martin-ipv6-addr-selection-updates.xml))

**Source of truth:** `draft-martin-ipv6-addr-selection-updates.md` is the only
authoritative source and the only draft file committed to git. Generated
`.xml`, `.txt`, and `.html` files are produced locally by `make` and on GitHub
Actions (idnits plus downloadable artifacts on every push/PR; the editor's copy
is published to GitHub Pages from `main`). Do not commit the generated files.

## Building

Install the toolchain (or let `make` bootstrap mmark and xml2rfc automatically):

- [Go](https://go.dev/dl/) — for mmark
- Python 3.10+ — for xml2rfc (a project `.venv` is created on first build)
- [idnits](https://github.com/ietf-tools/idnits) (Node.js 20+, optional for lint): `npm install -g @ietf-tools/idnits`

Then build from the repository root:

```sh
make          # install missing tools, then generate XML, text, and HTML
make lint     # run idnits on the generated XML
make clean    # remove generated outputs
make clean-all # also remove the local .venv
```

Before submitting, run `make` and upload the generated XML (for example
`draft-martin-ipv6-addr-selection-updates-01.xml`) to the
[IETF Datatracker submission tool](https://datatracker.ietf.org/submit/).
Do not add the generated `.xml`, `.txt`, or `.html` files to the commit.

When editing the Markdown source, use [mmark](https://github.com/mmarkdown/mmark)
conventions: internal links are `(#anchor)` (not `{{anchor}}`), and tables use
`col1 | col2` with a `---|---` header row (not GitHub-style `| col |` tables).

## License

All contributions to this draft are subject to the **IETF Trust Legal Provisions (BCP 78 / RFC 5378)**.
See [https://trustee.ietf.org/license-info](https://trustee.ietf.org/license-info) for details.

## Contributing

Pull requests are welcome! Please ensure your contributions comply with IETF policies.
