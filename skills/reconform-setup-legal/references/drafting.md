# Draft after the approved plan

Read [the API and workspace format](api.md) for schemas and the context's
organization/environment/mode. Read [the catalog](catalog.json), then only the
`template_files` of the chosen documents. [The pinned source manifest](templates.json)
records each template's source and license. Node 20+ is required for the helper.
If it is unavailable, return the confirmed brief and explain that preparation
needs the helper; do not claim unvalidated files are ready for import.

Use `node /path/to/reconform-setup-legal/scripts/legal.cjs` for the commands below.
The helper has its dependencies bundled; no npm install or model-provider key is
needed. It does not call an LLM. Credentials stay in `RECONFORM_API_KEY` in the
local environment. Preserve one checkpoint per organization/mode, and run its
commands sequentially.

## Write the confirmed facts

Create `legal-workspace.json` with `facts`, `subprocessors`, and
`subprocessor_document_id`. Retain existing vendor IDs. Each fact's `key` is the
question `id` from the question files, such as `backup-retention`; use the
question's `label` as its label. Every answered question becomes a `confirmed`
fact whose `confirmed_at` is the time the founder approved the plan, not a copied
example date. Its source says where the answer came from, for example "Founder
confirmed in plan approval; proposed from lib/stripe.ts". Vendors from the
subprocessor interview become `subprocessors` entries.

Run this before any save:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs validate-workspace --file legal-workspace.json
```

Correct local errors and repeat validation. Do not submit diagnostic one-fact
subsets to the API; workspace PATCH replaces the complete fact/vendor lists.

## Author the document bodies

For each chosen document, write one authored Markdown file from its
`template_files`, such as `terms-cover.md` or `privacy-body.md`. The catalog
`mode` says how:

- `cover+standard`: fill the cover page. Replace every bracketed field with the
  confirmed answer. Keep only the checkbox options the founder chose, written as
  plain statements. Remove drafting notes, instructions, HTML comments, and
  optional fields that don't apply. The helper appends the standard terms
  unchanged.
- `adapted-body`: rewrite the source text for this business. Remove products,
  names, links and commitments of the source company (the catalog's
  `source_terms` must not remain) and anything the founder's facts don't
  support. Keep the section headings a reader expects.

Where a cover field belongs to each customer, such as the customer's name or the
effective date, use the clickthrough wording the terms cover uses: the Customer
is the company or person who accepts, and the effective date is the date they
first accept. Do not assert transfer arrangements or certifications from a
guess. Put a `Last updated: <date>` line under the title of every policy and of
the terms, using the date you draft it. When a document is signed by each party instead of accepted online,
end it with a signature block of labels only, one per line for each party:
Company, Name, Title, Notice address, Signature, Date. Don't add underscores,
brackets, or notes about what gets filled in. Nothing unfinished may remain: if an answer is missing, ask the founder
now instead of leaving a gap. `review.md` records decisions, not open questions.

For terms, explicitly resolve whether customer content or usage data may be used
for model training. The standard grants permission. The helper requires the
confirmed choice and generates an overriding prohibition when it is prohibited.
Review all other source clauses against the facts too, especially promotional
usage-data use, deletion, refunds, audit promises, and incident notification.
Put proposed changes in the cover, leaving the standard terms intact.

For the DPA, fill the cover and relevant schedules. The helper adds the complete
pinned standard terms. Explain source clauses that need changes and put those
changes in the cover; do not summarize or shorten the standard terms.

Adapt the privacy body to actual practices, removing unrelated Automattic
products and commitments. Its attribution and ShareAlike license are distinct
from Common Paper's license.

## Assemble using specifications

Create one JSON specification per document. Body files must be Markdown files in
the same directory with names different from the assembled output.

The spec `kind` is the catalog id. `terms.json`:

```json
{
  "kind": "terms",
  "slug": "terms",
  "name": "Terms of Service",
  "body_file": "terms-cover.md",
  "model_training": "prohibited",
  "clause_reviews": {}
}
```

Use the customer's confirmed `model-training` answer: `prohibited` or
`permitted`. Only terms specs have `model_training`. For other documents use
the catalog id, name, slug and your body filename. Add `clause_reviews` when the
document's question file lists `clause_reviews`, or when you amend a clause of
its standard terms.

The empty `clause_reviews` in the example must be filled before assembly. Review
these clauses explicitly against the confirmed facts (the question file maps
each clause to the facts that decide it):

| Kind  | Required clause decisions                                                                        |
| ----- | ------------------------------------------------------------------------------------------------ |
| Terms | `1.4` usage-data promotion; `5.5(b)` deletion upon request; `5.6(b)` backup/retention exceptions |
| DPA   | `5.2` independent audit/report assertions; `6` deletion/return obligations                       |

Each entry is either `{ "decision": "keep", "reason": "Why the confirmed facts support this default" }`
or `{ "decision": "amend", "reason": "Why this change is approved", "text": "Actual contract amendment wording" }`.
Reuse the source's exact defined terms in amendments. For example, the DPA uses `Customer Personal Data`, not a newly invented `Customer Data` alias.

These quoted descriptions are shapes, not business facts. Use real reasons and
wording, not the placeholders. You may include additional numbered clauses when
needed. Terms section `1.6` is controlled by `model_training`, not this map.

The helper inserts each amendment into the cover and states that it controls
over conflicting source language. Merely saying "needs review" or "will be
overridden" is not an amendment. A keep decision means the default is supported
by the confirmed facts, not simply that you copied the template. If a required
choice is unresolved, ask the customer and retain the question rather than
inventing a keep reason. Record the actual clause decisions in `review.md`.

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs assemble --spec terms.json
```

