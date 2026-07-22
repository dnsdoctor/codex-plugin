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
| `enroll_monitoring_trial` | Emails a human a double-opt-in link that creates their free account; monitoring starts once they add the domain and verify it with a TXT record. |

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
