# Desktop changes

This is my `desktop` branch for Mango. Implementation changes and their commits
live in this repository. My configuration lives in `~/.config/mango/`.

## Building

With Mango's build dependencies installed, clone the public `desktop` branch
and use the native Meson build:

```sh
git clone --branch desktop https://github.com/jdujava/mango-jd.git
cd mango-jd
meson setup build --prefix=/usr/local --bindir=bin --wrap-mode=nofallback
meson compile -C build
sudo meson install -C build --no-rebuild --tags runtime
```

The runtime tag installs Mango, mmsg and mangonag in `/usr/local/bin` without
installing configuration files or systemd services. The new compositor takes
effect when you next start Mango.

For an Arch package, the standalone
[AUR-jdujava](https://github.com/jdujava/AUR-jdujava) repository contains the
canonical `mango-jd-git` recipe. From a separate directory:

```sh
git clone https://github.com/jdujava/AUR-jdujava.git
cd AUR-jdujava/mango-jd-git
makepkg -si
```

## Focus and window order

- Closing a window uses Mango's native focus-history fallback instead of
  choosing a neighbor in layout order. Closing a background window keeps focus.
- Pointer warping moves the cursor only if it is outside the focused window
  after the layout settles. The policy is in `src/input/pointer.c`.
- `exchange_stack_client,next,move` pushes a tiled window down the stack;
  `prev,move` pushes it up. Floating neighbors are skipped, and the ends wrap
  like dwm. Omitting `move` keeps Mango's original swap behavior.
- `tagmon,next,silent` sends a window to another monitor and keeps your view
  and focus where they are. Omitting `silent` follows the window.
- Moving a tiled window to another monitor clears fullscreen or maximized
  windows on its destination tag, just like a tag move. This also covers hidden
  destination tags. Other monitors and tags keep their fullscreen windows.
- `focusstack` stays on a real fullscreen window. Explicit tag changes,
  monitor changes and application activation requests still work.
- Application activation respects `focus_on_activate` and silent-window rules,
  including scratchpads and the special workspace. Focusing an already visible
  window does not replace the current tag view.
- Swallowing a terminal on a hidden tag keeps your current tag and focus.
- Application-owned XWayland popups keep their pointer coordinates current
  while moving, so fast drags do not overshoot the pointer.
- Entering an XWayland window also synchronizes its button device. Clicks then
  use the cursor's current position instead of repeating its previous movement.

Utility-window rules can opt out of pointer warping with `no_warp:1`, or
out of `focusstack` cycling with `skip_focusstack:1`. These are independent;
the window can still be clicked and explicitly focused. Native `no_focus:1`
prevents keyboard focus entirely. Cycling skips those clients too.

Shortcut keys belong to the compositor until released. A focus-changing
shortcut does not pass its held key or its release to the application.
Explicit `bindp` bindings still pass the key through.

Floating native Wayland windows can also be moved and resized with their
own controls. Tiled windows keep their layout.

Native fullscreen windows keep their fullscreen state and size from the first
configure through mapping and monitor assignment, without a floating-size dip.

## Scrolling

Holding Alt multiplies mouse and touchpad scrolling by four. Normal scrolling
still uses `axis_scroll_factor` and `trackpad_scroll_factor`.

`scroll_boost_modifier=ALT` and `scroll_boost_factor=4.0` customize this behavior;
`NONE` disables it. The factor accepts values from 1 to 10. Modifier combinations
must match exactly, and ordinary axis bindings take precedence.

The modifier is still passed to applications. Alt avoids replacing their usual
Shift+wheel horizontal scrolling and Ctrl+wheel zooming.

## Fake fullscreen

`is_fake_fullscreen:1` is a per-window preference, like in my dwm build. An
application requesting fullscreen stays within its tiled or floating area
when that preference is enabled; otherwise it gets real fullscreen.

`togglefakefullscreen` changes the preference. If the application is already
fullscreen, it also switches between fake and real fullscreen immediately.
The IPC field `is_fullscreen_requested` tracks the application's request;
`is_fullscreen` reports whether the window occupies the whole screen.

Use an ordinary `window_rule` to preset every matching window.
`window_rule_once` applies only to the first matching window in the session.

## Reviewing and carrying the changes

`desktop` is a small topical patch series over the official `upstream/main`.
Keep one coherent commit per customization, folding follow-up fixes into that
feature, with documentation and maintenance in the final commit. The public fork
is `origin`; fetch new upstream commits before implementation work and normally
rebase the series instead of merging upstream into it.

Applications can request a
particular monitor for native fullscreen, unless a window rule chooses one.
Upstream also adds window-title bars to the circular overview and switcher;
`title_message_bar_height=0` disables them. Initial sizing no longer applies
the maximum-size hint. The existing bar/tab click fix is included.

```sh
git log --oneline upstream/main..desktop
git diff upstream/main..desktop
```

Keep source changes on this repository's `desktop` branch and clone that branch
on another machine. Configuration is managed separately.

For an offline transfer, `git bundle create mango.bundle desktop` saves the
committed history. On the target, use
`git clone --branch desktop PATH/mango.bundle mango-jd`. Commit or separately
save any local edits before transferring them.

Before rewriting, preserve the previous tip privately and verify the backup.
A pure commit regrouping must reproduce the same complete tree before any
upstream changes are applied. After rebasing, build and check the custom behavior;
report changes to desktop behavior, configuration or build requirements and any
conflict decisions. Publish only the selected branch, never private recovery refs.
