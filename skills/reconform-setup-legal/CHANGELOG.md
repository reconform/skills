# 2.4.0

Update published documents after the app changes. When the saved workspace
already has confirmed facts, the skill loads them and the published version of
each document, reruns the codebase scout against them, reports new and removed
vendors, new data uses and facts the code no longer matches, and asks only
about those. The rebuilt drafts are shown as diffs against the published
versions, with each change tied to the fact that caused it. In live mode the
skill saves drafts and recommends a re-acceptance policy for each; publication
stays in the app. The helper adds two read-only commands, `read-published` and
`diff` (which also renders the subprocessor list the way the app does), and a
`--summary` option for `save-draft`. When the founder approves wording that
goes beyond a saved fact, the rerun updates the fact too, and the new
`check-facts` command fails when a saved vendor or data category is missing
from the vendor facts. The document-slot note now says Free allows four live
documents, which covers terms, privacy, DPA and the subprocessor list. The
first-run flow is otherwise unchanged. The 2.3.0 archive stays available.

# 2.3.0

Read the codebase only after the founder agrees, and never send anything from
it anywhere. The scout guide adds privacy rules: no secrets, env values or
customer data, no running or installing, and facts with file references only.

# 2.2.0

Fetch the helper and only the reference files it needs one at a time with
`curl`, instead of cloning the skills repository. API commands load the key
with `node --env-file`. No CLI steps.

# 2.0.1

Rebuild the helper against the current API schemas, which add the
`checkbox` session option and bundle acceptances. No change to the interview,
templates, or commands. The 2.0.0 archive stays available.

# 2.0.0

Plan first: pick documents from a catalog, have a read-only sub-agent propose
answers from the codebase, then run a structured 6-12 question interview per
document. The approved plan is the fact confirmation. Adds ten openly licensed
templates (cookie policy, acceptable use, refunds, SLA, mutual NDA, BAA, AI
addendum, professional services, pilot, software license). The helper looks
documents up in the catalog and refuses unusable drafts: template blanks,
drafting notes, checkboxes, source-business names, or missing sections. Adds
`check`. Unanswered questions are resolved with the founder instead of being
left in `review.md`.

# 1.2.0

Work from `.reconform/context.json` written by the Reconform CLI. Save the terms
draft as the next version of a sandbox's example terms instead of failing on the
existing slug. Add `publish-test`, which publishes an approved, unchanged draft in
test mode only and records the approval in `review.md`.

# 1.1.0

Separate the confirmation turn from drafting. Bundle a Node helper that validates
facts before writes, assembles intact standard terms and attribution, requires a
training choice, and verifies exact readbacks. Keep durable IDs and request keys
for recovery. Preserve prior 1.0.0 download URLs.

# 1.0.0

Interview business practices, reuse confirmed facts, adapt pinned Common Paper
and Automattic sources, and prepare documents and subprocessors for review.
Supports local-file output and authenticated draft saves. Publication stays in
the application.
