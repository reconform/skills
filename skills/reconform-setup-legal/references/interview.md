# Interview branches

Begin with product, buyer, and company identity. Ask two or three questions at a
time when that makes the conversation easier. Follow dependencies rather than
reciting this entire list.

| Topic              | Facts to establish                                                            | Useful follow-up                                                                 |
| ------------------ | ----------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| Company            | Legal entity, contact address, product name, privacy contact                  | Is the contracting entity the same entity handling the data?                     |
| Product            | What it does, B2B or consumer audience, age restrictions                      | Does it involve health, financial, or other sensitive data?                      |
| Contract           | Fees, billing period, renewal, cancellation, refunds, trials, support         | What happens to access and data after cancellation?                              |
| Legal choices      | Governing law, chosen courts, liability choices, attachments                  | Which choices are approved and which need outside review?                        |
| Data               | Categories collected, sources, purposes, account data versus customer content | Is any data used for advertising or model training?                              |
| Retention          | Retention by category, account deletion, logs, backups                        | Does the deletion promise include backups? How long until they expire?           |
| Vendors            | Actual processors, purpose, data shared, processing locations, public website | Is this a confirmed vendor or merely a package installed in the repository?      |
| Transfers          | Countries involved, processing roles, existing transfer arrangements          | Which supporting documents exist? Ask for them rather than choosing a mechanism. |
| Security           | Controls the business actually operates, access policy, incident contact      | Is this implemented now, or an aspiration?                                       |
| Rights and contact | How requests are received and fulfilled, applicable customer requirements     | Who handles the request, and what response process exists?                       |

For each fact retain a short source, such as "Founder confirmed in this
interview" or a repository path for a proposal. Record pending questions instead
of inferred answers. Confirmation timestamps are epoch milliseconds.

If an answer contradicts an existing document, show the conflict. Do not silently
change the source facts to match the document or the document to match a guess.
Reconfirm affected facts on later runs rather than assuming old answers remain
current. Suggest affected documents; changing a fact does not publish a revision.

For an empty subprocessor list, explicitly confirm that there are none to list.
An unfinished vendor interview is not an empty-list assertion.
