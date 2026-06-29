# Mobile Flow Node

The **Mobile Flow** node allows you to automate interactions with native mobile applications (Android and iOS) using **Appium**. It supports real devices, emulators, and cloud services such as BrowserStack, Sauce Labs, and LambdaTest.

---

## Overview

[Visão Geral](../../assets/images/nos-mobile-flow-visao-geral.mp4)

| Property | Value |
|----------|-------|
| **Type** | `mobile-flow` |
| **Category** | Mobile |
| **Color** | 🔴 Light Red (#f87171) |
| **Input** | `in` |
| **Output** | `out` |

---

## Prerequisites

### Appium

The Mobile Flow node requires an accessible **Appium** server:

| Situation | Solution |
|-----------|----------|
| **Desktop Edition** | Appium is installed automatically on first use and started when a flow with a mobile node runs |
| **Own server** | Install manually: `npm install -g appium` + driver: `appium driver install uiautomator2` (Android) or `appium driver install xcuitest` (iOS) |
| **Cloud (BrowserStack, etc.)** | No local Appium needed — use the provider credentials |

#### Recommended versions (QANode 0.2.2+)

```bash
npm install -g appium@3.2.0
appium driver install --source=npm appium-uiautomator2-driver@7.0.0
appium driver install --source=npm appium-xcuitest-driver@10.25.0
```

> For iOS, XCUITest requires macOS with Xcode configured.

### Appium Drivers

| Platform | Driver | Installation |
|----------|--------|--------------|
| Android | UiAutomator2 | `appium driver install uiautomator2` |
| iOS | XCUITest | `appium driver install xcuitest` (requires Xcode on Mac) |

---

## General Configuration

| Field | Type | Default | Description |
|-------|------|---------|-------------|
| **Credential** | `select` | — | Mobile credential with connection settings |
| **Session Mode** | `new` / `reuse` | `new` | Start a new session or reuse an existing one |
| **Session ID** | `string` | — | ID to identify/reuse the session |
| **Self Healing** | `boolean` | `false` | Tries to recover elements when mobile selectors no longer locate the original target |

### Session Mode

- **New Session (`new`)**: Opens a new Appium session on every execution. Ideal for isolated tests.
- **Reuse Session (`reuse`)**: Reuses a session already created by another Mobile Flow node in the same flow. Useful for splitting long automations across multiple nodes while keeping the same device connected.

### Self Healing

When **Self Healing** is enabled, Mobile Flow can automatically recover elements in actions such as tap and type when the recorded selectors stop working.

On mobile, the recovery considers signals such as:

- `accessibility_id`
- visible text
- `label`, `name`, `value`, and `hint`
- `resource-id`
- native class/type of the element
- nearby context in the hierarchy captured by Appium

This helps with common changes such as:

- small label adjustments
- uppercase/lowercase differences
- text with or without accents
- selector changes while the same element is still on screen

Mobile self healing also respects the active platform:

- it prioritizes iOS-compatible strategies when the execution runs on iOS
- it prioritizes Android-compatible strategies when the execution runs on Android

When a recovery is applied, it appears in the run details with:

- the affected step
- the recovery confidence
- the mobile selector used
- an **Apply to Flow** action to promote the recovered selector back to the original flow

---

## Mobile Credential

The credential centralizes connection settings and device capabilities. Configure at **Credentials → New Credential → Mobile**.

### Connection Modes

| Mode | Description |
|------|-------------|
| **Local** | Appium running on the same machine (default: `http://localhost:4723`) |
| **Self-hosted** | Appium on a custom server with URL and authentication token |
| **SaaS** | Cloud provider (BrowserStack, Sauce Labs, LambdaTest, or Custom) |

### Device Capabilities

| Field | Description | Example |
|-------|-------------|---------|
| **Platform** | `Android` or `iOS` | `Android` |
| **Device Name** | Device name or ID | `emulator-5554`, `Pixel 7` |
| **Platform Version** | OS version | `13.0`, `17.0` |
| **App Package** | Android app package | `com.myapp.android` |
| **App Activity** | Android initial activity | `.MainActivity` |
| **Bundle ID** | iOS app identifier | `com.myapp.ios` |
| **App Path** | Path to .apk/.ipa file | `/path/to/app.apk` |
| **UDID** | Unique device ID (real devices) | `R58M20XXXXX` |
| **No Reset** | Do not reset app between sessions | `true` |
| **Full Reset** | Fully reset app state | `false` |
| **Automation Name** | Automation driver | `UiAutomator2` (Android), `XCUITest` (iOS) |
| **Extra Capabilities** | JSON with additional Appium capabilities | `{"appium:language": "en"}` |

---

## Steps (Mobile Steps)

The Mobile Flow node executes a sequence of configurable **steps**. Each step represents an action on the device.

When the flow is recorded through MCP, steps appear in the visual editor with the same review pattern as Mobile Inspector, including interaction highlights and useful information for reviewing failures.

During execution, the canvas shows Mobile Flow step progress, helping you track which action is running, passed, or failed.

Use MCP to record or adjust mobile journeys when an AI client needs to interact with Appium, but review the visual editor to confirm that each recorded target matches the intended action.

### Available Actions

| Action | Description |
|--------|-------------|
| [tap](#tap) | Tap an element |
| [tap-coords](#tap-coords) | Tap at absolute coordinates |
| [type](#type) | Type text into a field |
| [clear](#clear) | Clear a field's text |
| [swipe](#swipe) | Swipe on the screen |
| [scroll](#scroll) | Scroll in a direction |
| [wait](#wait) | Wait for a condition or time |
| [assert](#assert) | Verify a condition in the app |
| [extract](#extract) | Extract text or attribute from an element |
| [double-tap](#double-tap) | Double-tap an element |
| [long-press](#long-press) | Long press on an element |
| [pinch-zoom](#pinch-zoom) | Pinch or zoom gesture on an element |
| [multi-touch](#multi-touch) | Multi-touch gesture with two simultaneous fingers |
| [permission](#permission) | Accept/dismiss system alerts or manage Android permissions |
| [push-file](#push-file) | Send a file to the device |
| [pull-file](#pull-file) | Download a file from the device to QANode |
| [reset](#reset) | Reset the application |
| [back](#back) | Press the Back button (Android) |
| [home](#home) | Press the Home button (Android) |
| [key-event](#key-event) | Send an Android key code |

---

## Mobile Selector Strategies

| Strategy | Description | Example |
|----------|-------------|---------|
| `id` | Element resource ID | `com.myapp:id/login_button` |
| `accessibility_id` | Content description / accessibility label | `Login button` |
| `xpath` | XPath expression in XML hierarchy | `//android.widget.Button[@text="Login"]` |
| `class_name` | Native class name | `android.widget.EditText` |
| `-android uiautomator` | UIAutomator2 selector (Android) | `new UiSelector().text("Login")` |
| `-ios predicate string` | NSPredicate (iOS) | `type == 'XCUIElementTypeButton' AND name == 'Login'` |
| `-ios class chain` | iOS Class Chain | `**/XCUIElementTypeButton[\`name == 'Login'\`]` |

> **Tip:** The [Mobile Inspector](mobile-inspector.md) automatically generates selectors when you tap elements, with no need to write XPath manually.

---

## Action Details

### tap

Taps an element using selector strategies.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Element selector strategies |
| **Retry Attempts** | `number` | Number of attempts (default: 3) |
| **Retry Delay (ms)** | `number` | Wait between attempts (default: 500ms) |

If all selector attempts fail and fallback coordinates exist (generated by the inspector), the tap will be performed at the direct coordinates.

---

### tap-coords

Taps at absolute screen coordinates.

| Field | Type | Description |
|-------|------|-------------|
| **X** | `number` | X coordinate in pixels |
| **Y** | `number` | Y coordinate in pixels |

---

### type

Types text into an input field.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Field selector strategies |
| **Text** | `string` | Text to type (supports `{{ }}`) |
| **Clear First** | `boolean` | Clear the field before typing |
| **Retry Attempts** | `number` | Number of location attempts |

For text fields, QANode tries to locate the target in layers:

1. strong identifiers such as resource-id, accessibility id, name, and content-desc;
2. `hint`, `label`, and `text` when they belong to the field itself;
3. contextual relationship between the visible label and the nearby input;
4. recorded coordinates as a visual fallback;
5. generic input class only as a last resort.

---

### clear

Clears the content of an input field.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Field selector strategies |

---

### swipe

Swipes on the screen between two points.

| Field | Type | Description |
|-------|------|-------------|
| **fromX / fromY** | `number` | Start coordinates |
| **toX / toY** | `number` | End coordinates |
| **Duration (ms)** | `number` | Gesture speed (default: 350ms) |
| **Direction** | `string` | Alternative: `up`, `down`, `left`, `right` |
| **Percentage** | `number` | Percentage of the screen to swipe (with direction) |

---

### scroll

Scrolls content in a direction.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | `direction` (default), `untilElement`, `untilText` |
| **Direction** | `string` | `up`, `down`, `left`, `right` |
| **Selectors** | `array` | Container selector to scroll (optional) |
| **Text** | `string` | Text to search for (mode `untilText`) |
| **Max Scrolls** | `number` | Maximum number of scroll attempts (modes `untilElement`/`untilText`) |

**Modes:**

| Mode | Description |
|------|-------------|
| `direction` | Scrolls in the specified direction (default behavior) |
| `untilElement` | Keeps scrolling until the element matching the selectors appears on screen |
| `untilText` | Keeps scrolling until an element with the specified text appears on screen |

---

### wait

Waits for a condition before proceeding.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | `string` | Wait type |
| **Timeout (ms)** | `number` | Maximum wait time |
| **Selectors** | `array` | Selectors (modes that require an element) |
| **Expected Text** | `string` | Expected text or value (text/attribute modes) |
| **Attribute Name** | `string` | Attribute to check (mode `attributeEquals`) |

**Modes:**

| Mode | Description |
|------|-------------|
| `timeout` | Wait for a fixed time in milliseconds |
| `elementVisible` | Wait for an element to appear on screen |
| `elementHidden` | Wait for an element to disappear from screen |
| `elementEnabled` | Wait for an element to become enabled/interactive |
| `textContains` | Wait for an element's text to contain the expected value |
| `textEquals` | Wait for an element's text to exactly match the expected value |
| `attributeEquals` | Wait for a specific attribute of an element to equal the expected value |
| `screenChanged` | Wait for the screen content to change after an action |

---

### assert

Verifies a condition in the application. If it fails, the step is marked as failed.

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Assertion identifier (output key) |
| **Mode** | `string` | Verification type |
| **Selectors** | `array` | Element selectors |
| **Expected Text** | `string` | Expected value (text modes) |
| **Continue on Fail** | `boolean` | Do not interrupt the flow if it fails |

**Modes:**

| Mode | Description |
|------|-------------|
| `elementExists` | Verifies the element is visible on screen |
| `textContains` | Verifies the element's text contains the expected value |
| `textEquals` | Verifies the element's text exactly matches the expected value |

Assertion results are available in outputs:
```
{{ steps["mobile-flow"].outputs.asserts.assertionName }}  →  true or false
```

---

### extract

Extracts text or an attribute from an element.

| Field | Type | Description |
|-------|------|-------------|
| **Name** | `string` | Extraction name (output key) |
| **Selectors** | `array` | Element selectors |
| **Attribute** | `string` | What to extract: `text` (default) or attribute name |

Extracted data is available in outputs:
```
{{ steps["mobile-flow"].outputs.extracts.extractionName }}
```

---

### double-tap

Double-taps an element using selector strategies.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Element selector strategies |
| **Retry Attempts** | `number` | Number of attempts (default: 3) |
| **Retry Delay (ms)** | `number` | Wait between attempts (default: 500ms) |

---

### long-press

Performs a long press on an element.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Element selector strategies |
| **X** | `number` | X coordinate (used when there are no selectors) |
| **Y** | `number` | Y coordinate (used when there are no selectors) |
| **Duration (ms)** | `number` | Press duration (default: 800ms) |

---

### pinch-zoom

Performs a pinch or zoom gesture on an element.

| Field | Type | Description |
|-------|------|-------------|
| **Selectors** | `array` | Element selector strategies |
| **Gesture** | `string` | `pinchIn` (pinch to zoom out) or `zoomOut` (spread to zoom in) |
| **Distance (px)** | `number` | Distance of finger movement in pixels |
| **Scale** | `number` | Scale factor for the gesture |
| **Duration (ms)** | `number` | Gesture duration in milliseconds |

---

### multi-touch

Performs a multi-touch gesture with two simultaneous fingers.

| Field | Type | Description |
|-------|------|-------------|
| **Points** | `array` | Array of touch points with `x`, `y`, `duration` |
| **X1 / Y1** | `number` | First finger coordinates (when not using points array) |
| **X2 / Y2** | `number` | Second finger coordinates (when not using points array) |

---

### permission

Accepts or dismisses system alerts, or grants/revokes Android app permissions.

| Field | Type | Description |
|-------|------|-------------|
| **Action** | `string` | `acceptAlert`, `dismissAlert`, `grantPermission`, `revokePermission` |
| **Permission Name** | `string` | Android permission name (actions `grantPermission`/`revokePermission`) |

**Actions:**

| Action | Description |
|--------|-------------|
| `acceptAlert` | Accepts the current system alert (OK / Allow) |
| `dismissAlert` | Dismisses the current system alert (Cancel / Deny) |
| `grantPermission` | Grants an Android permission to the app |
| `revokePermission` | Revokes an Android permission from the app |

> `grantPermission` and `revokePermission` are Android-only. The permission name follows Android format, for example: `android.permission.CAMERA`.

---

### push-file

Sends a file from QANode to the device before continuing the automation.

| Field | Type | Description |
|-------|------|-------------|
| **File** | `fileRef` | File to send |
| **Device Path** | `string` | Destination path. If empty, QANode uses a platform default |

Use this step when the application needs a file to exist on the device before opening a native picker, gallery, document picker, or upload flow.

On Android, common paths live under `/sdcard/Download/...`. On iOS, QANode tries to send the file to the application container when available and uses an Appium/XCUITest-compatible fallback.

---

### pull-file

Downloads a file from the device, saves it as a QANode artifact, and returns a `fileRef`.

| Field | Type | Description |
|-------|------|-------------|
| **Mode** | selection | Exact path, folder/pattern, or latest-file detection |
| **Device Path** | `string` | Exact file path, when known |
| **Folder** | `string` | Folder to inspect, such as `/sdcard/Download` |
| **Pattern** | `string` | Name filter, such as `*.pdf`, `*.txt`, `*.jpg`, or `*` |
| **Variable Name** | `string` | Key in `outputs.files` |
| **Output File Name** | `string` | Name saved in QANode. If empty, uses the found name |

Prefer **Exact path** when possible. When the application generates a dynamic filename, use folder/pattern to capture the most recent compatible file.

Images, PDFs, text files, spreadsheets, and videos can be captured as long as Appium can read the file from the device or application container.

---

### reset

Resets the application, clearing its state (equivalent to uninstalling and reinstalling).

---

### back

Presses the physical/virtual **Back** button (Android — keycode 4).

---

### home

Presses the **Home** button (Android — keycode 3).

---

### key-event

Sends an Android keycode directly to the device.

| Field | Type | Description |
|-------|------|-------------|
| **Keycode** | `number` | Android key code (e.g.: 4 = BACK, 3 = HOME, 66 = ENTER) |
| **Count** | `number` | How many times to send the event |

**Common keycodes:**

| Keycode | Key |
|---------|-----|
| 3 | HOME |
| 4 | BACK |
| 66 | ENTER |
| 67 | BACKSPACE/DEL |
| 82 | MENU |
| 84 | SEARCH |

---

## Evidence (Screenshots)

Each step can have individual evidence settings:

| Field | Type | Description |
|-------|------|-------------|
| **Take Screenshot** | `boolean` | Enable screen capture |
| **Mode** | `before` / `after` / `both` | When to capture |
| **Name Template** | `string` | Custom filename |
| **Wait for Stabilization** | `boolean` | Wait for stable screen before capturing |
| **Stabilization Timeout (ms)** | `number` | Max time waiting for stable screen (default: 2000ms) |
| **Delay (ms)** | `number` | Additional time before capturing (default: 300ms) |

> Generated screenshots are available in the **Evidence** tab of the execution result. Taps and swipes are highlighted with a green visual marker over the screenshot.

---

## Outputs

| Output | Type | Description |
|--------|------|-------------|
| `sessionId` | `string` | Mobile session ID for reuse in other nodes |
| `extracts` | `object` | Data extracted by `extract` steps (key → value) |
| `asserts` | `object` | Assertion results (key → `true`/`false`) |
| `files` | `object` | Files downloaded by `pull-file` steps |
| `fileRef` | `fileRef` | Shortcut to the last downloaded file |

### Downloaded files

When a **Download file** step runs, the output is available by variable name:

```
{{ steps["mobile-flow"].outputs.files.report.fileRef }}
{{ steps["mobile-flow"].outputs.files.report.name }}
{{ steps["mobile-flow"].outputs.fileRef }}
```

Use the `fileRef` to pass the file to **Extract File**, **File to Base64**, **HTTP Request**, components, or other nodes that accept files.

---

## Complete Example

### Android app login

Steps configured in the node:

1. **wait** → Mode: `elementVisible`, Selector: `id=com.myapp:id/email_field` (timeout: 10000ms)
2. **tap** → Selector: `id=com.myapp:id/email_field`
3. **type** → Selector: `id=com.myapp:id/email_field`, Text: `{{ variables.USER_EMAIL }}`
4. **type** → Selector: `id=com.myapp:id/password_field`, Text: `{{ variables.USER_PASSWORD }}`, Clear: true
5. **tap** → Selector: `accessibility_id=Login`
6. **wait** → Mode: `elementVisible`, Selector: `id=com.myapp:id/home_title` (timeout: 15000ms)
7. **assert** → Name: `loginOk`, Mode: `elementExists`, Selector: `id=com.myapp:id/home_title`
8. **extract** → Name: `welcomeText`, Selector: `id=com.myapp:id/welcome_message`, Attribute: `text`

**Result after execution:**
```json
{
  "sessionId": "my-android-session",
  "extracts": { "welcomeText": "Hello, John!" },
  "asserts": { "loginOk": true }
}
```

---

### Reusing a Session Across Nodes

**Node 1 — Mobile Flow (Login)**
- Session ID: `android-session`
- Steps: login to the app

**Node 2 — Mobile Flow (Check Order)**
- Session Mode: `reuse`
- Session ID: `android-session`
- Steps: navigate to orders, check status

Both nodes share the same device and session state.

---

## Tips

- Use the **[Mobile Inspector](mobile-inspector.md)** to record steps visually — it generates selectors automatically when you tap elements
- In the **Desktop edition**, Appium starts automatically when a flow with a mobile node runs — no manual configuration needed
- Prefer `accessibility_id` as the primary selector — it is more stable than XPath and works on both Android and iOS
- Configure **screenshots** on critical steps to make failure identification easier
- Use **Continue on Fail** in monitoring assertions to collect multiple results without interrupting the flow
- For apps that take a long time to start, add a `wait` step with `elementVisible` before interacting
- A custom `Session ID` makes it easy to reuse sessions across multiple Mobile Flow nodes in the same flow
