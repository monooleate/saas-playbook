# 18 — Example: SvelteKit + Supabase

> **Status:** Outline.

API routes under `apps/app/src/routes/api/v1/...`. Auth via a hook
that resolves `Authorization: Bearer <key>` against an `api_keys`
table (key hashed at rest). OpenAPI generated from Zod schemas. Webhook
delivery via a separate Fly Machine running BullMQ on Redis Cloud free
tier; or Inngest.
