---
name: 'hello'
description: 'A worked example showing dossier anatomy and the create-sign-verify-publish workflow.'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Hello World Dossier'
  dossier.version: '1.2.0'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Stable'
  dossier.objective: 'Demonstrate dossier structure and the create, sign, verify, publish workflow for authors writing their first dossier'
  dossier.category: '["documentation"]'
  dossier.tags: '["tutorial","example","getting-started"]'
  dossier.risk_level: 'low'
  dossier.risk_factors: '[]'
  dossier.requires_approval: 'false'
  dossier.content_scope: 'self-contained'
  dossier.authors: '[{"name":"Yuval Dimnik <yuval.dimnik@gmail.com>"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"d510fb567857ee5bd45aa4302cb21a255a6cd6b4bf7f7403936f26d902dd7424"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"S2N1QLgC/tOGWwtzWK082cQ7CLDuLEAzqaq/9+/4G7F7eg4L1pTWaKTeTQBbF9zRaCcfpdMfix+UsGdru8VvDg==","signed_at":"2026-10-07T12:37:02.493Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---
# Hello World Dossier

A dossier is a signed, versioned instruction file for an AI agent. This one is the smallest useful example: it shows what a dossier looks like on disk and how to create, sign, verify and publish your own.

## What you are looking at

A dossier is one `.ds.md` file in two parts:

1. **Frontmatter**: a YAML block between `---` lines. It follows the Agent Skills (agentskills.io) specification layout, so the file doubles as a skill any Agent Skills runtime can load. `name` and `description` sit at the top level. Everything dossier-specific (`title`, `version`, `risk_level`, `risk_factors`, `requires_approval`, the `checksum` and the `signature`) lives under `metadata` as `dossier.*` keys.
2. **Body**: the markdown an agent actually follows, like this page.

You do not write the frontmatter by hand. The CLI generates it, computes the checksum and signs the result.

## Create your own

Write the body as a plain markdown file, then let the CLI assemble the dossier:

```bash
# 1. Build a dossier from a plain markdown body
ai-dossier from-file my-task.md \
  --name my-task \
  --title "My Task" \
  --objective "What this dossier accomplishes, in a sentence or two" \
  --author "Your Name <you@example.com>" \
  -o my-task.ds.md

# 2. Sign it. The signature covers the frontmatter and the body together
ai-dossier sign my-task.ds.md --method ed25519 --key my-key --signed-by "Your Name <you@example.com>"

# 3. Check it before shipping
ai-dossier lint my-task.ds.md
ai-dossier verify my-task.ds.md

# 4. Publish to the registry
ai-dossier publish my-task.ds.md --namespace your-org/category
```

Publishing goes through the registry. There is no manifest to edit and nothing to commit by hand.

## Run it, or install it as a skill

```bash
ai-dossier run getting-started/hello
ai-dossier install-skill getting-started/hello
```

The second command copies the dossier into your skills directory unchanged, so its signature still verifies there.

## Good to know

- The `checksum` covers the body. The signature (`covers: "spec-frontmatter+body"`) also covers the frontmatter, so `risk_level` and `requires_approval` cannot be changed without breaking it.
- Bump `version` before you republish. The registry rejects a version that already exists, and it rejects a signature that does not verify.
- Keep each dossier focused on one task. If you are combining two jobs, write two dossiers.
- Older dossiers use a `---dossier` JSON header. They still verify and run; `ai-dossier format` converts an unsigned one to the layout above.
