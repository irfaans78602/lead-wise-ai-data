# LeadWise AI (Pty) Ltd — platform owner

One client, seeded by the loader, never updated by anything except
the loader.

- Country: ZA
- Segment: saas_it
- Currency: ZAR
- Language: en (default; ad leaves override per-leaf)
- Platform owner: yes; never billed

Channels shipped: web, WhatsApp, Messenger, Instagram, TikTok.
Locales shipped: en, ar, es, fr, zh.

The root ad (`source_ad_id: "leadwise"`) owns every piece of content.
All 25 leaves reuse it. `language_id` on the root is NULL; each leaf
declares its own.

## WhatsApp placeholder

`10-clients.json` currently carries:

    phone_number_id:       LEADWISE_PHONE_NUMBER_ID
    display_phone_number:  +27000000000

When the real number is registered in Meta Business Suite, replace
both values and re-run `10-clients.json --commit`. `phone_number_id`
is the primary key of `client_whatsapps`; the row is upserted in
place. No history to preserve on a Meta-issued ID.

## Updating an FAQ

To change a question's text, change its type, flip `mandatory`, or
add an option:

1. Edit the FAQ entry in `60-instance.json`.
2. Add `start_date` (the effective date) and, if you want a specific
   close date, `closes_previous`. Without `closes_previous`, the
   loader sets the old version's `end_date` to `start_date - 1 day`.
3. `python -m loader LeadWiseAI/60-instance.json` — preview.
4. `python -m loader LeadWiseAI/60-instance.json --commit`.

The old FAQ version stays in the table with its `end_date` set. The
runtime reads `active_faqs`, which filters to the current version
only. Nothing downstream sees the change until the effective date.

## Adding a new ad leaf

Add the channel to the `channels` array in `ad_matrix`, or the locale
to `locales`. Re-run `60-instance.json --seed --commit` if the ad is
genuinely new — the loader will insert it and leave existing leaves
untouched. Without `--seed`, either the leaf already exists (no-op)
or preflight errors with a clear message naming the missing key.