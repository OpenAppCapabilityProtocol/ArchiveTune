# OACP Integration -- ArchiveTune

How voice control was added to ArchiveTune using the OACP Kotlin SDK.

## Overview

Seven capabilities exposed via OACP:

| Capability | Dispatch | What it does |
|-----------|----------|-------------|
| `play_music` | **activity** | Search and play a song by name/artist (opens app) |
| `pause_music` | **broadcast** | Pause current playback (background) |
| `next_track` | **broadcast** | Skip to next track (background) |
| `previous_track` | **broadcast** | Go to previous track (background) |
| `toggle_shuffle` | **broadcast** | Toggle shuffle mode (background) |
| `toggle_repeat` | **broadcast** | Cycle repeat mode (background) |
| `toggle_like` | **broadcast** | Like/unlike current song (background) |

## Architecture

This app demonstrates the **hybrid** pattern. Only `play_music` needs the app to open (to search and display results). All other playback controls work in the background via MediaController without opening the app.

The `OacpActionReceiver` connects to the active MediaSession to control playback. The `play_music` action is handled by MainActivity which receives the intent and performs a music search.

## Files added

| File | Purpose |
|------|---------|
| `app/libs/oacp-android-release.aar` | OACP Kotlin SDK |
| `app/src/main/assets/oacp.json` | Capability manifest with 7 capabilities |
| `app/src/main/assets/OACP.md` | LLM context (streaming vs local, disambiguation) |
| `app/src/main/kotlin/.../oacp/OacpActionReceiver.kt` | Broadcast handler for playback controls |

## Files modified

### `app/build.gradle.kts`
```kotlin
implementation(files("libs/oacp-android-release.aar"))
```

### `AndroidManifest.xml`

Activity intent filter for play_music:
```xml
<intent-filter>
    <action android:name="${applicationId}.oacp.ACTION_PLAY_MUSIC" />
    <category android:name="android.intent.category.DEFAULT" />
</intent-filter>
```

Receiver for background playback controls:
```xml
<receiver android:name=".oacp.OacpActionReceiver" android:exported="true">
    <intent-filter>
        <action android:name="${applicationId}.oacp.ACTION_PAUSE_MUSIC" />
        <action android:name="${applicationId}.oacp.ACTION_NEXT_TRACK" />
        <action android:name="${applicationId}.oacp.ACTION_PREVIOUS_TRACK" />
        <action android:name="${applicationId}.oacp.ACTION_TOGGLE_SHUFFLE" />
        <action android:name="${applicationId}.oacp.ACTION_TOGGLE_REPEAT" />
        <action android:name="${applicationId}.oacp.ACTION_TOGGLE_LIKE" />
    </intent-filter>
</receiver>
```

## Testing

```bash
# Play a song (opens app, searches)
adb shell "am start -a moe.koiverse.archivetune.debug.oacp.ACTION_PLAY_MUSIC --es EXTRA_QUERY 'Lonely' --es EXTRA_ARTIST 'Akon' -p moe.koiverse.archivetune.debug"

# Pause (background, no UI)
adb shell am broadcast -a moe.koiverse.archivetune.debug.oacp.ACTION_PAUSE_MUSIC -p moe.koiverse.archivetune.debug

# Next track (background)
adb shell am broadcast -a moe.koiverse.archivetune.debug.oacp.ACTION_NEXT_TRACK -p moe.koiverse.archivetune.debug

# Toggle shuffle (background)
adb shell am broadcast -a moe.koiverse.archivetune.debug.oacp.ACTION_TOGGLE_SHUFFLE -p moe.koiverse.archivetune.debug
```

For release builds, replace `moe.koiverse.archivetune.debug` with `moe.koiverse.archivetune`.
