# Playwright TypeScript Reference

Use this reference when no repository framework exists or the user asks for a new Playwright TypeScript automation package.

## Default Structure

```text
automation/
  package.json
  playwright.config.ts
  tsconfig.json
  .gitignore
  tests/
    e2e/
      login.spec.ts
    api/
      auth.api.spec.ts
  pages/
    LoginPage.ts
    DashboardPage.ts
  fixtures/
    test-data/
      users.json
    auth.fixture.ts
  support/
    env.ts
    test-data.ts
    assertions.ts
  reports/
    .gitkeep
  README.md
```

## Guidance

- Put UI/E2E specs in `tests/e2e/` and API specs in `tests/api/`.
- Put reusable page interactions in `pages/` with one class per durable page or domain surface.
- Put shared fixtures and data loaders in `fixtures/` or `support/`.
- Keep secrets out of repository files. Use `.env.example` only.
- Configure `testIdAttribute` when the target app uses `data-test-id`, `data-cy`, or another stable convention.
- Keep assertions close to expected results from TC IDs.
- Prefer targeted commands during validation.

## Command Examples

```bash
npm init playwright@latest
npm run lint
npm run typecheck
npx playwright test tests/e2e/login.spec.ts
npx playwright test --project=chromium
npx playwright show-report
```

If repo scripts exist, use them instead of raw `npx` commands.
