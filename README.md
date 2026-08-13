# Filing Documents Showcase

Browse the filing output PDFs: https://mruff-aeq.github.io/filing-documents-showcase/

## What this is

One example of every filing the e2e API suite can produce in dev (CORPS 26/26, COOP 12/13,
FIRM 9/10 — the gaps are environment bugs, not suite gaps). The suite is fully self-contained:
it mints its own businesses (numbered corps; coop/firm via Name Requests approved with the
names_approver role) and auto-discovers aged businesses for the annual reports. Each run
REPLACES the documents here — this is a showcase of current output, not an archive.

## payloads/

Body-only request payloads for every filing type, exactly as POSTed to
`{gateway}/business-dev/api/v2/businesses/{identifier}/filings` (new-business filings go
through `POST /businesses?draft=true` then `PUT .../filings/{filingId}`). The `filing.header`
is stripped — add your own:

```json
"header": {"name": "<filingType>", "date": "YYYY-MM-DD", "certifiedBy": "...",
           "accountId": <billing org>, "authorizationReceived": true}
```

plus headers `Authorization: Bearer <jwt>`, `x-apikey: <gateway key>`, `Account-Id: <org>`.
File names are `<family>-<filingType>.json` (corps / coop / firm). Regenerated every suite run.
