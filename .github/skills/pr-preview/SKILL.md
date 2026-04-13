---
name: pr-preview
description: "Generates a ready-to-paste PR description by analyzing source control changes, extracting the CR number and type (story/bug) from the branch name, and fetching the test procedure from Jira. Triggers when the user says 'done', 'pr preview', 'generate PR', 'PR description', or anything indicating they finished coding and want a PR description. Always use this skill when the user says 'done' in the context of finishing their work."
---

# PR Preview Skill

Generate a copy-paste-ready PR description from the current branch state.

## Steps

### 1. Parse the branch

Run `git branch --show-current`. Extract:
- **Type**: first segment — `story` or `bug`. If neither, ask the user.
- **CR number**: the `CR-NNNNN` pattern (e.g. `CR-34203`). If not found, ask the user.

### 2. Get the changes

Use the `get_changed_files` tool. If it returns nothing, fall back to `git status --short` in the terminal to see what changed.

### 3. Fetch from Jira

Call `mcp_jira-local_jira_getIssue` with the CR number. Extract the **description/acceptance criteria** to use as the Test Procedure. Also note the issue summary to enrich the PR description.

If Jira is unavailable, use: `**{Refer to [CR-XXXXX](https://adlm.nielseniq.com/jira/browse/CR-XXXXX) for test procedure}**`

### 4. Output the PR description

Pick the correct template below based on type. Fill in `{NUMBER}`, the description (2–4 sentences: what changed and why), and the test procedure from Jira.

**Story:**
```markdown
# Description - Story

[CR-{NUMBER}](https://adlm.nielseniq.com/jira/browse/CR-{NUMBER})

**{Description of changes}**

### Test Procedure

**{Acceptance criteria / test steps from Jira}**

### Requirements - All should be checked - Reviewer must verify or fail the PR
- [x] I have documented old and new functions that I modified and each code block (you can use Co-Pilot for that)
- [x] After I finished coding, I reviewed the story definition and verified that the requirements had been met
- [x] I made sure that my changes do not affect other code (changes in common files, etc.)
- [x] I have run both BE+FE locally/over DOD and tested major scenarios with the UI like home page, reports, filters, etc. (estimation: 30sec-1min of testing)
- [x] I have changed a flyway script and tested the change on a fresh DOD DB (if not didnt change a db script, set n/a inside the brackets)
```

**Bug:**
```markdown
# Description - Bug

[CR-{NUMBER}](https://adlm.nielseniq.com/jira/browse/CR-{NUMBER})

**{What the error was and how it was fixed}**

### Test Procedure

**{Acceptance criteria / test steps from Jira}**

### Requirements - All should be checked - Reviewer must verify or fail the PR
- [x] I have documented old and new functions that I modified and each code block (you can use Co-Pilot for that)
- [x] I have reviewed the bug description and verified that it is not reproducible anymore
- [x] I made sure that my changes do not affect other code (changes in common files, etc.)
- [x] I have run both BE+FE locally/over DOD and tested major scenarios with the UI like home page, reports, filters, etc. (estimation: 30sec-1min of testing)
- [x] I have changed a flyway script and tested the change on a DOD DB (if not didnt change a db script, set n/a inside the brackets)
```

Wrap the final PR description in a fenced code block so it's easy to copy.

### 5. Code improvements (not part of PR)

After the code block, add a short `---` separated section with any code quality observations (missing `data-testid`, hardcoded strings instead of `useCiTranslation()`, missing `@PreAuthorize`, `console.log` left in, `any` types, etc.). If nothing found, say "No issues found."
