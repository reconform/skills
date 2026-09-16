# Framework recipes

Adapt these to the app's existing structure, auth helper, and naming. `getUserId` stands for however the app reads the signed-in user on the server. Every recipe requests `terms`.

## Shared backend helper

```ts
import { Reconform } from "@reconform/node";

const reconform = new Reconform(process.env.RECONFORM_API_KEY!, {
  baseUrl: process.env.RECONFORM_API_URL,
});

export async function createConsent(userId: string) {
  const session = await reconform.consentSessions.create({
    subject: { external_id: userId },
    documents: ["terms"],
    require_scroll: true,
  });
  return { status: session.status, client_secret: session.client_secret };
}

export async function needsConsent(userId: string) {
  return (await reconform.subjects.status(`ext:${userId}`)).needs_action;
}
```

## Next.js App Router

`app/api/reconform/session/route.ts`:

```ts
import { createConsent } from "@/lib/reconform";

export async function POST() {
  const userId = await getUserId(); // the app's auth
  if (!userId) return new Response("Unauthorized", { status: 401 });
  return Response.json(await createConsent(userId));
}
```

A client component where the signed-in area starts:

```tsx
"use client";
import { useEffect, useRef, useState } from "react";
import { ConsentGate, ReconformProvider } from "@reconform/react";
import type { ConsentEmbedHandle } from "@reconform/react";

export function TermsGate({ children }: { children: React.ReactNode }) {
  const handle = useRef<ConsentEmbedHandle>(null);
  const [complete, setComplete] = useState(false);
  const [busy, setBusy] = useState(false);
  const [error, setError] = useState("");
  const [consent, setConsent] = useState<{
    status: string;
    client_secret: string;
  } | null>(null);
  useEffect(() => {
    void fetch("/api/reconform/session", { method: "POST" })
      .then((response) => response.json())
      .then(setConsent);
  }, []);
  if (!consent) return null;
  if (consent.status === "nothing_required") return <>{children}</>;
  return (
    <ReconformProvider
      embedOrigin={process.env.NEXT_PUBLIC_RECONFORM_EMBED_ORIGIN}
    >
      <ConsentGate
        ref={handle}
        clientSecret={consent.client_secret}
        onChange={(event) => {
          setComplete(event.complete);
          setBusy(event.status === "submitting");
        }}
        error="Terms are unavailable. Try again."
      >
        {children}
      </ConsentGate>
      <button
        disabled={!complete || busy}
        onClick={async () => {
          const result = await handle.current?.submit();
          setError(result?.status === "error" ? result.error.message : "");
        }}
      >
        Continue
      </button>
      {error && <p role="alert">{error}</p>}
    </ReconformProvider>
  );
}
```

Call `needsConsent(userId)` in server components, route handlers, or middleware-backed checks that protect features.

## Next.js Pages Router

Use `pages/api/reconform/session.ts` with the same helper and `export default async function handler(req, res)`, returning `res.status(200).json(await createConsent(userId))` for `POST`. Reuse the `TermsGate` component.

## Express

```ts
app.post("/api/reconform/session", requireAuth, async (req, res) => {
  res.json(await createConsent(req.user.id));
});
app.use("/app", requireAuth, async (req, res, next) => {
  if (await needsConsent(req.user.id)) return res.redirect("/accept-terms");
  next();
});
```

## Hono

```ts
app.post("/api/reconform/session", async (c) => {
  const userId = c.get("userId");
  return userId
    ? c.json(await createConsent(userId))
    : c.text("Unauthorized", 401);
});
```

## React Router or Remix

Create the session in the loader of the signed-in layout route and return `{ consent }`. Render `TermsGate`'s gate with `useLoaderData()`. Check `needsConsent` in loaders and actions for protected routes.

## SvelteKit

`src/routes/api/reconform/session/+server.ts` returns `json(await createConsent(locals.user.id))`. In the layout component:

```svelte
<script lang="ts">
  import { onMount } from "svelte";
  import { Reconform } from "@reconform/js";
  let element: HTMLElement;
  let handle: ReturnType<typeof Reconform.mount> | undefined;
  let complete = false;
  let busy = false;
  let error = "";
  onMount(() => {
    let active = true;
    void fetch("/api/reconform/session", { method: "POST" })
      .then(response => response.json())
      .then(consent => {
        if (!active) return;
        handle = Reconform.mount(element, {
          clientSecret: consent.client_secret,
          onChange: event => { complete = event.complete; busy = event.status === "submitting"; },
          onError: cause => { error = cause.message; },
        });
      });
    return () => { active = false; handle?.unmount(); };
  });
  async function submit() {
    const result = await handle?.submit();
    if (!result) return;
    if (result.status === "error") error = result.error.message;
    else location.reload();
  }
</script>
<div bind:this={element}></div>
<button disabled={!complete || busy} on:click={submit}>Continue</button>
{#if error}<p role="alert">{error}</p>{/if}
```

Check `needsConsent` in `+layout.server.ts` for protected routes.

## Any other stack

Add one authenticated backend endpoint that calls `createConsent`, mount the consent screen with `@reconform/js` from its client secret, await `handle.submit()` from the host form, and call `needsConsent` wherever the app decides access. A non-Node backend can call `POST /v1/consent_sessions` and `GET /v1/subjects/ext:{id}/status` directly with `Authorization: Bearer $RECONFORM_API_KEY` and an `Idempotency-Key` header on the POST.
