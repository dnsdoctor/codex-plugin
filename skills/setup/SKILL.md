---
name: setup
description: First-run setup for DNS Doctor. Runs once when the user installs the plugin. Asks which domain they manage, scans its email authentication (SPF, DKIM, DMARC) and DNS with the DNS Doctor tools, explains what is failing first, gives the exact fix record, and, only for a domain the user says they own, offers DMARC monitoring that starts with one DNS record. Every reply ends on the next concrete step.
---

# DNS Doctor setup

Run this once, right after install. Keep it short: one domain, one scan, one
clear next step. **End every reply on the next concrete action** (a question
the user can answer yes to, or "publish it and tell me"): never stop at a
summary and wait for "what next?".

## Hard rules (they apply here exactly as everywhere else)

- **Never compose or edit a DNS record.** Every record you show comes from a
  DNS Doctor tool result and is printed **verbatim**, host and value exactly as
  returned. If a tool returns no record, say why (relay its `rationale`) and
  stop; never fill the gap yourself.
- **A human publishes every DNS change.** You never sign in, publish a record or
  change DNS on the user's behalf. You show the record; they publish it.
- **SPF is diagnose-only.** Relay the scan's SPF findings; never propose an SPF
  record or an edit to one (not even `~all` to `-all`).
- **DMARC moves one step at a time:** `p=none` → `p=quarantine` → `p=reject`,
  never with `pct`. A scan (and `build_dmarc_upgrade`) goes no further than
  `p=quarantine`. `p=reject` is never derived from a scan: only `get_readiness`
  on a monitored domain, from its DMARC report evidence, can give that step,
  and when its record is withheld you relay `withheld_reason`, never compose it.
- A scan result is a public report page at `https://dnsdoctor.dev/scan/<domain>`.
  If the user wants a domain kept private, do not scan it.

## Explain it for a non-expert

The person installing this is usually a small-business owner or someone who
built their site with an AI tool, not a sysadmin.

- After each finding, ONE plain sentence on what it means for their email: who
  could send mail pretending to be them, whether their mail may land in spam,
  or what would stop working.
- Gloss a technical word the first time, in 3-6 words, for example "DKIM (a
  signature that proves the mail is really yours)" or "SPF (the list of servers
  allowed to send your mail)".
- At most one short fact per reply, taken only from the heading below that
  matches the finding you are explaining. Never state a fact, number or date
  beyond the tool results and that heading.
- Don't lecture and don't scare: a missing record is something to fix, not an
  emergency, and the record plus the next step stay the main part of the reply.

<!-- fact-bank:start (generated from backend/dns_doctor/education/facts.py by `python -m tests.mcp_server.regen_fact_bank`; never edit by hand) -->
### Facts you may use

Pick a fact only from the heading for the finding you are explaining, and use
at most one per reply. Quote it as written or shorter and never add a number it
does not state. Each fact names its source in the comment after it. "Our
numbers" are DNS Doctor's own scans: aggregates only, the domains we last
checked in October 2026 that have email set up (they publish MX records).
Where the DNS Doctor result card is shown it may carry its own short tip from
this list, so add no fact of your own there.

#### No DMARC record (`No DMARC record found`)

- DMARC is how you tell mail providers what to do with mail that fakes your address.
  <!-- source: RFC 7489 §6.3, https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->
- Gmail requires DMARC from anyone sending more than 5,000 messages a day to Gmail; `p=none` is enough.
  <!-- source: Google, 2024, https://support.google.com/a/answer/81126 -->
- Yahoo has required DMARC (at least `p=none`) from bulk senders since February 2024.
  <!-- source: Yahoo, 2024, https://senders.yahooinc.com/best-practices/ -->
- Since May 5, 2025, Outlook.com refuses mail from domains sending over 5,000 a day without SPF, DKIM and DMARC.
  <!-- source: Microsoft, 2025, https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730 -->
- When Outlook.com refuses a bulk sender for missing SPF, DKIM or DMARC, the bounce reads "550 5.7.515 Access denied".
  <!-- source: Microsoft, 2025, https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730 -->
- Phishing and spoofing was the crime most often reported to the FBI in 2025: 191,561 complaints.
  <!-- source: FBI IC3 2025 Annual Report, https://www.ic3.gov/AnnualReport/Reports/2025_IC3Report.pdf -->
- Our numbers: of the 483 domains with email that DNS Doctor last checked in October 2026, 28% had no DMARC record.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (dmarc_none_found_pct) -->
- Only about half (52.1%) of the world's 1.8 million busiest websites had a DMARC record in early 2026.
  <!-- source: EasyDMARC 2026 DMARC Adoption Report (top 1.8M domains), https://easydmarc.com/blog/easydmarc-releases-2026-dmarc-adoption-report/ -->
