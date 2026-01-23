# .github/upgrades/plan.md

## Table of contents

- Executive Summary
- Migration Strategy
- Detailed Dependency Analysis
- Project-by-Project Plans
- Package Update Reference
- Breaking Changes Catalog
- Testing & Validation Strategy
- Risk Management
- Complexity & Effort Assessment
- Source Control Strategy
- Success Criteria

---

## Executive Summary

### Selected Strategy
**All-At-Once Strategy** - All projects upgraded simultaneously in a single coordinated operation.

Rationale:
- Solution size: 1 project (small)
- Project SDK-style and homogeneous (ASP.NET Core)
- Assessment indicates available package updates and no blocking security vulnerabilities

Scope:
- Repository root: `D:\Azure\DotNetCoreApplication-master\DotNetCoreApplication-master\DotNetCoreApplication`
- Solution: `DotNetCoreApplication.sln`
- Projects to upgrade (atomic scope):
  - `DotNetCoreApplication\DotNetCoreApplication.csproj`

Key findings from assessment:
- Current target framework: `netcoreapp3.1` (all projects)
- Proposed target framework: `net10.0`
- NuGet package updates required: `Microsoft.EntityFrameworkCore.SqlServer` (3.1.8 ? 10.0.2), `Microsoft.EntityFrameworkCore.Tools` (3.1.8 ? 10.0.2), `Microsoft.VisualStudio.Web.CodeGeneration.Design` (3.1.3 ? 10.0.2)
- Deprecated package: `Microsoft.EntityFrameworkCore.SqlServer.Design` (remove/replace)
- API issues reported: usage of `IHostingEnvironment` (source-incompatible), one behavioral change in `UseExceptionHandler` overload

Deliverable:
- `plan.md` (this document) specifying atomic upgrade steps, package matrix, breaking-changes catalog, testing and rollback guidance.

Decision note:
- Target framework confirmed by stakeholder: `net10.0`. Plan locked to `.NET 10.0 (LTS)` as requested.

### Assessment snapshot

Highlevel Metrics (from assessment):

| Metric | Count | Status |
| :--- | :---: | :--- |
| Total Projects | 1 | All require upgrade |
| Total NuGet Packages | 5 | 4 need upgrade |
| Total Code Files | 21 |  |
| Total Code Files with Incidents | 3 |  |
| Total Lines of Code | 788 |  |
| Total Number of Issues | 11 |  |
| Estimated LOC to modify | 6+ | at least 0.8% of codebase |

Package compatibility (summary):

| Status | Count | Percentage |
| :--- | :---: | :---: |
| ? Compatible | 1 | 20.0% |
| ?? Incompatible | 1 | 20.0% |
| ?? Upgrade Recommended | 3 | 60.0% |
| **Total NuGet Packages** | **5** | **100%** |

Top API issues (summary):

| Category | Count | Impact |
| :--- | :---: | :--- |
| ?? Binary Incompatible | 0 | High - Require code changes |
| ?? Source Incompatible | 5 | Medium - Needs re-compilation and potential conflicting API error fixing |
| ?? Behavioral change | 1 | Low - Behavioral changes that may require testing at runtime |
| ? Compatible | 1664 |  |
| **Total APIs Analyzed** | **1670** |  |


## Migration Strategy

Chosen approach: **All-At-Once (atomic) upgrade**.

Justification:
- Single, small solution (1 project) — low coordination overhead
- SDK-style project: project file edits are straightforward
- No security vulnerabilities flagged by assessment; package updates are available for target framework

Key principles for execution (for the executor):
- Perform all project TargetFramework updates and all package reference updates in a single commit/operation
- Restore and build the entire solution to reveal compilation errors caused by the framework and package changes
- Fix compilation errors and re-build until the solution builds with 0 errors
- Run automated tests after the atomic upgrade completes

Phases (informational; the atomic upgrade remains a single operation):
- Phase 0 — Preparation: validate SDK, update `global.json` if present, ensure branch `upgrade-to-NET10` is checked out and clean
- Phase 1 — Atomic Upgrade (single coordinated pass): update TargetFramework and package versions across all projects; restore; build; fix compilation errors
- Phase 2 — Test Validation: run unit/integration tests and address failures

## Detailed Dependency Analysis

Summary:
- Total projects: 1
- No project-to-project dependencies (leaf and root are the same project)
- No circular dependencies

Implication for All-At-Once strategy:
- With only one project and no inter-project dependencies, the atomic upgrade scope is the single project file and its package references.

Critical files to check for imported MSBuild logic:
- `Directory.Build.props` / `Directory.Build.targets` / `Directory.Packages.props` (search repository root for these files) — package versions or target frameworks may be defined there and must be updated in the same atomic pass if present.

Project list (atomic scope):
- `DotNetCoreApplication\DotNetCoreApplication.csproj` (current: `netcoreapp3.1`, target: `net10.0`)

## Project-by-Project Plans

### Project: `DotNetCoreApplication\DotNetCoreApplication.csproj`

