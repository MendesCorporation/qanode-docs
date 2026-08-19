# Mobile Inspector

The **Mobile Inspector** is a visual tool built into QANode for recording and building mobile automation steps interactively. Instead of writing XPath and selectors manually, you interact directly with the device screen and steps are generated automatically.

---

## How to Access



The Mobile Inspector is opened from the flow editor:

1. Add a **Mobile Flow** node to the flow
2. Configure the mobile credential (device, platform, app)
3. Click the **magnifying glass icon** button in the node's properties panel

> In the **Desktop edition**, Appium starts automatically when connecting. In the web/server edition, make sure Appium is running before opening the inspector.

[How to Access](../../assets/images/mobile-como-acessar.mp4)

---

## Interface

The Mobile Inspector is divided into three main areas:

```
┌──────────────────────────────────────────────────────────────┐
│  [Connect]   Mode: [Tap | Inspect]       [Close Session]     │
├─────────────────────────┬────────────────────────────────────┤
│                         │  Recorded Steps                    │
│   Device Screen         │  ─────────────────────────────     │
│   (live screenshot)     │  1. tap — Login Button             │
│                         │  2. type — email@...               │
│                         │  3. tap — Login                    │
│                         │  ─────────────────────────────     │
│                         │  [Clear]           [Save Steps]    │
├─────────────────────────┴────────────────────────────────────┤
│  Selected Element: id=com.app:id/login_btn                   │
└──────────────────────────────────────────────────────────────┘
```

### Device Screen Area

[Device Screen Area](../../assets/images/mobile-area-dispositivo.mp4)

Displays an updated screenshot of the device in real time. You can:
- **Tap** elements to generate `tap` steps
- **Drag** to generate `swipe` steps
- **Right-click** to inspect an element without tapping (works in any mode)

### Recorded Steps Area

Lists all steps recorded during the session. Each action shows:
- Icon and action color, following QANode's visual pattern
- Action type (tap, type, swipe, wait, extract, assert, etc.)
- Target description or text
- Actions to copy or remove the individual step

You can also reorder steps by dragging them in the list before saving to the node.

### Selected Element Panel

Displays information about the tapped element in **Inspect** mode:
- selected target summary;
- text, label, native class, and bounds;
- relevant element attributes;
- detected selectors, with the option to enable or disable which ones will be used;
- match count when this information is available.

The selector section starts collapsed to keep the screen clean. Open it when you need to review or adjust the strategy before adding an action.

---

## Operation Modes

### Tap Mode (default)

[Tap Mode](../../assets/images/mobile-modo-toque.mp4)

In **Tap** mode, each click on the device screen:
1. Sends the tap to the device via Appium
2. Records a `tap` or `tap-coords` step with the tapped element's selectors
3. Updates the screenshot automatically after the action

To **swipe**: click and drag on the device screen — the gesture is executed on the device and recorded as a `swipe` step.

When a text field is selected, the inspector can also record filling through its own dialog without switching modes. **Clear first** is enabled by default.

### Inspect Mode

[Inspect Mode](../../assets/images/mobile-modo-inspecionar.mp4)

In **Inspect** mode, clicking an element **does not send the tap to the device** — it only displays the element's information in the side panel. Use this mode to:
- Explore the element hierarchy without interacting with the app
- Copy selectors for manual use
- Identify the best selector before creating a step

> **Tip:** You can also **right-click** anywhere on the device screen to inspect an element without switching to Inspect mode.

---

## Recording Steps

### Tap Step

[Tap Step](../../assets/images/mobile-passo-toque.mp4)

1. Make sure you are in **Tap Mode**
2. Click the desired element on the device screen
3. The inspector detects the element, extracts its selectors, and records a `tap` step
4. The screen updates automatically

**Double-tap shortcut:** hold `Shift` while clicking the element to execute and record `double-tap`.

### Type Step

[Type Step](../../assets/images/mobile-passo-digitacao.mp4)

After tapping a text field:
1. The inspector detects that the field is active
2. Type the text on your physical keyboard — the inspector captures keystrokes in real time
3. The typed text is grouped and recorded as a `type` step
4. Pressing **Backspace** deletes characters and updates the step

### Swipe Step

[Swipe Step](../../assets/images/mobile-passo-swipe.mp4)

1. In **Tap Mode**, click and **drag** on the device screen
2. The gesture is executed on the device
3. A `swipe` step is recorded with start and end coordinates

---

## Visual Feedback

During inspector use, visual indicators are displayed over the device screen:

| Indicator | Meaning |
|-----------|---------|
| Pulsing green circle | Registered tap point |
| Green line with circles | Registered swipe gesture |

These indicators disappear automatically after a few seconds.

---

## Advanced Actions

In addition to recording taps and swipes, the Mobile Inspector provides buttons to add advanced actions directly to the flow without manually configuring the fields.

### Actions available in the panel

**With a selected element (Inspect mode):**