This writes `terms.md`, supplies the pinned convenience copy and license notice,
checks supported Markdown, and prints its SHA-256. It then checks the document
is usable and lists every problem: template blanks, drafting notes, checkboxes,
source-business names, and missing sections. Fix the authored cover/body and
assemble again until it reports `assembled`. Repeat for each chosen document.
Do not regex-strip HTML, manually paste the standard terms, or modify assembled
output independently of its specification.

To recheck an assembled file, run:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs check --spec terms.json
```

Read the assembled text yourself too. The check finds leftovers; it can't tell
whether a clause matches the founder's facts.

With no API key, return these assembled files, the fact file, and `review.md` for
manual import. The helper's printed hashes are the actual file hashes.

## Check the documents against each other

A set of documents is published together, so read them as a set before saving:

- Every confirmed fact appears in each document its question `fills`, in the
  founder's words. Don't widen or narrow it: if a vendor receives "names and
  email addresses", say exactly that everywhere.
- Documents point to each other: the terms link the privacy policy, and the
  DPA and subprocessor page when they exist; the DPA gives the subprocessor
  page's address; the privacy policy links the subprocessor page and the cookie
  policy. Use only addresses the founder confirmed. If you don't have one, ask.
- Use every part of an answer. If the pricing answer mentions a trial or a
  pricing page, the terms cover the trial and link the page; if the founder set
  a minimum age, the terms say who may use the product.
- Contract terms confirmed for one document hold in the others. Liability caps,
  uncapped claims, renewal price limits, and notice periods in an SLA, DPA or
  subprocessor page must match the terms, or the terms must say which document
  controls.
- No two documents disagree. For example, when the subprocessor objection
  right includes a refund, the terms' refund rule makes that exception.
- Commitments the founder confirmed and the reader expects are stated: no model
  training in both the terms and the DPA, self-service controls in the privacy
  policy, the security contact in the DPA.
- For a B2B-only product, the Customer is "the company" that accepts, not
  "the company or person".

Then have the documents fact-checked by a fresh, read-only sub-agent, since
it's hard to catch your own paraphrases. Give it `legal-workspace.json`, the
founder's answers as recorded in the plan, and the assembled documents (the
standard terms sections can be skipped). Ask it to quote every statement that
contradicts, narrows, widens, or adds to a confirmed fact: a changed period or
trigger ("30 days after cancellation" is not "30 days after the contract
ends"), a dropped or added data category, an extra detail such as "daily", a
feature the code doesn't have, or a wrong address. No sub-agents? Do this pass
yourself, one fact at a time.

Fix the authored files and assemble again. Anything that needs a new decision
goes back to the founder.

## Save authorized drafts

With `.reconform/context.json` (or the downloaded context file) and a scoped
local key, read the current workspace first. Run the commands from
`.reconform/legal/` so the checkpoint and `review.md` stay together:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs read-workspace --context ../context.json --state checkpoint.json
```

Use the returned revision for the complete workspace save:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs save-workspace --file legal-workspace.json --context ../context.json --state checkpoint.json --expected-revision 0
node /path/to/reconform-setup-legal/scripts/legal.cjs save-draft --spec terms.json --context ../context.json --state checkpoint.json
```

If a document with the same slug already exists, such as a sandbox's example
`terms`, `save-draft` adds the draft as that document's next version. It refuses
when the existing document has a different kind or is archived.

Replace `0` with the revision just read. Repeat `save-draft` for each chosen document.
Local testing additionally requires `--allow-local` and an explicitly selected
localhost API. Production/staging URLs must match the context's environment.
The helper checks the key's organization/mode before writing.

Use the helper instead of recreating curl save/hash commands. It persists each
idempotency key before a request, retains created IDs, and compares exact server
strings against assembled Markdown. It never appends a console newline to the
compared text and never silently rewrites a local file to match a response.

After an unknown write result, repeat the exact command and inputs. Keep the
checkpoint; do not invent a new key, slug, or document. The helper reuses the
pending key or recognizes an already-saved identical draft.

After a revision conflict, reread and compare the other editor's changes with
_your authored cover/body_. Present a merged diff for approval, then update the
cover/body, reassemble, and use `--expected-revision` with the reviewed current
revision. Preserve the other editor's text. Do not overwrite or automatically
retry a conflicting edit. `read-workspace` can inspect the latest facts; a stale
workspace save similarly needs a reviewed merge and its current revision.

## Publish in test mode after approval

Only in test mode, and only after the customer explicitly approves the exact
assembled text, publish it so their integration shows it:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs publish-test --spec terms.json --context ../context.json --state checkpoint.json --confirmed "Yes, publish these terms in test mode."
```

Pass the customer's actual words. The helper refuses live contexts, refuses a
saved draft that differs from the assembled file, publishes with required
re-acceptance, reads the version back, and appends the approval to `review.md`.
Repeat for each approved document. After an unknown result, repeat the same
command; it recognizes an already-published version.

## Hand back verified results

Copy document/version IDs, revisions, and SHA-256 values from successful helper
output into `review.md`; don't calculate them through shell output redirection.
Use the final helper `verified_at` timestamp for completion time. If timing is requested, record a real start timestamp before the interview and measure against that completion time; do not round or invent clock readings. If the start was not measured, say so.

Keep the specs, covers, assembled files, checkpoint and approval record together
for resumption. Distinguish saved documents from unfinished/failed steps.

An empty subprocessor list needs the founder to confirm there are none; an
unfinished vendor interview is not that confirmation. For vendors, save the structured data and return to **Legal pages → Preview
subprocessor draft**. The app produces that fourth document. Live publication
and public sharing happen in the app after review. The helper publishes only in
test mode and has no commands for sharing or creating acceptance records. To go
live, the customer claims the sandbox (`npx @reconform/cli@next claim`) and
promotes the reviewed test document in the app.
