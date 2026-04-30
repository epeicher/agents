---
name: desktop-mode
description: Guidance for authoring WordPress plugins that **consume** the `desktop-mode` plugin (under `src/wp-content/plugins/desktop-mode/`) as a vendor API. Use whenever editing or creating a sibling plugin that hooks into desktop mode — registering windows, dock icons, commands, palettes, settings tabs, wallpapers, AI tools, or any `wpdm_*` / `wp.desktop` integration. Enforces — never modify the `desktop-mode` source or WordPress Core; treat `desktop-mode/docs/` as the public contract; stop and escalate when a needed hook is missing rather than patching the vendor. Apply any time the user mentions desktop mode, wp.desktop, wpdm_, native windows, the desktop shell, the dock, wallpapers, or building plugins alongside desktop-mode — even if they don't explicitly say "skill" or name the integration.
---

# Plugins for desktop-mode — author's notes

You are helping the author of plugins that **consume** the `desktop-mode` API. They do not maintain `desktop-mode` itself — its source under `src/wp-content/plugins/desktop-mode/` is a read-only vendor dependency. Their plugins live alongside it under `src/wp-content/plugins/<my-plugin>/`.

## What that means in practice

- **Never edit files under `src/wp-content/plugins/desktop-mode/`.** Reading it to learn the API is fine; modifying it isn't. If a hook the author needs doesn't exist, **STOP and tell the user** so they can file it with the desktop-mode developer. Do not work around the gap by patching the vendor.
- **Never edit Core** (`src/wp-admin/`, `src/wp-includes/`, `src/js/_enqueues/`). Same reasoning, stronger: this repo is a Core checkout used as a dev host for the plugins.
- **The author's code lives under `src/wp-content/plugins/<my-plugin>/`** and integrates only through documented hooks, filters, and JS APIs.
- **`docs/` inside `desktop-mode` is the public contract.** Treat it as the source of truth — read it before assuming an API exists, before guessing a signature, and before designing around an absence. If `docs/` says something is `Planned` or `Experimental`, weight that into the design.

## Read before, never update

The `docs/` tree under `desktop-mode` documents the contract the author depends on. **Read** it; do not update it (that's the desktop-mode developer's job).

```
src/wp-content/plugins/desktop-mode/docs/
├── README.md                   Index + status legend (Stable / Experimental / Planned).
│                               READ FOR: getting oriented; checking what surfaces exist.
│
├── getting-started.md          5-minute quickstart for a minimal consumer plugin.
│                               READ FOR: bootstrapping a new plugin; minimum viable skeleton.
│
├── architecture.md             High-level design: shell vs iframe, bridge, lifecycle.
│                               READ FOR: understanding why something behaves the way it does;
│                                         deciding between iframe and native windows.
│
├── hooks-reference.md          Every PHP action + filter, with Stable/Experimental/Planned
│                               status + signatures + examples.
│                               READ FOR: any PHP-side integration. Confirm the hook exists,
│                                         its status, and its signature before writing code.
│
├── javascript-reference.md     CustomEvents, window.wp.desktop API, postMessage bridge.
│                               READ FOR: any JS-side integration — registering windows,
│                                         commands, palettes, settings tabs; reacting to
│                                         window lifecycle; cross-frame messaging.
│
├── native-windows-proposal.md  Contract for native windows + framework interop.
│                               READ FOR: anything that registers a native window (PHP or JS),
│                                         window tabs, framework integration story.
│
└── examples/                   Copy-paste recipes — ONE per surface.
    ├── README.md               Index of examples.
    ├── arrange-action.md                READ FOR: hooking into window-arrange.
    ├── chromeless-style-override.md     READ FOR: theming the in-iframe (chromeless) admin.
    ├── dock-badge.md                    READ FOR: registering a dock item / showing badges.
    ├── gate-by-role.md                  READ FOR: gating desktop mode by capability/role.
    ├── inject-shell-config.md           READ FOR: passing PHP→JS config to the shell.
    ├── layout-primitives.md             READ FOR: the <wpd-*> component kit.
    ├── native-windows.md                READ FOR: wp_register_desktop_window() shape and flow.
    ├── native-window-with-tabs.md       READ FOR: tabbed native windows.
    ├── react-to-window-events.md        READ FOR: window-lifecycle CustomEvents.
    ├── register-command.md              READ FOR: slash-commands (JS or PHP-declared).
    ├── ai-ask.md                        READ FOR: wp.desktop.ai.ask() and AI tool calling.
    ├── register-icon.md                 READ FOR: wallpaper desktop icons.
    ├── register-wallpaper.md            READ FOR: registering a wallpaper module.
    └── window-lifecycle.md              READ FOR: the window state machine.
```

