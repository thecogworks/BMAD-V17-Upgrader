# Upgrade Gate: v14 → v15

**Date:** 2026-02-27
**Branch baseline:** v14 complete (0 errors, 103 tests passing)
**Result:** ✅ PASS — 0 build errors, 103 tests passing

---

## Package Changes

| Package | v14 Version | v15 Version |
|---|---|---|
| Umbraco.Cms (all sub-packages) | 14.3.4 | 15.4.4 |
| Umbraco.Forms | 14.2.3 | 15.2.0 |
| Umbraco.Forms.Core | 14.2.3 | 15.2.0 |
| Umbraco.StorageProviders.AzureBlob | 14.1.0 | 15.0.0 |
| Umbraco.StorageProviders.AzureBlob.ImageSharp | 14.1.0 | 15.0.0 |
| Skybrud.Umbraco.Redirects | 14.0.0 | 15.0.1 |
| uSync.Complete | 14.2.0 | 15.1.9 |
| uSync.Forms | — (removed in v14) | 15.0.1 (re-added) |
| Microsoft.Extensions.Caching.SqlServer | 8.0.22 | 9.0.13 |
| Microsoft.Extensions.Caching.StackExchangeRedis | 8.0.22 | 9.0.13 |
| System.Runtime.Caching | 8.0.1 | 9.0.0 |
| Microsoft.Extensions.Http | 8.0.1 | 9.0.3 |

## Framework Change

All 12 projects: `net8.0` → `net9.0`

---

## Code Changes

### 1. Generated ModelsBuilder files (196 files)
**Issue:** `IPublishedSnapshotAccessor` was removed from Umbraco v15.
**Fix:** Bulk-replaced `IPublishedSnapshotAccessor publishedSnapshotAccessor` → `IPublishedContentTypeCache contentTypeCache` in all 196 generated `.cs` files under `TSW.Common/Models/CmsModels/Generated/`. The `using Umbraco.Cms.Core.PublishedCache` is retained (it contains `IPublishedContentTypeCache`).

### 2. Search Controllers (5 files)
**Issue:** `UmbracoApiController` was removed in v15.
**Files:**
- `TSW.Search/Controllers/ArticlesController.cs`
- `TSW.Search/Controllers/ContentPagesController.cs`
- `TSW.Search/Controllers/GlobalSearchController.cs`
- `TSW.Search/Controllers/OfficesController.cs`
- `TSW.Search/Controllers/PeopleController.cs`

**Fix:** Changed base class from `UmbracoApiController` to `ControllerBase` and added `[ApiController]` attribute.

### 3. TabNavigationHandler.cs
**Issue:** `IContentService.SaveAndPublish()` was removed in v15.
**File:** `TSW.Core/Notifications/TabNavigationHandler.cs`
**Fix:** Replaced `contentService.SaveAndPublish(page)` with:
```csharp
contentService.Save(page);
contentService.Publish(page, Array.Empty<string>());
```

### 4. BreadcrumbQueryHandler.cs
**Issue:** `AncestorsOrSelf()` was rewritten in v15 to use `IDocumentNavigationQueryService` (database-backed tree) and no longer falls back to `IPublishedContent.Parent`. Unit tests use mocked `Parent` chains that the new implementation ignores.
**File:** `TSW.Core/Features/Components/Breadcrumb/BreadcrumbQueryHandler.cs`
**Fix:** Replaced `request.CurrentPage.AncestorsOrSelf()` with explicit `Parent` traversal loop. `IPublishedContent.Parent` is deprecated in v15 (scheduled removal in v16) but still functional.
**Note:** At v16 upgrade, this must be replaced with `IDocumentNavigationQueryService.TryGetAncestorsOrSelfKeys`.

### 5. TestFixtureUmbraco.cs
**Issue:** v15's `FriendlyPublishedContentExtensions` static constructor and `AncestorsOrSelf` resolve additional services from `StaticServiceProvider` via `GetRequiredService`, causing `TypeInitializationException` / `InvalidOperationException` in tests.
**File:** `TSW.Tests.Common/TestFixtureUmbraco.cs`
**Fix:** Added mocks for v15 navigation services:
- `IDomainCache` (Umbraco.Cms.Core.PublishedCache) — resolved in static ctor
- `IPublishedContentCache` (Umbraco.Cms.Core.PublishedCache) — resolved in static ctor
- `IDocumentNavigationQueryService` (Umbraco.Cms.Core.Services.Navigation) — resolved in static ctor
- `IMediaNavigationQueryService` (Umbraco.Cms.Core.Services.Navigation) — resolved in static ctor
- `IPublishedContentStatusFilteringService` (Umbraco.Cms.Core.Services.Navigation) — resolved at runtime

---

## Deprecation Warnings to Address at v16

The following patterns produce `CS0618` warnings in v15 and must be updated at v16:

| File | Warning | Required Action |
|---|---|---|
| `TSW.Core/Features/Components/Breadcrumb/BreadcrumbQueryHandler.cs` | `Parent` obsolete | Use `IDocumentNavigationQueryService.TryGetParentKey` |
| `TSW.Core/Features/Components/RelatedContent/RelatedContentQueryHandler.cs` | `Parent` obsolete | Use `IDocumentNavigationQueryService.TryGetParentKey` |
| `TSW.Core/Features/Pages/InsightListingPages/InsightListingPageQueryHandler.cs` | `Children` obsolete | Use `IDocumentNavigationQueryService.TryGetChildrenKeys` |
| `TSW.Core/Services/UmbracoContentNodeService.cs` | `Children<T>()` obsolete | Use overload with `INavigationQueryService` |
| `TSW.Tests.Common/TestFixtureModel.cs` | `Parent`, `Children` on models obsolete | Use navigation service in test setup |

---

## Build Summary

```
dotnet build TSW.sln — 0 Error(s), 8 Warning(s)
dotnet test TSW.sln  — Failed: 0, Passed: 103, Skipped: 0
```

---

## Next Step: v15 → v16
