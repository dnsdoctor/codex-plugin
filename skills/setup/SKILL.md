---
name: setup
description: First-run setup for DNS Doctor. Runs once when the user installs the plugin. Asks which domain they manage, scans its email authentication (SPF, DKIM, DMARC) and DNS with the DNS Doctor tools, explains what is failing first, and, only for a domain the user says they own, offers DMARC monitoring that starts with one DNS record.
---

# DNS Doctor setup

Run this once, right after install. Keep it short: one domain, one scan, one
clear next step.

## Hard rules (they apply here exactly as everywhere else)

- **Never compose or edit a DNS record.** Every record you show comes from a
  DNS Doctor tool result and is printed **verbatim**, host and value exactly as
  returned. If a tool returns no record, say why (relay its `rationale`) and
  stop; never fill the gap yourself.
- **A human publishes every DNS change.** You never sign in, publish a record or
  change DNS on the user's behalf. You show the record; they publish it.
- **SPF is diagnose-only.** Relay the scan's SPF findings; never propose an SPF
  record or an edit to one (not even `~all` to `-all`).
- A scan result is a public report page at `https://dnsdoctor.dev/scan/<domain>`.
  If the user wants a domain kept private, do not scan it.

## Flow

1. **Hello.** One line: DNS Doctor checks a domain's email authentication and
   DNS and gives the exact record to fix anything broken. Ask which domain they
   manage.
2. **Scan.** Say in one line that the scan publishes a public report page for
   the domain, and scan only if that is fine. Call `scan_domain` with it.
3. **Summarize failing-first:** `fail`, then `warn`, then a one-line "the rest
   passed". Plain language: what is wrong and why it matters for their mail.
   - `temperror` is transient: "couldn't be checked right now", never a failure.
   - `info` is "not found / not applicable", never a failure.
   - **`not_registered: true` comes first:** the domain does not resolve (usually
     a typo). No check ran, so no failures does not mean healthy. Propose no
     records, do not offer monitoring, and ask whether they meant another domain.
   - Any record in the report is printed verbatim. For a DMARC fix, call
     `build_dmarc_upgrade` and present its `record` verbatim, or its `rationale`
     when `record` is null.
4. **Offer monitoring only to an owner.** Ask whether they own or manage the
   domain. If they don't say yes, stop after the summary: never offer
   monitoring for someone else's domain. If they do, offer it: daily re-scans,
   alerts when something changes, and DMARC reports showing who sends mail as
   their domain.
5. **If they want it, offer website setup or setup here.** Follow their choice.
   Keep the report and relevant dashboard links available for the full website view.
   - **Setup here** (the host lets you use `add_monitored_domain`): call
     `add_monitored_domain`, then `check_domain_verification` once to get the
     domain's DMARC record: print `enrollment.host` and `enrollment.record`
     verbatim. That **one DMARC record** proves ownership and starts the
     reports; it replaces their current DMARC record rather than sitting beside
     it. If they cannot edit DMARC, print the TXT record that
     `add_monitored_domain` returned (`challenge.host`, `challenge.value`)
     verbatim instead. The check's outcome is `not_found` until they publish,
     which is expected: "publish it, then tell me and I'll check again". When
     they do, call `check_domain_verification` again. To see whether the record
     itself is live yet, `check_record` with `expected_value` set to it answers
     `done`, `wait` (wait `check_again_after_seconds` before checking again) or
     `edit`. A `transient` or `fault` outcome is our lookup,
     not their DNS: try again shortly.
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
     Website links remain available whichever route they choose.
6. **Close** with one line: they can ask DNS Doctor about any domain, any time.
