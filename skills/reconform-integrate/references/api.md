# Reconform integration API, skill version 1.0.0

Credentials come from the local environment. `RECONFORM_API_KEY` is organization- and mode-scoped: `rk_test_` keys only see test documents, and `rk_live_` keys only live ones. Keep keys on the server.

| Environment          | `RECONFORM_API_URL` / Node `baseUrl`    | Browser `embedOrigin`                  |
| -------------------- | --------------------------------------- | -------------------------------------- |
| Production (default) | unset, or `https://api.reconform.co/v1` | unset, or `https://embed.reconform.co` |
| Staging              | `https://api-staging.reconform.co/v1`   | `https://embed-staging.reconform.co`   |

## Backend

`new Reconform(apiKey, { baseUrl })` from `@reconform/node`.

- `reconform.consentSessions.create({ subject: { external_id }, documents: ["terms"], require_scroll: true })` calls `POST /consent_sessions`. It returns `status` (`pending` or `nothing_required`), `client_secret`, and `expires_at` (one hour). Send only the client secret and status to the browser.
- `reconform.subjects.status(\`ext:${userId}\`)`calls`GET /subjects/{id}/status`and returns`needs_action` plus per-document states. Use it to decide access.
- `subject.external_id` is the app's stable user ID from the authenticated session. `email` and `name` are optional.

Errors throw `ReconformError` with `status` and `code`. A live Free organization past its monthly active user allowance returns 402 `quota_exceeded`; show a retry path and do not grant protected access on an unknown result. Test mode has no quota.

## Browser

- React: wrap content in `<ReconformProvider embedOrigin={...}>` and use `<ConsentGate clientSecret={secret} error="...">`, which hides children until consent is complete, or `<ConsentEmbed clientSecret={secret} onAccept={...} />`.
- Other frameworks: `Reconform.mount(element, { clientSecret, embedOrigin, onAccept, onError, onExpire })` from `@reconform/js` returns a handle with `submit()` and `unmount()`.

Test-mode consent screens work on any `localhost` or `127.0.0.1` port with no configuration. Live mode requires the app's exact origin under **Organization → Allowed origins** in Reconform.

## Documents

The integration requests documents by slug. A sandbox has published example terms under `terms`. Publishing a new version of `terms` changes what users see without code changes. Never create acceptance records through the API on a person's behalf, and never publish documents from this skill.

See the [API reference](https://www.reconform.co/docs/api/) and [quickstart](https://www.reconform.co/docs/).

## Manual submission in SDK 0.4.0

Both layouts require a host-owned submit button. Check/uncheck emits `onChange`;
never treat `complete` as recorded acceptance. Await `handle.submit()` in browser
integrations, or use a `ConsentEmbedHandle` ref in React. Keep the button outside
`ConsentGate`'s protected children. Disable it while incomplete or submitting.
Handle `accepted`, `already_accepted`, and `nothing_required` as successful results;
show `error.message` for errors. On `outcome_unknown`, retry `submit()` to confirm.
A known save failure clears selection and requires checking again. Backend status
checks still decide access. Upgrade all iframe clients from 0.3.x; legacy clients
fail with `sdk_upgrade_required`.
