# Playing Without WebView2

The Windows player does not need a browser engine to decode video. `libmpv`
(`libmpv-2.dll`) opens the stream, decodes it with hardware acceleration, and renders
straight into a Win32 child window owned by the app. That is the same playback core that
powers mpv and, indirectly, most VLC-style players.

WebView2 is used for exactly one thing: hosting the HTML control overlay
(`src/desktopMain/resources/player-ui/controls.html` + `controls.js`). It draws the seek
bar, buttons and menus and sends the user's actions back to Kotlin.

## What happens when WebView2 is missing

The player probes the Evergreen WebView2 runtime with
`GetAvailableCoreWebView2BrowserVersionString` before creating a player. When the runtime
is absent, the player is created without the overlay and mpv's built-in on-screen
controller (`--osc=yes`) takes over. Playback, seeking, audio/subtitle track selection,
external subtitles, resume position and hardware decoding all keep working.

When the runtime is present, the player behaves exactly as before.

## Forcing native controls

Useful for testing, or on a machine where the runtime is installed but broken:

```
# JVM system property
-Dnuvio.player.nativeControls=true

# or environment variable
set NUVIO_PLAYER_NATIVE_CONTROLS=true
```

Set it back to `false` (or unset it) to return to the HTML overlay.

## What native controls give you

mpv's OSC inside the video window: play/pause, seek bar, volume, fullscreen toggle and
the standard mpv keyboard bindings.

| Key | Action |
| --- | --- |
| `Space` / `p` | Play / pause |
| `←` / `→` | Seek 5 s |
| `↑` / `↓` | Seek 60 s |
| `9` / `0` | Volume down / up |
| `m` | Mute |
| `f` | Fullscreen |
| `j` | Cycle subtitle track |
| `#` | Cycle audio track |
| `s` | Screenshot |

## What it does not give you

The HTML overlay is the app's own skin, so the following are unavailable in native
mode: the custom control layout, the in-player track and quality menus, the watch-party
overlay, and the webview-specific shortcuts (such as the `F11` accelerator). Track
selection falls back to mpv's cycling keys, and the app's own resume and progress
tracking still work because they poll mpv directly.

## Building

The native bridge is compiled from source, so a build needs MSVC (Visual Studio Build
Tools), a JDK 17+ toolchain, and the WebView2 SDK headers. The SDK is still required at
build time even when the runtime is absent at run time, because the same DLL contains
both code paths.
