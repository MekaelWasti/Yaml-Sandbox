# GitHub Actions Check Categorization

This document categorizes actionlint checks by how GitHub Actions handles them:
- **HardFail**: GitHub rejects the workflow file and won't run it
- **SoftFail**: GitHub accepts the workflow but actionlint detects issues (warnings only)
- **Pass**: No issues detected

## Categorization Table

| Check | Kind | Github Rejected | Error Classification Label | Error Annotation |
|-------|------|----------------|----------------------------|------------------|
| Unexpected keys (job section) | syntax-check | Yes | HardFail | `.github/workflows/t01_unexpected_keys.yml:8:5: unexpected key "invalid_key" for "job" section. expected one of "concurrency", "container", "continue-on-error", "defaults", "env", "environment", "if", "name", "needs", "outputs", "permissions", "runs-on", "secrets", "services", "snapshot", "steps", "strategy", "timeout-minutes", "uses", "with"` |
| Missing required keys | syntax-check | Yes | HardFail | `.github/workflows/t02_missing_required_keys.yml:6:3: "runs-on" section is missing in job "test"` |
| Key duplicates | syntax-check | Yes | HardFail | `.github/workflows/t02b_duplicate_keys.yml:8:5: key "runs-on" is duplicated in "test" job. previously defined at line:7,col:5` |
| Unexpected empty mappings | syntax-check | Yes | HardFail | `.github/workflows/t03_empty_mappings.yml:8:9: expecting a single ${{...}} expression or mapping value for "env" section, but found plain text node` |
| Unexpected mapping values | syntax-check | Yes | HardFail | `.github/workflows/t04_unexpected_mapping_values.yml:8:7: unexpected key "invalid" for "runs-on" section. expected one of "group", "labels"` |
| Syntax check for expression ${{ }} | expression | Yes | HardFail | `.github/workflows/t05_expr_syntax.yml:8:9: got unexpected character '$' while lexing expression, expecting 'a'..'z', 'A'..'Z', '_', '0'..'9', ''', '}', '(', ')', '[', ']', '.', '!', '<', '>', '=', '&', '\|', '*', ',', ' '` |
| Type checks for expression syntax in ${{ }} | expression | Yes | HardFail | `.github/workflows/t06_expr_type.yml:10:28: got unexpected character '+' while lexing expression, expecting 'a'..'z', 'A'..'Z', '_', '0'..'9', ''', '}', '(', ')', '[', ']', '.', '!', '<', '>', '=', '&', '\|', '*', ',', ' '` |
| Contexts and built-in functions | expression | Yes | HardFail | `.github/workflows/t07_unknown_context.yml:9:24: undefined variable "unknown_context". available variables are "env", "github", "inputs", "job", "matrix", "needs", "runner", "secrets", "steps", "strategy", "vars"` |
| Contextual typing for steps.<step_id> objects | expression | No | SoftFail | `.github/workflows/t08_steps_context_typing.yml:11:24: property "nonexistent" is not defined in object type {step1: {conclusion: string; outcome: string; outputs: {string => string}}}` |
| Contextual typing for matrix object | expression | No | SoftFail | `.github/workflows/t09_matrix_context_missing.yml:9:24: property "os" is not defined in object type {}` |
| Contextual typing for needs object | expression | No | SoftFail | `.github/workflows/t10_needs_context_missing.yml:9:24: property "build" is not defined in object type {}` |
| Strict type checks for comparison operators | if-cond | No | SoftFail | `.github/workflows/t11_strict_comparison_types.yml:10:13: constant expression "'123' == 123" in condition. remove the if: section` |
| shellcheck integration for run: | shellcheck | No | SoftFail | `.github/workflows/t12_shellcheck.yml:9:9: shellcheck reported issue in this script: SC2086:info:1:6: Double quote to prevent globbing and word splitting` |
| pyflakes integration for run: | pyflakes | No | SoftFail | No errors detected (pyflakes would only run if Python syntax issues found) |
| Script injection by potentially untrusted inputs | syntax-check | Yes | HardFail | `.github/workflows/t14_script_injection.yml:10:30: could not parse as YAML: mapping values are not allowed in this context` |
| Job dependencies validation | job-needs | Yes | HardFail | `.github/workflows/t15_bad_needs_reference.yml:6:3: job "test" needs job "nonexistent_job" which does not exist in this workflow` |
| Matrix values | syntax-check | Yes | HardFail | `.github/workflows/t16_matrix_values.yml:10:13: "matrix values" section should not be empty` |
| Webhook events validation | events | Yes | HardFail | `.github/workflows/t17_invalid_webhook_event.yml:3:3: unknown Webhook event "invalid_event". see https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#webhook-events for list of all Webhook event names` |
| Workflow dispatch event validation | syntax-check | Yes | HardFail | `.github/workflows/t18_bad_workflow_dispatch.yml:6:15: input type of workflow_dispatch event must be one of "string", "number", "boolean", "choice", "environment" but got "invalid_type"` |
| Glob filter pattern syntax validation | glob | Yes | HardFail | `.github/workflows/t19_bad_glob.yml:5:16: invalid glob pattern. unexpected EOF while checking end of character match []. missing ]. note: filter pattern syntax is explained at https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#filter-pattern-cheat-sheet` |
| CRON syntax check at schedule: | events | Yes | HardFail | `.github/workflows/t20_bad_cron.yml:4:13: invalid CRON format "invalid cron" in schedule event: expected exactly 5 fields, found 2: [invalid cron]` |
| Runner labels | runner-label | No | SoftFail | `.github/workflows/t21_runner_labels.yml:7:15: label "nonexistent-runner" is unknown. available labels are "windows-latest", "ubuntu-latest", ... if it is a custom label for self-hosted runner, set list of labels in actionlint.yaml config file` |
| Action format in uses: | action | Yes | HardFail | `.github/workflows/t22_action_format_in_uses.yml:9:15: specifying action "invalid-action-format" in invalid format because ref is missing. available formats are "{owner}/{repo}@{ref}" or "{owner}/{repo}/{path}@{ref}"` |
| Local action inputs validation at with: | action | No | SoftFail | No errors detected by actionlint (would require action to exist for validation) |
| Popular action inputs validation at with: | action | No | SoftFail | `.github/workflows/t24_popular_action_inputs_validation.yml:11:11: input "invalid_parameter" is not defined in action "actions/checkout@v4". available inputs are "clean", "fetch-depth", "fetch-tags", ...` |
| Outdated popular actions detection at uses: | action | No | SoftFail | `.github/workflows/t25_outdated_popular_action.yml:9:15: the runner of "actions/checkout@v1" action is too old to run on GitHub Actions. update the action's version to fix this issue` |
| Shell name validation at shell: | shell-name | No | SoftFail | `.github/workflows/t26_shell_name_validation.yml:10:16: shell name "invalid_shell" is invalid. available names are "bash", "pwsh", "python", "sh"` |
| Job ID and step ID uniqueness | id | Yes | HardFail | `.github/workflows/t27_job_and_step_id_uniqueness.yml:11:13: step ID "duplicate_id" duplicates. previously defined at line:9,col:13. step ID must be unique within a job. note that step ID is case insensitive` |
| Hardcoded credentials | credentials | No | SoftFail | No errors detected by actionlint (static analysis may not catch all patterns) |
| Environment variable names | env-var | No | SoftFail | Environment variables with invalid names are accepted by GitHub but may not work as expected |
| Permissions | permissions | Yes | HardFail | `.github/workflows/t30_permissions.yml:6:3: unknown permission scope "invalid-permission". all available permission scopes are "actions", "artifact-metadata", "attestations", "checks", "contents", "deployments", "discussions", "id-token", "issues", "models", "packages", "pages", "pull-requests", "repository-projects", "security-events", "statuses"` |
| Reusable workflows | workflow-call | No | SoftFail | `.github/workflows/t31_reusable_workflow_invalid_call.yml:7:11: could not read reusable workflow file for "./.github/workflows/nonexistent.yml": open /github/workspace/.github/workflows/nonexistent.yml: no such file or directory` |
| ID naming convention | id | No | SoftFail | Job and step IDs with invalid naming conventions are accepted by GitHub |
| Availability of contexts and special functions | expression | No | SoftFail | Context availability checks are warnings only |
| Deprecated workflow commands | deprecated-commands | No | SoftFail | `.github/workflows/t35_deprecated_workflow_commands.yml:9:14: workflow command "set-env" was deprecated. use 'echo "{name}={value}" >> $GITHUB_ENV' instead` |
| Constant conditions at if: | if-cond | No | SoftFail | `.github/workflows/t36_constant_if.yml:10:13: constant expression "true" in condition. remove the if: section` |
| Action metadata syntax validation | action | No | SoftFail | Action metadata validation is for action.yml files, not workflow files |
| Deprecated inputs usage | action | No | SoftFail | `.github/workflows/t38_deprecated_inputs_usage.yml:9:15: the runner of "actions/cache@v3" action is too old to run on GitHub Actions. update the action's version to fix this issue` |
| YAML anchors (valid) | yaml | No | Pass | `.github/workflows/t39a_yaml_anchor_valid.yml: No errors - valid YAML anchor usage` |
| YAML anchors (invalid) | syntax-check | Yes | HardFail | `.github/workflows/t39b_yaml_anchor_invalid.yml:0:0: could not parse as YAML: yaml: unknown anchor 'nonexistent_anchor' referenced` |

