# test-copilot-repo

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

## Questions?

If you're unsure whether a config change is appropriate, consider:
- Does this change make the environment less secure without justification?
- Are environment-specific values correctly scoped to their folder?
- Would this configuration be safe if it were promoted to production?
