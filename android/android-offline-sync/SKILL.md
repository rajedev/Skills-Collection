---
name: android-offline-first-sync
description: Use this skill whenever building or reviewing an Android feature that involves offline-first data (DB as source of truth), periodic/background sync (WorkManager), cache staleness/TTL policy, or condition-based alerts/notifications (e.g. extreme weather, price drops, stock-outs, low battery). Trigger this for ANY Android app domain — weather, fitness, news, finance, IoT, delivery tracking — not just weather specifically. Also trigger when the user mentions "offline mode", "offline-first", "cache invalidation", "periodic sync", "background refresh", "stale data", "sync status", "WorkManager", or "alert system" in an Android/Kotlin context. This is a feature/capability skill (the pattern of HOW to build the behavior), not a tech-stack tutorial — pair it with whatever networking/DB/DI libraries the project already uses.
---

# Android Offline-First, Background Sync & Alerting

This skill covers three tightly-coupled capabilities that show up in almost every data-driven Android app: **offline-first caching**, **periodic background sync**, and **condition-based alerting**. They're grouped together because they share one architectural decision — the local DB is the single source of truth, and everything else (network, WorkManager, notifications) exists to keep that DB fresh and to react to what's in it.

Domain-agnostic by design: swap "weather" for "stock price", "flight status", "delivery ETA", "fitness metric" — the pattern doesn't change, only the entity and the alert condition do.

## When to go deeper

- Building the reactive cache/repository layer → read `references/offline-first-repository.md`
- Designing the periodic sync worker (scheduling, constraints, battery/network tradeoffs) → read `references/background-sync-workmanager.md`
- Designing condition-based alerts/notifications on top of synced data → read `references/alerting-notifications.md`

Read only the file(s) relevant to what the user is asking — don't load all three for a narrow question.

## Core mental model (always apply this first)

```
UI observes DB (Flow) ──never talks to network directly
        ▲
        │ writes
   Repository ── decides IF/WHEN to fetch, based on staleness policy
        │
        ▼
   Remote API (Retrofit/Ktor/whatever) ── only source of truth for "what's true right now"
                                           DB is only source of truth for "what UI shows"

Background Worker ── periodically forces the Repository's refresh path
                      (same code path as pull-to-refresh, just triggered by a scheduler, not a user gesture)

Alert Evaluator ── pure function over synced data → optional Notification
                    runs wherever fresh data lands (worker after sync, or DB write observer)
```

**The one rule that keeps all three capabilities consistent:** there must be exactly ONE code path that writes fresh data into the DB, and ONE code path that reads it out to the UI. Pull-to-refresh, WorkManager sync, and app-launch refresh should all call the *same* repository method — never let "background sync" become a parallel, slightly-different copy of "foreground refresh." That divergence is the #1 source of bugs in offline-first apps (stale UI after background sync, double-fetch races, notification firing on data the UI hasn't seen yet).

## Quick decision checklist

Ask these before writing any code — they determine which reference doc's advice applies:

1. **Staleness tolerance** — how old can displayed data be before it's "wrong enough" to force a refresh? (weather: 30–60 min; stock ticker: seconds; flight status: 1–5 min). This number drives your TTL constant and your WorkManager interval.
2. **Is the data multi-entity or single-entity?** Multiple cities/tickers/devices cached at once changes your Room schema (composite key) and your worker (loop per saved entity, not single fetch).
3. **Does staleness ever block the UI, or only badge it?** Offline-first apps should almost never block — show stale data with a "last updated" indicator, never a blank/loading screen if cache exists.
4. **Is the alert condition evaluated on every sync, or only on user-visible entities?** Evaluating alerts for entities the user isn't actively viewing (e.g. saved cities) is a background-worker responsibility, not a ViewModel responsibility — keep alert logic out of the UI layer entirely so it works even when the app is killed.
5. **What's the cost of a missed vs. duplicate sync?** Determines whether you need `ExistingPeriodicWorkPolicy.KEEP` vs `REPLACE`, and whether you need alert-debouncing (see alerting doc) to avoid re-notifying every sync cycle while a condition persists.

## Anti-patterns to flag if you see them in existing code

- ViewModel calling Retrofit directly "for the background refresh case" — breaks the single-write-path rule above.
- Using `AlarmManager`/`setExactAndAllowWhileIdle` for periodic sync that doesn't need exact timing — wastes battery, fights Doze, and is the wrong tool 95% of the time. Reserve exact alarms for genuinely time-critical events (a scheduled alarm clock, not "refresh every hour").
- No TTL/staleness field on cached rows — means every app launch either always refetches (defeats offline-first) or never refetches (shows permanently stale data).
- Alert evaluation living in Composable/ViewModel scope — means alerts stop firing the moment the user backgrounds or kills the app, which defeats the purpose of a background alert system.
- `NetworkType.UNMETERED` constraint on sync workers for apps expected to run on mobile-data-only user bases — silently starves sync for a large chunk of real-world users (relevant for e.g. Indian, or generally price-sensitive mobile markets).
