---
name: android-location-services
description: Use this skill whenever building or reviewing an Android feature that needs the device's current location — permission requesting, FusedLocationProviderClient usage, reverse geocoding (coordinates to place name), or graceful fallback when location is denied/unavailable. Trigger for ANY domain that uses "current location" as one input among others — weather, delivery/ride tracking, local search, fitness route logging, store locators — not just navigation apps. Also trigger on mentions of "use my location", "current location", "GPS", "FusedLocationProviderClient", "reverse geocode", "location permission", or "location not available" in an Android/Kotlin context. This is a feature/capability skill (the pattern for HOW to request, fetch, resolve, and degrade), not a tech-stack tutorial — pair it with whatever DI/architecture the project already uses.
---

# Android Location Services: Request, Fetch, Resolve, Degrade

Location is almost never the *only* way into a feature — it's one convenience path alongside a manual alternative (search, saved list, typed address). Every location feature in this skill's scope should be designed so the manual path works with zero location permission granted, ever. Location makes the app faster/more convenient; it should never be a hard dependency for core functionality.

## Core mental model

```
Cold start / feature entry point
        │
        ▼
  Explicit choice: "Use current location" vs. manual alternative
        │ (never auto-fire the permission dialog on app launch)
        ▼
  User taps "Use current location"
        │
        ▼
  checkSelfPermission() ── re-checked EVERY time, never cached as app state
        │
        ├── not granted → request → user denies → land on manual path
        │                          → user grants → continue below
        │
        ▼
  One-shot getCurrentLocation() (suspend, not a continuous stream)
        │
        ├── fails (GPS off / no fix / timeout) → inline error: Retry / Use manual path
        │                                          (show any cached prior result underneath, don't blank the screen)
        │
        ▼
  Reverse geocode lat/lon → place name (with fallback chain, see below)
        │
        ▼
  Feed resolved place into the same data path manual entry would have used
  (one downstream code path regardless of how the place was chosen)
```

**The one rule that keeps this consistent:** whatever the manual/search path ultimately produces (a place ID, a city name, an address) is exactly what the location path must also produce, and both must feed the same downstream function. Don't let "location-based lookup" become a parallel code path with its own data shape — that's how you end up with subtly different behavior depending on how the user got there.

## Permission requesting — Compose pattern

```kotlin
@Composable
fun LocationPermissionHandler(
    onGranted: () -> Unit,
    onDenied: () -> Unit
) {
    val context = LocalContext.current
    val launcher = rememberLauncherForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { result ->
        val granted = result[Manifest.permission.ACCESS_FINE_LOCATION] == true ||
                      result[Manifest.permission.ACCESS_COARSE_LOCATION] == true
        if (granted) onGranted() else onDenied()
    }

    LaunchedEffect(Unit) {
        val alreadyGranted = ContextCompat.checkSelfPermission(
            context, Manifest.permission.ACCESS_FINE_LOCATION
        ) == PackageManager.PERMISSION_GRANTED

        if (alreadyGranted) onGranted()
        else launcher.launch(arrayOf(
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.ACCESS_COARSE_LOCATION
        ))
    }
}
```

Only invoke this composable from an explicit user action (tapping "Use current location"), never from a screen's initial composition — that's what causes the "permission dialog fires the instant the app opens" pattern users dislike.

**Coarse vs. fine**: default to requesting both and accepting either. Most "current location" features (weather, nearby search, city-level personalization) only need coarse (city/neighborhood-level) accuracy — request fine only when the feature genuinely needs it (turn-by-turn navigation, precise geofencing). Coarse-only reduces the permission's perceived invasiveness and improves grant rates.

## Fetching location — wrapped as a one-shot suspend call

Never let `FusedLocationProviderClient` leak into a ViewModel directly — wrap it behind an interface like any other data source:

```kotlin
interface LocationProvider {
    suspend fun getCurrentLocation(): Result<LatLng>
}

class FusedLocationProviderImpl(private val context: Context) : LocationProvider {
    private val client = LocationServices.getFusedLocationProviderClient(context)

    @SuppressLint("MissingPermission") // caller must have already confirmed permission
    override suspend fun getCurrentLocation(): Result<LatLng> = suspendCancellableCoroutine { cont ->
        val cancelSource = CancellationTokenSource()
        client.getCurrentLocation(Priority.PRIORITY_BALANCED_POWER_ACCURACY, cancelSource.token)
            .addOnSuccessListener { loc ->
                if (loc != null) cont.resume(Result.success(LatLng(loc.latitude, loc.longitude)))
                else cont.resume(Result.failure(LocationUnavailableException()))
            }
            .addOnFailureListener { e -> cont.resume(Result.failure(e)) }
        cont.invokeOnCancellation { cancelSource.cancel() }
    }
}
```

