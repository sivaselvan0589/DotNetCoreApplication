# DotNetCoreApplication .NET 10 Upgrade Tasks

## Overview

This document tracks the atomic upgrade of `DotNetCoreApplication` from `netcoreapp3.1` to `net10.0` using a single coordinated update of project files and package references, followed by testing and validation. Tasks follow the plan's All-At-Once strategy and are executable in sequence.

**Progress**: 3/3 tasks complete (100%) ![0%](https://progress-bar.xyz/100)

---

## Tasks

### [✓] TASK-001: Verify prerequisites *(Completed: 2026-01-23 05:19)*
**References**: Plan §Phase 0, Plan §Detailed Dependency Analysis

- [✓] (1) Verify required .NET 10 SDK is installed on local/CI machines per Plan §Phase 0 (e.g., `dotnet --list-sdks`)
- [✓] (2) Runtime/SDK version meets minimum requirements (**Verify**)
- [✓] (3) If `global.json` exists, update its SDK version entry to the required SDK per Plan §Phase 0
- [✓] (4) Check configuration/import files (`Directory.Build.props`, `Directory.Packages.props`, `Directory.Build.targets`) for centrally-managed TargetFramework or package versions and ensure compatibility or update as required (**Verify**)

### [✓] TASK-002: Atomic framework and package upgrade with compilation fixes *(Completed: 2026-01-23 05:21)*
**References**: Plan §Project-by-Project Plans, Plan §Package Update Reference, Plan §Breaking Changes Catalog, Plan §Source Control Strategy

- [✓] (1) Update `<TargetFramework>` to `net10.0` in `DotNetCoreApplication\DotNetCoreApplication.csproj` per Plan §Project-by-Project Plans
- [✓] (2) Update NuGet package references per Plan §Package Update Reference (notably `Microsoft.EntityFrameworkCore.SqlServer`, `Microsoft.EntityFrameworkCore.Tools`, `Microsoft.VisualStudio.Web.CodeGeneration.Design` → target versions; remove deprecated `Microsoft.EntityFrameworkCore.SqlServer.Design`) and update central package management files if present
- [✓] (3) Restore dependencies (`dotnet restore`) for the solution per Plan §Testing & Validation Strategy
- [✓] (4) Build solution and fix all compilation errors caused by framework/package upgrades (address `IHostingEnvironment` → `IHostEnvironment`, `UseExceptionHandler` behavioral changes, EF Core API changes) per Plan §Breaking Changes Catalog
- [✓] (5) Solution builds with 0 errors (**Verify**)
- [✓] (6) Commit changes with message: "TASK-002: Atomic upgrade to net10.0 (project file and package updates + compilation fixes)"

### [✓] TASK-003: Run full test suite and validate upgrade *(Completed: 2026-01-23 05:22)*
**References**: Plan §Testing & Validation Strategy, Plan §Breaking Changes Catalog

- [✓] (1) Run tests for all test projects per Plan §Testing & Validation Strategy (`dotnet test`)
- [✓] (2) Fix any test failures caused by API or behavioral changes (reference Plan §Breaking Changes Catalog)
- [✓] (3) Re-run tests after fixes
- [✓] (4) All tests pass with 0 failures (**Verify**)






