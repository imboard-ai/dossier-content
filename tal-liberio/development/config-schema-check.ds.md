---
authors:
- name: tal
category:
- development
checksum:
  algorithm: sha256
  hash: 721ab8ddf9ad49df1e407c323f7bb771a9ffc8dbaecd35fdbda43e077e6728a6
estimated_duration:
  max_minutes: 8
  min_minutes: 2
name: config-schema-check
objective: Check if a configuration system supports a specific option or setting
schema_version: 1.0.0
status: draft
tags:
- configuration
- schema
- validation
- tooling
title: Config Schema Check
version: 1.0.0
---

# Config Schema Check

Check if a tool's configuration system supports a specific option or setting.

## Your Task

Determine if: **{option}**

### 1. Check JSON Schema (if available)

Many tools publish their config schema:
- `{tool}.schema.json` in the repo
- JSON Schema Store (schemastore.org)
- TypeScript types for config

Look for the option name, type, and allowed values.

### 2. Check Documentation

Review configuration docs:
- Official configuration reference
- Option listings with descriptions
- Default values documentation

### 3. Check Source Code

If schema isn't definitive:
- Look at config parsing code
- Find where the option is read and used
- Check for validation logic

### 4. Check Examples

Look for usage in:
- Official examples or starter templates
- Popular open source projects
- Documentation examples

## Output Format

### Answer
**[YES/NO]**: {option}

### Option Details

**Tool**: {name} v{version}+

**Option path**: `{path.to.option}`

**Type**: {string | boolean | number | array | object}

**Default value**: `{default}`

**Valid values**:
- `{value1}` - {description}
- `{value2}` - {description}

### Configuration Example

**JSON** (e.g., `.{tool}rc.json`):
```json
{
  "path": {
    "to": {
      "option": "value"
    }
  }
}
```

**YAML** (if supported):
```yaml
path:
  to:
    option: value
```

**JavaScript** (if supported):
```javascript
module.exports = {
  path: {
    to: {
      option: 'value'
    }
  }
};
```

### Documentation Reference

**Official docs**: {link}

**Added in version**: {version if known}

### Related Options

| Option | Description | Relationship |
|--------|-------------|--------------|
| `{related.option}` | {description} | {conflicts with / requires / see also} |

### Common Mistakes

- {Mistake 1}: {correct approach}
- {Mistake 2}: {correct approach}

## Notes

- Config options may be added in specific versions - check tool version
- Some options only work in certain file formats (JSON vs JS)
- CLI flags may override config file options
- Environment variables may provide alternative configuration