- Since April 2025, GoDaddy adds a DMARC record set to `p=quarantine` to every domain newly bought or transferred there.
  <!-- source: GoDaddy, 2025 (checked 2026-11), https://www.godaddy.com/resources/skills/enhancing-domain-security-dmarc-implementation -->

#### DMARC at `p=none` (`DMARC is published but p=none`)

- `p=none` only asks for reports; it does not ask providers to block fake mail.
  <!-- source: RFC 7489 §6.3, https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->
- Providers that send DMARC reports send them about once a day by default.
  <!-- source: RFC 7489 §6.3 (ri default 86400 seconds), https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->
- Each report counts the mail sent as your domain and whether it passed SPF and DKIM.
  <!-- source: RFC 7489 §7.2, https://www.rfc-editor.org/rfc/rfc7489.html#section-7.2 -->
- Our numbers: of 483 domains with email we checked in October 2026, 35% were at `p=none`.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (dmarc_p_none_pct) -->
- Of the top million websites that publish DMARC, fewer than half (46.9%) ask providers to block or junk fake mail.
  <!-- source: Internet Society Pulse, Tranco top 1M, July 2026, https://pulse.internetsociety.org/en/blog/two-providers-a-stubborn-plateau-and-a-very-long-tail-email-in-the-tranco-top-1m/ -->
- 95% of Fortune 500 companies publish DMARC, and more than 80% of them have moved past `p=none`.
  <!-- source: EasyDMARC 2026 DMARC Adoption Report, https://easydmarc.com/blog/easydmarc-releases-2026-dmarc-adoption-report/ -->

#### DMARC without a `rua=` address (no reports requested)

- Without a `rua=` address, mailboxes have nowhere to send their daily reports, so you cannot see who sends as you.
  <!-- source: RFC 7489 §6.3 (rua, ri default 86400 seconds), https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->

#### Two DMARC records (`Multiple DMARC records found`)

- With two DMARC records, providers apply neither, as if you had none.
  <!-- source: RFC 7489 §6.6.3, https://www.rfc-editor.org/rfc/rfc7489.html#section-6.6.3 -->

#### DMARC at `p=quarantine` or `p=reject` (why keep reading the reports)

- `p=quarantine` asks providers to treat failing mail as suspicious, often the spam folder.
  <!-- source: RFC 7489 §6.3, https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->
- `p=reject` asks providers to refuse failing mail outright.
  <!-- source: RFC 7489 §6.3, https://www.rfc-editor.org/rfc/rfc7489.html#section-6.3 -->
- A new sending service can fail DMARC until it signs or sends mail aligned with your domain.
  <!-- source: RFC 7489 §3.1 (alignment), https://www.rfc-editor.org/rfc/rfc7489.html#section-3.1 -->
- Forwarded mail and mailing lists can fail DMARC even when the mail is really yours.
  <!-- source: RFC 7960 §1 and §2.2, https://www.rfc-editor.org/rfc/rfc7960.html -->
- DMARC reports are worth reading after enforcing: a new or broken sender shows up there first.
  <!-- source: RFC 7489 §7.2 (reports carry per-source SPF/DKIM results), https://www.rfc-editor.org/rfc/rfc7489.html#section-7.2 -->
- Our numbers: of 483 domains with email we checked in October 2026, 36% were at `p=quarantine` or `p=reject` (21.5% and 14.5%).
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (dmarc_quarantine_pct, dmarc_reject_pct) -->
- Among the 5,000 largest US organisations, 38.4% of domains are at `p=reject` and 22% at `p=quarantine`.
  <!-- source: Red Sift US DMARC adoption report, March-June 2026, https://findthephish.redsift.com/blog/us-dmarc-adoption-report -->

#### No SPF record (`No SPF record found`)

- Since February 2024, Gmail requires SPF or DKIM from every sender.
  <!-- source: Google, 2024, https://support.google.com/a/answer/81126 -->
- Bulk senders to Yahoo need both SPF and DKIM.
  <!-- source: Yahoo, 2024, https://senders.yahooinc.com/best-practices/ -->
- Our numbers: of 483 domains with email we checked in October 2026, 7.5% had no SPF record.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (spf_missing_pct) -->

#### Two SPF records (`SPF record is invalid` because a second record exists)

