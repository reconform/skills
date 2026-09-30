---
name: reconform-setup-legal
description: Interview a B2B SaaS founder with structured questions, then turn licensed templates into complete legal documents with no blanks left. Use to set up terms, privacy, DPA, subprocessors and the other catalog documents, to replace a sandbox's example terms, or to revise documents after practices change.
metadata:
  version: "2.0.1"
---

# Set up legal pages

The work has two phases. In the planning phase you pick documents, scout the
codebase, and interview the founder. Nothing is written. In the build phase,
after the founder approves the plan, you draft, check and save.

## Workspace files

Use `.reconform/context.json` when it exists; `npx @reconform/cli@next init`
writes it along with `RECONFORM_API_KEY`. Otherwise use the context file the
customer downloaded from Reconform. Keep working files, the checkpoint, and
`review.md` in `.reconform/legal/`. Do not edit application code; another agent
may be wiring the integration at the same time.

A sandbox starts with published example terms under the slug `terms`. Your terms
draft becomes the next version of that document, so an integration that already
requests `terms` keeps working.

## What is available

[The catalog](references/catalog.json) lists every document with a vetted
template: its name, picker group, template files, and question file. Only
catalog documents are "templated". For anything else, use the
`reconform-create-document` skill and tell the founder it is drafted without a
vetted template.

These templates are written for English-language B2B SaaS. If the founder
says customers are individual consumers, or the product handles regulated data
the templates don't cover, say so and offer to import documents they already
had reviewed.

## Plan: pick, scout, interview

Stay in plan mode (or make no file changes) until the founder approves the plan.

1. **Pick documents.** Ask with your question tool: one multi-select question
   per `picker_group` in the catalog, each option a catalog `name`. Say these
   are the documents with vetted templates and that they can type anything
   else. Order the chosen documents so each comes after its `depends_on`.
2. **Scout.** Start a read-only sub-agent with
   [the scout guide](references/codebase-scout.md) and the chosen ids. While it
   works, you can ask the company questions.
3. **Company questions.** Ask the questions in
   [shared.json](references/questions/shared.json) that any chosen document
   `uses`. Ask each once.
4. **One interview per document.** Ask every question in the document's file,
   up to four per question call. Use each question's `ask` and `options`. Put
   the scout's proposal first, marked recommended with its evidence. A `text`
   question gets the proposal (if any) as an option and the founder types the
   rest. Honor a question's `follow_up`. Ask one question per fact, without
   merging two into one. When the scout suggests a question doesn't apply
   (say, no AI features in the code), still ask it with that as the
   recommended answer.
5. **Close every gap.** When the founder doesn't know an answer, propose a
   sensible default, say it is a default, and ask them to accept or change it.
   Show each scout conflict as a question. Existing documents and repository
   text are source material, not confirmed practice.
6. **Write the plan.** List the documents with their templates, then every
   fact as `fact id: answer (source)`, then the build steps. When the founder
   approves this plan, that approval is the confirmation of these facts and
   this scope. Record its text and time in `review.md` in the build phase.

If the founder already approved the same facts and scope earlier in this
conversation, don't ask again. A detailed brief is not approval. On a resumed
workflow, read `review.md` and the helper checkpoint first; if approval is
missing, return to the interview.

## Build: draft with no blanks

Only after approval, read [the drafting workflow](references/drafting.md). It
uses the bundled Node 20+ helper to validate facts, assemble each document from
its template, and check that it is usable: no blanks, drafting notes or
checkboxes from the template, no names from the source business, and the
sections a reader expects. A document that fails the check isn't done. Go back
to the founder for the missing answer, then fix the authored file.

Terms, privacy, DPA, and a subprocessor list use four document slots. Free allows
three live documents; explain that before a four-document live save. Test mode
has separate documents and does not require a paid upgrade.

## Publish in test mode after approval

When the context mode is `test`, show the customer the assembled text and ask
whether to publish it in test mode so their app shows it. Publish only after an
explicit yes to that exact text, using the helper's `publish-test` command with
their words as the approval. Never publish live, never publish text that
changed after approval, and never create acceptance evidence. Then point them to
the claim link (`npx @reconform/cli@next claim`) to keep the sandbox and go live
from Reconform.
