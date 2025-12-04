---
authors:
- name: tal
category:
- setup
checksum:
  algorithm: sha256
  hash: 966c420854306a1ff739ce1cf0586913848b6121328de9d0e44f0f4cffa9b7e0
estimated_duration:
  max_minutes: 5
  min_minutes: 2
name: validate-project-config
objective: Validate project configuration files and environment setup to ensure all
  required settings are properly configured
schema_version: 1.0.0
status: draft
tags:
- configuration
- environment
- validation
- onboarding
title: Validate Project Configuration
version: 1.0.0
---

# Validate Project Configuration

Validate that all project configuration files are properly set up and environment variables are correctly configured.

## When to Use

- After cloning a new repository
- When debugging "it works on my machine" issues
- When onboarding to a new project
- After environment changes

## Your Task

Check that the project's configuration is complete and valid by examining:
1. Configuration files exist and are readable
2. Environment variables are set
3. No placeholder values remain
4. No conflicting configurations

## Areas to Check

### 1. Identify Project Type

Detect the technology stack:
- **Node.js**: `package.json` present
- **Python**: `requirements.txt`, `setup.py`, or `pyproject.toml` present
- **Go**: `go.mod` present
- **Rust**: `Cargo.toml` present
- **Ruby**: `Gemfile` present

### 2. Configuration Files

Check for common configuration files:
- `.env`, `.env.local`, `.env.development`, `.env.production`
- `config.json`, `config.yaml`, `settings.json`
- Framework-specific: `next.config.js`, `vite.config.ts`, etc.

For each file found:
- Is it readable?
- Is it empty?
- Does it have placeholder values (`YOUR_`, `REPLACE_`, `xxx`, `TODO`)?

### 3. Environment Variable Templates

Check if template files exist:
- `.env.example` → should have corresponding `.env`
- `.env.sample` → should have corresponding `.env`
- `config.example.json` → should have corresponding `config.json`

### 4. Common Issues to Flag

- `.env.example` exists but `.env` doesn't
- Placeholder values in configuration files
- Hardcoded secrets or API keys in source code
- Conflicting values between config files
- Missing required environment variables referenced in code

## Output Format

### Project Type
**Detected**: [Node.js/Python/Go/etc.]
**Framework**: [Express/Django/etc. if detectable]

### Configuration Files Found

| File | Status | Notes |
|------|--------|-------|
| `.env` | Found | 245 bytes |
| `.env.local` | Missing | — |
| `config.json` | Found | Has placeholders |

### Environment Variables

**From `.env`:**
- `DATABASE_URL` - Set
- `API_KEY` - Placeholder value
- `SECRET_KEY` - Missing

### Issues Found

1. **Missing .env file**
   - `.env.example` exists but `.env` doesn't
   - Fix: `cp .env.example .env` and configure values

2. **Placeholder values detected**
   - File: `.env:12`
   - Value: `API_KEY=YOUR_API_KEY_HERE`
   - Fix: Replace with actual API key

3. **Potential secret in source code**
   - File: `src/config.ts:45`
   - Issue: Hardcoded API endpoint with key
   - Fix: Move to environment variable

### Recommendations

1. **Immediate**: Copy `.env.example` to `.env` and fill in values
2. **Immediate**: Replace placeholder values in configuration
3. **Consider**: Add `.env` to `.gitignore` if not already

## Notes

- This workflow only READS configuration files, it doesn't modify them
- Never commit actual `.env` files to version control
- Keep `.env.example` files up to date with required variables
- Use the output to document required configuration for your team