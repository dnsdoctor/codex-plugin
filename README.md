# DNS Doctor — Codex / ChatGPT plugin

Wraps the hosted DNS Doctor MCP server as an OpenAI plugin (Codex + ChatGPT) so
your agent can diagnose and fix a domain's email authentication (SPF, DMARC,
DKIM, MX, blacklist, domain/SSL expiry). Every fix record is generated and
validated by a deterministic engine — RFC grammar plus the SPF 10-lookup
counter — **never an LLM guess**.

## What's inside

```
codex-plugin/
├── .codex-plugin/
│   └── plugin.json          # manifest: metadata + interface + component pointers
├── .mcp.json                # the MCP connector (https://dnsdoctor.dev/mcp)
├── skills/dns-doctor/       # the bundled skill ($-invoked in Codex, @ in ChatGPT)
│   └── SKILL.md
├── LICENSE                  # Apache-2.0
└── README.md
```

> **Schema note:** the Codex plugin manifest format evolves. Validate
> `plugin.json` and `.mcp.json` against the current docs
> (learn.chatgpt.com → Build plugins) before each submission and correct any
> field names that changed — the MCP endpoint (`https://dnsdoctor.dev/mcp`,
> streamable HTTP) is the part that must survive.

## Tools it adds

| Tool | Does |
|---|---|
| `scan_domain` | Fresh scan of a domain; full report. |
| `get_report` | Persisted report (scans once if none exists). |
| `build_dmarc_upgrade` | A validated DMARC enforcement record — `p=reject` only when the server-derived alignment gate passes. |
| `count_spf_lookups` | The SPF DNS-lookup count against the RFC limit of 10. |
| `validate_dmarc_record` | Parse and validate a DMARC record, tag by tag. |
| `generate_dmarc_record` | Build a DMARC record from a policy + reporting address. |
| `check_dkim_selector` | Look up one DKIM selector and check the key. |
| `parse_dmarc_report` | Parse an aggregate (RUA) report file into rows. |
| `check_record` | Read any DNS record type for a name. |
| `check_reverse_dns` | PTR / forward-confirmed reverse DNS for an IP. |
| `audit_spf_includes` | The SPF include/redirect tree — who can transitively send as the domain, with typed findings (broken include, confirmed-unregistered include, expiring registration, nested `+all`). Analysis only; no SPF fix record. |
| `build_parked_domain_records` | The Null MX + `v=spf1 -all` + `p=reject; np=reject` hardening pack for a domain that sends no mail. The server re-checks DNS itself and refuses when it finds evidence of mail. |
| `start_monitoring_signup` | A sign-up link to hand to the human who owns the domain. Sends no email and creates nothing — they open it, sign in on our page themselves (a social provider or an emailed link, whichever that deployment offers), and the domain is carried over to their dashboard already filled in; monitoring starts once they verify it with a TXT record. |

The `dnsdoctor://domains` resource (your monitored domains) is always listed;
reading it needs an API token and is refused without one. Anonymous access is
enough for a one-off diagnosis.

## Optional: API token for monitored domains

Anonymous access covers scanning and fixes. For the `dnsdoctor://domains`
resource: sign in at <https://dnsdoctor.dev> → **Settings → API tokens** →
create a token (the `dnsd_…` plaintext is shown once) and add it as an
`Authorization: Bearer` header on the connector. Prefer an environment variable
over a committed literal; never commit the token.

## Learn more

- **Methodology:** <https://dnsdoctor.dev/methodology>
- **REST API / OpenAPI schema:** <https://dnsdoctor.dev/api/v1/openapi.json>

## License

[Apache-2.0](./LICENSE).
