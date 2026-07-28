---dossier
{
  "dossier_schema_version": "1.0.0",
  "protocol_version": "1.0",
  "category": [
    "documentation"
  ],
  "tags": [
    "tutorial",
    "example",
    "getting-started"
  ],
  "risk_level": "low",
  "risk_factors": [],
  "requires_approval": false,
  "content_scope": "self-contained",
  "description": "A worked example showing dossier anatomy and the create-sign-verify-publish workflow.",
  "name": "hello",
  "title": "Hello World Dossier",
  "version": "1.1.0",
  "status": "Stable",
  "objective": "Demonstrate dossier structure and the create, sign, verify, publish workflow for authors writing their first dossier",
  "authors": [
    {
      "name": "Yuval Dimnik <yuval.dimnik@gmail.com>"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "2caae4c1dc451f6ad544b4ce56a342705d8de9e690bbe3b8376d72dd76048d47"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "tleSPJ4TNRd75WsxpJF+fWbOpNS248hrROa4qvxAs7ppS72a4TQ4+psAv3299Zxb66ItJI0y74S/SfSreDdYDg==",
    "public_key": "m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=",
    "signed_at": "2026-07-28T06:23:15.375Z",
    "covers": "frontmatter+body",
    "key_id": "imboard-ai",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---
# Hello World Dossier

This is a sample dossier that demonstrates the basic structure and format.

## Purpose

Use this dossier as a reference when creating new dossiers for the registry.

## Anatomy of a Dossier

A dossier is a single `.ds.md` file with two parts:

1. **Frontmatter** — a `---dossier` fenced block containing JSON metadata. It carries
   the identity (`name`, `title`, `version`), the security declarations
   (`risk_level`, `risk_factors`, `requires_approval`), the body `checksum`, and an
   optional `signature`.
2. **Body** — the markdown instructions an agent actually executes.

You do not hand-write the frontmatter. The CLI generates it, computes the checksum,
and signs the result.

## Creating Your Own

Write the body as a plain `.md` file, then let the CLI assemble the dossier:

```bash
# 1. Build a dossier from a plain markdown body
ai-dossier from-file my-task.md \
  --name my-task \
  --title "My Task" \
  --objective "What this dossier accomplishes, in a sentence or two" \
  --author "Your Name <you@example.com>" \
  -o my-task.ds.md

# 2. Sign it (binds the frontmatter and body together)
ai-dossier sign my-task.ds.md --method ed25519 --key my-key --signed-by "Your Name <you@example.com>"

# 3. Check it before shipping
ai-dossier lint my-task.ds.md
ai-dossier verify my-task.ds.md

# 4. Publish to the registry
ai-dossier publish my-task.ds.md --namespace your-org/category
```

Publishing goes through the registry — there is no manifest to edit and nothing to
commit to a content repository by hand.

## Running a Dossier

```bash
ai-dossier run getting-started/hello
```

To install one as a Claude Code skill instead:

```bash
ai-dossier install-skill getting-started/hello
```

## Notes

- Frontmatter is `---dossier` + JSON. The CLI writes and maintains it for you.
- The `checksum` covers the body; a signature with `covers: "frontmatter+body"` binds
  the security metadata too, so `risk_level` and `requires_approval` cannot be altered
  without invalidating it.
- Bump `version` before republishing — the registry rejects a version that already exists.
- Keep each dossier focused on a single task. If you are combining two jobs, write two
  dossiers.
