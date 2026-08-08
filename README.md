# 🛡️ AgebypassX v2.3.0

A lightweight Tampermonkey userscript designed to modify X/Twitter's client-side age and sensitive-media feature state directly in the browser.

AgebypassX v2.3.0 uses a **multi-layer state interception system** covering:

- `window.__INITIAL_STATE__`
- `Object.assign`
- `JSON.parse`
- `Response.prototype.json`

The script runs entirely inside your browser. It does not send data to external servers, include analytics or tracking, or directly modify your X/Twitter account.

> **Note:** v2.3.0 no longer relies primarily on webpack chunk interception. The current implementation works by intercepting and patching application state and parsed response data before X's frontend consumes it.

---

## 🚀 Installation

### Recommended Setup

1. Install **Tampermonkey** or another compatible userscript manager.
2. Install the **AgebypassX** userscript.
3. Open or reload **X/Twitter**.
4. A small **green status dot** should appear in the upper-right corner when the script has loaded successfully.

The script runs at `document-start` so its hooks are installed as early as possible.

---

## 🔒 Privacy

AgebypassX is designed to operate entirely locally.

The script:

- Does **not** send browsing data anywhere.
- Does **not** contact external APIs or servers.
- Does **not** contain analytics.
- Does **not** contain advertisements.
- Does **not** contain telemetry or tracking.
- Does **not** directly modify your X/Twitter account.
- Does **not** modify your authentication cookies.
- Stores only the indicator visibility preference locally using `localStorage`.

Everything happens inside the browser through the userscript.

The full source code is available for inspection.

---

## ⚙️ How It Works

Modern X/Twitter pages receive application configuration and feature information through several different state and response paths.

AgebypassX hooks those paths and patches selected client-side values before the frontend uses them.

### Targeted Feature Flags

The current version watches for the following flags:

```js
rweb_age_assurance_flow_enabled
age_verification_gate_enabled
sensitive_tweet_warnings_enabled
sensitive_media_settings_enabled
grok_settings_age_restriction_enabled
rweb_mvr_blurred_media_interstitial_enabled
```

It also detects compatible feature structures such as:

```js
{
    feature: "feature_name",
    enabled: true
}
```

and feature objects containing:

```js
{
    value: true
}
```

The patcher recursively searches relevant state objects while using cycle protection to avoid repeatedly traversing the same object during a single patch operation.

---

## 🧩 Interception Layers

### 1. `__INITIAL_STATE__`

AgebypassX installs a setter on:

```js
window.__INITIAL_STATE__
```

When X assigns its initial application state, the object is patched before being returned to the application.

This allows relevant feature configuration to be modified very early during page initialization.

---

### 2. `Object.assign`

Some state is created or updated dynamically after the initial page load.

AgebypassX wraps:

```js
Object.assign
```

and checks state-like objects containing structures such as:

```js
featureSwitch
entities
users
```

Relevant objects are patched after the assignment completes.

---

### 3. `JSON.parse`

Some application state is created by parsing JSON strings.

AgebypassX wraps:

```js
JSON.parse
```

Before performing a recursive patch, the raw JSON string is checked using a single regular-expression gate.

Deep traversal only occurs when the payload contains one of the target feature names or related state fields.

This significantly reduces unnecessary processing.

---

### 4. `Response.prototype.json`

Modern applications frequently use:

```js
fetch(...)
```

followed by:

```js
response.json()
```

Browser-native `Response.json()` parsing does not necessarily pass through the page's overridden `JSON.parse`.

For this reason, AgebypassX also wraps:

```js
Response.prototype.json
```

Returned objects are first inspected using a lightweight, depth-limited structural scan.

Only potentially relevant responses are passed to the full patcher.

---

## ⚡ Performance

AgebypassX is designed to avoid blindly scanning every object X creates.

### Raw JSON Gate

Before recursively walking data parsed through `JSON.parse`, the script performs a single regex search for:

- Target feature flag names
- `birthdate`
- `featureSwitch`

If none of those values are present, the parsed object is returned immediately without a deep scan.

### Fetch Response Gate

Objects returned from `Response.json()` use a depth-limited key search before the full patcher is invoked.

This avoids recursively walking normal API responses that contain no relevant configuration.

### Event-Driven Architecture

AgebypassX does not use:

- Continuous polling
- Scroll listeners
- Repeating patch timers
- Per-tweet processing loops

Patching occurs only when one of the hooked data paths is triggered.

---

## 🔁 SPA Navigation Support

X is a Single Page Application, meaning large parts of the application can be replaced without a traditional page reload.

AgebypassX uses a new `WeakSet` for every patch operation.

This provides cycle protection during a single recursive traversal while still allowing the same state object to be patched again later if X repopulates or modifies it after SPA navigation.

---

## 🎭 Native Function Compatibility

AgebypassX wraps several native JavaScript functions.

To reduce compatibility problems, the replacement functions preserve the original function's:

- `.name`
- `.length`

For example, the wrapped `JSON.parse` continues to identify itself as:

```js
parse
```

and the wrapped `Object.assign` continues to identify itself as:

```js
assign
```

This helps prevent simple feature-detection checks from treating the functions differently from their native counterparts.

---

## 🟢 Status Indicator

AgebypassX displays a small status indicator in the top-right corner of the page.

### Green Dot

🟢 **ACTIVE**

The script loaded and no hook installation or patching error has been detected.

### Red Dot

🔴 **ERROR**

At least one hook or patch operation encountered an error.

Check the browser console for messages beginning with:

```text
[Nox]
```

---

## 🖱️ Status Popup

Click the indicator dot to display the current script status.

The popup shows:

- AgebypassX version
- Current ACTIVE / ERROR state
- Installed hooks
- Indicator hotkey

Current hooks:

```text
__INITIAL_STATE__
Object.assign
JSON.parse
Response.json
```

---

## ⌨️ Indicator Hotkey

Press:

```text
Alt + .
```

to show or hide the indicator.

The preference is stored locally using:

```js
localStorage
```

and remains applied across page reloads.

No external server receives this setting.

---

## 🛡️ Defensive Indicator Mounting

Because the userscript runs at:

```text
document-start
```

the page DOM may not yet exist when the script starts.

The indicator therefore waits for the document root before mounting.

A `MutationObserver` also watches for cases where X replaces or removes the indicator during page updates.

If the indicator disappears, AgebypassX automatically mounts it again.

---

## 🛠️ Troubleshooting

### No Indicator Appears

Try the following:

- Press **Alt + .** in case the indicator was previously hidden.
- Confirm Tampermonkey is enabled.
- Confirm AgebypassX is enabled.
- Confirm the script has permission to run on:

```text
https://x.com/*
https://twitter.com/*
```

- Reload the page.

---

### No `[Nox]` Messages in Console

Open DevTools:

```text
F12 → Console
```

Search for:

```text
[Nox]
```

You should normally see:

```text
[Nox] Loaded
[Nox] Ready
```

If there are no AgebypassX messages at all, the userscript is probably not executing.

Check your userscript manager and site permissions.

---

### Red Indicator

A red indicator means an exception occurred while installing or executing one of the hooks.

Check the browser console for messages such as:

```text
[Nox] __INITIAL_STATE__ hook failed
[Nox] Object.assign patch failed
[Nox] JSON.parse patch failed
[Nox] Response.json hook failed
```

Include those messages when opening a bug report.

---

### Green Indicator but Behaviour Has Not Changed

A green indicator means the hooks loaded without throwing an error.

It does **not** guarantee that X is still using the same feature names or application structures.

X may have:

- Renamed a feature flag.
- Changed its response format.
- Moved configuration to another state path.
- Added server-side enforcement.
- Changed how the frontend consumes feature state.

If this happens, open a GitHub issue with console information and reproduction details.

---

## 🌐 Browser Compatibility

AgebypassX has primarily been tested with **Chromium-based browsers**.

Examples include:

- Google Chrome
- Chromium
- Microsoft Edge
- Brave

Other browsers or userscript managers may behave differently.

---

## 🧪 Debugging

Open the browser developer console and filter messages using:

```text
[Nox]
```

Common messages include:

```text
[Nox] Loaded
```

The userscript has started executing.

```text
[Nox] Ready
```

All hook installation code has completed.

```text
[Nox] Patched __INITIAL_STATE__
```

An initial application state object was intercepted and passed through the patcher.

```text
[Nox] Indicator hidden
```

or:

```text
[Nox] Indicator visible
```

The indicator visibility was changed using **Alt + .**.

---

## 🐛 Reporting Issues

