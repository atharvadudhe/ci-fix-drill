# CI Pipeline Failure Diagnosis

## Failure 1 — Test job cannot find package.json

**Step:** `test → Run tests`

**Exact error from GitHub Actions:**

```text
npm error enoent Could not read package.json: Error: ENOENT: no such file or directory, open '/home/runner/work/ci-fix-drill/ci-fix-drill/package.json'
```

**Classification:** Type 3 — Configuration

**Root cause:** The `test` job runs on a fresh GitHub Actions runner but does not contain an `actions/checkout` step. Because the repository is not checked out in that job, `package.json` and the source files are unavailable when `npm test` runs.

**Fix:** Add the repository checkout step to the test job and configure the job so dependencies are installed before tests run.

---

## Failure 2 — package.json and package-lock.json are out of sync

**Step:** Local dependency installation using `npm ci`

**Exact error:**

```text
npm error `npm ci` can only install packages when your package.json and package-lock.json or npm-shrinkwrap.json are in sync. Please update your lock file with `npm install` before continuing.
```

The log also reported:

```text
npm error Missing: lodash@4.18.1 from lock file
```

**Classification:** Type 2 — Dependency

**Root cause:** The committed `package-lock.json` is not synchronized with the dependency information in `package.json`. `npm install` can reconcile dependency information, but `npm ci` requires an existing lockfile that exactly represents the dependency tree.

**Fix:** Regenerate the lockfile using `npm install`, commit the updated `package-lock.json`, and use `npm ci` in the CI workflow for reproducible installations.

---

## Failure 3 — Incorrect unit-test assertions

**Step:** Jest unit tests

Two incorrect assertions were found after inspecting the test files.

### calculateDiscount

The test contained:

```js
expect(calculateDiscount(100, 10)).toBe(100);
```

The implementation correctly calculates a 10% discount:

```text
100 - (100 × 10 / 100) = 90
```

Therefore, the expected value in the test was incorrect.

**Classification:** Type 1 — Assertion

**Fix:** Change the expected value from `100` to `90`.

### formatCurrency

The test contained:

```js
expect(formatCurrency(10.005, 'USD')).toBe({
  amount: 10.01,
  currency: 'USD'
});
```

The function returns an object with the expected properties. Jest's `toBe` matcher checks object identity rather than structural equality, so it is the wrong matcher for this assertion.

**Classification:** Type 1 — Assertion

**Fix:** Replace `toBe` with `toEqual` so Jest compares the contents of the returned object.
