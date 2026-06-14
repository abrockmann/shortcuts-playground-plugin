# Automation Triggers

OS 27 shortcuts can include native automation headers directly in exported shortcut XML. The portable carrier is the top-level `WFWorkflowTriggers` key, not a write to the user's live Shortcuts database.

Use this only when the user explicitly asks for an OS 27 automation shortcut or provides/exported automation samples. For ordinary shortcuts, omit `WFWorkflowTriggers`.

## Packaged Metadata

- ToolKit trigger metadata: `data/toolkit-v78-trigger-parameter-keys.json`
- Exported workflow trigger samples: `data/macos27-workflow-trigger-samples.json`
- ToolKit trigger variants: 42
- Variants with exported `WFWorkflowTriggers` samples: 37
- Missing exported samples: 5
- Minimum target: macOS/iOS 27

The lookup helper surfaces both ToolKit trigger parameters and exported plist samples:

```bash
python3 scripts/lookup_action_grounding.py --python-name when_low_power_mode_changes --target-macos 27
python3 scripts/lookup_action_grounding.py --identifier com.apple.shortcuts.WFTimeOfDayTrigger.at_time_on_recurring_day --target-macos 27 --json
python3 scripts/lookup_action_grounding.py --query "wi-fi trigger" --target-macos 27 --json
```

## Plist Shape

Add `WFWorkflowTriggers` as a root key beside `WFWorkflowActions`:

```xml
<key>WFWorkflowTriggers</key>
<array>
  <dict>
    <key>WFTriggerIdentifier</key>
    <string>WFLowPowerModeTrigger</string>
    <key>WFTriggerSerializedParameters</key>
    <dict>
      <key>WFLowPowerModeType</key>
      <string>both</string>
    </dict>
    <key>WFTriggerUUID</key>
    <string>GENERATE-A-FRESH-UPPERCASE-UUID</string>
  </dict>
</array>
```

Rules:

- Always generate a fresh `WFTriggerUUID` with `uuidgen | tr '[:lower:]' '[:upper:]'`.
- Validate and sign with `--target-macos 27`.
- Do not ship catalog placeholders such as `$placeholder`; they mark redacted local picker values.
- Raw exported automation XML can contain user-local contact, Mail, Messages, account, app, location, and device payloads. Sanitize those values before adding samples to the plugin.
- Do not infer a trigger header from ToolKit metadata alone. Use `workflowTriggerSample` from the lookup helper or an exported shortcut from the user.
- Local picker values such as apps, contacts, locations, alarms, devices, networks, Focus modes, Wallet merchants, and sounds must come from a user export or be selected manually in Shortcuts.

## Observed Defaults

- Change-style triggers serialize on/off or connect/disconnect as `both`.
- Wi-Fi connect-to-any can omit serialized parameters; Wi-Fi disconnect uses `WFConnectionType = disconnected`.
- Sleep Bedtime Begins uses `WFSleepMode = bedtime`.
- Time of Day sunrise/sunset use `WFTimeEvent = sunrise` / `sunset`; the observed at-time sample stores `WFTime` as a plist date.
- Notification predicates use `Conditions`; Email and Message predicates use `WFEmailConditions` / `WFMessageConditions`.
- Battery Equal exported with an empty serialized-parameters dictionary even though ToolKit exposes threshold/comparison keys.

## Current Support

These variants have exported `WFWorkflowTriggers` samples and can be generated for OS 27 targets, subject to picker-value availability:

Observed support means the shortcut header can be generated and imported from exported XML. Runtime support is still device-specific. For example, a Low Power Mode trigger imported into macOS Shortcuts can render as "not supported on Mac" even though the same header is valid for iPhone/iPad automation testing. When the user asks for a Mac-only automation, prefer triggers verified in the macOS editor/runtime and avoid treating ToolKit platform provenance as a runtime guarantee.