Bug reports are welcome.

Open an issue:

https://github.com/Saganaki22/AgebypassX/issues

Please include:

- Browser name
- Browser version
- Userscript manager
- Userscript manager version
- AgebypassX version
- Relevant `[Nox]` console messages
- Screenshot of the status popup if useful
- Description of what happened
- Steps required to reproduce the issue

Please remove any personal information from screenshots or console output before posting it publicly.

---

## 🧑‍💻 Source Code

AgebypassX is open source.

Repository:

https://github.com/Saganaki22/AgebypassX

The complete userscript can be inspected before installation.

---

## 📜 License

AgebypassX is licensed under the **MIT License**.

You are free to inspect, modify, fork, and redistribute the project under the terms of the license.

https://opensource.org/licenses/MIT

---

## ⭐ Support & Contributions

Feedback, bug reports, pull requests, and improvements are welcome.

GitHub:

https://github.com/Saganaki22/AgebypassX

If X changes its frontend architecture or feature configuration, reports containing useful console information can help identify what changed.

---

# 🔄 Version History

## v2.3.0 — Current

### ⌨️ Indicator Hotkey

Added:

```text
Alt + .
```

to show or hide the status indicator.

The preference persists locally across reloads.

### ⚡ Performance Gate

Added a single-pass regular-expression gate for raw JSON payloads.

Deep recursive walks are only performed when potentially relevant feature names are detected.

### 🔍 Smarter Fetch Inspection

`Response.json()` results now use a lightweight, depth-limited structural scan before the full object patcher runs.

This substantially reduces unnecessary traversal of unrelated responses.

---

## v2.2.0

### 🆕 `Response.json` Hook

Added interception for:

```js
Response.prototype.json
```

because browser-native fetch response parsing can bypass an overridden `JSON.parse`.

### 🔁 Re-Patch Support

Cycle detection is now scoped to individual patch operations.

State objects can therefore be patched again if X repopulates them after SPA navigation.

### 🛡️ Defensive Indicator

Improved indicator mounting for `document-start`.

The indicator is also automatically restored if the page removes it.

### 🎭 Native Function Spoofing

Wrapped native functions preserve their original:

```text
.name
.length
```

values.

### 🧩 GraphQL Feature Arrays

Added support for feature structures using:

```js
{
    feature: "feature_name",
    enabled: true
}
```

---

## v2.1.0

### 🎯 Multi-Hook Architecture

Introduced interception through:

```text
__INITIAL_STATE__
Object.assign
JSON.parse
```

### 🚦 Status Indicator

Added the green/red status indicator with click-for-status information.

### 🛡️ Hardened Patching

Improved protection against:

- Circular object graphs
- Throwing getters
- Non-writable properties
- Unexpected application objects
- DOM objects and browser globals

---

## v1.3.0 — Webpack Edition

### 🆕 Webpack Architecture

Introduced webpack chunk interception for the X frontend architecture used at the time.

### 🎯 Enhanced Detection

Added targeting for sensitive-media-related frontend modules and queries.

### 📊 Advanced Status

Added improved debugging and status information.

### 🔧 Multiple Fallbacks

Added additional interception strategies for network and GraphQL-related paths.

### 🎨 UI Improvements

Expanded the status indicator and debugging interface.

### 🐛 Improved Debugging

Added additional console information and debugging tools.

> The architecture introduced in v1.3.0 should not be confused with the current v2.3.0 implementation. Modern releases primarily use application-state and response interception instead of webpack chunk interception.

---

## v1.2 — Enhanced Edition

- Added configuration and state-management improvements.
- Added multiple patching strategies.
- Improved user interface behaviour.
- Improved SPA navigation handling.

---

## v1.1 — Reliability Update

- Added fallback patching methods.
- Improved error handling.
- Expanded patch targets.

---

## v1.0 — Original

- Initial release.
- Basic client-side age-state patching.
- Basic status indicator.

---

## ⚠️ Disclaimer

AgebypassX is an independent open-source project and is not affiliated with, endorsed by, or associated with X Corp., Twitter, or Tampermonkey.

The project modifies client-side application state only. X may change its frontend or introduce additional server-side enforcement at any time, which may cause some or all functionality to stop working.

Use the software at your own discretion and comply with the laws, platform rules, and age requirements applicable to you.
````
