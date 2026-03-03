# ArchBIM — Master Project Memory
> This file is read automatically by Claude Code at the start of every session.
> Keep it up to date as the project evolves.
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
├── CLAUDE.md                          ← THIS FILE (master memory)
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
## 7. Other Add-ins in RvtAddins repo
| Folder | Purpose |
|--------|---------|
| DimData | Dimension data extraction |
| DisDim | Display dimensions |
| MaterialsByRoom | Material takeoffs by room |
| NWC | Navisworks export helper |
| SeparationLines | Room separation line utilities |
| WallSleeves | Wall sleeve placement |
> These have not been discussed in detail yet. Each has a similar structure
> to PHE (IExternalApplication + IExternalCommand classes).
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
| 2026-03-03 | Recovered CLAUDE.md after Claude reinstall; committed to GitHub repo (SanjivKK/archbim) so it is never lost again |
| 2026-03-03 | Added logo image (38px) to navbar + footer, favicon link in head of index.html; images added at C:\victus\Website\images\ and pushed to main from Windows — COMPLETE |
| 2026-03-03 | Developer switching to local Windows drive (C:\victus\Website) for all future sessions; git repo is fully in sync on main branch as of this session |
