---
name: reconform-integrate
description: Add Reconform terms acceptance to a web app. Creates consent sessions on the backend for the signed-in user, shows the consent screen, and checks acceptance status before granting access. Use when asked to wire Reconform into an app; it can create a test sandbox with plain API calls first.
metadata:
  version: "2.3.0"
---

# Add terms acceptance to an app

Read [the API reference](references/api.md) and [the framework recipes](references/frameworks.md) before editing code.

## Before you start

1. Look for `RECONFORM_API_KEY` in the environment or the app's env file. If it is missing, ask the person whether to create a test sandbox, then follow [the sandbox steps](references/sandbox.md): `curl` calls that create a test sandbox with published example terms, save the key to the env file without printing it, and write `.reconform/context.json`.
2. Never print the key, put it in browser code, or commit the env file. Keep the env file and `.reconform/auth.header` in `.gitignore`.
3. You change the person's code, so keep to the files the integration needs and list every file you touched. Don't send their code or data to any service.
4. Do not edit `.reconform/legal/`. Another agent may be drafting documents there at the same time.

## Find the app's shape

Identify the framework, the package manager, where users sign in, and the first screen a signed-in user reaches. Find how backend code reads the current user's stable ID. Read the existing code before choosing a recipe; follow the app's conventions for routes, components, and environment variables.

If the app has no authentication, add a clearly marked development-only user ID, name it in your report, and say that it must be replaced before going live. Never let the browser choose the subject ID.

## Wire it

1. Install `@reconform/node@next` for the backend and `@reconform/react@next` for React apps, or `@reconform/js@next` otherwise.
2. Add a backend endpoint that creates a consent session for the signed-in user with `documents: ["terms"]` and returns only `client_secret` and `status`.
3. Show the consent screen with that secret where the signed-in experience starts. Use `onChange` to enable a host-owned submit button, and await `handle.submit()` or the React ref’s `submit()` from that button. Keep the button outside `ConsentGate`’s protected children. Checking a box never records acceptance. Skip the screen when the status is `nothing_required`.
4. Add a backend status check where the app grants access to protected features. Treat the browser `onAccept` callback as a UI signal, not authorization.
5. If `RECONFORM_API_URL` is set, pass it as the Node client's `baseUrl` and use the matching embed origin from the API reference.

Keep the change small. Do not restyle the app or refactor unrelated code.

## Check it

Run the setup check in [the sandbox steps](references/sandbox.md#5-check-the-setup). Then run the app's typecheck, tests, or build, whichever the project uses, and start the dev server. Open the page yourself only if you have browser tools, and never click accept: an acceptance records a person's consent, so the person must do it.

## Report back

Return this summary:

- **Files changed:** each path and a one-line reason.
- **Run:** the command that starts the app, and the URL where the consent screen appears.
- **Setup check:** the result of each check in the sandbox steps.
- **Needs attention:** development stubs, failing checks, or decisions for the person.

Tell the person they can accept the example terms in their browser now. Their real terms can replace the example later through `reconform-setup-legal` without code changes, because the integration requests the `terms` slug.
