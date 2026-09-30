# 2.1.0

Document `checkbox: "combined"` on consent sessions, which shows one checkbox
for several documents, such as terms and a privacy policy. The default
integration still requests `terms` only.

# 2.0.0

Use SDK 0.4.0 and the v2 hosted embed. The host form owns the submit button,
tracks completeness with `onChange`, and awaits `submit()`. Checking a box never
records acceptance.

# 1.0.0

Wire Reconform consent sessions, the consent screen, and backend status checks
into a web app. Includes recipes for Next.js, Express, Hono, React Router,
SvelteKit, and other stacks, and verifies the setup with `reconform doctor`.
