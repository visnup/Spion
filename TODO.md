# Todo

Delete items when they're done; the history is in git.

- Animate between circle and rectangle. `AnyShape` doesn't interpolate, so this probably
  wants one shape with an animatable corner radius (a rect whose radius goes to half the
  shorter side) plus a square frame in circle mode.
- Animate mirroring, if it can look good. A preview layer's `isVideoMirrored` snaps; a
  SwiftUI flip (`scaleEffect(x: -1)` or a `rotation3DEffect`) would animate. Iris flips the
  preview layer with a `CATransform3DMakeScale(-1, 1, 1)` transform instead of the connection.
- Research resizing and do it in the most idiomatic macOS way. Today it's AppKit's narrow
  borderless resize zone, a 4pt margin sized to roughly match it, and hand-set cursors.
  Look at how other borderless and shaped utility windows do it, whether SwiftUI or AppKit
  now has a resize affordance for plain windows, `NSPanel`, and how a titled window with a
  hidden title bar and buttons compares.
  - [Iris](https://github.com/ahmetb/Iris) (`Iris/Iris/CircularWindow.swift`), a circular
    camera overlay, does it all by hand, in about 250 lines: no `.resizable`, an 18pt ring
    inside the circle resizes, the rest moves with its own drag code. Resizing pins the
    opposite corner and sets a square frame from the mouse distance. Our hand-rolled
    attempt was janky; Iris disables implicit layer animations (`CATransaction`
    `setDisableActions`, `actions` set to `NSNull` on the preview layer) and calls
    `setFrame(_:display: false, animate: false)`, which may be the difference (untested).
- In circle mode, make the window square, or resize it to a square, so there's no dead
  transparent area beside the circle. Match the hover frame to the circle too. Iris keeps
  its window square by only ever setting square frames while resizing.
- Show over other apps' full-screen Spaces. Per Apple DTS this needs an `.accessory`
  activation policy and a non-activating `NSPanel`. It goes with moving to a menu bar extra,
  since accessory apps have no menu bar for these menus.
- If the saved frame is off-screen (a display was unplugged), check that it comes back on
  screen. Iris checks its saved origin against `NSScreen.screens` visible frames and centers
  the window when nothing intersects.
- Before sharing builds: a real bundle identifier (this resets camera permission), a README,
  and a signed, notarized release.
