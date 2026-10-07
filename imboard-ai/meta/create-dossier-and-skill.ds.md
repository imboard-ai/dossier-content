---
name: 'create-dossier-and-skill'
description: 'Guide an LLM agent to create a well-formed .ds.md dossier file and its companion Claude Code skill (SKILL.md) from user-provided parameters'
metadata:
  dossier.dossier_schema_version: '1.0.0'
  dossier.title: 'Create Dossier and Skill'
  dossier.version: '2.0.2'
  dossier.protocol_version: '"1.0"'
  dossier.status: 'Draft'
  dossier.last_updated: '2026-03-20'
  dossier.objective: 'Guide an LLM agent to create a well-formed .ds.md dossier file and its companion Claude Code skill (SKILL.md) from user-provided parameters'
  dossier.category: '["development","documentation"]'
  dossier.tags: '["dossier-creation","authoring","meta-dossier","skill-creation","scaffolding"]'
  dossier.estimated_duration: '{"max_minutes":10,"min_minutes":3}'
  dossier.risk_level: 'low'
  dossier.risk_factors: '["modifies_files"]'
  dossier.requires_approval: 'false'
  dossier.destructive_operations: '["Creates a new .ds.md dossier file at the user-specified path","Creates a SKILL.md file in ~/.claude/skills/<skill-name>/"]'
  dossier.inputs: '{"optional":[{"default":"<kebab-case-title>.ds.md","description":"Output file path for the dossier (must end in .ds.md)","example":"deploy-aws-production.ds.md","name":"file","type":"string"},{"description":"Dossier category (array)","example":"devops","name":"category","type":"string"},{"description":"Comma-separated tags for searchability","example":"aws, deployment, production","name":"tags","type":"string"}],"required":[{"description":"Human-readable title for the dossier","example":"Deploy to AWS Production","name":"title","type":"string"},{"description":"Clear statement of what the dossier accomplishes","example":"Deploy the application to AWS ECS with zero-downtime rollout","name":"objective","type":"string"},{"description":"Risk assessment level","example":"medium","name":"risk_level","type":"string","validation":"low | medium | high | critical"}]}'
  dossier.outputs: '{"files":[{"description":"The generated dossier file with valid JSON frontmatter and declarative body","format":"markdown","path":"${output_file}"},{"description":"Companion Claude Code skill that enables the dossier via slash command","format":"markdown","path":"~/.claude/skills/${skill-name}/SKILL.md","required":true}]}'
  dossier.authors: '[{"name":"Dossier Team"}]'
  dossier.checksum: '{"algorithm":"sha256","hash":"fba556e1ad79a72107f83fa005068edec99e3814d123bd57099e007293456890"}'
  dossier.signature: '{"algorithm":"ed25519","covers":"spec-frontmatter+body","key_id":"imboard-ai","public_key":"m97FPrnq/zKlQArLvJl3bTZCUMWWpp/d0UJ/OfUKZeE=","signature":"7Z4knhHYea9lpIV0OEZniq7tXhPIIv61snwr4Ih9InpCLwUXair08PVs86xkE2i8U1nqsq+8XQ4MRp+sK0q8Ag==","signed_at":"2026-10-07T12:03:44.498Z","signed_by":"Yuval Dimnik <yuval.dimnik@gmail.com>"}'
---
# Create Dossier and Skill

## Objective

Create a well-formed `.ds.md` dossier file and its companion Claude Code skill (`~/.claude/skills/<name>/SKILL.md`). Read parameters from the `USER-PROVIDED CONTEXT` header prepended above this dossier. Prompt interactively for any values marked "Not specified".

## Constraints

### Frontmatter format

- Frontmatter is **JSON** (not YAML) enclosed between `---dossier` and `---` delimiters
- New dossiers default to `"status": "Draft"`
- The file extension must be `.ds.md`

### Required frontmatter fields (all 9)

| Field | Value |
|-------|-------|
| `dossier_schema_version` | `"1.0.0"` (constant) |
| `title` | From user input |
| `version` | `"1.0.0"` for new dossiers |
| `protocol_version` | `"1.0"` |
| `status` | `"Draft"` |
| `objective` | From user input |
| `risk_level` | From user input — one of: `low`, `medium`, `high`, `critical` |
| `requires_approval` | `false` for low/medium, `true` for high/critical |
| `checksum` | `{"algorithm": "sha256", "hash": "0000...0000"}` placeholder — user must run `dossier checksum --update` after |

### Valid enum values

- **`category`**: `devops`, `database`, `development`, `data-science`, `security`, `testing`, `deployment`, `maintenance`, `setup`, `migration`, `monitoring`, `infrastructure`, `ci-cd`, `documentation`
- **`risk_level`**: `low`, `medium`, `high`, `critical`
- **`status`**: `Draft`, `Stable`, `Deprecated`, `Experimental`
- **`risk_factors`**: `modifies_files`, `deletes_files`, `modifies_cloud_resources`, `requires_credentials`, `network_access`, `executes_external_code`, `database_operations`, `system_configuration`

### Body style

The dossier body must follow declarative authoring guidelines:
- Tell agents **what** to achieve and **why**, not **how** step-by-step
- Sections: Objective, Constraints, Known Pitfalls, Decision Points, Validation (minimum)
- Avoid naming specific tools unless they are genuine constraints
- Avoid step-by-step bash commands for standard operations

## Schema Reference

### Recommended fields

Include these when relevant to the dossier's purpose:

| Field | Type | When to include |
|-------|------|-----------------|
| `category` | `string[]` | Always — aids discovery |
| `tags` | `string[]` | Always — free-form search terms |
| `inputs` | `object` | When the dossier accepts parameters |
| `outputs` | `object` | When the dossier produces files or artifacts |
| `estimated_duration` | `object` | When timing matters (`min_minutes`, `max_minutes`) |
| `authors` | `object[]` | For published dossiers |
| `risk_factors` | `string[]` | For medium+ risk dossiers |
| `destructive_operations` | `string[]` | For any dossier that modifies/deletes files |
| `last_updated` | `string` | Always — ISO 8601 date |

### Optional fields

| Field | Type | Purpose |
|-------|------|---------|
| `tools_required` | `object[]` | External tools needed (name, version, check_command) |
| `coupling` | `object` | Dependency level with other systems |
| `rollback` | `object` | How to undo changes |
| `relationships` | `object` | Links to related dossiers (preceded_by, followed_by, etc.) |
| `prerequisites` | `object[]` | Conditions that must exist before execution |
| `validation` | `object` | Machine-verifiable success criteria and commands |
| `mcp_integration` | `object` | MCP server integration metadata |
| `content_scope` | `string` | `self-contained` or `references-external` |

## Decision Points

### Risk-based defaults

| Risk level | `requires_approval` | Suggested `risk_factors` |
|------------|---------------------|--------------------------|
| `low` | `false` | `[]` or `["network_access"]` |
| `medium` | `false` | `["modifies_files"]` |
| `high` | `true` | `["modifies_files", "deletes_files"]` |
| `critical` | `true` | `["modifies_cloud_resources", "database_operations"]` |

### Inputs/outputs inclusion

- Include `inputs` if the dossier is parameterized (accepts user-provided values)
- Include `outputs.files` if the dossier creates or modifies files
- Include `outputs.state_changes` for infrastructure or configuration changes

### Skill creation

The companion skill file enables the dossier as a Claude Code slash command:
- Skill directory: `~/.claude/skills/<skill-name>/` where `<skill-name>` is the kebab-case title
- File: `SKILL.md` inside that directory
- The skill should describe when to trigger and reference the dossier by registry name or local path

## Known Pitfalls

- **JSON, not YAML**: The frontmatter must be valid JSON between `---dossier` / `---` delimiters. YAML is not supported.
- **`category` is a fixed enum**: Custom category values will fail schema validation. Only use the 14 values listed in the Constraints section.
- **`risk_factors` is also a fixed enum**: Only the 8 values listed above are valid.
- **Checksum must be updated separately**: Write the placeholder hash (`0000...`), then instruct the user to run `dossier checksum <file> --update`. The checksum cannot be computed inline.
- **Body should be declarative**: Avoid procedural step-by-step instructions. Agents perform better with goal-oriented guidance (what/why) than prescriptive instructions (how).
- **Skill name = kebab-case of title**: Convert the dossier title to kebab-case for the skill directory name (e.g., "Deploy to AWS" → `deploy-to-aws`).
- **`name` field controls registry path**: If the dossier will be published, set the `name` field explicitly. The publish command uses `name || title` for the registry path.

## Validation

- [ ] `.ds.md` file created at the specified output path
- [ ] Frontmatter parses as valid JSON with all 9 required fields present
- [ ] `risk_level` is one of the valid enum values
- [ ] `category` values (if present) are all valid enum values
- [ ] Body contains at minimum: Objective, Constraints, and Validation sections
- [ ] Body follows declarative style (goals and constraints, not step-by-step commands)
- [ ] `~/.claude/skills/<skill-name>/SKILL.md` created with appropriate trigger description
- [ ] Checksum field contains placeholder (user runs `dossier checksum --update` after)

## Next Steps

After the dossier and skill are created:

1. **Review** both files for accuracy and completeness
2. **Update checksum**: `dossier checksum <file> --update`
3. **Sign** (optional): `dossier sign <file>`
4. **Verify**: `dossier verify <file>`
5. **Publish** (optional): `dossier publish <file>`

## Example Output

A minimal but complete dossier for reference:

```markdown
---dossier
{
  "dossier_schema_version": "1.0.0",
  "title": "Set Up Redis Cache",
  "version": "1.0.0",
  "protocol_version": "1.0",
  "status": "Draft",
  "last_updated": "2026-03-20",
  "objective": "Configure Redis as the application cache layer with connection pooling and health checks",
  "category": ["infrastructure", "setup"],
  "tags": ["redis", "caching", "performance"],
  "risk_level": "medium",
  "requires_approval": false,
  "risk_factors": ["modifies_files", "system_configuration"],
  "estimated_duration": {"min_minutes": 10, "max_minutes": 30},
  "checksum": {
    "algorithm": "sha256",
    "hash": "0000000000000000000000000000000000000000000000000000000000000000"
  }
}
---
# Set Up Redis Cache

## Objective

Configure Redis as the application cache layer with connection pooling,
health checks, and graceful fallback when Redis is unavailable.

## Constraints

- Must use connection pooling (not single-connection mode)
- Health check endpoint must report Redis status
- Application must degrade gracefully if Redis is down (fall through to DB)
- Cache TTL defaults: 5 minutes for queries, 1 hour for configuration

## Known Pitfalls

- The default `maxmemory-policy` is `noeviction`, which causes write errors
  when memory is full. Set it to `allkeys-lru` for cache use cases.

## Validation

- [ ] Redis connection pool initializes on application startup
- [ ] Health endpoint reports Redis connectivity status
- [ ] Application functions correctly when Redis is stopped
- [ ] Cache hit/miss ratio is observable via metrics or logs
```
