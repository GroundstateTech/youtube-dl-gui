# Groundstate fork validation and manifest publishing

This fork's default branch is `master`. Vue and Rust checks target that branch; they do not require a `main` or `release` branch. Rust's frontend preparation uses Node 24, matching the Vue build and the package's TypeScript manifest scripts.

## Manifest workflow

**Build manifest and deploy pages** is manual. It checks out the branch selected in GitHub's workflow-run control for both the generator and documentation. It no longer runs a failing nightly job against absent branch names.

The default run only generates a manifest. It does not sign, upload an artifact, or deploy Pages, and its generated files disappear with the runner. To retain files locally, run `npm ci` and `npm run manifest:gen` in a checkout and keep `docs/manifest/manifest.json`. Generation downloads public tool metadata/binaries to calculate checksums, so it requires network access.

Only explicitly selecting **publish_pages** enables signing, signature verification, Pages artifact upload, and the separate deployment job. Publishing requires available GitHub artifact quota, correctly configured signing keys, and Pages setup; it is not a storage-quota workaround. This change does not run that path or change repository settings.

## Local checks

From the repository root:

```sh
npm ci
npm run lint
npm run test:unit
npx playwright install --with-deps chromium
npm run test:e2e
npm run build
```

Rust checks, from `src-tauri/`, require Cargo and the platform's Tauri development libraries:

```sh
cargo fmt --all -- --check
cargo clippy --all-targets -- -D warnings
cargo test
```

The CI frontend checks and native Rust checks are distinct; a passing frontend build does not prove a native installer works. Consult the workflow logs and verify a native release candidate before distribution.

## Dependency advisory status (2026-10-19)

The dependency refresh resolves the Vue/server-renderer and Vitest advisories and compatible transitive updates. Vitest and its coverage provider are pinned together. The build job audits production dependencies at high severity before producing the frontend bundle.

The full npm audit still reports four high-severity entries in one development-tool chain: `@vue/eslint-config-typescript` → `fast-glob` → `micromatch` → `braces`. These are affected-package counts, not four independent vulnerabilities. The published `braces` version remains affected, and npm's proposed automatic fix downgrades the ESLint configuration outside its declared range. No forced downgrade or unverified override is applied. Track this as unresolved development-tool risk and audit again before release; a passing production-only audit does not clear it.

Maintainer advisories: [Vitest](https://github.com/vitest-dev/vitest/security/advisories/GHSA-82fw-gwwq-j7x9), [Vue renderer](https://github.com/vuejs/core/security/advisories/GHSA-g2v6-rqmx-r4w6), and [braces](https://github.com/advisories/GHSA-vfj7-8cjw-p6xm).