If an example or a `docs/` page seems wrong or stale, **don't fix it in place** — flag it to the user. The desktop-mode developer owns those files.

If `docs/` doesn't exist yet in the working copy, fall back to grepping the `desktop-mode` source for the API you need — and surface that gap to the user, since the public contract is incomplete.

## Process reminders

- **Read before speculating.** Before claiming a hook, event, or method exists, grep `docs/` first. If `docs/` doesn't cover it, grep the source. Hand-waving gets caught.
- **Stop on missing hooks.** If a feature requires a hook, filter, event, or API that isn't in `docs/` and isn't in the source, **stop and tell the user** what's missing and why it's needed. Don't patch `desktop-mode` to add it; don't reach into private internals (anything not in `docs/` is private).
- **Stable > Experimental > Planned.** Prefer `Stable` surfaces. Use `Experimental` only when nothing stable fits, and call that out to the user. Never depend on `Planned` — it doesn't exist yet.
- **Same-origin shortcut is fair game.** The chromeless iframe and the parent shell are same-origin, so editor-iframe code may reach `window.top.wp.desktop` directly. This is documented and supported. Use it for things like opening native windows from inside Gutenberg.
- **Don't reinvent.** If WordPress has an API for it, use it (`wp_enqueue_script`, `wp_localize_script`, REST, `wp.data`, `wp.plugins`, `enqueue_block_editor_assets`, etc.). The same applies to the desktop-mode API surface — prefer documented helpers over DOM scraping.

## Code Is Poetry — WordPress Way

Every line should feel like it belongs in Core, even though the plugin is third-party.

- **PHP**: WordPress PHP Coding Standards — tabs, Yoda conditions, `snake_case`, PHPDoc, `defined( 'ABSPATH' ) || exit;` at the top of every PHP file. Sanitize on input, escape on output. Use nonces for AJAX.
- **JS**: WP JS standards — tabs, JSDoc on public methods, no jQuery in new code, prefer `@wordpress/*` packages.
- **CSS**: WP CSS standards — tabs, semantic class names prefixed with the plugin's slug. CSS logical properties for RTL.
- **A11y**: keyboard-reachable, ARIA-correct, focus-visible. Test with VoiceOver when the surface is non-trivial.

## Plugin layout

A typical consumer plugin lives at:

```
src/wp-content/plugins/<my-plugin>/
├── <my-plugin>.php          plugin bootstrap; defined('ABSPATH')||exit; enqueues; hook wiring
├── includes/                PHP partials when the bootstrap grows past one file
├── assets/
│   ├── editor.js            block-editor side (loaded via enqueue_block_editor_assets)
│   ├── shell.js             parent-shell side, when needed (loaded via admin_enqueue_scripts
│   │                        gated on wpdm_is_enabled() && ! wpdm_is_chromeless_request())
│   └── *.css
└── readme.txt               WordPress.org-style readme if shipping publicly
```

Helpers exposed by `desktop-mode` to lean on from PHP (all documented in `docs/`):

- `wpdm_is_enabled()` — is desktop mode active for the current user?
- `wpdm_is_chromeless_request()` — is the current request the in-iframe chromeless render?
- `wp_register_desktop_window()`, `wp_register_desktop_icon()`, `wp_register_desktop_widget()`, `wp_register_desktop_wallpaper()`, `wp_register_desktop_command()` / `wp_desktop_register_command_script()`, `wp_register_desktop_settings_tab()` / `wp_desktop_register_settings_tab_script()`, `wp_register_desktop_ai_tool()` — registration entry points. Confirm the exact shape in `hooks-reference.md` / the relevant example before calling.

JS surfaces to lean on (all documented in `javascript-reference.md`):

- `wp.desktop.windowManager` — `open`, `openNew`, `getById`, `focus`, `closeAll`, `getVisibleRects`, virtual desktops.
- `wp.desktop.registerWindow()` — open a native (DOM-rendered) window from JS.
- `wp.desktop.registerCommand()`, `registerPalette()`, `registerSettingsTab()`, `wp.desktop.ai.ask()`.
- `wp.desktop.hooks` + `wp.desktop.HOOKS` — subscribe to lifecycle actions (`WINDOW_OPENED`, `WINDOW_CLOSED`, etc.).
- `document.addEventListener( 'wp-desktop-init', … )` — wait for the shell to be ready before touching `wp.desktop`.
- postMessage bridge (`wp-desktop-title-change`, `wp-desktop-navigate`, `wp-desktop-notification`, `wp-desktop-focus-request`, …) — for iframe-window plugins that need to talk to the shell.

