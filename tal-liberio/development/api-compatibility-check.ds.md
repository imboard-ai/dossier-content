---
authors:
- name: tal
category:
- development
checksum:
  algorithm: sha256
  hash: b1b93323f387339673693ffe348ba2fa5df7f1d333b30ae789cdcc218d6c214c
estimated_duration:
  max_minutes: 10
  min_minutes: 2
name: api-compatibility-check
objective: Check if a library or service API supports a specific method, endpoint,
  or capability
schema_version: 1.0.0
status: draft
tags:
- api
- compatibility
- sdk
- documentation
title: API Compatibility Check
version: 1.0.0
---

# API Compatibility Check

Check if a library or service API supports a specific method or capability.

## Your Task

Determine if: **{capability}**

### 1. Check Type Definitions

For typed languages, examine:
- TypeScript definitions (`.d.ts` files, `@types/*` packages)
- Python type hints or stubs
- Go interfaces
- API client SDK source code

Look for the method signature, parameters, and return types.

### 2. Check Documentation

Review official documentation:
- API reference docs
- SDK documentation
- Method/endpoint listings
- Parameter descriptions

### 3. Check Implementation

If source is available:
- Look at the actual implementation
- Check for feature flags or conditions
- Verify the method is exported/public

### 4. Verify with Examples

Look for:
- Official examples in docs
- Test files in the SDK repo
- Community examples or Stack Overflow answers

## Output Format

### Answer
**[YES/NO/PARTIAL]**: {capability}

### API Details

**Library/Service**: {name} v{version}

**Method/Endpoint**:
```
{signature or endpoint}
```

**Parameters**:
| Name | Type | Required | Description |
|------|------|----------|-------------|
| {param} | {type} | {yes/no} | {description} |

**Returns**: {return type and description}

### Documentation Reference

**Official docs**: {link}

**Relevant section**: {quote or summary}

### Usage Example

```{language}
// Example showing how to use this capability
{working code example}
```

### Limitations

- **Rate limits**: {if applicable}
- **Required permissions/scopes**: {if applicable}
- **Not available in**: {regions, plans, versions}
- **Deprecated**: {yes/no, replacement if yes}

### Edge Cases

- {Edge case 1 and how to handle}
- {Edge case 2 and how to handle}

## Notes

- Type definitions may lag behind actual API capabilities
- Check both REST API docs and SDK docs (may differ)
- Beta/preview features may require opt-in or special headers
- Some features are plan/tier dependent (free vs paid)