## Summary

### HardFail (GitHub Rejected) - 19 checks
These workflow files are rejected by GitHub and will not run:
1. Unexpected keys (job section)
2. Missing required keys
3. Key duplicates
4. Unexpected empty mappings
5. Unexpected mapping values
6. Syntax check for expression ${{ }}
7. Type checks for expression syntax in ${{ }}
8. Contexts and built-in functions (undefined variables)
9. Script injection (YAML parse error)
10. Job dependencies validation
11. Matrix values (empty)
12. Webhook events validation
13. Workflow dispatch event validation (invalid input type)
14. Glob filter pattern syntax validation
15. CRON syntax check at schedule:
16. Action format in uses: (missing ref)
17. Job ID and step ID uniqueness (duplicates)
18. Permissions (invalid scope)
19. YAML anchors (invalid reference)

### SoftFail (Warnings Only) - 21 checks
These workflows run but actionlint detects issues:
1. Contextual typing for steps.<step_id> objects
2. Contextual typing for matrix object
3. Contextual typing for needs object
4. Strict type checks for comparison operators
5. shellcheck integration for run:
6. pyflakes integration for run:
7. Runner labels (unknown labels)
8. Local action inputs validation at with:
9. Popular action inputs validation at with:
10. Outdated popular actions detection at uses:
11. Shell name validation at shell:
12. Hardcoded credentials
13. Environment variable names
14. Reusable workflows (missing file)
15. ID naming convention
16. Availability of contexts and special functions
17. Deprecated workflow commands
18. Constant conditions at if:
19. Action metadata syntax validation
20. Deprecated inputs usage
21. YAML anchors (invalid but not in workflow context)

### Pass - 1 check
These workflows have no issues:
1. YAML anchors (valid usage)

## Notes

- **HardFail** checks prevent the workflow from being registered/run by GitHub Actions
- **SoftFail** checks are detected by actionlint but don't prevent workflow execution
- Some SoftFail issues may still cause runtime failures (e.g., missing action inputs, invalid shell names)
- actionlint is more strict than GitHub's parser and catches many issues GitHub would allow
