# ArchBIM — Master Project Memory

> This file is read automatically by Claude Code at the start of every session.
> Keep it up to date as the project evolves.
> **Master location: C:\Users\kapil\.claude\CLAUDE.md** (global, private — not accessible to Codex)
> Backup: C:\victus\Website\CLAUDE.md (committed to GitHub repo)

---

## 1. Who / What

**Owner:** Sanjiv Kapila
**Brand:** ArchBIM
**Domain:** archbim.co.in (registered on GoDaddy)
**Website:** https://archbim.co.in
**GitHub:** https://github.com/SanjivKK/archbim (username: SanjivKK)
**Vendor ID (Revit):** ARCHBIM
**Add-In GUID (Survey Coordinates):** 8A6A0D5E-0D52-4F4A-8C7C-FA8C0F6AE001

---

## 2. Folder Map

```
C:\victus\
├── CLAUDE.md                          ← STUB ONLY (master moved to C:\Users\kapil\.claude\CLAUDE.md)
├── Website\                           ← Git repo → GitHub Pages → archbim.co.in
│   ├── index.html                     ← Homepage (lists all add-ins)
│   ├── survey-coordinates\
│   │   └── index.html                 ← Survey Coordinates product page
│   └── CNAME                          ← archbim.co.in (GitHub Pages custom domain)
└── RevitAPI\
    └── 2026\
        ├── shared\
        │   └── skRevitAddins_sharedParameterFile.txt  ← master shared param file
        └── RvtAddins\                 ← Git repo root
            ├── RvtAddins.sln
            ├── PHE\                   ← Survey Coordinates add-in
            │   ├── Class1.cs          ← IExternalApplication, ribbon setup + trial init
            │   ├── PHELocations.cs    ← "Update Parameter Values" command
            │   ├── PHEUpdateLocations.cs ← "Update Element Locations" command
            │   ├── TrialManager.cs    ← 30-day trial (HMAC, Windows SID, HKCU registry)
            │   ├── PHE.csproj
            │   ├── icon\phe32.png
            │   └── Installer\
            │       ├── PHE.addin      ← Revit manifest
            │       ├── ArchBIM_SurveyCoordinates_SharedParams.txt ← bundled copy
            │       ├── SurveyCoordinates_Setup.iss  ← Inno Setup script
            │       └── output\        ← compiled .exe goes here
            ├── DimData\
            ├── DisDim\
            ├── MaterialsByRoom\
            ├── NWC\
            ├── SeparationLines\
            └── WallSleeves\
```

---

## 3. Survey Coordinates Add-in (PHE)

### Purpose
Reads and writes real-world survey coordinates (Northing / Easting) on Revit
family instances. Works on two categories: **Site** and **Planting**.

### Ribbon
- Panel: **Survey Coordinates**
- Button: **Northings and Eastings** (PulldownButton)
  - Dropdown 1: **Update Parameter Values** → `PHE.PHELocations`
  - Dropdown 2: **Update Element Locations** → `PHE.PHEUpdateLocations`

### Trial — TrialManager.cs
- 30-day free trial, tied to machine (not email)
- Storage: `HKCU\SOFTWARE\ArchBIM\SurveyCoordinates` — keys `TS` (start date ISO-8601), `TV` (HMAC)
- Machine fingerprint: Windows SID via `WindowsIdentity.GetCurrent().User.Value`
- HMAC-SHA256 with embedded byte key — date tampering or cross-machine copy → HMAC mismatch → Expired
- `InitIfNeeded()` called in `Class1.OnStartup()`; ribbon LongDescription shows days remaining
- Both commands check `TrialManager.Check()` at top of `Execute()` and block if Expired
- Price: **USD 12** one-time via LemonSqueezy

### PHELocations.cs — "Update Parameter Values"
- Collects all Site + Planting `FamilyInstance` elements
- For each: reads `LocationPoint`, applies survey-point offset (`ProjectPosition`),
  converts to project display units, writes to "Northing" / "Easting" shared params
- Hard-stops with a named exception if any element is missing the shared params
- Preflight `TaskDialog` reminds user to:
  - Assign correct categories (Site / Planting)
  - Add Northing + Easting shared params to BOTH categories

### PHEUpdateLocations.cs — "Update Element Locations"
- Collects same elements
- For each: reads stored Northing/Easting params, converts back to internal feet,
  reverses survey-point offset, moves element
- **Drift check:** skips elements whose current position already matches stored
  params within 1 mm tolerance (prevents mass-moving unedited elements)
- Calls `doc.Regenerate()` inside transaction + `uidoc.RefreshActiveView()` after
- Selects all moved elements after commit so they are visible/highlighted
- `TaskDialog` preflight warns about family geometry offset from origin
- Summary dialog: moved / unchanged / skipped counts

