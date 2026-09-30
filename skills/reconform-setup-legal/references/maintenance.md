# Update documents after the app changes

A rerun starts from what the founder already confirmed and what is published.
It finds what changed in the code, asks only about that, shows each document
as a diff against its published version, and saves drafts. Nothing is
published live from here.

Run helper commands from `.reconform/legal/`, as in
[the drafting workflow](drafting.md). Use `../../.env` in `--env-file` if that
is the app's env file. Add `--allow-local` only when the context names a
localhost API.

## Plan: find what changed

Enter plan mode now (in Claude Code, call the `EnterPlanMode` tool) and stay in
it until the founder approves the plan. Only if your agent has no plan mode:
tell the founder you won't change any file until they approve, and keep to
that.

1. **Load the last run.** Print the saved facts and vendors with the helper
   the earlier run downloaded. This only reads:

   ```sh
   node --env-file=../../.env.local ../skills/reconform-setup-legal/scripts/legal.cjs read-workspace --context ../context.json --state checkpoint.json
   ```

   Without a key or an earlier helper, read `.reconform/legal/legal-workspace.json`
   instead, and tell the founder the comparison is against local files, which
   may differ from what is live. Also read `review.md` for the earlier plan
   and clause decisions, and the specs in `.reconform/legal/` for the
   documents it made.

   The documents in scope are the catalog documents the earlier run made and
   the subprocessor list. Documents made with `reconform-create-document` stay
   as they are unless the founder asks, but say so when a change touches one.

2. **Rescout, if they agree.** Finding what changed means reading the
   codebase again, so ask first, as in the first run; nothing from it leaves
   the machine. If they agree, start the read-only scout with
   [the scout guide](codebase-scout.md), the document ids, and the saved facts
   and vendors. It follows the guide's privacy rules and rerun section, and
   reports new and removed vendors, new data uses, and saved facts the code no
   longer matches. If they don't agree, ask them what changed instead.

3. **Report the changes.** Before asking anything, show the founder a short
   list:
   - new vendors, with the file that calls each one;
   - removed vendors, whose code is gone;
   - new data uses: personal data sent somewhere new, a new category
     collected, a new purpose;
   - saved facts the code contradicts: fact id, saved answer, what the code
     shows;
   - how many facts are unchanged and carried over.

4. **Ask only about changes.** For each changed fact, ask its question from the
   question files, with the scout's proposal first (marked recommended, with
   its evidence) and the saved answer shown as "Previously: ...". For a new
   vendor, ask its purpose, data, and location. For a new data use, ask what
   data goes where and why. Ask a legal-choice question again (such as
   `model-training` or `advertising-sharing`) only when a change bears on it,
   with the saved answer as the recommended option. Also ask any question in
   the files that has no saved fact yet. Don't ask again about facts that
   haven't changed; the founder can change one by saying so.

5. **Write the plan.** List the documents that change and the ones that don't,
   with the reason. Then list changed facts as
   `fact id: saved answer → new answer (source)`, carried-over facts as
   `fact id: answer (confirmed <date>)`, and the build steps below. Exit plan
   mode by asking for approval (in Claude Code, `ExitPlanMode` with the plan).
   Approval confirms the changed facts. Carried-over facts keep their original
   `confirmed_at`.

## Build: rebuild and diff