Current state:
- TargetFramework: `netcoreapp3.1`
- SDK style: Yes
- Project type: ASP.NET Core application
- Key issues from assessment: API source-incompatibility (`IHostingEnvironment`), behavioral change for `UseExceptionHandler`, recommended NuGet package updates, deprecated package present

Target state:
- TargetFramework: `net10.0`
- All package references updated to versions compatible with net10.0 (see Package Update Reference)

Migration Steps (to be executed atomically):
1. Preparation (TASK-000 - prerequisites):
   - Validate .NET 10 SDK is installed on build machines and CI (verify `dotnet --list-sdks` or use `upgrade_validate_dotnet_sdk_installation` step)
   - If `global.json` exists, update SDK version entry to the required SDK (or ensure compatibility)
   - Ensure branch `upgrade-to-NET10` is created from `Dev/migration9` and checked out (no pending changes)
2. Update project file:
   - Change `<TargetFramework>netcoreapp3.1</TargetFramework>` to `<TargetFramework>net10.0</TargetFramework>` in `DotNetCoreApplication.csproj` (or append `net10.0` if multi-targeting is intentionally required)
   - Check for imported MSBuild files that define TargetFrameworks and update them accordingly
3. Update NuGet package references (all updates in the same pass):
   - `Microsoft.EntityFrameworkCore.SqlServer` 3.1.8 ? 10.0.2
   - `Microsoft.EntityFrameworkCore.Tools` 3.1.8 ? 10.0.2
   - `Microsoft.VisualStudio.Web.CodeGeneration.Design` 3.1.3 ? 10.0.2
   - Remove deprecated `Microsoft.EntityFrameworkCore.SqlServer.Design` (no replacement) — validate code does not rely on it
   - Leave `Dapper` as-is (compatible)
4. Restore and build the solution to discover compilation errors
5. Fix compilation errors caused by framework and package upgrades (see Breaking Changes Catalog)
6. Rebuild and verify solution builds with 0 errors
7. Execute all test projects (if present) and resolve test failures
8. Final verification: confirm no security vulnerabilities remain, run static analysis if available

Validation checkpoints:
- Solution builds with 0 compilation errors
- All tests pass
- No package dependency conflicts
- No remaining deprecated packages

#### Risk & Complexity

- Risk level: Medium (EF Core upgrades + source-incompatible API usages)
- Complexity: Low (small codebase, limited number of files to change)
- Areas requiring attention:
  - Environment API replacements (`IHostingEnvironment` ? `IHostEnvironment`)
  - EF Core migrations and query behavior
  - Scaffolding/tooling package updates

#### Files likely to change

- `Program.cs` — potential minor updates if moving to top-level statements or changed host building patterns (optional)
- `Startup.cs` — replace `IHostingEnvironment` references, verify middleware registration (`UseExceptionHandler`) and configuration
- Any DbContext or EF-related classes — update EF Core APIs and ensure migrations remain valid

#### Program.cs / Startup.cs specific guidance

- The repository currently uses the generic host with `CreateHostBuilder(...).UseStartup<Startup>()`. This pattern remains supported on .NET 10.0; migrating to the minimal hosting model is optional.
- `Startup.Configure` currently accepts `IWebHostEnvironment env`. `IWebHostEnvironment` is compatible with newer runtimes, but the assessment flagged legacy `IHostingEnvironment` usages elsewhere. Recommended actions:
  - Search and replace any `IHostingEnvironment` usages with `IHostEnvironment`.
  - `IWebHostEnvironment` may be left as-is; consider migrating to `IHostEnvironment` for consistency with `Microsoft.Extensions.Hosting` APIs.
- Verify `UseExceptionHandler` behavior after upgrade and add/adjust integration tests to cover error-handling paths.

Code notes (no automatic changes in plan):
- Example replacement (optional):

```csharp
// Before (existing code uses IWebHostEnvironment)
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment()) { ... }
}

// Alternative (using IHostEnvironment)
public void Configure(IApplicationBuilder app, IHostEnvironment env)
{
    if (env.IsDevelopment()) { ... }
}
```

[Details to be filled by executor during implementation: exact code changes discovered from compiler errors]

## Package Update Reference

Common package updates (affecting `DotNetCoreApplication.csproj`):

| Package | Current | Target | Notes |
|---|---:|---:|---|
| `Microsoft.EntityFrameworkCore.SqlServer` | 3.1.8 | 10.0.2 | EF Core must match runtime target; update required |
| `Microsoft.EntityFrameworkCore.Tools` | 3.1.8 | 10.0.2 | Update tools package to align with EF runtime |
| `Microsoft.VisualStudio.Web.CodeGeneration.Design` | 3.1.3 | 10.0.2 | Scaffolding/tooling package upgrade |
| `Microsoft.EntityFrameworkCore.SqlServer.Design` | 1.1.6 | (deprecated) | Remove — package deprecated for modern EF Core versions |
| `Dapper` | 2.0.35 | (compatible) | No change required per assessment |

Notes:
- Apply these package version changes in the same atomic update. If versions are centrally managed (e.g., `Directory.Packages.props`) update that file instead of individual project files.
- For each package update, consult package release notes for breaking changes during implementation.

## Breaking Changes Catalog

Top expected breaking changes and guidance:

