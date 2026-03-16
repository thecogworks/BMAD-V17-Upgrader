# Upgrade Gate: v15 → v16

**Date:** 2026-02-27
**Branch baseline:** v15 complete (0 errors, 103 tests passing)
**Result:** ✅ PASS — 0 build errors, 0 CS warnings, 103 tests passing

---

## Package Changes

| Package | v15 Version | v16 Version |
|---|---|---|
| Umbraco.Cms (all sub-packages) | 15.4.4 | 16.5.0 |
| Umbraco.Forms / .Core | 15.2.0 | 16.4.3 |
| Umbraco.StorageProviders.AzureBlob | 15.0.0 | 16.0.0 |
| Umbraco.StorageProviders.AzureBlob.ImageSharp | 15.0.0 | 16.0.0 |
| Skybrud.Umbraco.Redirects | 15.0.1 | 16.0.3 |
| uSync.Complete | 15.1.9 | 16.1.2 |
| uSync.Forms | 15.0.1 | 16.0.2 |
| Umbraco.Community.Contentment | — (removed at v14) | **6.1.1** (RE-ADDED) |

## Framework

All 12 projects remain on **net9.0** (unchanged — v17 is the net10.0 jump).

---

## Code Changes

### 1. IndexInitializeComposer.cs — IComponent → IAsyncComponent
**Issue:** `IComponent` was removed in v16. The component must implement `IAsyncComponent`.
**File:** `TSW.Search/Composers/IndexInitializeComposer.cs`
**Fix:** Changed interface from `IComponent` to `IAsyncComponent` and updated method signatures:
```csharp
// Before (v15)
public class IndexInitializeComponent : IComponent
{
    public void Initialize() { ... }
    public void Terminate() { }
}

// After (v16)
public class IndexInitializeComponent : IAsyncComponent
{
    public Task InitializeAsync(bool forceReinitialization, CancellationToken cancellationToken) { ... }
    public Task TerminateAsync(bool forceTermination, CancellationToken cancellationToken) => Task.CompletedTask;
}
```

### 2. Contentment re-added
**Package:** `Umbraco.Community.Contentment 6.1.1`
**Added to:** `TSW.Web/TSW.Web.csproj`
**Note:** 6.x is the correct major version for Umbraco 16 (net9.0). The package ships a `net9.0` build for Umbraco 16 and a `net10.0` build for Umbraco 17+. No code changes required — Contentment 6.x is data-compatible with prior content.

---

## Deprecation Status After v16 Upgrade

After upgrading to v16 packages, the `IPublishedContent.Parent` / `.Children` and related `PublishedContentWrapped.Parent/.Children` APIs that were marked "Scheduled for removal in V16" in v15 packages are **no longer flagged as obsolete** in v16 packages. They remain available without warnings. The removal schedule shifted; per the v16 package messages these are now targeted for v17/v18.

**Zero `CS0618` warnings remain.** Only `NU1608` NuGet constraint warnings from `Cogworks.AzureSearch.IoC.Umbraco` pulling in a transitive `Umbraco.Cms.Web.Website 13.3.2` reference — harmless, pre-existing, does not affect compilation or runtime.

---

## Remaining Deprecation Debt (address at v17)

| API | Current Deprecation Message | Action Required |
|---|---|---|
| `IPublishedCache.GetAtRoot()` | "Scheduled for removal in v17" | Use `IDocumentNavigationQueryService` |
| `BlockListItem.ContentUdi` | "Use ContentKey instead. Will be removed in V18" | Use `ContentKey` |
| `ObjectExtensions.SafeCast<T>` | "Will be removed in Umbraco 17" | Use direct casting |
| `HealthCheck.GetStatus()` | "Use GetStatusAsync. Will be removed in v17" | Implement `GetStatusAsync` |
| `PublishedContentExtensions.Children<T>(IVariationContextAccessor)` | "Use overload with INavigationQueryService. Scheduled for removal in v17" | Update call site |
| `BreadcrumbQueryHandler.cs: current.Parent` | Uses deprecated `Parent` property (v15 comment) | Use `IDocumentNavigationQueryService.TryGetParentKey` |

---

## Build Summary

```
dotnet build TSW.sln — 0 Error(s), 0 CS Warning(s), 4 NU1608 (harmless)
dotnet test TSW.sln  — Failed: 0, Passed: 103, Skipped: 0
```

---

## Next Step: v16 → v17