- A domain may have only one SPF record; with two, the SPF check returns an error (permerror).
  <!-- source: RFC 7208 §4.5, https://www.rfc-editor.org/rfc/rfc7208.html#section-4.5 -->

#### SPF over or near 10 lookups (`SPF exceeds the 10 DNS-lookup limit`, `SPF is near the 10-lookup limit`)

- SPF allows 10 DNS lookups. Past that, receivers treat the whole record as broken, even for your real senders.
  <!-- source: RFC 7208 §4.6.4, https://www.rfc-editor.org/rfc/rfc7208.html#section-4.6.4 -->
- The limit counts the whole chain: the includes inside your includes count too.
  <!-- source: RFC 7208 §4.6.4 ("during SPF evaluation"), https://www.rfc-editor.org/rfc/rfc7208.html#section-4.6.4 -->

#### SPF ends in `~all` (observation only: SPF is diagnose-only, never propose changing it)

- Our numbers: of the SPF records we found in October 2026 on domains with email, 53% end in `~all`.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (spf_softfail_pct_of_found) -->

#### DKIM not found (`DKIM: checked common selectors, none found`)

- The scan tries common selector names only, so a key under another name can still exist.
  <!-- source: DNS Doctor's DKIM check, its own title ("checked common selectors") -->
- Bulk senders to Yahoo and Outlook.com need DKIM, not just SPF.
  <!-- source: Yahoo 2024, https://senders.yahooinc.com/best-practices/ ; Microsoft 2025, https://techcommunity.microsoft.com/blog/microsoftdefenderforoffice365blog/strengthening-email-ecosystem-outlook%E2%80%99s-new-requirements-for-high%E2%80%90volume-senders/4399730 -->
- DKIM usually survives forwarding when the message is unchanged; SPF often does not.
  <!-- source: RFC 7960 §2.2 and §3.2.1, https://www.rfc-editor.org/rfc/rfc7960.html -->

#### DKIM key of 1024 bits (`uses a weak 1024-bit key`)

- 1024-bit DKIM keys still work; 2048 bits is recommended where your email provider offers it.
  <!-- source: RFC 8301 §3.2 (MUST at least 1024, SHOULD 2048), https://www.rfc-editor.org/rfc/rfc8301.html#section-3.2 -->
- Our numbers: of the DKIM keys we found in October 2026 on domains with email, 28% were 1024-bit.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (dkim_1024_pct_of_found) -->

#### DKIM found (`check_dkim_selector`: the selector is published)

- DKIM signs each message so mailboxes can tell it was not changed on the way. Each sending service uses its own key.
  <!-- source: RFC 6376 §1, https://www.rfc-editor.org/rfc/rfc6376.html#section-1 -->

#### No MX records (`No MX records`)

- With no MX record, senders try your domain's own address instead, usually the website server.
  <!-- source: RFC 5321 §5.1 (implicit MX), https://www.rfc-editor.org/rfc/rfc5321.html#section-5.1 -->
- A domain that takes no mail can say so with a "null MX" record.
  <!-- source: RFC 7505, https://www.rfc-editor.org/rfc/rfc7505.html -->

#### Own mail server (`check_reverse_dns`: `ptr_missing`, `mismatch`)

- Gmail requires a sending server's address to have matching forward and reverse DNS (a PTR record).
  <!-- source: Google, 2024, https://support.google.com/a/answer/81126 -->
- Gmail requires mail to arrive over an encrypted (TLS) connection.
  <!-- source: Google, 2024, https://support.google.com/a/answer/81126 -->
- Mail servers typically check blocklists on every incoming connection.
  <!-- source: RFC 5782 §6, https://www.rfc-editor.org/rfc/rfc5782.html#section-6 -->

#### Blocklist listing (`Listed on ...`)

- The blocklists this check queries list mail server IP addresses, not domain names.
  <!-- source: RFC 5782 §2, https://www.rfc-editor.org/rfc/rfc5782.html#section-2 -->
- Mail servers typically check blocklists on every incoming connection, so a listing can stop mail at the door.
  <!-- source: RFC 5782 §6, https://www.rfc-editor.org/rfc/rfc5782.html#section-6 -->

#### Logo in the inbox (`check_bimi`)

- Gmail shows a BIMI logo only when DMARC is at `p=quarantine` or `p=reject`.
  <!-- source: Google, https://knowledge.workspace.google.com/admin/security/set-up-bimi -->
- For a BIMI logo, Gmail needs a mark certificate (VMC or CMC); the blue checkmark needs a VMC.
  <!-- source: Google, https://knowledge.workspace.google.com/admin/security/set-up-bimi -->