### Build
```
cd C:\victus\RevitAPI\2026\RvtAddins
dotnet build PHE/PHE.csproj --configuration Release
```
> Must build from RvtAddins\ root — hint paths for RevitAPI.dll are relative
> and only resolve from that level (not from worktrees).

### Deploy (manual)
```
copy PHE\bin\Release\net8.0\PHE.dll  C:\ProgramData\Autodesk\Revit\Addins\2025\
```

### Shared Parameter File
- Source: `C:\victus\RevitAPI\2026\shared\skRevitAddins_sharedParameterFile.txt`
- Bundled copy in installer: `PHE\Installer\ArchBIM_SurveyCoordinates_SharedParams.txt`
- Installed to: `C:\ProgramData\ArchBIM\Survey Coordinates\Resources\`
- Key parameters (Group: "PHE Equipment"):
  - **Easting**  — GUID `a22c4239-3337-4222-9075-21d22cc3f7b1` — DATATYPE: NUMBER
  - **Northing** — GUID `32cb1bac-1901-49ee-85ca-0f321f7d3495` — DATATYPE: NUMBER

### Installer
- Script: `PHE\Installer\SurveyCoordinates_Setup.iss`
- Tool: Inno Setup 6 (https://jrsoftware.org/isinfo.php) — NOT yet installed
- Auto-detects Revit 2024 / 2025 / 2026 via registry
- Installs: PHE.dll + PHE.addin (per Revit version) + shared param file (once)
- Output: `PHE\Installer\output\SurveyCoordinates_Setup_v1.0.0.exe`

---

## 4. Website — C:\victus\Website\

### Current state
- `index.html` — Homepage: hero, add-ins grid (Survey Coordinates $12 + free trial), about, CTA, footer
- `survey-coordinates/index.html` — Full product landing page: 2-tier pricing (free trial + $12 full)
- `CNAME` — contains `archbim.co.in` for GitHub Pages custom domain
- Pure HTML + CSS (no build tools)
- **Pushed** to https://github.com/SanjivKK/archbim (main branch)
- GitHub Pages enabled; custom domain `archbim.co.in` set; DNS pending (GoDaddy WHOIS verification issue — support ticket open, ~24h resolution)

### Hosting & domain
- **GitHub Pages** — push repo to GitHub, enable Pages on `main` branch
- **Domain:** `archbim.co.in` — registered on GoDaddy
- GoDaddy DNS records needed (see Section 9)

### Payments
- **LemonSqueezy** for payment + license key delivery
  - Acts as Merchant of Record (handles global VAT automatically)
  - Built-in license key generation per purchase
  - Update checkout URL in `survey-coordinates/index.html` once LemonSqueezy product is created
  - Webhook → notify backend on purchase (future)

---

## 5. Licensing Strategy (planned, not yet implemented)

1. User buys on website → LemonSqueezy delivers a license key by email
2. On first Revit launch, add-in calls `POST /licenses/activate { key, machineId }`
3. Backend validates key, stores activation, returns signed token
4. Token cached locally; re-validated every 7 days
5. Machine fingerprint: SHA-256 of `MachineName + WindowsSID`
6. Admin dashboard shows: activations per key, last seen, Revit version, country

---

## 6. BIM Insights Platform (roadmap)

A broader web-based BIM intelligence platform. Document:
`C:\victus\NewFolder\viewer\ARCHBIM_Insights_AI_APS_OpenAI_Brief.docx`

### Architecture
```
React + TypeScript (APS Viewer + extensions)
        ↕ REST / WebSocket