1. **Get this version's helper.** Download `scripts/legal.cjs` again, and any
   reference file the chosen documents need that is missing, as in
   [Get the helper](drafting.md#get-the-helper). The earlier run's helper
   doesn't have the commands below.

2. **Download the published text.**

   ```sh
   node --env-file=../../.env.local ../skills/reconform-setup-legal/scripts/legal.cjs read-published --context ../context.json --state checkpoint.json --out published
   ```

   This writes `published/<slug>.md` for each published document and
   `published/legal-workspace.json` with the saved facts and vendors. It prints
   each document's version, publish date, the re-acceptance policy it used,
   and its SHA-256, and marks the subprocessor list. It changes nothing on the
   server.

3. **Update the facts.** Start `legal-workspace.json` from
   `published/legal-workspace.json`. Change only the approved facts and give
   them the approval time as `confirmed_at`. Keep vendor IDs, add new vendors
   with new UUIDs, and remove vendors that are gone. Run `validate-workspace`.

4. **Edit the documents.** Use the earlier authored cover or body when
   assembling it reproduces the published text (compare the SHA-256 that
   `assemble` prints with `read-published`). Otherwise start from the
   published text: the part above `## Changes to the Standard Terms`,
   `## Standard terms (convenience copy)`, or `## License and source` is the
   cover or body, and the spec's `clause_reviews` come from the published
   changes section and `review.md`. Change only what the changed facts
   require, and update the `Last updated` line. A document that no change
   touches gets no new version.

5. **Assemble and check** each changed document as in
   [the drafting workflow](drafting.md), including the cross-document check
   and the fact-check sub-agent.

6. **Diff against the published version.**

   ```sh
   node ../skills/reconform-setup-legal/scripts/legal.cjs diff --from published/privacy.md --to privacy.md
   node ../skills/reconform-setup-legal/scripts/legal.cjs diff --from published/subprocessors.md --workspace legal-workspace.json
   ```

   The second command renders the subprocessor list the way the app does.
   Paste each diff's output into your reply to the founder, unchanged, in a
   `diff` code block. A plain-language summary can follow, but it doesn't
   replace the diff, and nothing is saved before the founder has seen it.
   Under each diff, tie every numbered change to the fact that caused it, for
   example
   `change 2: vendor-list (OpenAI added for Auto-schedule)`. A change that no
   fact explains is a mistake: undo it, or ask the founder. The standard terms
   never change; if a diff shows them, assemble again. Record the
   change-to-fact list in `review.md`.

7. **Keep facts and documents in step.** When the founder approves wording
   that goes beyond a saved fact, for example "Postmark also gets the email
   content" when `vendor-purposes` says only names and email addresses, update
   that fact in `legal-workspace.json` with the approved words and a new
   `confirmed_at`, and the vendor's `data_categories` if it applies. The
   documents and the saved facts must say the same thing before anything is
   saved. Then run:

   ```sh
   node ../skills/reconform-setup-legal/scripts/legal.cjs check-facts --file legal-workspace.json
   ```

   It fails when a saved vendor isn't named in `vendor-list`, or when a
   vendor's data category isn't in its `vendor-purposes` entry. Fix the fact
   (in the founder's approved words) or the vendor, and run it again.

8. **Save drafts.** Save the workspace with `save-workspace`, then each
   changed document with `save-draft`, adding `--summary` with one line on what
   changed and why. The app shows that line in the version history. The
   subprocessor list draft comes from the saved vendors in the app.

## Hand off

In test mode, publish with `publish-test` only after an explicit yes to the
exact text, as in the first run.

In live mode, nothing is published from here. For each changed document,
recommend a re-acceptance policy and say why:

| Policy (app label)              | Use when                                                                                                                                                                                            |
| ------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `required` (Require acceptance) | The change affects what users agreed to or how their data is handled: a new subprocessor, a new data use or category, data going to a new country, changed rights, fees, liability, or termination. |
| `notify` (Notify only)          | Users should know, but earlier acceptance still covers it: a removed vendor with nothing else changed, new contact details, a clarification that doesn't change practice.                           |
| `none` (No reacceptance)        | Typo and formatting fixes.                                                                                                                                                                          |

When unsure, recommend `required`. It accepts a grace period in days. With
`notify`, Reconform reports an informational status and the app decides how
to tell users; Reconform sends no email. If the subprocessor page promises
notice before a new subprocessor starts (the `subprocessor-change-notice`
fact), remind the founder to give that notice; Reconform doesn't send it.

Then tell the founder where to publish: open each saved draft under
**Documents**, review it, choose **Publish**, and pick the policy. For the
subprocessor list, open **Legal pages → Preview subprocessor draft**, save the
draft, and publish it the same way.