## Testing

The plugin is responsible for testing **itself**, not `desktop-mode`.

- **PHPUnit** for any non-trivial PHP function added. Tests live under `src/wp-content/plugins/<my-plugin>/tests/phpunit/`. Tag with the plugin's group, not `desktop-mode`.
- **E2E (Playwright)** for user-facing flows where it matters.
- **Manual QA** in the Dockerised host: from the repo root, `npm run env:start` → `npm run env:install` → site at http://localhost:8889 (admin / password). Activate `desktop-mode` and the consumer plugin from the Plugins screen, then toggle desktop mode on via the admin-bar button.

## Non-goals (for the consumer)

- NOT modifying `desktop-mode` source.
- NOT modifying WordPress Core.
- NOT depending on private/undocumented internals — anything outside `docs/` is private and may break without notice.
- NOT shipping work that requires a `Planned` surface; if genuinely needed, escalate to the user so the desktop-mode developer can prioritise it.

## Extracting (externalizing) an in-tree plugin

When the user asks to take an in-tree feature (something that lives under `desktop-mode/includes/<feature>/`) and turn it into a sibling plugin, follow this playbook. It is built from real failure modes — copy-pasting the in-tree pattern naïvely will look fine on your machine and break on the second user's.

### Naming rules (non-negotiable)

