---
applyTo: "**/*.{ts,tsx}"
---

## Project Overview

This is a **configuration map repository** that maintains environment-specific configuration files for a test application across multiple deployment environments: Development (dev), System Integration Testing (sit), and User Acceptance Testing (uat).

## Repository Structure

```
.
├── dev/
│   └── config.yaml      # Development environment configuration
├── sit/
│   └── config.yaml      # System Integration Testing environment configuration
└── uat/
    └── config.yaml      # User Acceptance Testing environment configuration
```

## Environment Progression

Configurations are organized by environment maturity:

1. **dev/** - Local development environment with relaxed security settings
2. **sit/** - System Integration Testing environment with moderate security
3. **uat/** - User Acceptance Testing environment with production-like security

## Automatic Code Review Guidelines

When reviewing changes to this repository, Copilot should focus on identifying **discrepancies and erroneous code modifications** by examining:

### 1. Environment-Specific Consistency

- **URLs and Hostnames**: Ensure URLs match the target environment
  - `dev/` should use `http://dev.example.local`
  - `sit/` should use `https://sit.example.local`
  - `uat/` should use `https://uat.example.local`
  
- **Ports**: Verify appropriate port usage
  - Development typically uses `:3000`, `:8000`, `:8080`
  - UAT should use standard HTTPS port `443`

### 2. Security Configuration Alignment

#### Critical Security Flags (`unsafe_for_prod` section)

These flags **MUST** align with environment safety levels:

| Setting | dev/ | sit/ | uat/ |
|---------|------|------|------|
| `DEBUG` | `true` | `false` | `false` |
| `ALLOW_HTTP` | `true` | `false` | `false` |
| `SKIP_SSL_VERIFY` | `true` | `false` | `false` |
| `ENABLE_SQL_LOGGING` | `true` | `false` | `false` |
| `SEED_DEMO_DATA` | `true` | `false` | `false` |
| `RELAXED_CSP` | `true` | `false` | `false` |

**⚠️ WATCH FOR**: Any `unsafe_for_prod` flag set to `true` in `sit/` or `uat/` environments

#### Security Settings

- **CSRF Protection**: Must be `true` in `sit/` and `uat/`, can be `false` in `dev/`
- **TLS/SSL**: 
  - `dev/` can use `http://` and `tls: false`
  - `sit/` and `uat/` must use `https://` and `tls: true`
  - Database connections in `sit/` and `uat/` must use `sslmode=require`
- **CORS Origins**: Should only allow environment-specific domains
- **Content Security Policy**: Should be restrictive in `sit/` and `uat/`

#### Password Policies

Progressive password strength requirements:
- `dev/`: `min_length: 6`, `require_special: false`
- `sit/`: `min_length: 8`, `require_special: true`
- `uat/`: `min_length: 10`, `require_special: true`

### 3. Logging Configuration

Appropriate logging levels by environment:
- `dev/`: `level: debug` (verbose)
- `sit/`: `level: info` (moderate)
- `uat/`: `level: warn` (production-ready)

**Stack traces** should be disabled in `sit/` and `uat/` (`enable_stack_traces: false`)

### 4. Feature Flags

- `payment_sandbox_mode`: Should be `true` in `dev/` and `sit/`, but `false` in `uat/`
- Beta features should typically be disabled in `uat/` to match production

### 5. Common Mistakes to Flag

❌ **Accidental cross-environment pollution**:
- Dev secrets/keys copied to sit/uat
- HTTP URLs in sit/uat configs
- Debug mode enabled in sit/uat

❌ **Inconsistent naming**:
- Database names should follow pattern: `{env}_db` (underscores per SQL conventions)
- S3 buckets should follow pattern: `{env}-test-bucket` (hyphens per AWS conventions)
- OAuth redirect URIs must match the environment's domain

❌ **Security downgrades**:
- Changing `https://` to `http://` in sit/uat
- Disabling CSRF protection in sit/uat
- Relaxing password policies as environments mature

❌ **Missing environment alignment**:
- The `environment` field at the top must match the folder name
- Asset hosts, API URLs should all use the same environment prefix

### 6. Validation Checklist for Reviews

When a config file is modified, verify:

- [ ] The `environment` field matches the directory name
- [ ] All URLs use the correct environment subdomain
- [ ] Protocol matches environment (HTTP in dev only, HTTPS in sit/uat)
- [ ] All `unsafe_for_prod` flags are `false` in sit/uat
- [ ] Security settings are appropriate for the environment maturity
- [ ] Placeholder values are clearly marked (e.g., "PLACEHOLDER", "changeme")
- [ ] No real production secrets are committed
- [ ] Database connection strings match the environment
- [ ] OAuth redirect URIs align with the environment domain
- [ ] Logging level is appropriate for the environment

## Best Practices

1. **Never copy-paste between environments** - Each environment has specific requirements
2. **Secrets are placeholders** - Real secrets should come from secure vaults/CI systems
3. **Progressive hardening** - Security should increase from dev → sit → uat
4. **Consistency matters** - All references to the same service should use the same naming pattern
5. **Backmerge when neccessary** - If a config change is made in one env, consider if it should be backmerged to others. If it should, do so in a separate PR automatically.
6. **Test identifiers** - This project commonly uses real-world-like identifiers as test placeholders (e.g., IBAN-like strings, test emails, account numbers). Do not raise alarms for reasonable testing values found in `dev/`, `sit/`, or PR diffs (within reason). However, if apparent production credentials or identifiers appear in `sit/` or `uat/`, or a value in a production-scoped file looks like a real, incorrect production value, flag it and suggest a safe replacement or placeholder.

## Questions?

If you're unsure whether a config change is appropriate, consider:
- Does this change make the environment less secure without justification?
- Are environment-specific values correctly scoped to their folder?
- Would this configuration be safe if it were promoted to production?

## Assistant Targets

These instructions are intended for both GitHub Copilot and Amazon Q (Code Review/PR comments). Where behavior differs, see the notes below.

## Amazon Q Notes

- Review scope: Only comment on files/lines that are part of the diff. Do not flag issues in other environment folders unless those files are modified in the PR.
- Environment inference: Infer the target environment(s) from the changed file paths, e.g., changes under `dev/`, `sit/`, `uat/`. If only `uat/` files are changed, treat the PR as UAT-scoped.
- PR intent signals: Also consider PR title/description and branch names containing `dev`, `sit`, or `uat`. Use these as hints, but the changed paths are authoritative.
- Cross-env issues: If you detect issues in a different environment not touched by the PR, do not create blocking comments. At most, leave a single optional FYI summary in a non-blocking general comment, clearly marked as out-of-scope.
- Confidence & severity: Do not suppress high-severity security misconfigurations within the changed files. Use high confidence for violations of the rules in this document. Avoid low-confidence noise on untouched files.
- Suggestion style: When flagging a mismatch (e.g., password vs connection string), include a minimal, precise fix that references the exact key path(s) and environment folder.

## Env-Aware Review Scope

When reviewing a PR, first determine the environments actually touched by the diff and apply the rules below accordingly.

- Scope to changed envs: If only `uat/` files are changed, limit comments to `uat/`. Do not block on `dev/`/`sit/` issues unless those files/lines are changed in this PR.
- Cross-file checks within env: It is appropriate to compare related values within the same environment (e.g., `uat/` password vs `uat/` connection string) and flag mismatches.
- Cross-env parity: Do not require parity across environments in the same PR unless the PR changes those environments. If parity is important, add a single non-blocking note with a concrete follow-up suggestion.
- New env folders: If a new environment directory is introduced (e.g., `qa/`), validate completeness against the schema below and the hardening progression, and provide a concise checklist of missing/incorrect keys.

### Env Change Detection Heuristics

- Primary: Paths of changed files (e.g., `uat/config.yaml`).
- Secondary: PR title/description and branch naming (e.g., `feature/uat-…`).
- Rule: If signals disagree, paths of changed files win.

### Minimal Schema Checklist (per env/config.yaml)

Require presence and correct values per environment maturity; flag missing keys or misaligned values:

- `environment`: Must equal folder name (`dev`, `sit`, `uat`).
- URLs/hosts: Env-specific domain and protocol per rules above.
- `unsafe_for_prod` block: All keys present; values match the matrix.
- Security: `csrf_protection`, `tls`, DB `sslmode`, CORS origins restricted to env domain.
- Password policy: Matches environment progression.
- Logging: Level and `enable_stack_traces` correct.
- Feature flags: Especially `payment_sandbox_mode` set per env.
- DB connection string: Matches discrete fields (host, db name `{env}_db`, user, password, ssl settings) and is consistent with any separately declared password.

### Commenting Policy Examples

- UAT-only PR with a `sit/` localhost issue unchanged: Do not comment on the `sit/` file. Optionally add a single non-blocking general FYI noting the finding is out-of-scope for this PR.
- UAT PR where `uat/` password updated but connection string not updated: Leave a blocking comment on `uat/config.yaml` with a concrete fix to update the connection string password.

### Example Fix Suggestion (mismatch within same env)

"The database connection string in `uat/config.yaml` uses the old password while `database.password` is updated. Update the connection string to use the new password to avoid connection failures."

Provide the corrected connection string segment inline if feasible.
