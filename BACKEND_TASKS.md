# Kabadiwala Connect backend task plan

This plan separates the runnable UI prototype from the Firebase work needed for a safe, multi-user pilot. The app currently has Firebase client configuration and service code, but the dashboards and their demo records are local preview data. Do not treat the current preview as a production backend.

The client now uses Firebase phone verification consistently for browser and
native sign-in, then routes using the saved profile role. Firebase provider,
domain, live-profile, and privileged-role behavior still need staging/emulator
validation before this flow is production-ready.

## Delivery order

| When | Priority | Workstream | Exit condition |
|---|---|---|---|
| Phase 0 · before storing real user data | P0 | Firebase project, environments, identity, authorization | A test user can sign in; role access is server-authoritative; emulator rules tests pass. |
| Phase 1 · first backend sprint | P0 | Firestore model, lot creation, pricing, offline sync | A collector can create a lot offline, sync once online, and see district prices with source and freshness. |
| Phase 2 · second backend sprint | P0 | Matching, offers, pickup, handover, receipt | An authorized recycler can accept a matched lot and both parties can retrieve the same immutable receipt. |
| Phase 3 · pilot readiness | P1 | Admin review, ledger, storage, monitoring, operations | Admin actions are auditable; users can export receipts; security and failure paths are exercised in staging. |
| Later · after pilot evidence | P2 | AI, voice, push, analytics, scale tuning | Optional features have measured value and safe manual fallbacks. |

Dates should be assigned once the team and pilot districts are confirmed. Each phase is ordered by dependencies; work must not skip Phase 0 to reach a visual demo.

## Phase 0 — security and environments

| ID | Task | What it will do | How to implement | When / depends on | Done when |
|---|---|---|---|---|---|
| BE-001 | Create Firebase environments | Keep local development, staging, and production data separate. | Create/select Firebase projects; configure FlutterFire options per flavor; commit only non-secret client identifiers; document project aliases and emulator ports. | First; no dependency. | `dev`, `staging`, and `prod` builds target the intended project and cannot silently share data. |
| BE-002 | Use phone OTP consistently on every platform | Ensure sign-in succeeds only after Firebase Phone Auth verifies the user. | Use `verifyPhoneNumber` callbacks on Android/iOS and `signInWithPhoneNumber` with reCAPTCHA on Web; pass the Firebase SMS code (never a locally generated code or password); remove synthetic-user success fallbacks; load the saved profile and role after verification; surface provider/domain errors and retry state. Use Firebase test phone numbers for development. | First; BE-001. | Invalid, expired, and network-failed OTP attempts do not create a signed-in app session; browser domains and native apps are configured in Firebase Auth. |
| BE-003 | Establish trusted roles | Prevent a public user from selecting admin/recycler privileges in the client. | Store roles in server-controlled custom claims or a protected membership collection. Set/change them through a trusted callable/admin function. Resolve role after authentication and route by the verified role. | First; BE-001, BE-002. | Changing local app state or editing a profile document cannot promote a user. |
| BE-004 | Run Firebase emulators | Make rules and functions testable without production data. | Configure Auth, Firestore, Storage, and Functions emulators; add documented start/seed commands and deterministic fixture users. | First; BE-001. | A developer can run the stack locally and reset it to known data. |
| BE-005 | Rewrite and test Firestore/Storage rules | Enforce least privilege for private data and each transaction state. | Define ownership and role checks; separate public recycler summaries from private contact/profile data; restrict writes to validated fields; deny arbitrary object paths; add emulator tests for allowed and denied cases. | Before any real data; BE-003, BE-004. | Tests cover cross-user reads, role escalation, invalid transitions, oversized uploads, and unauthorized receipt edits; default is deny. |
| BE-006 | Add App Check and environment restrictions | Reduce abuse of public Firebase endpoints. | Configure App Check providers for supported app platforms; enforce it in staging after monitoring; restrict API keys by app and API; never place service-account credentials in Flutter. | Before pilot; BE-001, BE-004. | Staging rejects unverified clients and the release package contains no server credentials. |

## Phase 1 — data, prices, and offline lots

| ID | Task | What it will do | How to implement | When / depends on | Done when |
|---|---|---|---|---|---|
| BE-101 | Define Firestore schema and indexes | Store users, recycler profiles, price zones, lots, offers, handovers, and ledger entries consistently. | Publish a versioned schema document; use IDs and timestamps; include `state/district` partition fields; define composite indexes from actual queries; validate numeric ranges and enums in functions/rules. | Sprint 1; BE-005. | Schema, indexes, migration/seed approach, and sample documents are reviewed and checked in. |
| BE-102 | Build price board service | Return verified buying rates with unit, district, source, and update time. | Read only published rate documents; distinguish recycler-submitted, admin-reviewed, and stale values; cache last successful response locally. | Sprint 1; BE-101. | UI never presents a stale or unverified number as current; offline view names its sync time. |
| BE-103 | Persist lot drafts and sync queue | Let collectors record core lot data with no connection. | Use a local SQLite/Drift store; enqueue photo metadata, category, weight, coarse location, and client operation ID; retry idempotently with conflict status and visible sync state. | Sprint 1; BE-101. | Create/list/edit drafts works offline and reconnect does not duplicate a lot. |
| BE-104 | Implement secure lot creation | Publish a validated lot and matchable status. | Submit through a callable function or tightly scoped transaction; validate category/weight/owner; store coarse location; write server timestamp; keep immutable creation fields. | Sprint 1; BE-101, BE-103. | Only the owning collector can create/update allowed fields and lots become visible only under intended matching rules. |
| BE-105 | Secure photo upload | Attach compressed lot photos without public write access. | Compress on device; use authenticated Storage paths tied to UID/lot ID; validate size/type; store metadata; use short-lived authorized reads where appropriate. | Sprint 1; BE-003, BE-005, BE-104. | A user cannot overwrite another user’s media or upload arbitrary file types/size. |

