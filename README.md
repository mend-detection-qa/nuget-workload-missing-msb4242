# nuget-workload-missing-msb4242

## Purpose

Reproduction probe for **TKA-10392** — `.NET 10 MAUI Android` project where the
required workload version `10.0.101.1` is not installed on the scan runner.

Previously, the Mend scanner attempted `dotnet restore` before scanning, which
triggered `MSB4242` (missing workload) and caused the entire scan to fail with no
dependency tree output. TKA-10049 introduced a fix; this probe validates it holds
for workload version `10.0.101.1`.

## Failure mode (pre-fix)

```
error MSB4242: The SDK 'Microsoft.NET.Sdk.Maui' specified could not be found.
Waiting for this workload to be installed: microsoft-maui (version 10.0.101.1)
```

The build fails → no `project.assets.json` → scanner returns empty dep tree.

## Expected scanner behaviour (post-fix)

| Behaviour | Expected |
|---|---|
| `dotnet restore` is NOT called | ✅ (`nuget.runPreStep=false`) |
| Scanner parses `.csproj` XML directly | ✅ (fallback path from TKA-10049) |
| MSB4242 does NOT appear in scan logs | ✅ |
| Dep tree contains ≥4 direct packages | ✅ |
| `Microsoft.Maui.Controls` is detected | ✅ |
| Scan status is `RESOLVED` | ✅ |

## Configuration

- `nuget.runPreStep=false` — no `dotnet restore`, avoids MSB4242 entirely
- `nuget.resolveSystemPackages=true` — ensures `Microsoft.*` workload packages are visible
- `.whitesource` → `dotnet: ">=10 <11"` — forces .NET 10 SDK selection

## Customer context

- **Customer:** Fidelity National Financial / ServiceLink  
- **Product:** `ServiceLink/Auction//AuctionMobileApp`  
- **Platform:** Mend Dev Platform for AzDo  
- **Environment:** Mend SaaS US  
- **Related tickets:** TKA-10392 (customer), TKA-10049 (.NET 10 support fix)
