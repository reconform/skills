# Draft after the confirmed interview

Read [the API and workspace format](api.md) for schemas and the context's
organization/environment/mode. Read [the pinned source manifest](templates.json)
and only the relevant template/cover files. Node 20+ is required for the helper.
If it is unavailable, return the confirmed brief and explain that preparation
needs the helper; do not claim unvalidated files are ready for import.

Use `node /path/to/reconform-setup-legal/scripts/legal.cjs` for the commands below.
The helper has its dependencies bundled; no npm install or model-provider key is
needed. It does not call an LLM. Credentials stay in `RECONFORM_API_KEY` in the
local environment. Preserve one checkpoint per organization/mode, and run its
commands sequentially.

## Write the confirmed facts

Create `legal-workspace.json` with `facts`, `subprocessors`, and
`subprocessor_document_id`. Retain existing vendor IDs. Fact keys use lowercase
hyphenated slugs, such as `backup-retention`, never underscores. Confirmation
records use the actual time the customer confirmed, not a copied example date.

Run this before any save:

```sh
node /path/to/reconform-setup-legal/scripts/legal.cjs validate-workspace --file legal-workspace.json
```

Correct local errors and repeat validation. Do not submit diagnostic one-fact
subsets to the API; workspace PATCH replaces the complete fact/vendor lists.

## Author covers and the privacy body

Create `terms-cover.md`, `dpa-cover.md`, and `privacy-body.md` as needed. Keep
questions and unfinished choices in `review.md`; do not call incomplete drafts
ready for publication. Preserve the required cover fields and identify referenced
attachments. Do not assert transfer arrangements or certifications from a guess.

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

`terms.json`:

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

Use the customer's confirmed training choice: `prohibited` or `permitted`.
Do not infer it from the example. For `privacy.json` and `dpa.json`, use the
matching kind/name/slug/body filename and omit `model_training`. Privacy also
omits `clause_reviews`.

The empty `clause_reviews` in the example must be filled before assembly. Review
these clauses explicitly against the confirmed facts:

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
checks supported Markdown, and prints its SHA-256. Repeat for selected documents.
Review the assembled text and semantic changes. Edit the authored cover/body and
reassemble when changes are needed. Do not regex-strip HTML, manually paste the
standard terms, or modify assembled output independently of its specification.

With no API key, return these assembled files, the fact file, and `review.md` for
manual import. The helper's printed hashes are the actual file hashes.

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

Replace `0` with the revision just read. Repeat `save-draft` for privacy and DPA.
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

For vendors, save the structured data and return to **Legal pages → Preview
subprocessor draft**. The app produces that fourth document. Live publication
and public sharing happen in the app after review. The helper publishes only in
test mode and has no commands for sharing or creating acceptance records. To go
live, the customer claims the sandbox (`npx @reconform/cli@next claim`) and
promotes the reviewed test document in the app.
