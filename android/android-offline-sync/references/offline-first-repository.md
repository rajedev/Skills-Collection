# Offline-First Repository Pattern (DB as Source of Truth)

## The pattern

The Repository is the only class that talks to both DB and network. It never returns network data directly to the UI — it writes network data to DB, and the UI always reads from DB. This means the UI has exactly one thing to observe (a `Flow` from Room) instead of juggling network-loading-state and cache-state separately.

```kotlin
class EntityRepositoryImpl(
    private val dao: EntityDao,
    private val api: EntityApi,
    private val staleDurationMs: Long   // set per Quick Decision Checklist item 1
) : EntityRepository {

    override fun observe(id: String): Flow<Resource<EntityModel>> = channelFlow {
        // 1. Immediately emit whatever is cached — offline-first means never blocking on network
        val cached = dao.getById(id)
        cached?.let { send(Resource.Success(it.toModel(), isStale = isStale(it.timestamp))) }

        // 2. Kick a refresh if missing or stale — fire-and-forget into the same DB write path
        if (cached == null || isStale(cached.timestamp)) {
            launch { refresh(id) }
        }

        // 3. Stay subscribed to DB so any later write (from this refresh, from WorkManager,
        //    from a different screen) republishes to this collector automatically
        dao.observeById(id).collectLatest { entity ->
            entity?.let { send(Resource.Success(it.toModel(), isStale = isStale(it.timestamp))) }
        }
    }

    // The ONE method background sync, pull-to-refresh, and app-launch refresh all call.
    override suspend fun refresh(id: String): Result<Unit> = try {
        val fresh = api.fetch(id)
        dao.upsert(fresh.toEntity(timestamp = System.currentTimeMillis()))
        Result.success(Unit)
    } catch (e: IOException) {
        Result.failure(e)   // DB retains last-known-good; UI keeps showing it with isStale=true
    }

    private fun isStale(timestamp: Long) = System.currentTimeMillis() - timestamp > staleDurationMs
}
```

## Room schema notes

```kotlin
@Entity(tableName = "entity_cache")
data class EntityRow(
    @PrimaryKey val id: String,          // composite/domain key if multi-entity (see checklist item 2)
    val payload: String,                  // JSON blob if the shape is read-as-a-unit; normalize into
                                           // child tables only if you need partial queries across entities
    val timestamp: Long,                   // drives staleness — index this if you query "all stale rows"
)

@Dao
interface EntityDao {
    @Query("SELECT * FROM entity_cache WHERE id = :id")
    suspend fun getById(id: String): EntityRow?

    @Query("SELECT * FROM entity_cache WHERE id = :id")
    fun observeById(id: String): Flow<EntityRow?>

    @Upsert
    suspend fun upsert(row: EntityRow)

    // Run periodically (e.g. piggyback on the sync worker) to prevent unbounded cache growth
    @Query("DELETE FROM entity_cache WHERE timestamp < :expiryThreshold")
    suspend fun evictOlderThan(expiryThreshold: Long)
}
```

Blob vs normalized: default to a JSON blob column (kotlinx.serialization `@TypeConverter`) when the payload is always read/written as one unit for that entity. Normalize into child tables only when you need to query across entities (e.g. "all cities currently above 40°C" for a dashboard) — that's a real requirement, not a default.

## `Resource` wrapper

```kotlin
sealed class Resource<out T> {
    data class Success<T>(val data: T, val isStale: Boolean) : Resource<T>()
    data class Loading<T>(val cachedData: T?) : Resource<T>()   // still show cache while loading
    data class Error<T>(val cachedData: T?, val exception: Throwable) : Resource<T>()
}
```

Key discipline: `Loading` and `Error` both still carry the last-known cached value. An offline-first UI should almost never render a bare spinner or bare error screen if any cache exists — always render stale-but-present data alongside a small "last updated Xm ago" / "couldn't refresh, showing saved data" indicator.

## When to reach for a library instead

Hand-rolling the above is the right call for demonstrating understanding (portfolio projects, interviews) and for small-to-medium apps. For larger apps with many entity types repeating this exact pattern, `org.mobilenativefoundation.store:store5` formalizes `Fetcher` + `SourceOfTruth` + TTL and adds request de-duplication (concurrent calls to `observe(id)` for the same id share one in-flight network call instead of firing N times). Mention this as a "next iteration" note if hand-rolling for a portfolio piece — it signals you know the pattern well enough to know when not to reinvent it.

## Common mistakes

- Returning network data directly from a "refresh and return" method that bypasses the DB write — creates two sources of truth and desyncs the UI from what background sync later writes.
- Forgetting `collectLatest`/`distinctUntilChanged` on the DB flow — causes redundant recompositions on every identical re-emission from Room's invalidation tracker.
- Making `isStale` a one-time check at emission time instead of recomputing — if the UI holds a `Resource.Success` in state for a while (screen left in background), the staleness badge won't update without either a periodic recompute or recomputing on `onResume`.