| Trigger | Python Name | Template Status |
|---|---|---|
| Airplane Mode changes | `when_airplane_mode_changes` | copyable with fresh UUID |
| Alarm any alarm | `when_alarm_any_alarm` | copyable with fresh UUID |
| Alarm specific alarm | `when_alarm_specific_alarm` | requires user values |
| App opened | `when_app_opened` | requires user values |
| Apple Watch Workout any | `when_apple_watch_workout_any` | copyable with fresh UUID |
| Apple Watch Workout specific | `when_apple_watch_workout_specific` | requires user values |
| Arrive location | `when_arrive_enter_location` | requires user values |
| Arrive location between times | `when_arrive_enter_location_between` | requires user values |
| Battery Level equal | `when_battery_level_equal` | copyable with fresh UUID |
| Bluetooth any connection changes | `when_bluetooth_any_connection_changes` | copyable with fresh UUID |
| Bluetooth selected connection changes | `when_bluetooth_selected_connection_changes` | requires user values |
| CarPlay changes | `when_car_play_changes` | copyable with fresh UUID |
| Charger changes | `when_charger_changes` | copyable with fresh UUID |
| Email senders are | `when_email_senders_are` | requires user values |
| Email senders are and subject contains | `when_email_senders_are_and_subject_contains` | requires user values |
| Email subject contains | `when_email_subject_contains` | requires user values |
| Focus enable | `when_focus_enable` | requires user values |
| Keyboard connection changes | `when_keyboard_connection_changes` | copyable with fresh UUID |
| Leave location | `when_leave_leave_location` | requires user values |
| Leave location between times | `when_leave_leave_location_between` | requires user values |
| Low Power Mode changes | `when_low_power_mode_changes` | copyable with fresh UUID |
| Message contains | `when_message_contains` | copyable with fresh UUID |
| Message senders are | `when_message_senders_are` | requires user values |
| Message senders are and contains | `when_message_senders_are_and_contains` | requires user values |
| NFC scan tag | `when_nfc_scan_tag` | copyable with fresh UUID |
| Notification received | `when_notification_received` | requires user values |
| Screenshot saved | `when_screenshot_saved` | copyable with fresh UUID |
| Sleep | `when_sleep_wfsleeptrigger` | copyable with fresh UUID |
| Sound Recognition | `when_sound_recognition_sound_recognition` | requires user values |
| Time of Day around sunrise | `when_time_of_day_around_sunrise_on_recurring_day` | copyable with fresh UUID |
| Time of Day around sunset | `when_time_of_day_around_sunset_on_recurring_day` | copyable with fresh UUID |
| Time of Day at time | `when_time_of_day_at_time_on_recurring_day` | copyable with fresh UUID |
| Wallet tap | `when_wallet_tap` | requires user values |
| Wi-Fi connect to any | `when_wi_fi_connect_to_any` | copyable with fresh UUID |
| Wi-Fi connect to selected | `when_wi_fi_connect_to_selected` | requires user values |
| Wi-Fi disconnect from any | `when_wi_fi_disconnect_from_any` | copyable with fresh UUID |
| Wi-Fi disconnect from selected | `when_wi_fi_disconnect_from_selected` | requires user values |

These variants still need exported automation-bearing XML before the plugin should generate them:

- Display / external display: `when_display_wfexternaldisplaytrigger`
- External Drive: `when_external_drive_external_drive`
- File Modified: `when_file_file_modified`
- Folder Changed: `when_folder_folder_changed`
- Stage Manager On: `when_stage_manager_on`

## Validation

`validate_shortcut.py` checks `WFWorkflowTriggers` when present:

- OS target must be 27 or later.
- The root value must be an array of dictionaries.
- Each trigger must include `WFTriggerIdentifier`, `WFTriggerSerializedParameters`, and `WFTriggerUUID`.
- UUIDs must be uppercase real UUIDs, not placeholders.
- Serialized parameters must be dictionaries.
- Placeholder values from the static catalog are rejected.
- Known trigger identifiers and serialized parameter keys are checked against the exported sample catalog.
