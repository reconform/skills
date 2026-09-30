# Codebase scout

Use this only after the founder has agreed to let you read their codebase. Give
this file to a read-only sub-agent (an Explore-type agent where one exists)
together with the chosen document ids and the path or URL of this
`references/` directory. The scout proposes answers from the repository so the
founder confirms instead of typing. It never decides anything and never
changes a file.

## Privacy rules

- Work only on this machine. Don't send code, file contents, or anything you
  read to Reconform or to any other service, and don't fetch anything.
- Look only for what the questions ask. Don't open env files, keys,
  certificates, credential stores, database dumps, or customer data. A variable
  name such as `STRIPE_SECRET_KEY` in code or an example env file is enough
  evidence of a vendor; its value is never needed.
- Don't run the app, its scripts, or its tests, and don't install anything.
- Report facts with a file and line as evidence. Don't copy code or text from
  the repository into your report beyond a short quote of a claim you are
  flagging as a conflict.

## Instructions for the scout

You are reading an application's repository to propose answers for a legal
document interview. For each chosen document id, read
`questions/<id>.json`, and read `questions/shared.json` for the ids listed in
each file's `uses`.

For every question whose `codebase` hint is not null, search the repository
the way the hint describes. Skip questions with `legal_choice: true`; those
are the founder's decisions.

Report only what the code shows:

- A package in `package.json` that is never imported or called is not a
  vendor. Look for the call site.
- A hosted service's default setting is not proof of the actual setting.
- Existing legal pages, READMEs and marketing copy are claims, not facts.
  Report what they say as `"confidence": "claimed"`, and report any place
  where they disagree with the code under `conflicts`.
- If nothing answers a question, leave it out.

Finish with one JSON object and nothing else:

```json
{
  "answers": {
    "billing-period": {
      "value": "Monthly or annual, customer chooses",
      "evidence": "lib/stripe.ts:6",
      "confidence": "found"
    }
  },
  "conflicts": [
    {
      "fact": "vendor-list",
      "claim": "We do not share your data with any third parties.",
      "claim_source": "public/legal/privacy.md:9",
      "code": "Stripe and Postmark are called in src/billing.js and src/mail.js."
    }
  ]
}
```

`confidence` is `found` (the code does it), `configured` (a config file sets
it), or `claimed` (text says so but code doesn't show it). Keys are the
question `id`s.

## How the main agent uses the result

Keep the scout's JSON; you can't write files while planning. When you ask a
question the scout answered, put its value first among the options, labeled
with the evidence, for example
`Monthly or annual (Recommended, found in lib/stripe.ts)`. Still ask every
question. A proposal becomes a fact only when the founder confirms it.

Show each conflict to the founder as its own question before drafting, with
the claim, the code, and a request to say which is true.
