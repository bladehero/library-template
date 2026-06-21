# library-template

A .NET library template (`netstandard2.0`) that publishes a NuGet package on every merge to `master`. By default it publishes to **GitHub Packages**; it can additionally publish to **nuget.org** via Trusted Publishing (opt-in, see below).

## Versioning

The published package version is **not** hardcoded in the `.csproj`. It comes from a GitHub Actions **repository variable** named `VERSION` (Settings → Secrets and variables → Actions → Variables). The publish workflow (`.github/workflows/publish.yaml`) reads `${{ vars.VERSION }}`, publishes with `-p:Version`, then increments the patch and writes it back via `gh variable set`. So each merge to `master` publishes the next version (`x.y.z` → `x.y.(z+1)`), gap-free, with no commit-back to the repo.

- **To set or jump the version** (e.g. a minor bump): edit the `VERSION` variable in repo settings to `1.1.0`.
- The patch auto-increments on its own; you only touch the variable to change major/minor.

## Publishing destinations

- **GitHub Packages** — always on for generated repos. Uses the `GH_PAT` secret. (The template repo itself never publishes, guarded by `if: github.repository != 'bladehero/library-template'`.)
- **nuget.org** — opt-in. The nuget.org steps run only when the repo sets a `NUGET_USER` variable. They use **Trusted Publishing (OIDC)** via `NuGet/login@v1`, which exchanges the workflow's short-lived GitHub OIDC token for a temporary nuget.org key — **no long-lived API-key secret is stored**. This is nuget.org's recommended approach over API keys.

## Naming the package

- Replace `LibraryTemplate` everywhere with your library name, and keep the **PackageId, assembly name, root namespace, project file + directory, and solution** all consistent — otherwise the published package is misleading (e.g. a `Foo.Bar` package whose DLL/namespace is still `LibraryTemplate`).
- **Do not use a reserved ID prefix.** nuget.org reserves `Microsoft.`, `System.`, `NuGet.`, etc.; pushing a new package under one returns **HTTP 403**. Use a prefix you own (e.g. `Bladehero.`) and consider reserving it on nuget.org after the first publish.

## One-time setup for a new repo created from this template

GitHub does **not** copy repository variables or secrets when you use this template, so each generated repo needs:

1. Create the version variable (must exist before the first workflow run):
   ```bash
   gh variable set VERSION --body "1.0.0" --repo <OWNER>/<REPO>
   ```
2. Add a `GH_PAT` repository secret with **two** capabilities (for GitHub Packages push + the version bump):
   - package push (classic `write:packages`), **and**
   - **variable-write** (classic `repo`, or fine-grained *Variables: Read and write*) — required because the built-in `GITHUB_TOKEN` cannot modify Actions variables.
3. Set your `PackageId` (and the matching assembly name + root namespace) in `LibraryTemplate/LibraryTemplate.csproj`, replacing `LibraryTemplate`.
4. **(Optional) Publish to nuget.org too** — using Trusted Publishing, no API key:
   - On nuget.org → your username → **Trusted Publishing** → add a policy bound to this repo's owner, repository name, and workflow `publish.yaml`.
   - Set the `NUGET_USER` variable to your nuget.org account username:
     ```bash
     gh variable set NUGET_USER --body "<nuget-username>" --repo <OWNER>/<REPO>
     ```
   - If `NUGET_USER` is unset, the nuget.org steps are skipped and only GitHub Packages publishing runs.
