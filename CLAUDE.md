# library-template

A .NET library template (`netstandard2.0`) that publishes a NuGet package to GitHub Packages on every merge to `master`.

## Versioning

The published package version is **not** hardcoded in the `.csproj`. It comes from a GitHub Actions **repository variable** named `VERSION` (Settings → Secrets and variables → Actions → Variables). The publish workflow (`.github/workflows/publish.yaml`) reads `${{ vars.VERSION }}`, publishes with `-p:Version`, then increments the patch and writes it back via `gh variable set`. So each merge to `master` publishes the next version (`x.y.z` → `x.y.(z+1)`), gap-free, with no commit-back to the repo.

- **To set or jump the version** (e.g. a minor bump): edit the `VERSION` variable in repo settings to `1.1.0`.
- The patch auto-increments on its own; you only touch the variable to change major/minor.

## One-time setup for a new repo created from this template

GitHub does **not** copy repository variables or secrets when you use this template, so each generated repo needs:

1. Create the version variable (must exist before the first workflow run):
   ```bash
   gh variable set VERSION --body "1.0.0" --repo <OWNER>/<REPO>
   ```
2. Add a `GH_PAT` repository secret with **two** capabilities:
   - package push (classic `write:packages`), **and**
   - **variable-write** (classic `repo`, or fine-grained *Variables: Read and write*) — required because the built-in `GITHUB_TOKEN` cannot modify Actions variables.
3. Set your own `PackageId` in `LibraryTemplate/LibraryTemplate.csproj`.
