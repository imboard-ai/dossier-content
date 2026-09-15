---dossier
{
  "dossier_schema_version": "1.0.0",
  "name": "evidence-smoke-736",
  "title": "Evidence Smoke Test",
  "version": "1.1.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-09-15",
  "objective": "Throwaway dossier used to exercise the authoring-evidence workflow for issue #736. Safe to delete.",
  "category": [
    "development"
  ],
  "tags": [
    "smoke-test",
    "throwaway"
  ],
  "risk_level": "low",
  "risk_factors": [],
  "requires_approval": false,
  "authors": [
    {
      "name": "Yuval Dimnik"
    }
  ],
  "checksum": {
    "algorithm": "sha256",
    "hash": "665969792931caba0ed29d0640dd3017300f4da45f5a5a0298b3c94f18c6dcc5"
  },
  "signature": {
    "algorithm": "ed25519",
    "signature": "LEw0TfpuQM8S67Qmt9ki0aKi6AHMLfOTjHjsx3OB7/E9Thil1xO2W8VsOuf7Ylx7eJlcWzjIbUd5ijUU4rhhDQ==",
    "public_key": "XFfJpThOYVckpc5i6LLPF7Jk/7bHGlgxUwqoV7g9So4=",
    "signed_at": "2026-09-15T11:41:33.980Z",
    "covers": "frontmatter+body",
    "key_id": "smoke-test",
    "signed_by": "Yuval Dimnik <yuval.dimnik@gmail.com>"
  }
}
---

# Evidence Smoke Test

## Objective

Throwaway dossier, published under the `yuvaldim` namespace, to prove the
authoring-evidence workflow end-to-end for issue #736. Not for real use.

## Actions to Perform

### Step 1: Print a greeting, exactly once

Print `hello from the evidence smoke test` exactly once. Do not repeat the
greeting even if the step is re-entered — a prior draft of this step had no
"exactly once" qualifier and an agent re-ran it on every retry, printing the
greeting N times for N attempts.
