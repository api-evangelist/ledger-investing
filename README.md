# Ledger Investing

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Ledger Investing is a New York-based specialist broking and advisory firm for casualty insurance-linked securities (ILS), connecting insurers, reinsurers and fronting carriers to institutional capital. The firm has securitized more than $2.5 billion of casualty premium across 170+ transactions since graduating Y Combinator in 2017.

Its wholly owned SaaS subsidiary Korra Tech, LLC operates the Korra data and analytics platform for the reinsurance and ILS market. The **Ledger Analytics API** (`https://api.korra.com/analytics/`, currently in beta) gives actuaries remote compute access to Bayesian loss development, tail and forecasting models over insurance loss triangles, alongside the open-source Bermuda and BayesBlend Python libraries.

- Website — https://www.ledgerinvesting.com/
- Korra — https://www.korra.com
- Analytics API documentation — https://ledger-investing-ledger-analytics.readthedocs-hosted.com/en/stable/index.html
- GitHub — https://github.com/LedgerInvesting
- Trust center — https://trust.korra.com

Backed by: accel

## Note on the OpenAPI

Ledger/Korra publish no OpenAPI description. `openapi/ledger-investing-analytics-openapi.yml` was **derived** by API Evangelist from the request construction in the first-party, MIT-licensed open-source `ledger-analytics` Python client and from the published documentation. It is not an authoritative provider artifact.
