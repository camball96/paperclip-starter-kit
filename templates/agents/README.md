# Agent instruction templates

Paste each template into the matching agent's **Instructions** page in Paperclip, after replacing the placeholders. Keep the result short. Every line is read on every run.

| Placeholder | Example |
|---|---|
| `{{PROJECT}}` | `Acme Notes` |
| `{{ONE_LINE_DESCRIPTION}}` | `a note-taking app for small teams` |
| `{{OWNER}}` | `Alex` |
| `{{BASE_BRANCH}}` | `dev` |
| `{{TYPECHECK_COMMAND}}` | `` `npm run type-check` `` |
| `{{UNIT_TEST_COMMAND}}` | `` `npm run test:unit` `` |
| `{{E2E_COMMAND}}` | `` `npm run test:e2e` `` |
| `{{PROJECT_QA_SKILL}}` | `` `acme-qa` `` |
| `{{PROJECT_E2E_SKILL}}` | `` `acme-e2e` `` |
| `{{PROJECT_PR_SKILL}}` | `` `acme-pr` `` |
| `{{SECURITY_CHECKS}}` | The project's top security rules, in one line. For example: every query is scoped to the signed-in user; endpoints never trust a user id from the client. |
| `{{BOUNDARY_CHECKS}}` | Framework rules that compile but break at runtime. For example: server-only code is never imported into client code; values that can't be serialised never cross the server/client boundary. |

If the project has no e2e suite, drop QA's e2e sections and the "one run at a time" rules, but keep `maxConcurrentRuns: 1`.