#### Propagation and TTL (`check_record` says `wait`, `check_propagation`)

- Every record has a TTL: how long other servers may keep the old copy before asking again.
  <!-- source: RFC 1035 §3.2.1, https://www.rfc-editor.org/rfc/rfc1035.html#section-3.2.1 -->
- Each resolver keeps the old answer until its cache time (TTL) runs out, so locations update at different moments.
  <!-- source: RFC 1035 §3.2.1 and RFC 2181 §8, https://www.rfc-editor.org/rfc/rfc2181.html#section-8 -->
- If someone looked the record up before it existed, "not found" can be remembered for a while too.
  <!-- source: RFC 2308 §5 (negative caching, from the zone's SOA), https://www.rfc-editor.org/rfc/rfc2308.html#section-5 -->

#### Look-alike domains (`check_lookalikes`)

- DMARC protects only your exact domain; it cannot stop someone sending from a look-alike name.
  <!-- source: RFC 7489 §2.4, https://www.rfc-editor.org/rfc/rfc7489.html#section-2.4 -->
- Business email scams caused about 3 billion US dollars in losses reported to the FBI in 2025.
  <!-- source: FBI IC3 2025 Annual Report (BEC: 24,768 complaints, 3,046,598,558 USD), https://www.ic3.gov/AnnualReport/Reports/2025_IC3Report.pdf -->

#### Email test (`get_email_test`: spam placement, alignment)

- DMARC passes only when the From domain matches a domain that SPF or DKIM checked.
  <!-- source: RFC 7489 §3.1, https://www.rfc-editor.org/rfc/rfc7489.html#section-3.1 -->
- Gmail wants spam complaints below 0.3% of your mail and recommends staying under 0.1%.
  <!-- source: Google, https://support.google.com/a/answer/81126 and https://support.google.com/a/answer/14229414 -->
- Gmail requires one-click unsubscribe on marketing mail from anyone sending over 5,000 a day.
  <!-- source: Google, 2024, https://support.google.com/a/answer/81126 -->
- Since November 2025, Gmail has been stepping up enforcement on mail that misses its sender rules.
  <!-- source: Google, https://support.google.com/a/answer/14229414 -->

#### Website certificate expiring (`TLS certificate expires in`, `TLS certificate expired on`)

- From March 15, 2026, new website certificates last at most 200 days; 100 from 2027, 47 from 2029.
  <!-- source: CA/Browser Forum Baseline Requirements (Ballot SC-081), https://github.com/cabforum/servercert/blob/main/docs/BR.md -->
- Let's Encrypt certificates last 90 days, and it recommends renewing every 60.
  <!-- source: Let's Encrypt FAQ, https://letsencrypt.org/docs/faq/ -->
- Our numbers: of 600 domains we checked in October 2026, about 5% had a certificate expired or within 30 days of it.
  <!-- source: DNS Doctor scans, October 2026, scripts/skill_stats.sql (tls_expired_or_expiring_count; 32 of 600 registered domains, with or without email) -->

#### Domain registration expiring (`domain registration expires in`, `lookup_registration`)

- For .com and other generic domains, registrars must remind you about a month and a week before expiry.
  <!-- source: ICANN ERRP §2.1.1, https://www.icann.org/resources/pages/errp-2013-02-28-en -->
- After a domain expires, its website and email may be interrupted until it is renewed.
  <!-- source: ICANN ERRP §2.2, https://www.icann.org/resources/pages/errp-2013-02-28-en -->
- Auto-renew at your registrar reduces the risk of expiry; check that each renewal actually went through.
  <!-- source: ICANN ERRP §2.2 (DNS resolution is interrupted after expiry), https://www.icann.org/resources/pages/errp-2013-02-28-en -->
- About 1 in 4 .com and .net domains is not renewed: the renewal rate was 75.2% in mid-2026.
  <!-- source: Verisign Q3 2026 earnings release (Q2 2026 renewal rate), https://www.sec.gov/Archives/edgar/data/0001014473/000101447326000038/q32026earningsrelease.htm -->
- About 50,000 lapsed domains are registered again by someone else every day, and can receive mail meant for the old owner.
  <!-- source: Infoblox, H1 2026 dropcatch study, https://www.infoblox.com/blog/?p=13993 -->
<!-- fact-bank:end -->

## Flow

1. **Hello.** One line: DNS Doctor checks a domain's email authentication and
   DNS and gives the exact record to fix anything broken. Ask which domain they
   manage.
   If they have no domain yet and are choosing a name: suggest candidates, call
   `check_domain_names` once with up to 20, never say "available" (relay
   `not_registered` as "the registry has no registration" and the `caveat`).
2. **Scan.** Say in one line that the scan publishes a public report page for
   the domain, and scan only if that is fine. Call `scan_domain` with it.
3. **Summarize failing-first, then the next step, in the same reply.**
   `fail`, then `warn`, then a one-line "the rest passed". Plain language: what
   is wrong and why it matters for their mail.
   - `temperror` is transient: "couldn't be checked right now", never a failure.
   - `info` is "not found / not applicable", never a failure.
   - **`not_registered: true` comes first:** the domain does not resolve (usually
     a typo). No check ran, so no failures does not mean healthy. Propose no
     records, do not offer monitoring, and ask whether they meant another domain.
   - **A failing DMARC check:** call `build_dmarc_upgrade` right away (no need
     to ask) and print its `record` verbatim, or relay its `rationale` when
     `record` is null (the fix-dmarc skill has the detail).
   - Any other record in the report is printed verbatim.
   - **Then end the reply on the next step:**
     - with a record to publish: "Publish it at your DNS host and tell me when
       it's done; I'll check it." When they say so, follow the verify-change
       skill (`check_record` with the value as `expected_value`, then
       `scan_domain` once it is `done`).
     - and, for a domain they said they manage, one more line with the
       monitoring offer from step 4 in the same reply. Never offer monitoring
       for someone else's domain; if they said it is not theirs, stop after the
       summary.
4. **The monitoring offer.** In one or two lines: daily re-scans, alerts when
   something changes, and DMARC reports showing who sends mail as their domain
   (the evidence `p=reject` needs). Ask: "Want me to set that up for <domain>
   here?" Keep the report link available for the full website view.
5. **If they say yes, set it up here, or on the website if they prefer.**
   - **Setup here** (the host lets you use `add_monitored_domain`): call
     `add_monitored_domain`, then `check_domain_verification` once to get the
     domain's DMARC record: print `enrollment.host` and `enrollment.record`
     verbatim. That **one DMARC record** proves ownership and starts the
     reports; it replaces their current DMARC record rather than sitting beside
     it. If they cannot edit DMARC, print the TXT record that
     `add_monitored_domain` returned (`challenge.host`, `challenge.value`)
     verbatim instead. If the result has `dns_connect.apply_url`, offer that
     one-click link first. End with: "Publish it, then tell me and I'll check
     again." The check's outcome is `not_found` until they publish, which is
     expected. When they say it is done, call `check_domain_verification`
     again:
     - `verified`: one line that monitoring is on; mail providers send DMARC
       reports about once a day, so the first ones arrive within a few days.
       Done.
     - `not_found` or `mismatch`: `check_record` with `expected_value` set to
       the record answers `done`, `wait` (wait `check_again_after_seconds`
       before checking again) or `edit` (relay `found_at`). End with when to
       look again.
     - A `transient` or `fault` outcome is our lookup, not their DNS: try
       again shortly.
     Tell them they don't have to come back: for a week after the domain is
     added, DNS Doctor re-checks it on its own and emails them once it's
     verified.
     If the user cannot make the DNS change themselves, offer the ready
     `help.ticket` message for their DNS host's support, or `help.web_person`
     for whoever manages their DNS, verbatim; never compose a record.
     `help.support_url`, when present, is their host's support page.
   - **Website setup**: call `start_monitoring_signup` and show its `signup_url`
     as a clickable link with its `message`, using the returned URL without changing
     it. A description without printing the URL is not enough. Use this path
     also when linking is not
     available here, or the connect prompt fails, is declined or never appears.
     Do not retry the linked tools after that. If the signup tool is unavailable,
     use the result's `next_steps.signup_url` if present, or link to
     <https://dnsdoctor.dev/dashboard>; never invent a signup URL. Never ask for
     their email address.
     Opening the link does not start monitoring; publishing the record their
     dashboard shows them does.
   - When a result carries `next_steps.in_chat` (or top-level `in_chat`), use it
     for the setup-here choice: add only with the user's agreement; show any
     attached `record` verbatim and wait for them to publish it before checking;
     if already verified, read readiness and alerts rather than adding again.
     If they ask which domains they already monitor, call `list_my_domains`;
     never guess the list from alerts.
     If a linked user wants to be told when a record change lands, call
     `watch_record_change` with the same arguments as their `check_record`
     and relay `watch.note` (one email to their account); never ask for an
     email address.
     Website links remain available whichever route they choose.
6. **Close** with one line: they can ask DNS Doctor about any domain, any time,
   for example "did my change land?" or "test an email I send".
