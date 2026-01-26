# Testing LLMs to catch erroneous config updates

We've added the following LLM agents to the repo to test it's implementation of headless-config-maps:

 - Github Copliot
 - Amazon Q Agent

## Scenarios tested

Various, real world examples were selected and simulated on this repo. The following scenarios were tested:

1. [Upgrading configs to UAT with wrong indentation](https://github.com/christoffellis/test-copilot-repo/pull/6)
2. [Incorrectly introduce SIT config to UAT](https://github.com/christoffellis/test-copilot-repo/pull/7)
3. [Spelling mistakes in variables (booleans & urls)](https://github.com/christoffellis/test-copilot-repo/pull/8)
4. [Attempting automatic backmerges](https://github.com/christoffellis/test-copilot-repo/pull/9)
5. [Added non-prod BIC code to test for correction of value](https://github.com/christoffellis/test-copilot-repo/pull/11)

These are discussed further below.

### Upgrading configs to UAT with wrong indentation

##### Target

Commit a PR with the config indentation incorrect. This will normally break a BFF, but could potentially only be picked up when BVT has commenced and the config values are actually needed. 

Ideally the agent would pick up this indentation issue and point it out.

##### Result

|  | GH Copilot | Amazon Q |
|---|---|---|
| Note | This pull request adds new service configuration URLs to the UAT environment configuration file. The change introduces a newConfigs section under the features block with URLs for feature toggle, monitoring, and error tracking services. | YAML Syntax Error: Inconsistent indentation will cause YAML parsing to fail. The newConfigs section uses inconsistent spacing compared to other feature flags. |
| Target passed | ❌ | ✅ |
| Side Effects | GHC Identified issues with the naming convention instead of the target issue trying to be illustrated. | In addition to the syntax error, Q did identify the varying casings used in the new addition |


### Incorrectly introduce SIT config to UAT

##### Target

Configs were updated from SIT to UAT. In this migration, the actual env of the target config was not updated, eg:
`https://sit.website.com` vs `https://uat.website.com`.

##### Result

|  | GH Copilot | Amazon Q |
|---|---|---|
| Note | The CORS allowed origins list includes a SIT environment domain in the UAT configuration. CORS origins should only allow environment-specific domains to maintain proper environment isolation. The https://sit-admin.example.local domain should be removed from the UAT configuration, as only UAT domains like https://uat.example.local or https://uat-admin.example.local should be permitted. | Security Configuration Error: Adding a SIT environment URL to UAT CORS configuration creates cross-environment access that violates security boundaries. This allows the SIT admin interface to make requests to UAT services, which could lead to data leakage or unauthorized access between environments. |
| Target passed | ✅ | ✅ |
| Side Effects | None | Q interpreted it as an intentional link between SIT and UAT, instead of a human mistake, but did identify the critical vulnerability in this case |

### Spelling mistakes in variables (booleans & urls)

##### Target

Two variables being upgraded were intentionally misspelled. Twi examples were made, one with a boolean and one with a url, where the environment in the URL was misspelled, not matching previous env convention.

##### Result

|  | GH Copilot | Amazon Q |
|---|---|---|
| Note | The value trues is not a valid boolean. This should be true (without the 's'). This syntax error will cause the configuration to fail parsing. | Configuration Error: Invalid boolean value trues will cause runtime configuration parsing errors. Boolean values must be true or false. |
|  | The URL subdomain should be dev to match the development environment, but it is set to devs (with an extra 's'). This inconsistency could cause the feature toggle service to fail connection. Change to https://dev-feature-toggle.example.local. | Inconsistent naming convention in service URLs. Line 84 uses devs- prefix while lines 85-86 use dev- prefix. This inconsistency could cause service discovery issues if the actual endpoints follow different naming patterns. |
| Target passed | ✅ | ✅ |

### Attempting automatic backmerges

##### Target

A new variable was added to the UAT environment (to simulate a hotfix). This variable should ideally exist on DEV and SIT to ensure future code is aligned to the relevant change. 

##### Result

|  | GH Copilot | Amazon Q |
|---|---|---|
| Note | The ALLOW_WEAK_PASSWORDS field is being added only to the UAT environment's unsafe_for_prod section. However, this field is not present in the dev or sit config files, creating a schema inconsistency across environments. For maintainability and consistency, this field should be added to all three environments with appropriate values: true in dev (where weak passwords are acceptable for testing), false in sit, and false in uat (production-like environments). This aligns with the progressive security hardening pattern documented in the guidelines. | Security Vulnerability: Hardcoded database credentials are exposed in the configuration file. This creates a significant security risk as credentials are stored in plaintext and committed to version control |
| Target passed | ✅ | ❌ |
Side Effects | Along with the message generated by GHC, a button was presented that automatically generated a PR to add this config to DEV and SIT | None |

### Added non-prod BIC code to test for correction of value

##### Target

Throughout our testing envs, the BIC code SBZAZAJ0 was used. When updating to prod, the agent should identify that this is a testing value, and instead suggest the real BIC, SBZAZAJJ.

##### Result

|  | GH Copilot | Amazon Q |
|---|---|---|
| Note | The BIC code 'SBZAZAJ0' appears to be a test identifier used across all environments (dev, sit, uat). For production, this should be replaced with the actual production BIC code for the payment system. Using the same test BIC in production as in lower environments could lead to payment processing issues or failures. | Security Vulnerability: Hardcoded bank BIC code in production configuration creates security and compliance risks. Bank identifiers should be externalized to environment variables or secure configuration management systems to prevent exposure in version control and enable proper rotation |
| Target passed | ⚠️ | ❌ |
Side Effects | GHC did identify that the value being promoted to prod was indeed a testing value, but did not provide a specific value to be updated to. This could be due to it believing the real bic may be a security vulnerability, or it doesnt have the context of SBG. | Q frequently dislikes the project structure, and will need to be configured to accept that this is an org repo. |