| Action | Description |
|--------|-------------|
| **Tap** | Creates a `tap` step on the selected element |
| **Fill** | Opens a dialog to enter text and creates a `type` step |
| **Double Tap** | Creates a double-tap step on the selected element |
| **Long Press** | Creates a `long-press` step on the selected element |
| **Pinch In** | Creates a pinch gesture over the element |
| **Zoom Out** | Creates a spread gesture over the element |
| **Wait Visible** | Creates a wait until the element appears |
| **Wait Enabled** | Creates a wait until the element is enabled |
| **Extract** | Creates an `extract` step using stable selectors |
| **Assert** | Creates an assert step for the selected element |

**System actions (without a selected element):**

| Action | Description |
|--------|-------------|
| **Send File** | Sends a file to the device and creates a `push-file` step |
| **Download File** | Captures a file from the device and creates a `pull-file` step |
| **Accept Alert** | Adds a `permission` step that accepts the current system alert |
| **Dismiss Alert** | Adds a `permission` step that dismisses the current system alert |
| **Grant Permission** | Prompts for the Android permission name and adds a `permission` step to grant it |
| **Revoke Permission** | Prompts for the Android permission name and adds a `permission` step to revoke it |

> Grant/Revoke Permission actions are Android-only and open a prompt requesting the permission name, for example: `android.permission.CAMERA`.

### Sending a file during recording

[Sending a file](../../assets/images/mobile-enviar-arquivo.mp4)

Use **Send File** when the app needs to receive a file before opening a native picker or continuing the flow. The inspector sends the file to the device during recording and adds the corresponding step to Mobile Flow.

On Android, the default destination is usually the downloads folder. On iOS, the inspector uses the application container when available and applies an Appium-compatible fallback.

### Downloading a file during recording

[Downloading a file](../../assets/images/mobile-baixar-arquivo.mp4)

Use **Download File** when the app generated a file and the test needs to capture it as a `fileRef`.

The dialog lets you provide:

- exact path on the device;
- folder and filename pattern to find the most recent file;
- output variable name;
- output filename saved in QANode.

Use exact path when the app shows or defines the filename. Use folder/pattern when the filename is dynamic.

---

## Saving Steps

[Saving Steps](../../assets/images/mobile-salvar-passos.mp4)

When you click **"Save Steps"**:
1. All recorded steps are transferred to the Mobile Flow node
2. The Appium session remains open (you can continue recording if needed)
3. The inspector closes and the steps are visible in the node panel

> Steps generated by the inspector can be manually edited in the Mobile Flow node after saving — add evidence, adjust selectors, configure retry attempts, etc.

---

## Managing the Session

| Action | Description |
|--------|-------------|
| **Connect** | Starts a new Appium session with the credential settings |
| **Close Session** | Terminates the Appium session on the device |
| **Clear Steps** | Removes all recorded steps (the session remains active) |

> Closing the inspector without clicking "Save Steps" discards the steps recorded in the current session. Steps previously saved to the node are not affected.

---

## Generated Selectors

The inspector automatically creates more than one strategy when the application provides enough information. It prioritizes:

1. accessibility identifiers and the application's own IDs;
2. the control's text, label, hint, or name;
3. nearby context, which is useful for fields without their own identifier;
4. selectors compatible with the platform;
5. the tap position as a visual fallback.

When recording screens with repeated elements, use Inspect Mode to confirm that the highlight matches the intended item. If the app provides only a position, the step may be more sensitive to major layout changes.

> For more stable flows, prefer controls with accessibility identifiers. Position is useful for components that Android or iOS do not expose as identifiable elements, but it should be treated as a last resort.

---

## Evidence Generated by the Inspector

When saving, the inspector configures evidence capture according to the action type:

- actions that usually change the screen, such as taps, gestures, permissions, back, opening, or resetting the app, use a longer window to wait for stability;
- typing, extraction, assertions, and other actions without a transition use a short window;
- `wait` and `pull-file` steps do not receive a screenshot automatically;
- taps and gestures can record evidence before and after; other actions capture only after by default.

These values are only the defaults created during recording. In the step panel, you can enable or disable the screenshot and adjust:

| Field | Purpose |
|-------|---------|
| **Wait timeout** | Total limit for looking for stability |
| **Capture delay** | Minimum time before the first image |
| **Polling interval** | Interval between checks |
| **Stable frames** | Number of equal captures required to consider the screen stable |

QANode follows the panel values exactly during execution. Increase the timeout or delay for screens with long animations; keep the window short for actions that do not change the screen to avoid unnecessarily slow executions.

---

## Tips

- Use **Inspect Mode** (or right-click) to identify the best selector before recording, especially on screens with many similar elements
- **Record a complete flow** in one go — connect, execute the use case in the app, save — to get steps with real context
- After saving, review the steps in the node and add **evidence screenshots** on critical steps
- For apps with long animations, add `wait` steps (`elementVisible`) after taps to ensure the screen has loaded before the next action
- In the **Desktop edition**, local device connection is faster — use for development. For cloud tests (BrowserStack, etc.), configure the SaaS credential before opening the inspector
