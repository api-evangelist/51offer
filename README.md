# 51offer

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
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

51offer (Shanghai Huizhi Business Consulting Co., Ltd. / 上海汇紫商务咨询有限公司, Xuhui District,
Shanghai) is a one-stop online study-abroad platform for Chinese students applying to universities in
the United Kingdom, Australia, the United States, New Zealand, Japan and Singapore. Its surface covers
DIY application filing, AI/big-data school and course matching, personal statements and application
materials, IELTS/TOEFL language training, adviser and channel-partner services, a study-abroad mall
with online contract signing, payment and refund, a GPA calculator, and a student content section.

## The contract

51offer runs **no developer programme** — no portal, no documentation, no SDKs, no support channel for
API consumers. It does, however, serve an unauthenticated **Swagger 1.2** API listing on its own
official site:

- <https://www.51offer.com/api-docs> — HTTP 200, `application/json`, `swaggerVersion: 1.2`,
  `info.title: "Horizon Site APIConfig List"`, `info.description: "51offer官网所有开放接口清单"`,
  `info.contact: woodrow.w@51offer.com`. 24 resource declarations, 452 operations.
- The same documents are served at <https://m.51offer.com/api-docs>.

Those 25 documents are saved verbatim under `openapi/_original/`. `openapi/51offer-horizon-site-openapi.yml`
is a mechanical conversion of them to OpenAPI 3.1 — 221 paths, 389 operations, 159 schemas — carrying
their provenance in `info.x-provenance` and per-operation `x-swagger-1-2-source`. Nothing in it was
invented; 14 models the provider's own documents reference but never define are flagged
`x-undefined-in-source` rather than filled in.

The API base is `https://www.51offer.com` — the marketing site and the API share one host. Verified
live on 2026-09-05: `GET https://www.51offer.com/ngGpaCalc/constants` returned HTTP 200 and
`application/json`.

## What an integrator needs to know

- One envelope on every response — `{message, code, data, header, page, apiInfo}`. The HTTP status is
  200 on success **and** on most failures; the outcome is the `code` field.
- Authentication is a `token` request header, evidenced only by CORS headers. Undocumented.
- **No idempotency mechanism** on any of the 118 mutating operations, including `createOrder`,
  `payOrder` and `submitEntranceRefund`.
- Many mutating operations are declared `GET` (`addCart`, `submitIntentions`, every `diym` delete).
- A refund path exists (`POST /ngrefund/sub`, `GET /ngrefund/progress`) but **no reversal window is
  published anywhere**, so reversibility grades `documented`, not `verified`.
- Unknown paths return HTTP 200 with an HTML error page rather than a 404.

## What is absent

Probed and not found on 2026-09-05: any `/.well-known/` document on any of five hosts (every path
returns a soft 404, a real 404, or `{"error":"Document not found"}`), `security.txt`, `llms.txt`,
`robots.txt`, an A2A agent card, an MCP server, OAuth or OIDC metadata, GraphQL, WSDL, AsyncAPI or
webhooks, a status page (`status.51offer.com` is NXDOMAIN, as are `developer.`, `open.` and `docs.`),
a changelog, a deprecation policy, an SLA, published rate limits, an error-code reference, a trust
centre or vulnerability-disclosure programme, and any API pricing.
