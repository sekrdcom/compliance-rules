# Sekrd compliance citation corpus

The closed-set citation library used by [Sekrd](https://sekrd.com)'s
$39 Pre-Launch Audit. Every compliance finding in a generated audit
report cites a `citation_id` that resolves to a line in this repo.

## Why this exists

When Sekrd's audit pipeline tells you "your privacy policy is
missing GDPR Art. 13(1)(c) data category enumeration", you should be
able to:

1. Verify that GDPR Art. 13(1)(c) actually says what we claim it says
2. Verify that we're not making up article numbers
3. Verify the regulation text is current

This repo answers all three. It's a read-only mirror of
`internal/compliance_audit/corpus/` from
[github.com/sekrdcom/sekrd](https://github.com/sekrdcom/sekrd) — the
closed-source application repository. The mirror updates quarterly
on the same cadence as the corpus refresh runbook.

## How to verify a finding

Every Sekrd audit PDF references citations by `citation_id` (e.g.
`gdpr-art-13-1-c`). To verify:

1. Open [`citations.yaml`](./citations.yaml)
2. Search for the citation_id
3. Read the verbatim quote + jurisdiction + article number

If the quote in the PDF matches the quote here, the citation is
authentic. If it doesn't match — or if the citation_id isn't in
this file — flag it; that's a fabrication and we want to fix it.

The Sekrd audit pipeline includes a whitelist filter that drops any
LLM-generated citation_id not present in this corpus. The filter is
how we keep the LLM from inventing article numbers; this repo is how
you verify the filter is working.

## Anti-fabrication mechanism

The audit pipeline never lets an LLM generate citation IDs. The
flow is:

1. Pattern detection (deterministic Go) finds technical issues:
   "this app has no Do Not Sell link"
2. LLM extracts facts from the audited site (controller name,
   data categories, etc.)
3. **Deterministic Go code** maps facts → citation IDs from this
   closed-set list
4. The PDF cites only IDs that resolved against this whitelist

The LLM never sees this list and never proposes citation IDs of its
own. Any LLM output that mentions an unknown citation gets logged as
a fabrication metric (currently 0/100 across the eval corpus) and
dropped before reaching the customer.

Read more in the methodology page on
[sekrd.com/compliance/methodology](https://sekrd.com/compliance/methodology).

## What's in this repo

| File | Contents |
|---|---|
| `citations.yaml` | Flattened citation map: `citation_id → {jurisdiction, article, title, quote, web_app_relevance}` for every cited article. Verbatim quotes from source documents. |
| `jurisdictions.md` | One section per jurisdiction with source URL, effective date, retrieval date, and citation count. |
| `LICENSE` | Apache 2.0 — fork it, mirror it, audit it. |

## What's NOT in this repo

- The full text of the regulations (cite the official source — EUR-Lex,
  California AG, planalto.gov.br, etc.)
- The detection rules / pattern library (those are in the closed-source
  Sekrd repo and are part of the product)
- The LLM prompts / system instructions (same)
- Customer audit data (lives in customer dashboards only)

## Refresh cadence

Quarterly. Corpus refresh checklist in the closed-source repo at
`internal/compliance_audit/corpus/REFRESH.md` includes:

- EU GDPR + UK GDPR: automated EUR-Lex SPARQL diff
- California CCPA / CPRA: manual annual review against California AG text
- Brazil LGPD: manual annual review against planalto.gov.br
- App Store + Play guidance: manual review on Apple / Google
  developer policy updates

Each `metadata.yaml` per jurisdiction in the source repo carries a
`retrieved_at` timestamp shown in audit PDFs as "Rules current as of".

## Known limitations

- **5 jurisdictions covered**: EU GDPR, UK GDPR, US CCPA, US CPRA,
  Brazil LGPD. We expand based on customer demand.
- **Web-app focused**: relevance triggers in `web_app_relevance` lean
  toward web/mobile SaaS patterns. If you're auditing infrastructure
  (network, IoT, etc.) — different corpus needed.
- **Not legal advice**: this is technical attestation infrastructure,
  not a substitute for qualified privacy counsel. Sekrd is not a
  certified auditor.

## Reporting issues

- **Citation text wrong** (quote doesn't match source): file an issue
  with link to authoritative source + diff
- **Citation missing** (article exists but isn't here): file an issue
  with article number + jurisdiction
- **Citation should be retired** (regulation amended / superseded):
  file an issue with effective date + replacement reference

Issues here trigger a corpus update in the source repo + a sync run
to this mirror. Mean time to resolution depends on whether the change
is automated (EUR-Lex) or manual (US state law).

## License

[Apache 2.0](./LICENSE). Fork freely. Attribution to Sekrd appreciated
but not required. If you fork to maintain a different jurisdiction
mix, file an issue and we'll cross-link.