`PRIORITY_BALANCED_POWER_ACCURACY` over `PRIORITY_HIGH_ACCURACY` for the large majority of "current location as one input" features — balanced priority favors WiFi/cell-tower fixes over GPS, which is faster indoors and materially better for battery. Reserve `HIGH_ACCURACY` for features where meter-level precision changes the outcome (navigation, geofencing), not for "which city/area am I roughly in."

This is a **one-shot fetch**, not a location stream — `getCurrentLocation()`, not `requestLocationUpdates()`. Don't set up continuous tracking for a feature that only needs "where am I right now" once per user action; continuous tracking has different permission tiers (background location), different battery cost, and different privacy disclosure obligations (Play Store data-safety section, background-location justification in review).

## Reverse geocoding — with a real fallback chain

```kotlin
class AndroidGeocoderService(private val context: Context) : GeocodingService {
    override suspend fun resolvePlaceName(latLng: LatLng): Result<String> =
        withContext(Dispatchers.IO) {
            try {
                val geocoder = Geocoder(context, Locale.getDefault())
                val address = geocoder.getFromLocation(latLng.lat, latLng.lng, 1)?.firstOrNull()
                val name = address?.locality
                    ?: address?.subAdminArea   // district/county — needed for smaller towns not tagged as "locality"
                    ?: address?.adminArea      // state/province — last resort before giving up
                name?.let { Result.success(it) } ?: Result.failure(GeocodingFailedException())
            } catch (e: IOException) {
                Result.failure(e)  // Geocoder throws IOException when its backend service is unavailable —
                                     // common on non-Google/AOSP-based devices and some OEM Android skins
            }
        }
}
```

**Known gotcha**: Android's on-device `Geocoder` depends on a backend service that isn't reliably present across all OEMs — plan for `IOException` as an expected, not exceptional, outcome, and always have the fallback chain above rather than assuming `locality` is populated. If the app already talks to a domain-specific geocoding API (a maps provider, a weather provider's own geocoding endpoint, etc.), prefer normalizing through that API's canonical result instead of raw `Geocoder` output — it keeps place identifiers consistent with whatever the manual-search path also produces (see "Core mental model" above).

## No-location fallback flow (the part usually skipped)

Handle each of these as a **distinct** case — collapsing them into one generic "location error" loses the ability to give the user the right next action:

| Situation | User-facing handling |
|---|---|
| Cold start / feature entry, no permission decision yet | Explicit choice screen: "Use current location" / manual alternative. Never auto-request. |
| Permission denied | Land on the manual path (must be fully functional alone). Small dismissible banner offering to enable location later via settings — don't re-prompt repeatedly. |
| Permission granted, but the fix itself fails (GPS off, no signal, timeout) | Inline error with **Retry** / **Use manual path** actions. If prior data exists for a previously-resolved location, show it under the error rather than blanking the screen. |
| Permission was granted earlier, revoked later (system settings, between sessions) | Re-check `checkSelfPermission` on every invocation — never trust a cached "granted" boolean in app state. |
| User declined once, don't want to nag every session | Persist a one-time `hasSeenLocationOnboarding` flag; surface "enable location" as a settings toggle instead of a recurring prompt. |

## Common mistakes

- Requesting permission on `Activity`/screen launch instead of on explicit user action — the single biggest cause of permission-dialog fatigue complaints.
- Using `requestLocationUpdates()` (continuous) when the feature only ever needs a one-shot fix — unnecessary battery cost and background-location review/disclosure burden.
- `PRIORITY_HIGH_ACCURACY` by default — slower indoors, worse battery, rarely actually needed for "roughly where is the user."
- Trusting `Geocoder.getFromLocation()` to always return a populated `locality` — smaller towns/rural areas frequently need the `subAdminArea`/`adminArea` fallback.
- Caching "permission granted" as a boolean in app state instead of re-checking — breaks the moment a user revokes permission from system settings without reopening the app.
- Building the location path and the manual/search path as separate features with different output shapes — creates two slightly different behaviors depending on entry point instead of one consistent downstream flow.