## Phase 2 — offers, pickups, and verified handover

| ID | Task | What it will do | How to implement | When / depends on | Done when |
|---|---|---|---|---|---|
| BE-201 | Maintain verified recycler directory | Match collectors only to authorized facilities by accepted material and service zone. | Admin-curated recycler records with authorization ID, status, expiry, materials, zone, capacity, and pickup availability; expose a public-safe view separate from private contact data. | Sprint 2; BE-003, BE-101. | Expired/pending recyclers are excluded from the default match list. |
| BE-202 | Match lots and create offers | Let recyclers make traceable, comparable bids. | Cloud Function queries eligible verified recyclers, ranks by material/rate/distance/availability, creates offer documents with idempotency keys, and publishes status events. | Sprint 2; BE-102, BE-104, BE-201. | Collector sees up to three eligible offers with rate, distance, validity, and pickup window. |
| BE-203 | Enforce offer and pickup state machine | Stop conflicting accepts and invalid status jumps. | Callable functions own transitions such as `open → offered → accepted → scheduled → received`; use transactions/version checks and server timestamps. | Sprint 2; BE-202. | Concurrent accept attempts yield one winner; clients cannot write arbitrary status values. |
| BE-204 | Create immutable handover receipt | Capture custody details and recycler receipt confirmation. | On receipt, server generates a unique handover ref and records lot, parties, final weight/price, payment method/status, timestamp, and permitted coarse coordinates. Append corrections as events; do not overwrite the original receipt. | Sprint 2; BE-203. | Both authorized parties see the same reference and only the recycler can confirm receipt for their assigned lot. |
| BE-205 | Cash-first settlement and ledger | Record cash or optional digital settlement without requiring a wallet. | Write immutable ledger entries from successful handover/payment events; support paid/pending/partial and `cash`/`digital`; make retries idempotent. | Sprint 2; BE-204. | Cash completion needs no UPI setup; totals reconcile to handovers. |
| BE-206 | Realtime updates and notifications | Tell users when a relevant offer or pickup changes. | Listen only to owner/assigned-recycler scoped documents; send FCM from trusted Functions; keep polling-free UI fallback and notification preferences. | Sprint 2; BE-203. | No cross-user notification data leaks; disabling notifications is respected. |

## Phase 3 — admin, export, and pilot operations

| ID | Task | What it will do | How to implement | When / depends on | Done when |
|---|---|---|---|---|---|
| BE-301 | Admin queues and audit trail | Review recycler credentials, disputes, stale rates, and suspicious prices. | Build admin-only callable actions; record actor, action, before/after summary, reason, and timestamp in append-only audit events; require explicit confirmation in UI for decisions. | Pilot readiness; BE-003, BE-201, BE-204. | Admin actions are role-gated, reversible where possible, and queryable by audit reference. |
| BE-302 | Export and share receipts | Give both parties a saved proof of handover. | Generate a receipt from server-verified data; provide PDF/image download/share after authorization; avoid exposing unrelated contact or location details. | Pilot readiness; BE-204. | Receipt includes handover ref and verified facts and remains accessible offline after download. |
| BE-303 | Monitoring and recovery | Detect function failures, abusive traffic, and sync backlogs. | Add structured logs without unnecessary PII, error reporting, budget alerts, backups/export, retention policy, and an incident runbook. | Pilot readiness; BE-001, BE-004. | Staging alert test, restore rehearsal, and owner for each alert are documented. |
| BE-304 | Privacy and retention review | Limit personal data and set deletion/retention behavior. | Inventory fields and access; prefer district-level location; define retention by record type; review user export/deletion obligations with the project owner before pilot. | Pilot readiness; BE-101, BE-204. | Data map, retention schedule, and user-facing privacy notice are approved. |
| BE-305 | End-to-end staging verification | Prove the actual collector-to-recycler flow. | Seed test identities and verified facility; exercise offline create, sync, offer, accept, pickup, weigh-in, receipt, cash ledger, dispute, and admin review against emulators/staging. | Before field demo; BE-001–BE-304. | Repeatable test run passes on a representative Android device and a clean staging dataset. |

## Later enhancements

| Priority | Work | Implementation guardrail |
|---|---|---|
| P1 | AI material classification and abnormal-price flags | Call a protected backend endpoint; keep manual category/price override and show confidence/limitations. Never block lot creation when offline. |
| P1 | Hindi/Marathi text and spoken price guidance | Externalize strings, use bundled/offline-capable voice assets where possible, and test with intended users. |
| P2 | Learning mode, richer trends, chat, and advanced analytics | Add only after field research identifies the highest-value screens; keep core collector actions low-text. |

## Local developer sequence

1. Configure the Firebase emulator suite and seed test users before connecting UI screens to shared data.
2. Add repository/service interfaces so UI widgets do not call Firestore directly.
3. Implement one vertical slice at a time: offline lot → sync → verified match → offer → handover → receipt → ledger.
4. Add emulator rules/function tests for every new transition before staging deployment.
5. Use only staging identities and non-production data for the live demonstration until BE-305 passes.

