# Changelog

All notable changes to this project are documented here. Each entry records **what** changed and **why** we made the decision.

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)

---

## [2026-07-17] — Summer feature depth & alpha-readiness completion

**Why:** Baseline's alpha vertical-slice window (late June–mid July 2026) shipped negotiation depth, user-playable two-way contracts, infrastructure hardening, E2E coverage for the core game loop, and QA-driven refinements. The engine now enforces ownership atomicity, FA-round bidding routes through negotiation, minimum-salary signing works, and season-long play is supported end-to-end with session restore. Owner-requested player designation brackets (block/untouchable/target) add front-office control.

### Added

- **Negotiation v1.1 — FA-round competing-offers panel** (#400, #401) — surfaces per-free-agent competing bids during FA rounds; human player can Match, Counter, or Withdraw. `collectRoundOffers` and `previewFaRound` expose engine-computed bids; `submitFaOffer` gains idempotency fix.
- **Negotiation v1.1 — personality-driven dialogue** (#412) — dialogue now varies by derived agent temperament (mercenary/loyal/ambitious/weary from age/potential/OVR/archetype) and shifts tone with the human GM's reputation signal. Pure `temperament.ts` computes signals; no schema changes.
- **Negotiation v1.1 — Table routed through FA-round bidding** (#440) — closing a deal at The Table during rounds 1–3 now submits terms as a verbal-commitment offer (0.15–0.30 bonus weighting); players can still be outbid; walking away locks the table until the next round. Post-rounds instant-sign path unchanged.
- **Two-way contract user lifecycle — convert and promote** (#403) — human players can convert eligible roster players to two-way deals or promote two-way players to standard minimum. Gates reuse cap-exclusion and dev-boost logic; `twoWayDaysInNba` counter resets properly during offseason.
- **Two-way contract user lifecycle — G-League send-down and recall** (#414) — players on two-way deals can be assigned to G-League (excluded from lineup, stop accruing NBA days) and recalled at any time. Adds save migration v20→v21 (backfill `gLeagueAssigned: false`). Offseason `processGLeagueLoop` integration.
- **Supermax / Rose Rule extension eligibility & extend-and-trade validation** (#429) — new pure `cbaExtensions.ts` module adds `isDesignatedVeteranEligible`, `isRoseRuleEligible`, `computeDesignatedMaxSalary`, and `validateExtendAndTrade`. Fixes four extension bugs: 0-sentinel misreads, moratorium-day overflow, Rose Rule recency window, and designated-max vs. Bird-rights ceilings.
- **League integrity inspector UI** (#392) — `/game/inspector` page renders non-blocking validator errors/warnings in dev/E2E builds. Shows error code, message, and clean-state badge. Gated to dev/E2E; production shows "unavailable." Unblocks debugging during alpha testing.
- **Draft-class picker at offseason transition** (#370) — allows human player to choose draft class (2025, 2026, etc.) during league's advance into draft-lottery phase. Supports replayability and scenario testing.
- **Smart roster filters** (#371) — role-based filtering (starter/bench/bench-deep), multi-select archetypes, URL-shareable state. Extends inline Roster.tsx filters.
- **Onboarding tour** (#346) — first-time flow on Dashboard with Spotlight anchors guiding new players through roster review, sim, and save export via `useOnboarding` hook.
- **Playoff Fit & Clutch Rating productization** (#360) — Player Profile gains dedicated "Playoff Fit" card (archetype modifier, thrives/struggles label, role split) and "Clutch Rating" card (radial gauge + bonus breakdown by game situation).
- **Contention window & roster mood surfaces** (#362) — Dashboard displays team's contention window (peak years) and roster-mood band (morale aggregation). Enables long-term franchise strategy visibility.
- **Season news feed** (#363) — timeline of All-Star Weekend and game results integrated into Dashboard, surfacing narrative beats during sim play.
- **Era picker** (#381) — league generation now gates era selection; registry extension point for future historical eras (rules, cap values, player-pool flavor).
- **Honest settings page** (#377) — two sim styles labeled with intended audience; inert controls (cap tiers, num games) no longer presented as editable; clearer language on deferred features.
- **Async sim with progress overlay** (#443) — `simToDay` and `simPhase` now yield to the event loop every 3 days, driving a blocking "Day X / Y" progress bar. Cancel halts at the last completed day. Fixes perceived freezes during long sims reported in QA playtests.
- **Sim interruption tuning — headline-offer routing** (#442) — only headline offers (top-3-OVR or 80+ OVR players) interrupt sims; routine offers flow silently to Trade Inbox with live NavRail badge. New Settings section "Sim interruptions" (All/Big-only/Never + toggles for injury/deadline). Reduces dismissals from ~100–150/season to <10.
- **Minimum player salary exception** (#449) — capped-out teams no longer locked out of FA. Engine accepts salary ≤ player minimum as `signingType: 'minimum'` (unbounded reservation). `generateFaOffer` bids minimum for capped CPU teams. FA row button no longer disables on market-value affordability. Save migration v21→v22 adds `'minimum'` to all signingType unions.
- **Player designation brackets — block / untouchable / target** (#454) — new `Player.stance` enum (save migration v21→v22) replaces `isUntouchable` boolean. On-the-block players draw 2× AI trade interest. Untouchables demand ≥1.4× overpays on inbox trades. `Team.targetPlayerIds` with Crosshair toggles on Players/Profile pages; TradeHub Targets tab with build-trade deep-link; targets ranked first in opponent-panel suggestions and FA re-sign lists.
- **Deterministic draft-start fixture support** — "Draft" tab in StartPhasePicker gated behind dev/E2E flag (`?e2e=1`), enabling controlled draft-phase reproducibility.
- **Production reload persistence** (#452) — `useSessionRestore` hook auto-loads the most recent IndexedDB save on AppShell mount when no league is in memory. Page reload returns to the same league/day. Quiet "Restoring session…" overlay masks the gap. `?e2e=clean=1` skips restore for Playwright wipes.

### Changed

- **Autosave serialization via coalescing gate** (#396) — all autosave writes route through one serializing gate (one save at a time, newest-pending-wins). Eliminates duplicate-'Autosave'-slot race and stale-snapshot-out-of-order race. Concurrent tests confirm stability.
- **Write-time ownership transaction layer** — all roster mutations (trades, FA signings, draft, two-way convert/promote, 10-day callups, option exercises, waivers, RFA matches) now route through `applyOwnershipAndContractTransaction`. Eliminates direct writes to `player.teamId` and `team.roster`, enabling deterministic replay and atomicity.
- **Alpha usability feedback** (#398) — disabled Propose/Sign/Offer buttons now explain WHY via tooltip + `aria-label`. Draft phase gate names current phase + unlock condition. WhatNextCard gains "Draft Rookies" suggestion for unsigned first-round cap holds. Apron check folded into trade Propose disabled state.
- **CalendarCell modernization to Cinematic Dark** (#450) — flat dark squircle opponent badge (quiet team tint + self-colored edge) replacing gradient tile with glow; rounded inset home/away accent bar replacing borderLeft; tonal hover overlay replacing translate-lift + shadow bloom. Behavior/props unchanged.
- **CI e2e job** (#446) — `checks.yml` now runs Playwright suite (chromium, `CI=true`) on every PR. Uses dev server + 60s test / 10s expect budgets. 24 live specs + pre-existing skip; reload specs re-enabled via autosave-settled wait.
- **E2E trade & free-agency success paths use adaptive search** (#394) — both happy paths walk the real seed-42 roster for the first legal move instead of hand-crafted fixtures. Fixes 3 bugs en route: Build-tab reset no-op, BKN MLE room too tight (→ switched FA test to UTA), and `extractFullNameFromRowText` mis-parse of glued position/RFA prefix.
- **Three-way trade builder coverage & mutation fix** (#433) — new unit + E2E coverage for `ThreeTeamTradeBuilder.tsx` surfaced a real bug: CPU-to-CPU leg now routes through `applyOwnershipAndContractTransaction` instead of manual mutation. Fixes phantom `twoWayPlayers` entries and missed bird-rights updates on that leg.

### Fixed

- **Save migration hardening and validation** (#392 integration, completed.md DONE-3) — save import validates the league via `validateLeague` (non-destructive); corrupted saves fail gracefully with user-facing error. Migration chain replay tests cover v1 backfill → v22. Schema versioning (Dexie v3, save format v22) stays in sync.
- **Rookie-scale year-4 option anchored to CBA table** (#409) — fourth-year option years now stamped from official per-slot CBA rookie-salary table, not flat +1 math. Fixes edge cases for late-round picks.
- **Trade eligible day moratorium boundaries** (#406) — added regression tests for free-agent trade moratorium timing; clarifies phase-driven eligibility windows.
- **CPU apron and hard-cap legality** — CPU trade packages audit apron/hard-cap limits at suggestion time (`tradeAiShop`) before ranking; rejects illegal aggregations up front. Paired with contract legality regression tests.
- **Position-weighted OVR and market-value corrections** (#DONE-5) — market value now uses position-weighted OVR proxy across all price-discovery surfaces, fixing discrepancies between compute paths.
- **Save/load E2E validates exported league integrity** (#435) — integrity check in export→reimport spec; `validateLeague` runs directly against freshly-simulated exported league, failing test if structural errors detected.
- **Playoff jump wiring fix** (#447) — all three playoff-jump surfaces (TopBar, Dashboard, Schedule) now route through `jumpToPlayoffs`, which blanks stop conditions so jump completes in one action. Injury/return modals queue at completion. Fixes observed multi-day jump sequences (Day 81→112→132→144).
- **Dashboard record fallback and dead code** (#432) — removed dead `simulateToNextEvent` function; dashboard no longer hangs on missing game results.
- **UI QA sweep** (#430) — button-state tooltips, trade rejection reason copy, phase-gate language, WhatNext rookie suggestion.

### Infrastructure

- **Alpha vertical-slice E2E coverage** — 65+ passed tests across onboarding, trade, free-agency, draft, sim, save/load, and negative paths on bundled seed-42 roster. Mobile flow intentionally skipped; one legacy spec skip remains.
- **Deterministic browser state and import/load race hardening** — E2E pre-test clear-state and import→reimport race regression targets in place.
- **QA playtest findings triaged and fixed** — 2-season playtest by 3 parallel QA agents (sim/playoffs, FA/Table, trade surfaces) surfaced and fixed 5 immediate issues: headline-offer routing, async-sim freezes, minimum-salary lockout, playoff-jump incompletion, and prod-reload missing. New backlog items filed; false positives disposed (negotiation routing, War Chest phase scope, progress-overlay invisibility all verified working). Console clean; trade/FA e2e deterministic across full season.

---

## [2026-07-11] — Negotiation v1.1, two-way contracts, and alpha-readiness foundation

**Why:** Baseline's alpha vertical-slice window (late June–mid July 2026) shipped negotiation depth, user-playable two-way contracts, and the infrastructure hardening required for deterministic first-season testing. The engine now enforces ownership atomicity via write-time transaction layers, save migrations validate on import, and the E2E suite covers the core game loop (draft → free agency → sim → save/load) on real bundled roster data.

### Added

- **Negotiation v1.1 — FA-round competing-offers panel** (#400, #401) — surfaces per-free-agent competing bids and allows the human player to Match, Counter (with custom offer), or Withdraw during the simulated FA round. `collectRoundOffers` and `previewFaRound` expose bids computed by the engine; `submitFaOffer` now has idempotency fix and two-way control flow. Personality dialogue trees branch by derived agent temperament and GM warmth signal (#412, DONE-16).
- **Negotiation v1.1 — personality-driven dialogue** (#412) — dialogue now varies by derived agent temperament (mercenary/loyal/ambitious/weary from age/potential/OVR/archetype) and shifts tone with the human GM's reputation. Pure `temperament.ts` computes signal; no new schema fields, no migrations.
- **Two-way contract lifecycle — user-playable convert + promote** (#403) — human players can now convert an eligible roster player to a two-way deal or promote an existing two-way player to a standard minimum contract. Gates reuse existing cap exclusion and dev-boost logic; `twoWayDaysInNba` counter resets properly during offseason.
- **Two-way send-down / G-League recall** (#414) — completes the two-way system. Players on two-way deals can be sent down to the G-League and recalled without roster slots or salary implications. Offseason integration via `processGLeagueLoop`.
- **League integrity inspector** (#392) — diagnostics page (`/game/inspector`) renders non-blocking validator errors and warnings in dev/E2E builds. Shows error code, message, and clean-state badge. Gated to dev/E2E; production builds show "unavailable." Unblocks debugging during alpha testing.
- **Draft-class picker at offseason transition** (#370) — allows the human player to choose a draft class (2025, 2026, etc.) as the league advances into draft-lottery phase, supporting replayability and scenario testing.
- **Smart roster filters** (#371) — role-based filtering (starter/bench/bench-deep), multi-select archetypes, and URL-shareable state. Extends the existing inline Roster.tsx filters; no standalone hook.
- **Onboarding tour** (#346) — first-time flow on Dashboard with Spotlight anchors guiding new players through roster review, sim, and save export. Data-driven via `useOnboarding` hook.
- **Playoff Fit & Clutch Rating productization** (#360) — Player Profile gains dedicated "Playoff Fit" card (archetype modifier, thrives/struggles label, starter/bench split) and "Clutch Rating" card (radial gauge + close-game/elimination/finals bonus breakdown).
- **Contention window & roster mood surfaces** (#362) — Dashboard surface shows the team's contention window (peak years) and roster mood band (morale aggregation). Enables long-term franchise strategy visibility.
- **Season news feed** (#363) — timeline of All-Star Weekend and game results integrated into the Dashboard, surfacing narrative beats during sim play.
- **Era picker** (#381) — league generation now gates era selection; registry extension point for future historical eras (rules, cap values, player-pool flavor).
- **Honest settings page** (#377) — two sim styles labeled with their intended audience; inert controls (cap tiers, num games) no longer presented as editable; clearer language on deferred features.

### Changed

- **Autosave serialization via coalescing gate** (#396) — all autosave writes now route through one serializing gate (one save at a time, newest-pending-wins). Eliminates the duplicate-'Autosave'-slot race (concurrent first-time saves both created a record) and the stale-snapshot-out-of-order race. Concurrent tests confirm stability; `sim-save-load` e2e green.
- **Alpha usability feedback** (#398) — disabled Propose/Sign/Offer buttons now explain WHY via tooltip + `aria-label`. Draft phase gate names current phase + unlock condition. WhatNextCard gains "Draft Rookies" suggestion for unsigned first-round cap holds. Apron check folded into trade Propose disabled state.
- **Write-time ownership transaction layer** (DONE-1) — all roster mutations (trades, FA signings, draft, two-way convert/promote, 10-day callups, option exercises, waivers, RFA matches) now route through `applyOwnershipAndContractTransaction`. Eliminates direct writes to `player.teamId` and `team.roster`, enabling deterministic replay and atomicity guarantees. Earlier direct-write paths in auto-draft now audit-logged for future closure.

### Fixed

- **Save migration hardening and validation** (DONE-3) — save import now validates the league via `validateLeague` (non-destructive); corrupted saves fail gracefully with a user-facing error. Migration chain replay tests cover v1 backfill → v20. Schema versioning (Dexie v3, save format v20) stays in sync.
- **E2E trade & free-agency success paths always execute** (#394) — both E2E happy paths (trade build + FA signing) now use adaptive search on the real bundled roster, walking for the first legal move instead of hand-crafted fixtures. Fixes 3 bugs found en route: Build-tab reset no-op, BKN MLE room too tight (→ switched FA test to UTA), and `extractFullNameFromRowText` mis-parse of glued position/RFA prefix.
- **Rookie-scale year-4 option anchored to real CBA table** (#409) — fourth-year option years are now stamped from the official per-slot CBA rookie-salary table, not a flat +1 math. Fixes edge cases for late-round picks.
- **tradeEligibleDay moratorium boundaries** (#406) — added regression tests for free-agent trade moratorium timing; clarifies phase-driven eligibility windows.
- **Hard-cap signing mutation audit** (DONE-9) — post-draft integrity verified: `validateLeague` confirms zero errors after draft phase completion.
- **CPU apron and hard-cap legality** (DONE-4) — CPU trade packages now audit apron/hard-cap limits at suggestion time (`tradeAiShop`) before ranking; rejects illegal aggregations up front. Paired with contract legality regression tests.
- **Position-weighted OVR and market-value corrections** (DONE-5) — market value now uses position-weighted OVR proxy across all price-discovery surfaces, fixing discrepancies between compute paths.

### Infrastructure

- **Deterministic draft-start fixture support** (DONE-6) — "Draft" tab in StartPhasePicker gated behind dev/E2E flag (`?e2e=1`), enabling controlled draft-phase reproducibility.
- **Deterministic browser state and import/load race hardening** (DONE-7) — E2E pre-test clear-state and import→reimport race regression target in place.
- **Alpha vertical-slice E2E coverage** (DONE-8) — 65 passed tests across onboarding, trade, free-agency, draft, sim, save/load, and negative paths on the bundled seed-42 roster. Mobile flow intentionally skipped; one legacy spec skip remains.

---

## [2026-06-02] — Closed alpha support surface

**Why:** Baseline is shifting from broad feature work to alpha hardening. The next validation target is first-season fun for a controlled community waitlist, with support paths that let testers report confusing flows and attach reproducible save files.

### Added
- **Closed alpha public page** (`/alpha`) — explains the alpha scope, waitlist path, feedback expectations, deferred features, and legal posture.
- **In-game Support page** (`/game/support`) — collects bug-report checklist, known issues, smoke path, feedback categories, and release-note links.
- **First-season guide on Dashboard** — lightweight action checklist for roster review, rotation, trade, cap pressure, and save export.
- **Full save export/import** in Settings -> Save Support — exports league state, game results, and playoff state in a versioned JSON envelope for bug reports and imported repro saves.
- **`docs/ALPHA_ROADMAP.md`** — alpha scope, readiness criteria, manual smoke path, feedback categories, and deferred V2 systems.

### Changed
- Landing page now reflects closed-alpha positioning and links to the alpha waitlist.
- README quick-start port corrected to `5175`.

---

## [2026-05-29] — Free Agency shadcn refactor

**Why:** The free agency UI was built on custom containers and ad hoc forms that didn't leverage the shadcn component system. This refactor modernizes the FA experience onto Dialog, Form, Slider, Select, and Card primitives while preserving all existing player domain components and progression visuals.

### Added
- **`Form` component** from shadcn (`src/components/ui/form.tsx`) — react-hook-form wrapper for form validation with Zod schemas.
- **`contractOfferSchema`** in `src/types/contract.ts` — Zod schema for contract offers (salary, years, raisePercent).
- **`contractOptionSchema`** in `src/types/contract.ts` — Zod schema for contract option decisions.
- **`react-hook-form` and `@hookform/resolvers`** dependencies — form state management and Zod validation.

### Changed
- **ContractOfferModal** — rebuilt on shadcn `Dialog` + `Form` with `Slider` for salary/years and `Select` for contract structure. Replaced custom modal implementation and manual state management with react-hook-form validation.
- **ContractOptionsModal** — migrated to `Form` with react-hook-form, using `Button` components for exercise/decline decisions instead of raw buttons with manual state.
- **ContractExtension page** — refactored from manual state to `Form` with `Slider` for salary input and `Select` for years selection, using Zod validation.
- **InjuryPreventionPanel** — now uses shadcn `Card` components instead of custom bordered divs, with `Button` components for plan selection.
- **ProgressionSummaryCards** — rebuilt on shadcn `Card` components with proper header/content structure instead of custom divs.
- **FreeAgency page** — preserved all existing functionality, now properly integrates with refactored modals.

### Preserved
- **Player domain components** — `PlayerAvatar`, `AnimatedOvrCircle`, `PositionBadge`, `DeltaRatingBar` remain unchanged.
- **Progression visuals** — OVR displays, rating deltas, focus badges, and progression indicators retain their existing styling and behavior.
- **Business logic** — FA market value computation, offer submission, option decisions, and injury prevention all use existing engine functions.

---

## [2026-05-20] — Draft page shadcn war-room refactor

**Why:** The draft page still used older ad hoc containers and a loose layout that made scouting, board review, and live pick management feel fragmented. This refactor moves the page onto the app's shadcn primitives while tightening the screen into a denser draft-room workflow.

### Changed
- **Draft page UI rebuilt around shadcn composition** — `Tabs`, `Dialog`, `Table`, `Card`, `Input`, `Progress`, and `ScrollArea` now drive the Draft page instead of mixed custom sections and one-off containers.
- **Pre-draft workspace redesign** — added a board-centric layout with class pulse, scouting budget, owned picks, searchable prospect board, and lottery odds table.
- **Live draft war room** — added on-clock status, progress tracking, next-user-pick visibility, integrated auto-pick confirm dialog, and a persistent draft order panel.
- **Prospect board search and fit view** — the draft board now supports inline search plus a table-based team-fit/scouting view without changing any scouting or draft logic.
- **`ProspectCard` container refresh** — prospect rows now use shadcn `Card` and `Button` while preserving the existing draft-specific visuals, scouting visibility, ratings, and risk signals.

---

## [2026-05-04] — Fix: Player profile shows full career history via ESPN search fallback

**Why:** Player profiles only showed 2 seasons (simmed) for veterans like Curry because the ESPN historical stats fetch required a pre-existing `espnPlayerIds` mapping that was only populated during live ESPN data import. Without that import, the mapping was null and the ESPN fetch was silently skipped.

### Changed
- **ESPN search fallback**: Added `searchEspnPlayerByName()` to `liveNbaData.ts` — searches ESPN by first/last name when no stored ESPN ID mapping exists. Searches player's team first, then falls back to all 30 teams (handles traded players).
- **ESPN ID caching**: Found ESPN IDs are now cached back into the store via `updateEspnPlayerId()` so subsequent profile views don't need to re-search.
- **http:// → https:// ref fix**: ESPN roster API returns `http://` refs which browsers block as mixed content. Now converts all refs to `https://`.
- **Staleness guard**: Added cleanup to the ESPN history `useEffect` to prevent stale async results from overwriting current state when players switch rapidly.
- **PlayerProfile stats fetch**: No longer requires `espnPlayerIds` to be populated. Falls back to name search via ESPN API, fetching career history for any player regardless of import state.
- **Career range estimation**: Uses `bornYear + 20` (estimated debut) instead of `currentSeason - 10` when no stats exist yet, so veterans like Curry (debut 2009) get their full 17-season GSW history fetched.

---

## [2026-05-04] — Shot Tendency refactor: behavior-driven shot selection (NBA 2K-style)

**Why:** The sim engine previously used a fixed 58–66% handler keep-rate for shoot vs pass decisions — a pass-first PG and an ISO scorer were treated identically. The `ShotProfile` type had unused accuracy fields. This refactor introduces per-player behavioral tendencies (0–100 scale) following the 2K design principle: *Ratings = ability, Tendencies = behavior*.

### Added
- **`ShotTendency` type** replaces `ShotProfile`. Contains zone distribution (rim/mid/corner3/above3 rates, same as before) plus four behavioral tendencies on a 0–100 scale:
  - `shootTendency` — shoot vs pass. Maps to per-player handler keep-rate in sim (50 → 58%/66% baseline, 80 → 70%/78%, 20 → 40%/48%).
  - `driveTendency` — high values shift inside shot rate up and 3PT down.
  - `postTendency` — high values shift inside + mid up, 3PT down.
  - `foulDrawTendency` — high values increase free throw rate.
- **Per-player keep-rate** in `chooseShooter()`: now derived from the handler's `shootTendency` instead of a fixed 58%, allowing role players to pass more and stars to shoot more.
- **Behavioral tendency effects** in `computePossessionEventRates()`: drive/post/foul draw tendencies shift shot distribution and free throw rates after normalization.
- **`ARCHETYPE_SHOT_TENDENCIES`** — per-archetype defaults (e.g., scorer=80 shoot, rimProtector=25 shoot, playmaker=65 drive, bigFundamental=75 post).
- **`getShotTendencyForArchetype()`** — canonical lookup for archetype defaults.
- **UI tendency sliders** in `PlayerPlaystyleCard`: four new 0–100 sliders (Shoot vs Pass, Drive, Post-Up, Draw Fouls) with archetype-aware defaults.
- **Save migration** (v3 → v4): `shotProfile` on players automatically converts to `shotTendency` with neutral behavioral defaults (50/50/50/50).
- **12 regression tests** for tendency-driven behavior (shootTendency keep-rate, drive/post/foulDraw effects, archetype defaults).

### Changed
- **`chooseShooter()`**: replaces fixed `keepHandlerRate` with per-player formula derived from `shootTendency`.
- **`computePossessionEventRates()`**: blends tendency zone rates, then applies drive/post/foulDraw shifts after normalization.
- **`SimPlayerInput.shotProfile`** → `SimPlayerInput.shotTendency` (type change).
- **`Player.shotProfile`** → `Player.shotTendency` (with deprecated `shotProfile` kept for migration).
- **`PlayerPlaystyleCard`**: removed `rimAccuracy`/`midAccuracy`/`corner3Accuracy`/`above3Accuracy` sliders (were unused by sim).
- **Smoke test range**: 3PA share test widened from 35–42% to 35–46% due to tendency-driven shot variance.

### Deprecated
- `ShotProfile` type (kept for save-game migration).
- `getShotProfileForArchetype()` (aliases `getShotTendencyForArchetype`).
- `shotProfileToRates()` (aliases `shotTendencyToRates`).

---

## [2026-05-04] — Shop Player tab overhaul: rich cards, trade heat, section grouping

**Why:** Player list was unsorted (new acquisitions landed at array bottom), cards showed only name/position/OVR/salary (no avatar, age, contract status, or trade demand signal), and all styling was inline `style={{}}`. The tab felt sparse and colorless compared to the rest of the Trade Hub.

### Changed
- **Sort by OVR descending** — fixes "newly traded player at bottom" bug. Roster always ranked by talent.
- **Rich player cards** — each card now shows: `PlayerAvatar` (facesjs face), color-coded OVR badge (`getOvrHex`), age, contract years remaining, secondary positions, and contract status chips (Expiring/Rookie/NTC).
- **Trade heat indicator** — green dot (3+ offers), amber (1-2), red (0) pre-computed per player via `generateOffersForPlayer`. Shows market demand at a glance before clicking.
- **Section grouping** — roster split into "Core" (75+ OVR) and "Role players" (<75) with eyebrow headers.
- **Inline styles → Tailwind** — both `ShopPlayerTab.tsx` and `OfferCarousel.tsx` converted from `style={{}}` objects to Tailwind classes + design tokens.

---

## [2026-05-04] — Trade Hub refactor: consolidate directories + kill dead code

**Why:** Two directories (`components/trade/` with 13 files and `components/trade-hub/` with 3 files) served one feature. 11 of 13 old components were dead code (zero external imports). The "Build Trade" tab rendered an entire page component (`Trade.tsx`) with its own header, scroll container, and a "Propose Trade" button that navigated to itself (no-op). All styling used inline `style={{}}` objects instead of Tailwind.

### Removed
- **11 dead components** from `components/trade/`: FinderModeToggle, FinderOfferCard, InterestHeatBar, LeaguePlayerList, PlayerSearchPanel, PositionNeedBadges, CounterOfferPanel, OfferWhatIfPreview, RecentTradesPanel, TradePanel, TradeValueMeter, PickSelector (~2000 lines).
- **`src/ui/pages/Trade.tsx`** — replaced by `BuildTradeTab` component.

### Changed
- **Consolidated to single directory** — moved `TradeInbox` and `TradeInterruptionCard` from `components/trade/` into `components/trade-hub/`, deleted empty `trade/` dir.
- **`BuildTradeTab.tsx`** — new component replacing Trade.tsx page: stripped page header (parent provides tabs), converted inline styles → Tailwind, extracted `TeamPanel` sub-component to deduplicate SEND/RECEIVE columns.
- **"Propose Trade" button** — now calls `executeTrade` store action (was a self-navigation no-op), resets state on success, disabled when trade is illegal.

---

## [2026-05-04] — Shop Player: 6 trade improvements

**Why:** After fixing the zero-offers bug, the Shop Player tab needed deeper trade logic: under-cap teams couldn't absorb salary gaps, 2-for-1 packages were too narrow in dump mode, no explanation when offers were empty, no pick labels or quality grades in the UI, and no deadline urgency.

### Added
- **`diagnoseNoOffers()`** — exported engine function that explains *why* no offers came back (NTC, low OVR, salary block, negative trade value). Displayed in ShopPlayerTab when offers array is empty.
- **`gradeOffer()`** — exported engine function returning `'great' | 'fair' | 'slight overpay' | 'salary dump'` based on valueDelta. Shown as colored badge in OfferCarousel header.
- **`ShopPlayerOptions.currentDay`** — enables trade deadline pressure: when day ≥ 85, interest OVR thresholds drop by 5, age limits widen by 2, and proposal acceptance floor widens from -40 to -50.
- **OfferCarousel pick labels** — resolves pick IDs to human-readable labels ("2027 1st — LAL (top-5 protected)") using team draftPicks data.
- **Under-cap absorption sort** — under-cap buyers now sort candidates cheapest-first (cap room covers the gap) instead of salary-closeness, enabling trades where a $5M filler matches a $25M player via cap space.
- **Wider 2-for-1 in dump mode** — `buildTwoPlayerInitiatingPackage` maxValueGap parameterized (default 25, 60 in dump mode), allowing more 2-for-1 salary dump packages.
- **6 new tests** — deadline pressure, under-cap absorption, gradeOffer, diagnoseNoOffers (NTC, low OVR, salary block).

---

## [2026-05-04] — Shop Player fix + findOfferedPlayer tank/rebuild fix

**Why:** `generateOffersForPlayer` generated random trade proposals from each CPU team and then filtered for ones that happened to include the target player. Since each team only generated 3 proposals across all 29 teams, the chance of any targeting the specific shopped player was near zero — users got 0 offers for every player. Separately, `findOfferedPlayer` blanket-rejected all trades where the target team was tank/rebuild, meaning AI-to-AI trades could never target rebuilding teams.

### Fixed
- **`src/engine/tradeAi.ts` — `generateOffersForPlayer`** — rewrote from generic-then-filter to a targeted approach: for each CPU team, directly evaluate interest in the specific player, find salary-matchable return packages (single-player or 2-for-1), check CBA rules, and balance with picks. Now reliably produces offers when viable trades exist.
- **`src/engine/tradeAi.ts` — `findOfferedPlayer`** — tank/rebuild target teams now accept youth (age ≤ 26) or players on expiring contracts instead of blanket-rejecting everything. This also fixes AI-to-AI trade market where rebuilding teams were unreachable.

### Added
- **`src/engine/__tests__/tradeAi.test.ts`** — 9 regression tests for `generateOffersForPlayer`: viable offers, NTC guard, no-contract guard, tank interest in youth, contend skip low-OVR, tank skip old players, proposal structure, multi-team offers, sort order.

---

## [2026-05-03] — Rotation depth scaling bugfixes + UI cleanup

**Why:** Three bugs found via code review: (1) upward depth scaling loop had two flaws — gate `> 10` allowed transfers that left backups with 6 min (violating the ≥10 invariant), and `else break` aborted the entire loop on the first thin position rather than skipping that candidate. (2) Dead function `getPenaltyBorderColor` left in CourtView after a dot-refactor. (3) `onSimToDay` prop declared in `CalendarCellProps` but never used — passed from CalendarGrid and ScheduleListView unnecessarily.

### Fixed
- **`src/engine/positionRotation.ts`** — upward scaling: gate changed from `> 10` to `>= THIRD_TRANSFER + MIN_BACKUP_AFTER_TRANSFER` (≥ 15) so backup retains ≥ 10 min after transfer. `else break` replaced with `skippedCandidates` set — loop continues to next candidate instead of aborting.
- **`src/ui/components/team/CourtView.tsx`** — deleted dead `getPenaltyBorderColor` function (replaced by dot-color refactor in prior sprint).
- **`src/ui/components/schedule/CalendarCell.tsx`** — removed unused `onSimToDay` from `CalendarCellProps`.
- **`src/ui/components/schedule/CalendarGrid.tsx`**, **`ScheduleListView.tsx`** — removed `onSimToDay` from props and all call sites (full chain cleanup).
- **`src/ui/pages/Schedule.tsx`** — rank-10 label was "above play-in" but rank 10 IS the last play-in slot (7–10 = play-in). Split into 3 tiers: ranks 1–6 → "above play-in cut", ranks 7–10 → "in play-in · X up on #11", ranks 11+ → "X GB to play-in".
- **`src/ui/pages/PlayerComparison.tsx`** — search dropdown clipped by shadcn Card's `overflow-hidden`. Fixed with `overflow-visible` on the search Card + `z-[100]` on the dropdown.
- **`src/engine/__tests__/positionRotation.test.ts`** — `makeDepthPlayer` used wrong rating key `end` (should be `endu`) and was missing `fg` and `stl` keys. Cast hid the bug since tests used uniform OVR scaling.

### Added
- **`src/engine/__tests__/positionRotation.test.ts`** — 4 regression tests: backup ≥ 10 invariant on balanced roster, premature-break on uneven roster (depth=14 yields > 11 active), backup invariant on uneven roster, and correct rating keys.

---

## [2026-05-03] — Rotation auto-generate: slot penalty, scarcity ordering, philosophy sync

**Why:** `buildDefaultPositionRotation` ranked backup candidates by raw OVR, ignoring positional fit. A 6'1" guard with 82 OVR could be slotted at C backup over a 76 OVR natural center. Separately, Pass 2 processed positions alphabetically (C→PF→PG→SF→SG), so positions listed later could have their only eligible candidate "stolen" by an earlier position with many alternatives. Philosophy changes also didn't regenerate the rotation (depth changes did).

### Changed
- **`src/engine/positionRotation.ts`** — imported `computeSlotPenalty` from `positionPenalty.ts`. All 3 backup-selection sorts in `findBestFallback` and Pass 2 now rank by slot-adjusted OVR (`computeOvr - computeSlotPenalty`). Pass 2 backup ordering changed from alphabetical to scarcity-first (positions with fewer eligible unassigned candidates processed first). Removed dead `bestPos` null-check in Pass 3.
- **`src/ui/components/team/PositionRotationEditor.tsx`** — `handlePhilosophyChange` now calls `buildDefaultPositionRotation` (matching depth-change behavior).

### Added
- **`src/engine/__tests__/positionRotation.test.ts`** — 2 regression tests: (1) slot-adjusted backup selection verifies natural-fit player beats higher raw-OVR out-of-position player; (2) scarcity-ordering verifies sole C-eligible backup (PF|C) goes to C, not PF.

---

## [2026-05-01] — Dashboard visual unification to Schedule "cinematic dark" style

**Why:** Dashboard components used raw Tailwind opacity values (`bg-white/[0.02]`, `border-white/[0.06]`, `bg-white/[0.05]`) instead of the design system CSS tokens established by the Schedule page. This created visual inconsistency between the two most-used pages. Goal was to bring Dashboard into the same "cinematic dark" grammar — glass tokens, team-hue flat fills, SeasonArcBar integration.

### Changed
- **`src/ui/pages/Dashboard.tsx`** — replaced all raw `white/[0.0x]` opacity values with design system tokens (`var(--glass-bg)`, `var(--glass-border)`, `var(--ink-hair)`). Fixed gradient fills on the matchup banner win-prob column to flat `hsl(var(--team-hue) 38% 16% / 0.94)` solid fill matching CalendarCell upcoming. Added SeasonArcBar component above the matchup panel hero card (computed from `gameResults` + `schedule` via `arcDays` useMemo). All section footer "view all" borders, progress bars, hover states, and GlassCard hovers now use CSS variable tokens consistently.

### Added
- **SeasonArcBar on Dashboard** — surfaces the season arc visualization from the Schedule page on the Dashboard hero card. Computed inline from existing store data — no new store state required.

## [2026-04-30] — Saves UI: add sidebar SavePanel to gameplay

**Why:** Saves had no UI during gameplay — auto-save fired silently at ~30 trigger points with zero visual feedback. Players couldn't switch saves, rename, or delete without returning to the landing page. The game already has a 160px sidebar (NavRail), making it a natural home for persistent save management.

### Added
- **`src/ui/components/layout/SavePanel.tsx`** — collapsible saves section in the 160px left sidebar. Shows auto-save indicator (green dot + relative time, pulses on save), expandable save list with TeamDot + abbreviation + W-L record, inline rename (Enter/Escape), delete with confirmation, save switching (load + navigate), and "New Save" fork. Respects 160px width constraint.
- **`src/ui/components/layout/__tests__/SavePanel.test.tsx`** — 31 tests across 8 categories (rendering, expand/collapse, save items, load/rename/delete/new-save, auto-save indicator).
- **`src/ui/components/layout/__tests__/fixtures.ts`** — shared fixture data for SavePanel tests.

### Changed
- **`src/ui/components/layout/AppShell.tsx`** — integrated `<SavePanel />` between NavRail and team identity in the sidebar. Sidebar structure is now: Wordmark → hair → NavRail → hair → **SavePanel** → hair → Team identity. Landing page `SaveOverlay` modal stays as-is.

### Design rationale
Modal was the right pattern when saves were just "pick up where you left off" on the landing page. But Baseline has grown — saves are now a persistent concern during gameplay. The sidebar approach provides ambient auto-save confidence, in-game save management (rename/delete/switch/fork), and naturally fits the existing layout grammar without introducing a new interaction paradigm. Collapsed by default for immersion; one click reveals the full list.

## [2026-04-30] — Landing page refactor: extract components, add tests

**Why:** `Landing.tsx` had grown to 2,042 lines — 6x beyond the spec threshold of ~350 lines. Six spotlight sections (Engine, Trade, Draft, FA, Season, Dev) duplicated the same 2-column grid + scroll-reveal pattern. No tests existed. Inline styles everywhere violated UI conventions.

### Changed
- **Landing.tsx** — reduced from 2,042 → 323 lines (84%). Now imports 9 extracted components instead of defining them inline.
- **Extracted 12 new files** into `src/ui/components/landing/`:
  - `SpotlightSection.tsx` — shared 2-column grid layout with scroll-reveal, accent-driven card chrome, and render-prop children.
  - `EngineSpotlight.tsx`, `TradeSpotlight.tsx`, `DraftSpotlight.tsx`, `FreeAgencySpotlight.tsx`, `SeasonSpotlight.tsx`, `PlayerDevSpotlight.tsx` — individual spotlight sections.
  - `ClosingSection.tsx`, `SectionNav.tsx`, `SaveOverlay.tsx` — standalone components.
- **spotlight-card glass** — card container now uses `className="spotlight-card glass"` for backdrop-filter blur + diagonal glass shine (was missing before).
- **Dead code removed** — unused `phaseNote` function deleted.

### Added
- **`src/ui/data/landing.ts`** — 8 data constants extracted from Landing.tsx (NAV_SECTIONS, STATS, ROADMAP, STAT_ACCENTS, LOTTERY_ODDS, FA_BOARD, STANDINGS_PREVIEW, DEV_ARC).
- **`src/ui/hooks/useScrollReveal.ts`** — IntersectionObserver-based scroll-reveal hook extracted from Landing.tsx.
- **`src/ui/data/__tests__/landing.test.ts`** — 14 tests covering all data constants (structure, expected values, invariants).
- **`src/ui/components/landing/__tests__/Landing.test.tsx`** — 4 smoke tests verifying component exports are functions.

## [2026-04-29] — OpenCode compatibility: config, agents, and 18 command migrations

**Why:** Enable using OpenCode Go models (DeepSeek V4 Pro, Qwen 3.6 Plus) for orchestration and daily development. OpenCode natively reads `.claude/skills/` and `CLAUDE.md` but NOT `.claude/commands/`, so all 18 commands needed migration to `.opencode/commands/`.

### Added
- **`opencode.json`** — project config with `opencode-go/deepseek-v4-pro` default model, permissions, and CLAUDE.md instructions reference.
- **`.opencode/agents/` (8 files)** — custom subagents for the orchestrator pipeline: `orchestrator`, `task-writer`, `dev-worker` (qwen3.6-plus), `code-reviewer`, `researcher`, `security-reviewer`, `advisor-sonnet` (L1), `advisor-opus` (L2). Each with role-specific permissions and `steps` limits.
- **`.opencode/commands/` (18 files)** — all CLI commands migrated from `.claude/commands/` with translated tool calls (`Agent()` → `Task()`, `AskUserQuestion` → `question`), updated model references, and removed Claude-specific frontmatter (`name`, `allowed-tools`).

### Migration details
- **Skills**: No migration needed — OpenCode reads `.claude/skills/` and `.agents/skills/` natively.
- **Commands**: Frontmatter simplified (filename = command name), `model: claude-sonnet-4-6` → `model: opencode-go/deepseek-v4-pro`.
- **Orchestrator escalation**: L0 = `dev-worker` (qwen3.6-plus), L1 = `advisor-sonnet` (deepseek-v4-pro), L2 = `advisor-opus` (deepseek-v4-pro max thinking), L3 = human (`question` tool). No Opus-level model on Go, so L2 surfaces to human earlier than before.
- **Limitations**: No worktree isolation (mitigated by strict no-file-overlap rule), no per-call model selection (agent config handles it), visual verification (Step 4.6) unavailable without browser MCP.

---

## [2026-04-24] — Frontend redesign: screens, shell, and page restores

**Why:** Token foundation from 04-23 needed actual screen-level rewrites. Dashboard drifted from design spec (missing conference rank, NET rating, weekly delta, Today decisions). Shell (AppShell/NavRail/TopBar) was basic — needed keyboard-first nav, ⌘K palette, progress bar, static team identity. Rotation/Roster/PlayerProfile were gutted by the redesign commit and needed recovery from develop.

### Changed
- **Dashboard.tsx** — full spec alignment: "Good evening, GM" header with contender chip (NET rating), conference rank in record eyebrow, GB replacing payroll, weekly delta badge, PROJ spread on Next Tipoff, metric tiles (Payroll/Net Rtg/Roster Hlth/Morale), Today decisions section with trade inbox + low-morale + injury alerts via UrgeBar.
- **AppShell.tsx** — removed SeasonRing and TeamSwitcher; added top-edge season progress bar (CSS grid row, phase-aware gradient colors); static team identity block (TeamDot + city + W-L); global keyboard shortcuts (single-key nav D/S/T/L/R/O/C/P/X/F, N for sim, ⌘K for palette).
- **NavRail.tsx** — exported `NAV_GROUPS` for keybinding map; Link uses `cn('nav-item', active && 'active')` classNames.
- **TopBar.tsx** — dynamic season mood chip (early season/midseason/trade deadline/playoff push), Day counter, styled ⌘K chip, accent SIM button.
- **CommandPalette.tsx** — arrow key + Enter keyboard navigation, mouse hover selection, query-reset selection, ESC keyhint chip.
- **index.css** — added `.nav-rail`, `.nav-group`, `.nav-item` (+ hover/active/keyhint) rules ported from glass-screens.jsx GLASS_CSS.
- **Landing.tsx** — spotlight cards visual overhaul + lottery draw animation.
- **MainMenu.tsx** — team preview cascade animations.

### Restored from develop
- **Rotation.tsx** — user's polished version with tabs, court view, settings (152 lines → recovered from develop).
- **Roster.tsx** (1,012 lines), **PlayerProfile.tsx** (1,958 lines) — full pages recovered.
- **src/ui/components/team/** (12 files) — CapBar, CourtView, LineupAnalysisPanel, LineupSlotAssigner, PlayerPopover, PositionRotationEditor, RatingGradeTable, RosterTable, RotationHeader, SettingsTab, TimelineView, TrainingTab.
- **src/ui/components/player/** — AnimatedOvrCircle, PlayerAvatar, CareerAwardsTab, PlayerPlaystyleCard, and others.
- **src/ui/components/common/** — EmptyState, InterruptionModal, InjuryReportCard, BoxScore, and others.
- **src/ui/components/trade/** (14 files) — TradeInterruptionCard, TradeInbox, TradePanel, FinderOfferCard, CounterOfferPanel, etc.
- **src/ui/hooks/** — useLeague.ts, useToast.ts.

### Removed
- **TeamSwitcher** — orphaned, no longer imported. Static team identity block replaces it.
- **SeasonRing** — replaced by top-edge progress bar in CSS grid.

---

## [2026-04-23] — Frontend redesign: Direction 04 · Deep Space · Liquid Glass (design tokens + doc)

**Why:** Handoff bundle from Claude Design (`docs/baseline design/`) establishes a new visual direction — aurora backdrop, frosted-glass surfaces, team-hue rotation, Space Grotesk + Geist Mono — that supersedes the prior "Cinematic Dark / OKLCH / Bricolage" direction. This commit lays the token + utility foundation; screen-level rewrites follow.

### Changed
- **`DESIGN.md` replaced** with a 20-section Direction 04 rulebook covering density floor, typography, HSL team-hue color tokens, three glass surface tiers, aurora + starfield + noise backdrop, five named motion primitives (`auroraDrift`, `breathe`, `tilePulse`, parallax, stroke-draw), voice, keyboard, team identity, shadcn composition rules, and a screen review checklist.
- **`src/index.css` tokens** — `--bg-0`, `--ink`, `--team-hue`, `--accent`/`--accent-hi`/`--accent-2`, `--glass-bg[/-2/-border/-border-hi/-shine]`, `--ok`/`--warn`/`--bad`. Legacy `--color-*` tokens retargeted to the new palette so the 26 existing pages keep rendering. shadcn HSL tokens (`--background`, `--foreground`, `--primary`, `--ring`, etc.) remapped to the deep-space palette with team-hue-driven primary/ring.
- **`src/index.css` utilities** (+338 lines) — `.gframe`, `.aurora` (+`::after` noise overlay), `.stars`, `.glass[.hero.strong.sunken.flat]`, `.display`, `.mono`, `.eyebrow`, `.micro`, `.hair`, `.hatch`, `.urge.red|amber|green`, `.chip[.accent.ok.warn.bad]`, `.tile-pulse`. Keyframes: `auroraDrift` (18s), `breathe` (6s), `tilePulse` (4.5s). Reduced-motion guard disables all three.
- **Body font** → Space Grotesk Variable with `letter-spacing: -0.005em` and `font-feature-settings: 'tnum' 1`.
- **`tailwind.config.ts`** — `font-sans` → Space Grotesk, `font-mono` → Geist Mono. New color tokens: `bg-0`, `bg-1`, `ink{.dim,.faint,.hair}`, `accent-hi`, `accent-2`, `ok`, `warn`, `bad`.

### Added
- `@fontsource-variable/space-grotesk`, `@fontsource-variable/geist-mono` — imported in `src/main.tsx`.
- Ambient module declarations in `src/vite-env.d.ts`.

### Not yet done (follow-up work)
- React primitives (`AppShell`, `AuroraBG`, `TeamDot`, `GlassCard`, `Chip`, `Sparkline`, `UrgeBar`, `MetricTile`, `SeasonRing`, `CommandPalette`, `TeamSwitcher`) per DESIGN.md §15.
- Rewriting the six hero screens (Landing, Dashboard, Rotation, Trade, Player, Matchup).
- `ThemeProvider` + `--team-hue` wiring from favorite team.

---

## [2026-04-22] — Route-level code splitting via TanStack Router lazy routes

**Why:** Single 3.3 MB JS chunk (883 KB gzip) loaded all 26 pages upfront. Users landing on Dashboard paid the full bundle cost of Draft, Trade, Analytics, and FA UIs they may never visit. TanStack Router's `createLazyFileRoute` splits each page into its own chunk with zero runtime logic changes.

### Changed
- All 26 `src/routes/game/*.tsx` pages split into eager shell + `*.lazy.tsx` component file using `createLazyFileRoute`.
- `routeTree.gen.ts` regenerated — router now code-splits automatically at route boundaries.
- `contract-extension` keeps `validateSearch` in the eager file; only `component` moves to the lazy file.
- Initial bundle: **3,385 KB → 1,336 KB unminified** (gzip: **883 KB → 315 KB**) — **64% reduction**.
- Each page now ships as an independent cache unit (e.g. `trade.lazy-*.js`, `draft.lazy-*.js`).

---

## [2026-04-22] — Perf & hygiene: knip, bundle visualizer, fast-check, VirtualDataTable

**Why:** Three independent low-risk wins: (1) no dead-code detector or bundle analyzer existed; recharts shipped as barrel import defeating tree-shaking. (2) All 2,597 engine tests were example-based — cap/trade math invariants (125%+$100K, apron monotonicity) needed property coverage. (3) Players page rendered 450+ rows with no scroll limit.

### Added
- **`knip`** — unused exports/deps detector; `npm run knip`; CI job (non-blocking, `continue-on-error: true`).
- **`rollup-plugin-visualizer`** — bundle treemap via `npm run build:analyze`; stats.html gitignored; guarded by `ANALYZE=true` env flag.
- **`fast-check`** — property-based tests for engine cap/trade math (13 properties, 100 runs each):
  - Payroll non-negative, monotonic, two-way contracts excluded
  - Luxury tax = 0 below threshold, > 0 above
  - Apron status monotonic (once over threshold, can't drop back under)
  - Equal salary always matches trade rule
  - Bird max ≥ non-Bird max; non-Bird cap hold > 0
  - Trade value non-negative, higher OVR yields higher value
  - Cap hierarchy: salaryCap < luxuryTaxLine < firstApron < secondApron
- **`@tanstack/react-virtual`** — `VirtualDataTable` component (single-table, sticky header, padding-top/bottom virtualization); swapped onto `Players.tsx` (450+ rows → ~12 DOM rows rendered at any scroll position).
- **`src/engine/__tests__/arbitraries.ts`** — `arbPlayer`, `arbContract`, `arbTeam`, `arbSettings` generators for fast-check.

### Changed
- **`src/components/ui/chart.tsx`** — replaced `import * as RechartsPrimitive from 'recharts'` with granular named imports (`ResponsiveContainer`, `Tooltip`, `Legend` + 3 type imports). Enables tree-shaking.
- **`src/ui/pages/Players.tsx`** — `DataTable` → `VirtualDataTable` (drop-in, same props API).

### Design decisions
- `knip` non-blocking in CI initially; promote to blocking after one clean run.
- VirtualDataTable uses padding spacer rows (not absolute positioning) to maintain a single `<table>` — column widths align correctly with sticky `<thead>`.
- `arbTeam` stores player IDs in `roster` (not Player objects) per `Team.roster: string[]` contract.

---

## [2026-04-22] — Multi-team trade engine (3-team deals)

**Why:** Two-team trades are limiting for realistic roster construction. 3-team deals unlock more creative acquisitions and match real NBA front-office dynamics.

### Added
- **`src/types/transaction.ts`** — `TradeLeg` and `MultiTradePackage` interfaces. Circular execution model: `legs[i]` assets go to `legs[(i+1) % N]`.
- **`src/engine/trade.ts`** — `validateMultiTrade()`: validates ≥3-leg packages against CBA matching (125%+$100K per leg) and Stepien Rule per team.
- **`src/engine/tradeExecution.ts`** — `applyMultiTrade()`: pure function applying circular player/pick movements; creates TPE for uneven legs.
- **`src/engine/tradeAi.ts`** — `generateThreeTeamProposals()`: finds value-neutral 3-team swaps where all teams gain; tries both orderings per CPU pair; adds 2nd-round picks to bridge small gaps; returns top 3 by worst-leg value.
- **`src/engine/__tests__/tradeMultiTeam.test.ts`** — 11 integration tests covering the full validate → apply → generate pipeline.

### Design decisions
- All existing 2-team functions untouched — additive approach.
- `MultiTradePackage` types exported from `src/types/` (not engine modules) per project conventions.
- AI generation is engine-only; no UI surface in this PR.

---

## [2026-04-19] — Schedule page body refactor

**Why:** Calendar body was a visual outlier — opp-color glow failed on dark teams (CHA/MEM/ORL), empty cells wasted vertical space, game# counter duplicated SeasonArcBar info, and selected-game detail was a bare box score vs Trade-style rich panels elsewhere in the app.

### Added
- **`src/ui/utils/teamColorContrast.ts`** — `lightenForDarkBg()` + `hasLowContrastOnDark()` using WCAG luminance formula. Guarantees readable team colors on `#18181b` backgrounds.
- **`src/ui/components/schedule/MatchupDetailPanel.tsx`** — Rich matchup panel showing team badges, records, rest days, box score (played) or "Sim to this game" CTA (upcoming). Replaces bare `MatchupBoxScore` wrapper.
- **`src/ui/components/schedule/ScheduleListView.tsx`** — Vertical chronological list of user games, shown when List tab selected.

### Changed
- **`CalendarCell.tsx`** — Upcoming-game hero uses gradient team-badge pill (fixes CHA/MEM/ORL contrast). Removed `G{n}` counter (redundant with SeasonArcBar). Score font bumped to 14px. Clicking upcoming games opens detail panel instead of immediately simming.
- **`CalendarGrid.tsx`** — Empty cells collapsed from `min-h-[64px]` to `min-h-[28px]`. Added Grid | List view-switcher tabs matching Rotation's tab pattern. Month tabs hidden in List view.
- **`Schedule.tsx`** — `selectedGameResult` memo replaced with `selectedMatchup` (adds opp, isHome, restDays). Box-score wrapper replaced with `MatchupDetailPanel`.

### Preserved
`TeamSummaryBar`, `PlayoffPictureStrip`, `NextGameCard`, `SeasonArcBar`, month-tab pagination, W-L week column, GameDay banner.

---

## [2026-04-18] — shadcn adoption: Data-table pages (Issue #35)

### Added
- **`src/ui/components/common/DataTable.tsx`** — Generic `DataTable<T extends object>` component built on shadcn `Table` primitives. Supports column config with sort indicators (ChevronUp/Down/ChevronsUpDown), row click, custom row classNames, alignment, and empty state. Zero `any` types — typed column keys with narrowed fallback render.

### Changed
- **`src/ui/pages/Standings.tsx`** — Main standings table → `DataTable`. Raw `<table>` in stats breakdown detail view preserved (nested expansion pattern).
- **`src/ui/pages/Stats.tsx`** — Main player stats table → `DataTable`; category tabs → shadcn `Tabs`. `GameLogRow` raw table preserved (expandable rows not supported by DataTable).
- **`src/ui/pages/Players.tsx`** — Player list → `DataTable`; filter bar raw `<input>`/`<select>` → shadcn `Input`/`Select`; sort wiring added.
- **`src/ui/pages/Analytics.tsx`** — 3 raw tables → `DataTable`; `<select>` → shadcn `Select`; `TimelineCell` extracted as inline sub-component preserving all SVG/CSS contention window visuals.
- **`src/ui/pages/TransactionLog.tsx`** — Timeline list wrapped in shadcn `ScrollArea`.
- **`src/ui/pages/Schedule.tsx`** — Main content wrapped in shadcn `ScrollArea`; all domain visuals (`CalendarGrid`, `CalendarCell`, `TeamSummaryBar`, `BottomMetricsBar`, `LeagueTicker`, `StrengthOfSchedule`) preserved unchanged.

### Why
Establishes a single reusable DataTable pattern across all data-heavy pages — consistent sort UX, shadcn styling, accessible markup. Domain-specific visualizations (game log expansion, contention timeline, calendar) preserved as-is since they require layout patterns DataTable doesn't support. 2,828 tests passing, build clean.

---

## [2026-04-18] — Trade page UI polish + Stepien bug fix

### Changed
- **`src/ui/pages/Trade.tsx`** — Team partner selector replaced with color-coded abbreviation chip grid. Added sticky bottom summary bar showing send/receive chips, salary totals, validity badge, and submit button. Trade Impact section redesigned as aligned 3-column table (label | your team | their team) replacing cramped 2-column layout. Center empty state shows 3-step workflow illustration.
- **`src/ui/components/trade/TradePanel.tsx`** — Added `hideHeader` prop to suppress panel header when parent provides its own.

### Fixed
- **`src/ui/pages/Trade.tsx`** — Stepien Violation false positive on player-only trades. `checkStepieniRule` was called even when no picks were being sent, returning false for teams missing consecutive first-round picks. Now short-circuits: no picks sent = no violation.

### Why
UI improvements make the trade builder more visual and scannable — chip grid shows all teams at a glance, sticky bar keeps trade context visible while scrolling rosters. Stepien fix unblocks star-player trades (e.g. Luka Doncic) that don't involve picks. 2,828 tests passing, build clean.

---

## [2026-04-18] — shadcn adoption: Trade page (Issue #32)

### Changed
- **`src/ui/pages/Trade.tsx`** — Custom pill tab bar → shadcn `Tabs`/`TabsList`/`TabsTrigger` for Builder/Finder mode. Raw `<select>` for team partner → shadcn `Select`. Added `Sheet` slide-out for Trade Inbox (was previously unused). Inbox badge shows pending offer count.
- **`src/ui/components/trade/FinderModeToggle.tsx`** — Custom pill toggle → shadcn `Tabs` composition.
- **`src/ui/components/trade/PlayerSearchPanel.tsx`** — Raw `<input>` → shadcn `Input`; raw `<select>` → shadcn `Select`/`SelectTrigger`/`SelectValue`/`SelectContent`/`SelectItem`.
- **`src/ui/components/trade/CounterOfferPanel.tsx`** — Raw `<button>` → shadcn `Button variant="ghost"`.
- **`src/ui/components/trade/OfferWhatIfPreview.tsx`** — Raw toggle button → shadcn `Button variant="ghost"`.
- **`src/ui/components/trade/LeaguePlayerList.tsx`** — `max-h + overflow-y-auto` → shadcn `ScrollArea`.
- **`src/ui/components/trade/RecentTradesPanel.tsx`** — Custom filter toggle → shadcn `Tabs`.
- **`src/ui/components/trade/TradeInbox.tsx`** — Raw action buttons → shadcn `Button` (default/outline/ghost variants).
- **`src/ui/components/trade/TradeInterruptionCard.tsx`** — Raw action buttons → shadcn `Button` (gradient Accept, outline Counter, destructive Decline).
- **`src/ui/components/trade/FinderOfferCard.tsx`** — Raw accept button → shadcn `Button`.

### Preserved (out of scope per spec)
`TradeValueMeter`, `InterestHeatBar`, `PositionNeedBadges`, `OfferWhatIfPreview` domain visuals, court/player graphics.

### Why
Part of epic #29 — consolidate trade UI on shadcn/ui primitives (Issue #32). Removes parallel custom button/select/tab layers across 10 trade components. Trade Inbox now accessible via Sheet slide-out from the header. 2,828 tests passing, build clean.

---

## [2026-04-17] — shadcn adoption: Player Profile, Comparison & Rivalries (Issue #36)

### Changed
- **`src/ui/pages/PlayerProfile.tsx`** — Tab bar (overview/stats/ratings/analytics/awards) replaced with shadcn `Tabs`/`TabsList`/`TabsTrigger`/`TabsContent`. Hero header and 13 section containers replaced with shadcn `Card`. Compare button replaced with shadcn `Button`. `cn()` import migrated from `@/ui/utils/cn` to `@/lib/utils`.
- **`src/ui/pages/PlayerComparison.tsx`** — Section containers, badge spans, and layout chrome replaced with shadcn `Card` and `Badge` primitives.
- **`src/ui/pages/Rivalries.tsx`** — Section containers replaced with shadcn `Card`/`CardContent`; badge spans replaced with shadcn `Badge`.

### Preserved (out of scope per spec)
`PlayerAvatar` (facesjs SVG — not replaceable with shadcn Avatar), `StatCell`, `BoxScore`, `AnimatedOvrCircle`, domain rating visuals.

### Why
Part of epic #29 — consolidate app on shadcn/ui primitives (Issue #36). Removes parallel custom container layer across the player detail pages; tab structure now uses consistent shadcn Tabs composition. 2,828 tests still passing, build clean.

---

## [2026-04-17] — Roster & team quick-wins: render-setState bug + type-safe ContractOption

### Fixed
- **`src/ui/pages/Coaching.tsx`** — Replaced render-phase `setCurrentCoach` / `setLastUserTeamCoachId` pattern with a proper `useEffect`. The prior code mutated state during render, which React 18 tolerates in narrow shapes but can cause spurious extra renders and stale-closure bugs. Preserved original "init once per userTeamCoach id" semantic so firing a coach does not revive them.
- **`src/ui/components/team/RatingGradeTable.tsx`** — Removed `as string | null | undefined` cast on `player.contract.option`. The cast widened the `ContractOption` union and hid a bug: the code compared against `'eto'` but the union value is `'earlyTermination'`, so the ETO badge never rendered. Now compares against the correct enum member and benefits from exhaustive type narrowing.

### Why
Follow-up pass after shadcn refactor (#31) audit. Two small, surgical fixes — one render-time side effect, one dead code path that TypeScript could not catch through an unsafe cast. 2,828 tests still passing.

---

## [2026-04-17] — shadcn adoption: Roster & Team Management (Issue #31)

### Changed
- **`src/ui/pages/Roster.tsx`** — `SectionCard` → shadcn `Card`/`CardHeader`/`CardTitle`/`CardContent`.
- **`src/ui/pages/Rotation.tsx`** — Custom `<button>` tab bar → shadcn `Tabs`/`TabsList`/`TabsTrigger`/`TabsContent`.
- **`src/ui/pages/Development.tsx`** — Custom filter buttons and focus pills → shadcn `Tabs` + `Button` variants.
- **`src/ui/pages/Coaching.tsx`** — Custom card containers → shadcn `Card`; custom action buttons → shadcn `Button`; archetype badge spans → shadcn `Badge`.
- **`src/ui/components/team/RosterTable.tsx`** — Raw HTML `<table>`/`<thead>`/`<tbody>`/`<tr>`/`<th>`/`<td>` → shadcn `Table`/`TableHeader`/`TableBody`/`TableRow`/`TableHead`/`TableCell`.
- **`src/ui/components/team/SettingsTab.tsx`** — `<input type="range">` → shadcn `Slider` (value wrapped in array, extracted via `onValueChange`); custom toggle button → shadcn `Switch`.
- **`src/ui/components/team/TrainingTab.tsx`** — Custom focus selection buttons → shadcn `Button` with `variant` prop.
- **`src/ui/components/team/PlayerPopover.tsx`** — Internal custom buttons → shadcn `Button`; badge spans → shadcn `Badge`. `createPortal` positioning mechanism preserved.

### Preserved (out of scope per spec)
`CourtView`, `CapBar`, `PositionBadge`, `RatingGradeTable` internal visuals, `PhaseStepIndicator`.

### Why
Part of epic #29 — consolidate app on shadcn/ui primitives. Removes parallel custom primitive layer in these pages; improves accessibility and composition ergonomics. Closes #31.

---

## Older entries archived to docs/CHANGELOG-ARCHIVE.md

Entries before 2026-04-17 archived (includes 04-16 Playoff Awards page, 04-15 shadcn Awards/HoF, 04-14 Dashboard redesign + animations). See docs/CHANGELOG-ARCHIVE.md for full history.
