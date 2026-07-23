# Background Sync with WorkManager

## Why WorkManager over the alternatives

| Tool | Use it when |
|---|---|
| **WorkManager (periodic)** | Default choice for "keep cached data fresh in the background." Doze/battery-optimization compliant, survives process death and reboot (if rescheduled), supports constraints. |
| `AlarmManager` (exact) | Only for genuinely time-critical, user-visible-at-a-specific-moment events (an alarm clock, a calendar reminder). Wrong tool for "refresh every N minutes" — fights the OS's battery management instead of cooperating with it. |
| Foreground Service | Only if sync must run continuously with user awareness (e.g. live GPS tracking during a workout). Periodic weather/price/status sync should never need this. |
| FCM push + on-receipt sync | Better than polling if your backend can push "data changed" events — pulls sync frequency down from "every hour just in case" to "only when something actually changed." Consider this if you control the backend and update frequency is unpredictable. |

## Minimum interval reality check

`PeriodicWorkRequest` has a **15-minute minimum** interval — WorkManager will silently clamp anything lower. If a feature genuinely needs tighter-than-15-min freshness while foregrounded, that's a job for a foreground `Flow`-based poll or FCM push, not `PeriodicWorkRequest` — don't fight the platform.

## Worker implementation

```kotlin
@HiltWorker
class SyncWorker @AssistedInject constructor(
    @Assisted context: Context,
    @Assisted params: WorkerParameters,
    private val repository: EntityRepository,
    private val alertEvaluator: AlertEvaluator,
    private val savedIdsProvider: SavedEntityIdsProvider
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result = coroutineScope {
        val ids = savedIdsProvider.getSavedIds()
        if (ids.isEmpty()) return@coroutineScope Result.success()

        // Fan out per-entity, but cap concurrency — don't blast N simultaneous requests
        // if the saved list can be large (e.g. many tracked cities/tickers)
        val outcomes = ids.chunked(5).flatMap { chunk ->
            chunk.map { id -> async { id to repository.refresh(id) } }.awaitAll()
        }

        val anyFailure = outcomes.any { (_, result) -> result.isFailure }

        outcomes.forEach { (id, result) ->
            if (result.isSuccess) {
                repository.getCached(id)?.let { entity ->
                    alertEvaluator.evaluate(entity)?.let { alert ->
                        NotificationHelper.showAlert(applicationContext, alert)
                    }
                }
            }
        }

        // Retry (with backoff) only if EVERY entity failed — partial failure shouldn't
        // trigger a full retry storm; log/skip individual failures instead
        if (anyFailure && outcomes.all { it.second.isFailure }) Result.retry() else Result.success()
    }
}
```

## Scheduling

```kotlin
val constraints = Constraints.Builder()
    .setRequiredNetworkType(NetworkType.CONNECTED)   // NOT .UNMETERED — see anti-pattern note
    .build()

val syncRequest = PeriodicWorkRequestBuilder<SyncWorker>(
    repeatInterval = 1, TimeUnit.HOURS     // set from Quick Decision Checklist item 1
).setConstraints(constraints)
 .setBackoffCriteria(BackoffPolicy.EXPONENTIAL, WorkRequest.MIN_BACKOFF_MILLIS, TimeUnit.MILLISECONDS)
 .build()

WorkManager.getInstance(context).enqueueUniquePeriodicWork(
    "entity_sync",
    ExistingPeriodicWorkPolicy.KEEP,   // KEEP = don't reset the schedule/lose progress on every app launch
                                        // REPLACE only if constraints/interval actually changed (e.g. user
                                        // changed a "sync frequency" setting)
    syncRequest
)
```

`KEEP` vs `REPLACE` is a common source of subtle bugs: calling `enqueueUniquePeriodicWork` with `REPLACE` on every `Application.onCreate()` resets the periodic timer on every app launch, meaning a user who opens the app frequently may never actually get a background sync between launches. Use `KEEP` for the steady-state case, and only issue a `REPLACE` call from the specific place where sync settings actually change.

## Surfacing sync status to the UI

Expose `WorkManager`'s state as a `Flow` so the ViewModel can `combine()` it with the DB flow (see main repository pattern) to show a subtle "syncing…" indicator without blocking:

```kotlin
fun observeSyncState(context: Context): Flow<SyncState> =
    WorkManager.getInstance(context)
        .getWorkInfosForUniqueWorkFlow("entity_sync")
        .map { infos ->
            when (infos.firstOrNull()?.state) {
                WorkInfo.State.RUNNING -> SyncState.Running
                WorkInfo.State.FAILED -> SyncState.Failed
                else -> SyncState.Idle
            }
        }
```

## Testing

Use `WorkManagerTestInitHelper` with a `SynchronousExecutor` in instrumented tests — never test periodic timing logic itself (WorkManager's scheduling is Google's problem, not yours); test that your `doWork()` correctly calls the repository, handles partial failure, and triggers alerts.

## Common mistakes

- Rescheduling with `REPLACE` on every app launch (see above) — silently prevents background sync from ever running.
- `NetworkType.UNMETERED` constraint when your target users are commonly on mobile-data-only connections — sync simply never fires for that population, with no visible error to debug.
- Doing all N entity refreshes as one giant unbounded `async` fan-out with no chunking — fine for 5 saved cities, a problem at 50+.
- Putting alert-notification logic inline in the worker with no debounce — re-fires the same notification every sync cycle while a condition persists (see `alerting-notifications.md`).
