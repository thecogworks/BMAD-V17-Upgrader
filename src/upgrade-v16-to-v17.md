# Upgrade Gate: v16 → v17

**Date:** 2026-02-27
**Branch baseline:** v16 complete (0 errors, 103 tests passing)
**Result:** ✅ PASS — 0 build errors, 0 CS warnings (CS errors), 103 tests passing

---

## Package Changes

| Package | v16 Version | v17 Version |
|---|---|---|
| Umbraco.Cms (all sub-packages) | 16.5.0 | 17.2.1 |
| Umbraco.Forms / .Core | 16.4.3 | 17.1.3 |
| Umbraco.StorageProviders.AzureBlob | 16.0.0 | 17.0.0 |
| Umbraco.StorageProviders.AzureBlob.ImageSharp | 16.0.0 | 17.0.0 |
| Skybrud.Umbraco.Redirects | 16.0.3 | 17.0.2 |
| uSync.Complete | 16.1.2 | 17.1.1 |
| uSync.Forms | 16.0.2 | 17.0.0 |
| Umbraco.Community.Contentment | 6.1.1 | 6.1.1 (unchanged — ships net10.0 TFM for v17) |
| Microsoft.Extensions.Caching.SqlServer | 9.0.13 | 10.0.3 |
| Microsoft.Extensions.Caching.StackExchangeRedis | 9.0.13 | 10.0.3 |
| System.Runtime.Caching | 9.0.0 | 10.0.3 |
| Microsoft.Extensions.Http | 9.0.3 | 10.0.3 |

## Framework

All 12 projects: **net9.0 → net10.0**

---

## Code Changes

### 1. UmbracoContentNodeService.cs — Remove IVariationContextAccessor, fix GetAtRoot()

**Issues:**
- `IPublishedCache.GetAtRoot()` removed in v17 — use `IDocumentNavigationQueryService.TryGetRootKeys()`
- All `Children<T>(variationContextAccessor)`, `FirstChild<T>(variationContextAccessor)`, `Descendants<T>(variationContextAccessor)`, `DescendantsOrSelf(variationContextAccessor)`, `Children(variationContextAccessor, filter)` overloads removed in v17
- `IVariationContextAccessor` no longer needed as constructor dependency

**File:** `TSW.Core/Services/UmbracoContentNodeService.cs`

**Constructor change:**
```csharp
// Before (v16)
public class UmbracoContentNodeService(
    IUmbracoContextFactory umbracoContextFactory,
    IVariationContextAccessor variationContextAccessor) : IUmbracoContentNodeService

// After (v17)
public class UmbracoContentNodeService(
    IUmbracoContextFactory umbracoContextFactory,
    IDocumentNavigationQueryService documentNavigationQueryService) : IUmbracoContentNodeService
```

**GetRoot() change:**
```csharp
// Before (v16)
return content.GetAtRoot();

// After (v17)
if (!documentNavigationQueryService.TryGetRootKeys(out var rootKeys))
{
    return [];
}
return rootKeys.Select(key => content.GetById(key)).Where(x => x is not null)!;
```

**All variationContextAccessor overloads replaced with no-arg forms:**
- `FirstChild<T>(variationContextAccessor)` → `FirstChild<T>()`
- `Children<T>(variationContextAccessor)` → `Children<T>()`
- `Descendants<T>(variationContextAccessor)` → `Descendants<T>()`
- `DescendantsOrSelf(variationContextAccessor)` → `DescendantsOrSelf()`
- `Children(variationContextAccessor, m => filter)` → `Children().Where(m => filter)`

**GlobalUsings.cs addition:**
```csharp
global using Umbraco.Cms.Core.Services.Navigation;
```

### 2. RobotsCheck.cs — GetStatus() → GetStatusAsync()

**Issue:** `HealthCheck.GetStatus()` removed in v17.
**File:** `TSW.Core/Services/HealthChecks/RobotsCheck.cs`

```csharp
// Before (v16)
public override async Task<IEnumerable<HealthCheckStatus>> GetStatus() =>

// After (v17)
public override async Task<IEnumerable<HealthCheckStatus>> GetStatusAsync() =>
```

**Test file updated:** `TSW.Tests.Core/HealthChecks/RobotsCheckTests.cs` — all 5 `await handler.GetStatus()` calls → `await handler.GetStatusAsync()`

### 3. PromoBlockViewModelBuilder.cs + SearchDocumentsBuilder.cs — SafeCast<T>() removed

**Issue:** `ObjectExtensions.SafeCast<T>()` removed in v17 — use direct cast.
**Files:**
- `TSW.Core/Builders/ViewModelBuilders/PromoBlockViewModelBuilder.cs`
- `TSW.Search/Builders/SearchDocumentsBuilder.cs`

```csharp
// Before (v16)
?.GetValue()?.SafeCast<MediaWithCrops>()
?.GetValue()?.SafeCast<MediaWithCrops>()?.Id

// After (v17)
?.GetValue() as MediaWithCrops
(?.GetValue() as MediaWithCrops)?.Id
```

---

## Build Summary

```
dotnet build TSW.sln — 0 Error(s), 88 CS Warning(s) (all harmless — see below), 0 MSB errors
dotnet test TSW.sln  — Failed: 0, Passed: 103, Skipped: 0
```

Note: First `dotnet build` attempt (after package bump, before `dotnet clean`) produced 6 MSB3030 "file not found" errors for old AngularJS AzureIndexer files in the obj cache. `dotnet clean TSW.sln` cleared stale artifacts; subsequent build produced 0 errors.

---

## Deprecation Status After v17 Upgrade

The 88 CS0618 warnings fall into three categories, **all targeted for v18 or later**:

| Warning | Message | Removal Target |
|---|---|---|
| `IPublishedContent.Parent` property | "Scheduled for removal in Umbraco 18" | v18 |
| `IPublishedContent.Children` property | "Scheduled for removal in Umbraco 18" | v18 |
| `PublishedContentWrapped.Parent` | "Scheduled for removal in V16" (stale message, still works) | v18 |
| `PublishedContentWrapped.Children` | "Scheduled for removal in V16" (stale message, still works) | v18 |
| `BlockListItem.ContentUdi` | "Use ContentKey instead. Will be removed in V18" | v18 |
| `ForwardedHeadersOptions.KnownNetworks` | ASP.NET ASPDEPR005 — use `KnownIPNetworks` | N/A |

Plus harmless CS8632 (nullable annotation context), CS9113 (unused parameter), CS0114 (hide inherited member) in test helpers.

---

## Remaining Deprecation Debt (address at v18 if upgrading further)

| API | Current Deprecation Message | Action Required |
|---|---|---|
| `IPublishedContent.Parent` property | "Scheduled for removal in Umbraco 18" | Use `IDocumentNavigationQueryService.TryGetParentKey` |
| `IPublishedContent.Children` property | "Scheduled for removal in Umbraco 18" | Use `IDocumentNavigationQueryService.TryGetChildrenKeys` |
| `BlockListItem.ContentUdi` | "Use ContentKey instead. Will be removed in V18" | Use `ContentKey` |
| `ForwardedHeadersOptions.KnownNetworks` | ASPDEPR005 | Use `KnownIPNetworks` |

---

## Next Step: Application boot on v17

Run the application once to verify runtime correctness (migrations, startup, backoffice). This is the final validation gate.