- **Plugin slug must NOT start with `wp-`.** Plugin guidelines disallow it. Use `desktop-mode-<feature>` (e.g. `desktop-mode-cron-manager`, `desktop-mode-recycle-bin`).
- **PHP constants**: `DESKTOP_MODE_<FEATURE>_*` (e.g. `DESKTOP_MODE_CRON_MANAGER_DIR`). Not `WP_DESKTOP_*`.
- **Function prefix**: `desktop_mode_<feature>_*`. Not `wpdm_*` (that's a framework-internal prefix the consumer should not claim).
- **Textdomain**: matches the slug exactly.
- **REST namespace**: `<your-slug>/v1`. Never reuse `wp-desktop/v1` — that's the framework's namespace.
- **Plugin header**: include `Requires Plugins: desktop-mode` (WordPress 6.5+) so deactivating desktop-mode auto-deactivates the consumer.

### What you can rename. What you can't.

When extracting from in-tree, the prebuilt JS bundle hardcodes some identifiers and you cannot rebuild it from outside `desktop-mode/`. Keep these on whatever spelling the in-tree bundle uses (typically the framework's `wpdm-` prefix):

- The JS-side config global the bundle reads (e.g. `window.wpDesktopCronManagerConfig`).
- The native window id passed to `desktop_mode_register_window()` and looked up via `window.wpDesktopNativeWindows[id]` inside the bundle.
- CSS class names (`wpdm-<feature>`, `wpdm-<feature>__*`) and DOM data attributes (`data-wpdm-<feature>-*`) the bundle's render callback queries.

Everything else (PHP function names, constants, textdomain, REST namespace, AJAX action names, script/style handles, filter hook names, error codes) is fair game and SHOULD be renamed to the new prefix for cleanliness.

### The config-delivery trap (this is THE bug to avoid)

In-tree native-window plugins typically ship their config like this:

```php
wp_localize_script( 'wp-desktop-foo', 'wpDesktopFooConfig', [ ... ] );
```

**This works on the eager-load path and silently fails on the lazy-load path.** Desktop-mode loads native-window bundles two ways:

1. **Eager** — `desktop_mode_enqueue_native_window_scripts()` enqueues the handle, WordPress prints it via `wp_print_scripts`, inline data prints alongside.
2. **Lazy** — desktop-mode's JS calls `loadVendorScript()` which dynamically appends a `<script src="…">` tag. **This bypasses `wp_print_scripts` entirely.** Any `wp_localize_script` / `wp_add_inline_script` data attached to the handle is dropped on the floor.

In-tree plugins like `recycle-bin/` use the fragile pattern and "work" because the eager path triggers on developer machines. They break on consumer environments that take the lazy path. The bundle then throws `"<feature>Config is missing - bundle was loaded outside of desktop mode"`. The error message is misleading: the bundle WAS loaded inside desktop mode; its config just didn't survive the trip.

**The robust fix: serve the bundle through `admin-ajax.php` with the config baked into the JS response body.**

```php
// In your bootstrap:
add_action( 'wp_ajax_<your_action>', 'desktop_mode_<feature>_serve_bundle' );

function desktop_mode_<feature>_register_assets() {
    $bundle_url = add_query_arg(
        array( 'action' => '<your_action>' ),
        admin_url( 'admin-ajax.php' )
    );
    wp_register_script(
        '<your-handle>',
        $bundle_url,
        array( 'wp-i18n', 'wp-desktop' ),
        DESKTOP_MODE_<FEATURE>_VERSION,
        true
    );
}

function desktop_mode_<feature>_serve_bundle() {
    if ( ! desktop_mode_<feature>_user_can_use() ) {
        status_header( 403 );
        exit;
    }
    nocache_headers();
    header( 'Content-Type: application/javascript; charset=utf-8' );
    header( 'Vary: Cookie' );
    echo 'window.wpDesktopFooConfig = ' . wp_json_encode( $config ) . ";\n";
    readfile( DESKTOP_MODE_<FEATURE>_DIR . 'assets/js/<bundle>.js' );
    // Optionally append a customElements.whenDefined() wrapper to
    // guard against custom-element upgrade races on first window-open.
    exit;
}
```

The config is now part of the bundle's HTTP response body, set on `window` before the IIFE runs. There is no `<script>` tag to lose, no hook to fire, no admin template to bypass. This works on every environment because it doesn't depend on any WordPress page-rendering hook firing.

`Vary: Cookie` + `nocache_headers()` ensure each session gets its own nonce — cached responses are never shared across users.

### What NOT to carry across when extracting

Things that look tempting to copy but should stay in-tree:

- **TypeScript sources** (`src/<feature>/index.ts`, etc.) — they reference the in-tree shell components and the desktop-mode vite config. Re-creating that build pipeline outside is large; ship the prebuilt JS bundle as-is.
- **PHPUnit tests** that depend on desktop-mode's test bootstrap. Either rewrite for the new plugin's own test setup or skip.
- **Bug fixes to shared `<wpd-*>` components** (e.g. `wpd-select`, `wpd-table`). If the in-tree feature shipped alongside a fix to a desktop-mode component, that fix has to land in `desktop-mode` itself — surface this to the user; do not patch the vendor.

### Step-by-step

1. Read the in-tree feature directory completely. Note: which functions are public-facing (filters, REST routes, registration calls) vs framework-internal (helpers, error codes, option keys).
2. Choose the new plugin slug (`desktop-mode-<feature>`).
3. Create `src/wp-content/plugins/<slug>/` and the standard layout (`<slug>.php` bootstrap, `includes/`, `assets/{js,css}/`).
4. Copy prebuilt `.js` (and `.min.js`) and `.css` files verbatim.
5. Move PHP partials into `includes/`, swap function prefixes, constants, textdomain, REST namespace, filter names, AJAX action.
6. Replace the in-tree `wp_localize_script(...)` config-delivery with the admin-ajax-served bundle pattern above.
7. Add `Requires Plugins: desktop-mode` to the plugin header.
8. **Test on at least two environments.** The eager path masks the bug — a fix that works locally may still hit the lazy path on the recipient's machine. If you can only test one machine, ship the admin-ajax pattern from the start; it removes the variable.

### Diagnostic snippet for "the bundle says config is missing"

When a consumer reports `"<feature>Config is missing"`, paste this into their browser console:

```js
({
  bundleVer:    document.querySelector('script[src*="<bundle>.js"]')?.src.split('?ver=')[1],
  configGlobal: typeof window.wpDesktop<Feature>Config,
  inlineTag:    !!document.getElementById('<your-config-tag-id>'), // if you used a static tag
  htmlContains: document.documentElement.outerHTML.includes('wpDesktop<Feature>Config'),
})
```

- `configGlobal: "object"` → fixed.
- `configGlobal: "undefined"` and `htmlContains: false` → config never made it into the page (you're hitting the lazy-load path; switch to the admin-ajax-served bundle).
- `htmlContains: true` but `configGlobal: "undefined"` → config tag is in the DOM but isn't running (parse error or stripped by an output filter).

### Iteration discipline

If "this should work" doesn't on the first deploy, **switch architectures, don't add hooks**. The session that produced this skill burned nine versions trying to find a head/footer hook that fires reliably on every consumer's environment. The right answer was: stop relying on hooks firing, serve the config inside the bundle response. Skip directly to admin-ajax if there's any doubt.
