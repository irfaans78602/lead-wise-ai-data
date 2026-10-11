# lead-wise-ai-seed-json

Per-client JSON for the LeadWise AI loader.

Layout:

    LeadWiseAI/
    +-- .client-slug         one line: the log directory name
    +-- 10-clients.json      client identity, message, WhatsApp numbers
    +-- 60-instance.json     root ad, ad matrix, pricing
    +-- notes.md             free-form client context

    logs/                    runtime logs (gitignored)
    +-- <slug>/              one folder per client, named by .client-slug

Add a new client by creating a sibling folder with the same shape.
The folder name is for humans; `.client-slug` decides where logs land.

## Loading

From a checkout of `lead-wise-ai-loader`:

    export DATABASE_URL=postgres://...
    export LW_DATA_ROOT=/path/to/lead-wise-ai-seed-json
    export LW_SCHEMA_ROOT=/path/to/lead-wise-ai-schema

    # First load (--seed: default require is "new")
    python -m loader LeadWiseAI/10-clients.json --seed
    python -m loader LeadWiseAI/10-clients.json --seed --commit
    python -m loader LeadWiseAI/60-instance.json --seed
    python -m loader LeadWiseAI/60-instance.json --seed --commit

    # Later updates (preview, then commit)
    python -m loader LeadWiseAI/60-instance.json
    python -m loader LeadWiseAI/60-instance.json --commit

Without `--commit`, a load runs in preview mode: the full transaction
is opened, every write is performed, state is read back, and the whole
thing is rolled back. Nothing is committed.

Preview and commit produce byte-identical logs except for the `mode:`
line and the transaction verb. Diffing the two proves a commit did
exactly what its preview said it would.

## Rules

- `--seed` flips the default `require` from `existing` to `new`.
  Use it only for the initial load into an empty database.
- A JSON field `"new": true` explicitly demands a new row.
  `"new": false` demands an existing row. Absent means "use the
  CLI default".
- Every key in the file must resolve. A missing key errors before
  any write. No silent inserts.
- Versioned content (faqs, ad_confirmations, ad_reopen_prompts,
  qualification_score, client_messages) uses `closes_previous` when
  a new version should close the old one on a specific date. Absent,
  the loader infers `start_date - 1 day`.

See `docs/decision-34-loader-json-semantics.md` in the docs repo for
the JSON format in full.