ASP.NET Core 8 Minimal API
  ├── APS OAuth + OSS + Model Derivative
  ├── Rule Engine (deterministic C# rules)
  └── OpenAI (server-side only, gpt-4o / gpt-4o-mini)
        ↕
PostgreSQL (Projects, Models, Elements, RuleResults, AiAuditLogs)
        ↑
Revit Add-in (.NET 8) — exports structured data, pulls issues
```

### Phases
| Phase | Goal | Status |
|-------|------|--------|
| 1 | APS auth + file upload + Viewer working | Not started |
| 2 | Revit add-in data bridge | Not started |
| 3 | Deterministic rule engine + viewer heatmaps | Not started |
| 4 | OpenAI layer (NL queries, explain, summarize) | Not started |
| 5 | Multi-tenant SaaS + subscriptions (Stripe) | Not started |

### Key tech decisions
- APS tokens: server-side cache only, never sent to frontend
- AI model: gpt-4o for explain/summarize, gpt-4o-mini for classify/interpret
- Rules: code-based C# classes (deterministic, not AI)
- Multi-tenancy: row-level (TenantId column), PostgreSQL RLS
- Auth: Auth0
- Payments (platform): Stripe
- Hosting: Azure App Service + Azure Static Web Apps
- CI/CD: GitHub Actions

---

## 7. SheetTransfer Add-in

### Purpose
Copies sheets from one open Revit project into the active document, including:
- Titleblock detection, copy from source, and resolution in target
- Sheet parameter transfer (all writable parameters)
- Titleblock instance parameter transfer
- Level token detection in sheet numbers/names with mapping UI

### Add-In GUID
`2F8D4B7A-E3C1-4A96-B5D8-F7E2A9C0D4B3`

### Price
**USD 18** one-time via LemonSqueezy
Checkout: https://archbim.lemonsqueezy.com/checkout/buy/c1aa68f7-155a-41fc-89ed-d1fe5c1e8f0b

### Ribbon
- Panel: **Sheet Transfer**
- Button: **Transfer Sheets** → `SheetTransfer.SheetTransferCommand`

### Trial
- Registry: `HKCU\SOFTWARE\ArchBIM\SheetTransfer` — keys `TS`, `TV`
- HMAC key phrase: `ARCHBIM_ST_2026_KEY_V1_X`

### Key Files
```
SheetTransfer\
  App.cs                         ← IExternalApplication, ribbon setup + trial init
  SheetTransferCommand.cs        ← IExternalCommand, full Phase 1 logic
  TrialManager.cs                ← 30-day trial (same pattern as PHE, different registry key)
  SourceSelectionWindow.xaml/.cs ← WPF: pick source document from open docs
  LevelMappingWindow.xaml/.cs    ← WPF: map source level tokens → target levels + preview
  Installer\SheetTransfer.addin  ← Revit manifest
```

### Build
```
cd C:\victus\RevitAPI\2026\RvtAddins
dotnet build SheetTransfer/SheetTransfer.csproj --configuration Release
```

### Phases
| Phase | Scope | Status |
|-------|-------|--------|
| 1 | Sheet transfer + titleblock resolution + level mapping | ✅ Built |
| 2 | Conflict handling, sheet number auto-rename, detailed report | Not started |
| 3 | Legends, drafting views, schedules | Not started |
| 4 | Plan/section/elevation recreation, viewport placement | Not started |

### Roadmap doc
`C:\victus\NewFolder\viewer\revit_sheet_transfer_addin_roadmap_with_titleblock_strategy.docx`

---

## 8. WallSleeves Add-in

### Purpose
Detects intersections between MEP services (ducts, pipes, cable trays) in a
**linked** MEP model and walls in the **host** architectural/structural model,
then places a `Wall_Sleeve` face-hosted Generic Model family instance at each
intersection and writes clearance/metadata parameters.

### Add-In GUID
`2F8D4B7A-E3C1-4A96-B5D8-F7E2A9C0D4B8` ← placeholder; update when finalised

### Price
TBD (likely USD 18–25 one-time via LemonSqueezy)

### Ribbon
- Panel: **Sleeve Cuts**
- Button: **MEP Sleeves** → `WallSleeves.SleeveCuts` (wo.png)

> **Note:** "Load Wall Sleeve Family" button was removed (2026-03-08).
> The MEP Sleeves command now auto-loads `Wall_Sleeve.rfa` from the assembly folder
> if the family is not already loaded in the project.

### Current Codebase State (as of 2026-03-10)

| File | Status | Notes |
|------|--------|-------|
| `Class1.cs` | ✅ Complete | Single "MEP Sleeves" button (wo.png); TrialManager.InitIfNeeded(); trial days in LongDescription |
| `SleeveCuts.cs` | ✅ Complete | Full core algorithm; auto-loads family; Mark=D-ME-001/P-HW-002/CT-001; TaskDialog CommandLinks; zoom+select; Z-offset fix (void centred on MEP centreline) |
| `LinkedProcessor.cs` | ✅ Complete | Lists all loaded RevitLinkInstances; auto-picks if single; shows picker dialog if multiple |
| `LinkedModelPickerWindow.xaml/.cs` | ✅ Complete | WPF dialog listing all loaded links; double-click or OK to select |
| `LoadFamily.cs` | ⚠️ Kept but not in ribbon | Auto-loads from assembly dir; not wired to ribbon button (removed 2026-03-08) |
| `TagFormat.cs` | ❌ Empty stub | Not needed for Phase 1 |
| `Clearances.xaml/.cs` | ✅ Complete | Labels, Scope radio buttons, Cancel, public properties, validation |
| `ProgressWindow.xaml/.cs` | ✅ Complete | WPF progress bar, Dispatcher yield pattern |
| `TrialManager.cs` | ✅ Complete | 30-day trial, HKCU\SOFTWARE\ArchBIM\WallSleeves, HMAC_WS key |
| `Installer\WallSleeves.addin` | ✅ Complete | Revit manifest, GUID 2F8D4B7A-E3C1-4A96-B5D8-F7E2A9C0D4B8 |
| `Resources\Wall_Sleeve.rfa` | ✅ Copied | From C:\victus\NewFolder\viewer\Wall_Sleeve.rfa |
| Inno Setup `.iss` | ❌ Missing | Needs `WallSleeves_Setup.iss` (Phase 5) |

### Wall_Sleeve.rfa — Family Contract

Family type: **Generic Model, wall-hosted** (wall-based template, FamilyPlacementType.OneLevelBasedHosted).
Cutting: **"Cut with Voids When Loaded" = Yes** — automatic cutting via `doc.Regenerate()` inside the transaction. `InstanceVoidCutUtils.AddInstanceVoidCut` is NOT used (that API is for non-hosted families only).
Placement: `NewFamilyInstance(ptCenter, symbol, wall, level, StructuralType.NonStructural)` where `ptCenter = faceIntersectionPt + faceNormal.Negate() * (wall.Width/2.0)` (wall centreline).
Shared parameter group: **Wall Sleeves**
RFA file: `C:\victus\RevitAPI\2026\RvtAddins\WallSleeves\Resources\Wall_Sleeve.rfa`

> **⚠️ Parameter names are full English strings, NOT short codes.**
> Earlier assumption of `wd`/`ht`/`tClr` etc. was WRONG. Actual names confirmed from screenshots:

**Dimensions group — all `Length` (stored in feet):**

| Parameter Name | Value Written |
|----------------|---------------|
| `Width` | MEP width + 2 × side clearance (total sleeve width) |
| `Height` | MEP height + top clearance + bottom clearance |
| `Depth` | Wall thickness (`wall.Width`, already in feet) |
| `Top Clearance` | Top clearance in feet |
| `Bottom Clearance` | Bottom clearance in feet |
| `Side Clearance` | Side clearance in feet (each side) |

**Data group:**

| Parameter Name | Data Type | Value Written |
|----------------|-----------|---------------|
| `Comments` | Text | (reserved for user notes — not written by command) |
| `Host Id` | Integer | `wall.Id.Value` (element ID of the host wall/floor) |
| `Linked Model Name` | Text | `linkedDoc.Title` |
| `Mark` | Text | Auto-generated: `WS-001`, `WS-002`, … |
| `Service Type` | Text | "Duct", "Pipe", or "Cable Tray" |
| `System Name` | Text | MEP system name (or "Undefined") |

### Wall Face Extraction — Critical Technical Notes

**⚠️ KNOWN ISSUE — `HostObjectUtils.GetSideFaces()` string comparison always fails.**
The stable representation format differs between HostObjectUtils references and solid geometry
PlanarFace references — their ConvertToStableRepresentation() strings never match.

**Current approach: `wall.Orientation` dot product (reliable exterior face filter):**
```csharp
XYZ wallOutward = wall.Orientation; // outward normal of exterior face
// Keep only faces whose normal aligns with wall.Orientation (exterior face)
if (pf.FaceNormal.DotProduct(wallOutward) < 0.9) continue;
```
Uses `ViewDetailLevel.Medium` (not Fine — Fine returns per-layer solids with wrong interface normals).
Fallback: if no exterior face found (e.g. curved wall), return all vertical side faces.

**Compound wall / Fine detail issue:**
`ViewDetailLevel.Fine` → per-layer solids → interior interface face normals wrong → ptCenter
lands outside wall → placement fails. Always use `ViewDetailLevel.Medium` for wall geometry.

### MEP Curve Extraction

Uses **LocationCurve** (not connector endpoints) for accurate centerline:

```csharp
Curve raw = (duct.Location as LocationCurve)?.Curve
            ?? GetConnectorCurve(duct.ConnectorManager.Connectors);
if (raw == null) continue;
Curve curve = raw.CreateTransformed(linkTransform);
```

For **round ducts**: `ductWidth = ductHeight = duct.Diameter`
For **rectangular ducts**: `ductWidth = duct.Width; ductHeight = duct.Height`

### Linked Model Transform

MEP element geometry is in **linked model coordinates**. Must transform to
host coordinates before intersection testing:

```csharp
Transform linkTransform = linkInst.GetTotalTransform();
Curve transformedCurve = raw.CreateTransformed(linkTransform);
```

**Coordinate system confirmed (2026-03-09):** Host and Snowdon Towers HVAC linked model share
the same coordinates — "Acquire Coordinates" returns "already same coordinates" error.
`GetTotalTransform()` returns effectively identity in this case, but transform must still
be applied for models where coordinates differ.

### Clearance Defaults (mm)

| MEP Type | Top | Bottom | Side |
|----------|-----|--------|------|
| Standard (all) | 50 | 50 | 50 |
| Fire wall (any) | 75 | 75 | 75 |
| Cable Tray | 100 | 25 | 50 |

Fire wall detection: `wall.Name.ToLower().Contains("fire")` → add 25 mm to all.
Unit conversion: `mmToFeet = 1.0 / 304.8` (≈ 0.003281)

### Wall Cutting

Family is wall-hosted with "Cut with Voids When Loaded = Yes". Cutting is automatic
via `doc.Regenerate()` inside the transaction. Do NOT call `InstanceVoidCutUtils.AddInstanceVoidCut`
(that API is for non-hosted families only — causes errors for wall-hosted families).

Guard with `InstanceVoidCutUtils.CanBeCutWithVoid(wall)` before processing each wall.
Soffit-Beam Wrap walls return `true` but fail with "not cutting anything" warning — minor,
3 occurrences with Revit sample model, not a real-world issue.

### Placement API (wall-hosted overload)

```csharp
Level level = doc.GetElement(wall.LevelId) as Level
              ?? FindNearestLevel(doc, pt.Z);
FamilyInstance inst = doc.Create.NewFamilyInstance(
    pt,       // XYZ point at wall centreline (face intersection pt + normal.Negate() * wall.Width/2)
    symbol,
    wall,
    level,
    Autodesk.Revit.DB.Structure.StructuralType.NonStructural);
```

### Vertical Displacement — RESOLVED (2026-03-10)

**Root cause:** The wall-based family (`Metric Generic Model Wall Based`) places its
origin at the **bottom** of the void (at Ref. Level). When `NewFamilyInstance(pt, ...)`
is called with `pt.Z = MEP centreline Z`, the void *bottom* lands at the centreline —
void appears entirely above the duct.

**Fix applied in `SleeveCuts.cs` → `TryPlace()`:**
```csharp
// Shift insertion point so the void CENTRE aligns with the MEP centreline.
// (tFt - bFt) / 2 adjusts for asymmetric clearances (e.g. cable trays).
XYZ ptInsert = new XYZ(pt.X, pt.Y, pt.Z + (tFt - bFt) / 2.0 - finalH / 2.0);
FamilyInstance inst = doc.Create.NewFamilyInstance(ptInsert, symbol, wall, level, NonStructural);
```

**Result:** 86 sleeves placed, void centred on duct, wall cuts working, 3 Soffit-Beam
Wrap modal dialogs (expected — `CanBeCutWithVoid()` returns true for these but they
genuinely can't be cut; accepted by owner as is).

### Reference Source Code Location

`C:\victus\RevitAPI\ref\LnT Source Code\Sleeves\`
- `DuctWallIntersecs.cs` — main reference (core algorithm)
- `WallSleevesView.cs` — enhanced version with room detection
- `TagFormat.cs` — tag generation (LnT-specific, not used in Phase 1)
- `DimData.cs` — clearance data class

### WallSleeves Product Page — Key Feature Notes

> **Do not reveal internal implementation rules** (e.g. the abbreviation algorithm).

| Feature | What to say on the product page |
|---------|----------------------------------|
| **Smart Mark generation** | Each sleeve receives a unique Mark value that encodes the service type (duct, pipe, cable tray) and the MEP system name — making it easy to identify and filter sleeves in the schedule and in Revit's element properties. |
| **Auto-populated schedule** | A Wall Sleeve Schedule is created automatically. All key parameters are pre-populated: service type, system name, linked model, dimensions, clearances, and host wall ID. |
| **Linked model picker** | If the project has multiple linked models, a clean dialog lists all loaded links so you can choose the MEP model without having to click elements in the viewport. |
| **Clearance control** | Customisable top, bottom, and side clearances per run. Fire walls automatically receive an additional clearance margin. Cable trays use asymmetric clearances (more headroom, less underside). |
| **Scope selection** | Run on the entire project or limit to the currently active 3D view — useful for checking individual levels. |
| **Zoom and select after placement** | After placement, choose to zoom directly to all placed sleeves in a 3D view, or open the Wall Sleeve Schedule — all from a single post-run dialog. |

### Phase Plan

| Phase | Scope | Status |
|-------|-------|--------|
| 1 | Core intersection + placement; scope selection (view/project); progress bar; schedule; TrialManager; .addin | ✅ Built 2026-03-08 |
| 1.1 | Ribbon cleanup: "Load Wall Sleeve Family" button removed; main button renamed "MEP Sleeves"; icon swapped to wo.png; auto-load family in SleeveCuts; diagnostic summary dialog | ✅ Built 2026-03-08 |
| 1.2 | Mark format (D-ME-001/P-HW-002/CT-001); LinkedModelPickerWindow; schedule fix (return ViewSchedule + CommandLink UX); zoom/select fix (ComputeCombinedBBox + ZoomAndCenterRectangle) | ✅ Built 2026-03-09 |
| 1.3 | Wall cutting: InstanceVoidCutUtils (superseded — wrong API for wall-hosted); RefreshActiveView after commit | ✅ Built 2026-03-09 (superseded by 1.4) |
| 1.4 | Placement fixes: ptInsert=wall centreline; CanBeCutWithVoid(wall) skip check; LocationCurve for MEP; removed TryAttachVoidCut; removed FamilyPlacementType guard; wall.Orientation dot-product for exterior face | ✅ Built 2026-03-09/10 |
| 1.5 | Z-offset fix: ptInsert.Z = pt.Z + (tFt-bFt)/2 - finalH/2; void now centred on MEP centreline | ✅ Built 2026-03-10 |
| 2 | Floor penetrations + duplicate prevention | Not started |
| 3 | Structural clash detection | Not started |
| 4 | Summary report (placed / skipped / errors per level) | Not started |
| 5 | TagFormat — structured tags, level codes, system codes | Not started |
| 6 | Installer (Inno Setup), website product page | Not started |

### Phase 2 — Floor Penetrations Strategy

**Key complications for floors:**

| Issue | Strategy |
|-------|---------|
| Structural floors are in a separate structural model | Need a second link picker or auto-detect structural + architectural floors across all loaded links |
| Architectural floors may not have a slab | Filter by floor thickness (skip `floor.Width < 50mm`) |
| Structural slab compound layers | Only test the **top face** of the combined solid; filter `pf.FaceNormal.Z > 0.9` |
| Floor face orientation differs from walls | `refDir` for floor placement should be along the MEP run direction |

**Face filter for floors (top face only):**
```csharp
if (Math.Abs(pf.FaceNormal.Z) < 0.9) continue;  // skip side faces
if (pf.FaceNormal.Z < 0) continue;               // skip bottom faces
```

### Build
```
cd C:\victus\RevitAPI\2026\RvtAddins
dotnet build WallSleeves/WallSleeves.csproj --configuration Release
```

---

## 8a. Other Add-ins in RvtAddins repo (not yet developed)

| Folder | Purpose |
|--------|---------|
| DimData | Dimension data extraction |
| DisDim | Display dimensions |
| MaterialsByRoom | Material takeoffs by room |
| NWC | Navisworks export helper |
| SeparationLines | Room separation line utilities |

---

## 9. GoDaddy DNS Setup for GitHub Pages

Add these records in GoDaddy DNS for `archbim.co.in`:

| Type  | Name | Value                    |
|-------|------|--------------------------|
| A     | @    | 185.199.108.153          |
| A     | @    | 185.199.109.153          |
| A     | @    | 185.199.110.153          |
| A     | @    | 185.199.111.153          |
| CNAME | www  | SanjivKK.github.io |

Then in GitHub repo Settings → Pages:
- Source: `main` branch, `/ (root)`
- Custom domain: `archbim.co.in`
- Enable "Enforce HTTPS" (after DNS propagates, ~10 min)

---

## 10. Session Notes

| Date | What happened |
|------|--------------|
| 2026-02-28 | Full PHE review; added Planting category; renamed panel to Survey Coordinates; converted to PulldownButton; fixed mass-move bug (1mm drift check); added Regenerate + RefreshActiveView + Selection; Inno Setup script + .addin manifest created; CLAUDE.md + Website created |
| 2026-02-28 | Rebranded NeoBIM → ArchBIM everywhere; shared param file bundled into installer; Website git repo initialised with CNAME; domain archbim.co.in registered on GoDaddy |
| 2026-02-28 | Dropped "Insights" suffix → brand is now simply "ArchBIM"; website restructured: homepage at index.html + product page at survey-coordinates/index.html; GitHub repo to be named "archbim" |
| 2026-03-02 | Added 30-day free trial (TrialManager.cs, HMAC+SID fingerprint); price changed $49→$12; website updated with 2-tier pricing (trial + full); pushed to GitHub SanjivKK/archbim; DNS pending GoDaddy WHOIS verification support call |
| 2026-03-03 | LemonSqueezy account approved (Merchant of Record live); ImprovMX email forwarding set up (forwarding pending DNS); logo/favicon sizes increased (nav 48px, footer 36px); homepage restructured into two product categories: Revit Add-ins (desktop) + Web Apps (browser); Core Layout (SanjivKK/core-site, Three.js parametric 3D core generator) added as first Web App card |
| 2026-03-03 | Removed last NeoBIM reference (launch.json); About heading → "Built by an Architect, for Architects"; homepage cards shortened to one-liner hooks (detail stays on product pages); Core Layout description updated from LinkedIn post (UK residential regs, concept-stage 3D volumes); all changes pushed to GitHub |
| 2026-03-06 | SheetTransfer add-in built — Phase 1 complete (titleblock resolution, level mapping UI, parameter transfer); added to RvtAddins.sln; builds clean. Price $18. TrialManager copied with SheetTransfer-specific registry key and HMAC. |
| 2026-03-07 | SheetTransfer published on LemonSqueezy ($18 USD, 2 activations, lifetime); checkout URL live in sheet-transfer/index.html; screenshots section added to product page (3 image slots awaiting screenshots). |
| 2026-03-07 | WallSleeves add-in deep-dive: analysed existing codebase; read all 5 reference files in LnT Source Code; recorded full technical notes in CLAUDE.md. Wall_Sleeve.rfa to be created by owner; Phase 1 implementation pending. |
| 2026-03-08 | Wall_Sleeve.rfa received from owner. Confirmed exact parameter names (full English strings). Phase 1 fully implemented: TrialManager, Clearances dialog, ProgressWindow, LinkedProcessor, full SleeveCuts core algorithm. Builds clean. |
| 2026-03-08 | Deployed and tested — schedule was empty (0 sleeves placed). Fixes: switched NewFamilyInstance overload; added diagnostic summary; auto-load Wall_Sleeve.rfa; removed redundant ribbon button; renamed "MEP Sleeves"; swapped icon to wo.png. |
| 2026-03-09 | WallSleeves Phase 1.2–1.4 built+deployed: Mark format (D-ME-001/P-HW-002/CT-001); LinkedModelPickerWindow; schedule CommandLinks UX; zoom+select fix; wall.Orientation dot-product exterior face filter (ViewDetailLevel.Medium); wall-hosted NewFamilyInstance overload; LocationCurve for MEP; CanBeCutWithVoid guard. |
| 2026-03-09 | WallSleeves live test: 86 placed, 3 Soffit-Beam Wrap errors (expected), cuts working, but lateral displacement persists. Family geometry confirmed correct (EQ markers). Coordinates confirmed same (Acquire Coordinates → "already same"). Hypotheses narrowed to: pt calculation, LocationCurve vs visual center discrepancy. |
| 2026-03-10 | Master CLAUDE.md moved to C:\Users\kapil\.claude\CLAUDE.md (global Claude Code location, inaccessible to Codex). Stub left at C:\victus\CLAUDE.md. Backup maintained at C:\victus\Website\CLAUDE.md (in GitHub repo). Adding log file to SleeveCuts.cs for displacement diagnosis. |
| 2026-03-10 | WallSleeves Phase 1.5 RESOLVED. Root cause of vertical displacement: wall-based family origin is at void bottom (Ref. Level), not void centre. Fix: ptInsert.Z = pt.Z + (tFt - bFt)/2 - finalH/2. Confirmed working — sleeve now centred on duct. 86 placed, cuts working, 3 Soffit-Beam Wrap dialogs (accepted). |
| 2026-03-10 | WallSleeves launch: updated Wall_Sleeve.rfa in Resources folder; created WallSleeves_Setup.iss (Inno Setup — installs DLL+addin+RFA to Addins\2025\ and \2026\); product page updated ($22→$48, mark format pills in Smart Mark card); homepage card updated ($22→$48); all pushed to GitHub. |

## 11. Pending Owner Action Items

| Priority | Item | Notes |
|----------|------|-------|
| ✅ Done | Create Survey Coordinates product on LemonSqueezy | Published — checkout URL live |
| ✅ Done | Create SheetTransfer product on LemonSqueezy ($18) | Published — checkout URL: archbim.lemonsqueezy.com/checkout/buy/c1aa68f7… |
| ✅ Done | Add SheetTransfer icon (32×32 PNG) | st32.png embedded as EmbeddedResource in App.cs |
| ✅ Done | Add screenshots to Sheet Transfer product page | 3 PNGs saved to Website\sheet-transfer\images\ |
| ✅ Done | Compile SheetTransfer_Setup.exe + upload to GitHub Release | Uploaded to SanjivKK/archbim v1.0.0 alongside SurveyCoordinates_Setup.exe |
| ✅ Done | Push website to GitHub | Screenshots + sheet-transfer page + LemonSqueezy URL all live |
| ✅ Done | Create Privacy Policy, Terms of Service, Refund Policy pages | All three live — committed 2026-03-03, footer links working |
| 🟡 Medium | Crop logo.png / favicon.png — remove excess transparent padding | Images at C:\victus\Website\images\ — crop tight to artwork |
| 🟡 Medium | Edit demo videos — replace Autodesk App Store URL with archbim.co.in | YouTube videos exist; audio needs re-recording or editing |
| 🟡 Medium | Create Core Layout product page (core-layout/index.html) | Embed YouTube demo video; mirror survey-coordinates page structure |
| 🟡 Medium | Add remaining Revit add-ins to homepage as Coming Soon cards | DimData, DisDim, MaterialsByRoom, NWC, SeparationLines (WallSleeves now live on homepage) |
| 🟢 Low | DNS propagation for archbim.co.in | GoDaddy WHOIS verification support ticket — check status |
| 🟢 Low | ImprovMX email forwarding | Will auto-activate once DNS propagates |
| 🟢 Low | Enable "Enforce HTTPS" on GitHub Pages | Do after DNS resolves |
| ✅ Done | Provide Wall_Sleeve.rfa | RFA received 2026-03-08; copied to WallSleeves\Resources\ |
| ✅ Done | Implement WallSleeves Phase 1–1.4 | Built + deployed 2026-03-08/09 |
| ✅ Done | Resolve WallSleeves vertical displacement (Phase 1.5) | Z-offset fix: ptInsert.Z = pt.Z + (tFt-bFt)/2 - finalH/2. Void now centred on MEP centreline. |
| ✅ Done | Save screenshots to Website\wall-sleeves\images\ | 5 PNGs committed: settings-dialog, progress-window, summary-dialog, link-selection, readme.txt |
| ✅ Done | Create WallSleeves installer script | WallSleeves_Setup.iss created in WallSleeves\Installer\ |
| ✅ Done | Push wall-sleeves product page + homepage card to GitHub | wall-sleeves/index.html live at archbim.co.in/wall-sleeves/; price $48 everywhere |
| 🔴 High | Create WallSleeves product on LemonSqueezy ($48) | Then update checkout URL in wall-sleeves/index.html (currently #pricing placeholder) |
| 🔴 High | Compile WallSleeves_Setup.exe with Inno Setup 6 | Install Inno Setup 6 → right-click WallSleeves_Setup.iss → Compile → upload EXE to GitHub Release |
| 🔵 Future | Split website into private source repo + public deploy repo | See Section 12 — do when a build step or secrets are introduced |

---

## 12. Repo Split Strategy (Future — Not Urgent)

Follow the same pattern as **Core** (private source) → **core-site** (public deploy):

**Trigger:** Do this when any of the following are introduced:
- A build step (Vite, webpack, etc.)
- A backend or serverless functions
- API keys, `.env` files, or any secrets
- Anything in the source that should not be publicly visible

**Target pattern:**
| Repo | Visibility | Contents |
|------|-----------|----------|
| `archbim-source` (new, private) | Private | Source code, CLAUDE.md, build config, `.env`, dev notes |
| `archbim` (existing, public) | Public | Compiled output only — what GitHub Pages serves |

**Right now — no action needed:**
- The site is pure HTML/CSS with no build step and no secrets
- CLAUDE.md contains no sensitive keys or credentials
- Keep the current single-repo setup until a build step is introduced

**When the time comes:**
1. Create new private repo `archbim-source`
2. Move all source + CLAUDE.md there
3. Keep `archbim` (public) as the deploy target only — push built output via CI/CD (GitHub Actions)
4. Update this CLAUDE.md accordingly

---

## 13. Agentic AI Embedded in Revit (Future Trigger: 10–20 Add-ins)

**Concept**

Each ArchBIM add-in automates one focused Revit task. When the library reaches
10–20 published tools, those commands become **tools** for an AI agent embedded
directly inside Revit — a docked chat panel where an architect types a natural
language instruction and the agent plans and executes a sequence of add-in calls
to complete it.

**Example commands the agent could handle**
| Instruction | Agent action |
|---|---|
| "Copy all Level 3 sheets from the source project and update their survey coordinates" | SheetTransfer (level-filtered) → SurveyCoordinates (UpdateParameterValues) |
| "Create a sheet for each level using the standard A1 titleblock" | Future SheetCreate tool |
| "Export this model to Navisworks and update the coordination model" | NWC add-in |
| "Tag all site elements with northings and eastings then export a schedule" | SurveyCoordinates → future ScheduleExport tool |

**Architecture (when the time comes)**
```
Revit ribbon button → WPF Chat Panel
      ↕
  Claude API (tool_use) — server-side or local key
      ↓ selects tools
  ArchBIM Tool Registry (JSON manifests per add-in)
      ↓ dispatches
  Each add-in's IAgentTool.Execute(JsonNode params) → Result
```

- Each add-in exposes a lightweight `IAgentTool` interface (name, description,
  JSON schema for parameters, Execute method)
- The agent add-in loads all installed ArchBIM tools at startup, builds the
  Claude tool-use manifest dynamically
- Claude selects tools, passes structured params, receives structured results,
  and can chain multiple calls
- The chat panel shows each step with its result so the user can follow along

**Design principles to apply now (before the agent exists)**
- Keep command inputs and outputs clean and parameterised
- Summary dialogs already return structured counts (created / skipped / errors) —
  these become agent-readable results
- Avoid hard-coded UI-only flows; the core logic should be callable without
  showing dialogs (future: headless mode for agent calls)

**Trigger:** Start building the agent add-in when 10–20 ArchBIM tools are
published and stable. The groundwork (clean commands, structured summaries)
is being laid right now.
