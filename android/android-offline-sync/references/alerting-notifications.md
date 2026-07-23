# Condition-Based Alerting & Notifications

## The pattern

Alert logic is a **pure function** over synced data: `(EntityModel) -> Alert?`. It has no dependency on Android framework classes, which makes it trivially unit-testable and reusable whether it's invoked from a background worker, a DB-write observer, or a push-notification payload handler.

```kotlin
class AlertEvaluator {
    fun evaluate(entity: EntityModel): Alert? = when {
        entity.value > CRITICAL_HIGH_THRESHOLD -> Alert.CriticalHigh(entity.value)
        entity.value < CRITICAL_LOW_THRESHOLD -> Alert.CriticalLow(entity.value)
        entity.statusFlag == "severe" -> Alert.SevereCondition
        else -> null
    }
}

sealed class Alert {
    data class CriticalHigh(val value: Double) : Alert()
    data class CriticalLow(val value: Double) : Alert()
    object SevereCondition : Alert()
}
```

Swap in your domain's real thresholds — extreme temperature, price drop %, stock-out flag, delivery delay minutes, whatever `entity` represents.

## Debouncing — the part people forget

Without debouncing, a condition that persists across multiple sync cycles (e.g. a multi-hour heatwave, a multi-day price dip) re-fires the identical notification every single cycle. Store the last-alerted timestamp/value per entity and only notify on a **transition**, or after a cooldown window:

```kotlin
class DebouncedAlertDispatcher(
    private val alertStateDao: AlertStateDao,   // small Room table: id, lastAlertType, lastAlertedAt
    private val cooldown: Duration = Duration.ofHours(6)
) {
    suspend fun dispatchIfNeeded(id: String, alert: Alert, notify: (Alert) -> Unit) {
        val last = alertStateDao.get(id)
        val sameAlertType = last?.alertType == alert::class.simpleName
        val withinCooldown = last != null &&
            (System.currentTimeMillis() - last.lastAlertedAt) < cooldown.toMillis()

        if (sameAlertType && withinCooldown) return   // suppress repeat

        notify(alert)
        alertStateDao.upsert(AlertStateRow(id, alert::class.simpleName!!, System.currentTimeMillis()))
    }
}
```

This also naturally handles the **resolution case**: when a fresh sync evaluates to `null` (condition cleared), clear the stored alert state so the next occurrence isn't treated as "within cooldown" of a stale, already-resolved alert.

## Notification delivery

```kotlin
object NotificationHelper {
    private const val CHANNEL_ID = "critical_alerts"

    fun ensureChannel(context: Context) {
        val channel = NotificationChannel(
            CHANNEL_ID, "Critical Alerts", NotificationManager.IMPORTANCE_HIGH
        ).apply { description = "Time-sensitive condition alerts" }
        context.getSystemService(NotificationManager::class.java).createNotificationChannel(channel)
    }

    fun showAlert(context: Context, alert: Alert) {
        val (title, text) = alert.toDisplayText()   // map sealed subtype -> user-facing copy
        val notification = NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_alert)
            .setContentTitle(title)
            .setContentText(text)
            .setPriority(NotificationCompat.PRIORITY_HIGH)
            .setAutoCancel(true)
            .build()

        NotificationManagerCompat.from(context).notify(alert.hashCode(), notification)
    }
}
```

`IMPORTANCE_HIGH` is appropriate for genuinely critical/time-sensitive alerts only — don't use it for routine "sync completed" style notifications, which should either not notify at all or use a low-importance silent channel.

## Where evaluation should live

Run the evaluator from the background worker (right after each entity's `repository.refresh()` succeeds — see `background-sync-workmanager.md`), **not** from ViewModel/Composable scope. Alert logic tied to UI lifecycle stops working the moment the user backgrounds or force-kills the app, which defeats the entire purpose of a background alert system. The worker is the only component guaranteed to run regardless of UI state.

## Permissions (Android 13+)

`POST_NOTIFICATIONS` is a runtime permission from API 33 onward — request it at a sensible moment (after the user opts into "alerts" in a settings screen, not on cold app launch) and handle graceful degradation if denied: alert evaluation and in-app badging can still work, just skip the OS notification.

## Common mistakes

- No debounce — identical notification fires every sync cycle for a persisting condition.
- No "resolution" clear — after a condition ends, stale alert-state can suppress the *next distinct* occurrence if the debounce key isn't cleared.
- Evaluating alerts in the UI layer — silently stops working when the app isn't foregrounded.
- Using `IMPORTANCE_HIGH`/heads-up notifications for non-critical routine updates — trains users to disable notifications for the app entirely.