1) `IHostingEnvironment` ? `IHostEnvironment`
   - Assessment identified usages of `Microsoft.AspNetCore.Hosting.IHostingEnvironment`, which is removed/renamed in newer ASP.NET Core versions.
   - Mitigation: Replace `IHostingEnvironment` references with `IHostEnvironment` and update using statements if necessary. Typical change sites: `Startup`, controllers, services using environment checks.

2) `UseExceptionHandler` behavioral change
   - One behavioral change reported related to `UseExceptionHandler` overload semantics. Verify middleware invocation and error handling behavior after upgrade.
   - Mitigation: Run integration tests for error handling scenarios and review middleware configuration in `Startup`.

3) EF Core API changes (3.1 ? 10.x)
   - EF Core 10 may introduce API surface changes and behavior differences. Review usages of low-level APIs, migration code, and any custom conventions.
   - Mitigation: Update any obsolete EF APIs, regenerate migrations if necessary, and run data-related integration tests.

4) Deprecated package removal
   - `Microsoft.EntityFrameworkCore.SqlServer.Design` should be removed. Confirm no direct compile-time references; replace code that relied on tool-specific types or move logic to supported packages.

General discovery approach:
- Compilation will reveal source-incompatible changes. Use compiler errors to guide code modifications.
- Use `upgrade_get_member_info` or code search to find all usages of affected types and update them consistently.

Concrete code change examples (common fixes):

- Replace `IHostingEnvironment` usages:

```csharp
// Before
public class Startup
{
    public Startup(IHostingEnvironment env) { }
}

// After
public class Startup
{
    public Startup(IHostEnvironment env) { }
}
```

- Replace `env.EnvironmentName` checks if API surface changed (verify exact member names remain)

- Exception handling middleware: ensure `UseExceptionHandler` configuration still matches expected behavior. Example:

```csharp
// Verify existing pattern
app.UseExceptionHandler("/Home/Error");
```

If behavior differs after upgrade, prefer explicit error-handling middleware or use `UseExceptionHandler(options => { ... })` overload.

## Testing & Validation Strategy

Testing levels and checkpoints:

- Unit tests: run all unit test projects after atomic upgrade. Expect to fix API breakages first.
- Integration tests: run integration tests that exercise middleware, EF Core interactions, and error handling (especially `UseExceptionHandler` scenarios).
- Build verification: solution must build with 0 errors and should aim for 0 warnings where feasible.

Automated validation steps (executor):
1. `dotnet restore`
2. `dotnet build --no-restore` (validate 0 errors)
3. `dotnet test` for all discovered test projects

If tests fail, fix issues caused by API changes and re-run tests until green.

Testing checklist (to mark during execution):
- [ ] `dotnet restore` completes successfully
- [ ] `dotnet build` completes with 0 errors
- [ ] `dotnet test` all tests pass
- [ ] No deprecated packages remain referenced
- [ ] Manual verification of error-handling scenarios (if no automated test exists)
- [ ] EF Core migrations apply successfully in a safe environment

## Risk Management

Risk summary:

- Medium risk: package upgrades for EF Core and scaffolding packages; API source-incompatibilities present but limited in count
- Low risk: single-project solution, small LOC impact

Mitigations:
- Run the atomic upgrade in an isolated branch `upgrade-to-NET10`
- Ensure CI has .NET 10 SDK available before merging
- Prefer single atomic commit to simplify rollback (revert the commit on failure)
- Add targeted test coverage for behavioral change areas (error handling, environment-based logic, EF interactions)

Rollback strategy:
- If the atomic upgrade introduces irrecoverable regressions, revert the single upgrade commit and restore prior branch state. Preserve a branch with the failing upgrade for investigation.

## Complexity & Effort Assessment

Relative complexity (per project):
- `DotNetCoreApplication.csproj` — Low
  - LOC impact: ~6+ lines
  - Risk: Medium (EF Core package updates)

Overall plan complexity: Low (single project, limited API changes)

## Source Control Strategy

All-At-Once source control guidance:
- Create and switch to branch: `upgrade-to-NET10` (already proposed)
- Keep workspace clean (no pending changes) before starting
- Apply all changes in a single atomic commit (project file TargetFramework change + package updates + code fixes)
- Push branch and open a single PR for review. PR checklist should include:
  - Build passes in CI with .NET 10 SDK
  - All automated tests pass
  - No deprecated packages remain

Note: If repository uses protected branches, ensure PR has required reviewers and CI gates configured.

## Success Criteria

The migration is complete when all of the following are true:
1. `DotNetCoreApplication.csproj` targets `net10.0` (or equivalent `net10.0` TFM used across project files)
2. All package updates from the Package Update Reference are applied (exact versions specified)
3. Solution builds with 0 compilation errors
4. All tests pass
5. No deprecated NuGet packages remain referenced
6. No outstanding security vulnerabilities reported for referenced packages

Additional acceptance notes:
- Ensure CI pipeline has been updated to use .NET 10 SDK and all build agents are compatible.
- Document any behavior changes observed (especially around exception handling) in PR description for reviewers.

--

[End of plan initial version]
