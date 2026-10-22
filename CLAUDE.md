# Spion

A macOS app that shows the camera feed as a small floating overlay on the screen. That's the whole
scope. What's next is in `TODO.md`.

## Code

- Keep the code as small and tight as possible. This is a single-purpose app: it should do that
  one thing very well and nothing else. Reach for whatever macOS and SwiftUI already provide
  before writing code, and prefer deleting code to adding it.
- Latest Swift (language mode 6) with strict concurrency. Default actor isolation is `MainActor`.
- SwiftUI first, using the newest APIs the deployment target allows: scene modifiers such as
  `.windowStyle(.plain)`, `.windowLevel`, `.defaultWindowPlacement`, `WindowDragGesture`.
  Drop to AppKit only for what SwiftUI can't do yet, and say why where it happens.
- Commands go in menus, per the HIG, not behind hidden gestures like double-click.
- Prefer async/await and structured concurrency over GCD and completion handlers.
- macOS only (`SUPPORTED_PLATFORMS = macosx`), deployment target macOS 27.

## Layout

- `Spion/SpionApp.swift`: the single overlay `Window` scene.
- `Spion/OverlayView.swift`: the overlay, the camera-access check, and the camera view with
  its capture session. The camera, mirroring and shape are `@AppStorage`: shape from the View menu, the rest from the Camera menu.
- `Spion/Cameras.swift`: the camera list for the Camera menu, kept current as devices connect.
- `Spion/WindowBehavior.swift`: AppKit fixes for the borderless window, as a background view.
- `Spion/Icon.icon`: the Icon Composer app icon, set by `ASSETCATALOG_COMPILER_APPICON_NAME`.
  It's compiled by the Resources build phase, so keep that phase even when it looks empty.
- Sources are a file-system-synchronized group, so new files in `Spion/` join the target without
  editing `project.pbxproj`.
- The `.plain` window is borderless, so its style mask is `0`. `.windowResizeBehavior(.enabled)`
  on the overlay inserts `.resizable` (checked on macOS 27.0, 2026-10-22). `WindowBehavior` works
  around the rest of what borderless costs:
  - SwiftUI has no scene modifier for Spaces (none in the macOS 27.0 SDK either), so
    `collectionBehavior` gets `.canJoinAllSpaces`. It doesn't show over other apps' full-screen Spaces, and `.fullScreenAuxiliary` doesn't
    change that. Per Apple DTS that takes an `.accessory` activation policy (no Dock icon or
    menu bar) plus a non-activating `NSPanel`.
  - It can't become key, so `.pointerStyle` and cursor rects never show. An `.activeAlways`
    tracking area sets the `NSCursor.frameResize` cursors instead.
  - Clicks on fully transparent pixels fall through to the window behind. The margin around
    the video, corners included, has a 1% black fill so it catches them. Transparent corners
    break corner resizing.
  - AppKit's resize zone is only a few points wide and there's no public API to hand it a
    drag (`performDrag`/`WindowDragGesture` only move), so the 4pt `margin` is sized to
    roughly match it. Resizing the window by hand from mouse events was janky; don't.
- SwiftUI never restores the window with macOS's default "Close windows when quitting an
  application" setting, and resizes it on launch itself (to 900×450 without `.defaultSize`).
  `WindowBehavior` sets a frame autosave name one task after it gets its window, so the saved
  frame lands after SwiftUI's sizing. The window stays transparent until then, or it flashes
  at the default placement first. `defaultWindowPlacement` only applies before there's a
  saved frame.
- Camera access: `ENABLE_RESOURCE_ACCESS_CAMERA` (sandbox entitlement) and
  `INFOPLIST_KEY_NSCameraUsageDescription` in build settings.

## Before committing

Review the whole change, not just the last edit, for accretion: approaches tried and then
abandoned but still in the code, two mechanisms for one job, code that ended up in the wrong
place because it was convenient, and comments or this file describing an earlier approach.
Remove or fix them before the commit.

## Verify

```sh
xcodebuild -project Spion.xcodeproj -scheme Spion -destination 'platform=macOS' -derivedDataPath DerivedData/Spion build
open DerivedData/Spion/Build/Products/Debug/Spion.app
```

`DerivedData/Spion` is where Xcode itself builds (project-relative DerivedData), so both share
one build.
