# Tesla Camp Comfort V1 (README)

An Apple Shortcut that automatically adjusts your Tesla's cabin temperature while camping, responding to outside temperature changes throughout the day. Powered by the [Tessie API](http://share.tessie.com/cj00fYEccCb).

Download the shortcut here: [Tesla Camp Comfort V1.0.1](https://www.icloud.com/shortcuts/4eca3da6aa1e4160bfef5bd632d289d5)

---

## What It Does

When you're sleeping in your Tesla with Camp Mode on, the cabin holds whatever temperature you set — regardless of what's happening outside. As the morning sun heats things up, the temperature you set the night before may not be enough to keep you cool. On a cold night that keeps dropping, you might want more heat than what you originally set.

On top of that, when you're sleeping in the back of the cabin with the seats folded down, you're far from the vents — the thermostat up front might read fine, but the back where you're actually sleeping doesn't get the same airflow. As the outside temperature climbs, this gap gets worse. You wake up sweating and have to grab your phone to lower the temperature — which defeats the point of a comfortable setup.

This shortcut fixes that. It monitors the outside temperature and automatically adjusts the cabin temp in steps as conditions change — pushing the AC harder as the heat climbs, or raising the heat as it gets colder. You set the thresholds once and it tracks the temperature throughout the day. No waking up, no manual adjustments.

## How It Works

The shortcut runs on a schedule via iOS Time of Day automations. You control the frequency — hourly is a good default, but you can set shorter intervals (like every 30 minutes) during times when temperature changes quickly, and longer intervals when conditions are more stable. Each time it runs, it:

1. Checks if Camp Mode is active on your Tesla (via Tessie API)
2. Reads the current outside temperature
3. Compares it to the previous reading to determine if the temperature is rising or falling
4. Evaluates your configured tiers to see if any thresholds have been crossed
5. Adjusts the cabin temperature if needed

Each tier fires at most once per day. The shortcut resets automatically at midnight.

---

## Quick Start

**You'll need:**
- A Tesla with Camp Mode capability
- A [Tessie](https://tessie.com) account and API key
- An iPhone or iPad (to run the shortcut via automations)

**Setup:**

1. **Add the shortcut** to your Shortcuts app. When you install it, you'll be prompted with setup questions in order:
   - `VIN` — your Tesla's VIN
   - `TESSIE_API_KEY` — your Tessie API key (to find this: open the Tessie app → tap the **☰** menu at the top left → **Settings** → **Developer** → **Generate Access Token**)
   - `USE_WIFI_GATE` — set to `1` to enable Wi-Fi checking, `0` to disable (see Wi-Fi Gate section below)
   - `SSID_MAIN` — your vehicle's Wi-Fi network name, exactly as it appears when you connect to Wi-Fi on your phone (e.g., "Starlink"). If you don't use the Wi-Fi gate, leave as default.
   - `SSID_2GHZ` — your 2.4 GHz Wi-Fi name if separate (e.g., "Starlink_2GHZ"). If you only have one network, leave as default — it won't cause any issues.
   - `TEMP_UNIT` — set to `C` for Celsius or `F` for Fahrenheit. All tier thresholds and adjustments use whichever unit you choose. Conversion to/from the API is automatic.
   - **Tier dictionary** — each row is `threshold:adjustment`, using your chosen temperature unit. Use `-N` to cool (e.g., `30:-3` lowers cabin by 3° when outside hits 30°), `+N` to heat (e.g., `10:+2` raises cabin by 2° when outside drops to 10°), a plain number to set an absolute target (e.g., `35:18` sets cabin to exactly 18° when outside hits 35°), or `0:0` (or just `0`) to disable a tier. See Configuring Tiers below for more details and examples.
2. **Create automations** in the Shortcuts app (Automation tab → Time of Day). Create one for each time you want the shortcut to check. Hourly is a good starting point (e.g., 8 AM through 4 PM), but you can use any interval — for example, every 30 minutes during peak heat hours for faster response. Set each automation to run this shortcut with **"Ask Before Running" turned OFF**.
3. **Turn on Camp Mode** in your Tesla and go to sleep. The shortcut handles the rest.

To change any of these settings later, open the shortcut in the editor and modify the values directly.

The shortcut only does anything when Camp Mode is active. If Camp Mode is off, it quietly exits. It resets automatically each day.

**Important:** Before your first camping trip, read the **First Run — Permissions Setup** section below. iOS requires you to manually approve several permission dialogs before the shortcut can run unattended — if you skip this, the shortcut will fail silently while you're asleep.

---

## Configuring Tiers

The tier dictionary is the heart of the shortcut. It appears as a single block in the Shortcuts editor with 8 rows labeled `a` through `h`. Each row contains a `threshold:adjustment` pair.

### Format

Each tier value is a string with two numbers separated by a colon or space:

```
threshold:adjustment
```

- **Threshold** — The outside temperature that triggers this tier (in your configured unit — °C or °F)
- **Adjustment** — What to do with the cabin temperature when triggered

> **Note:** The examples below use Celsius. If you've set `TEMP_UNIT` to `"F"`, use Fahrenheit values instead (e.g., `86:-3` instead of `30:-3`).

### Adjustment Types

| Format | Meaning | Example |
|--------|---------|---------|
| `-N` | Cooling delta — lower cabin by N°C | `30:-3` → when outside hits 30°C, drop cabin by 3°C |
| `+N` | Heating delta — raise cabin by N°C | `10:+2` → when outside hits 10°C, raise cabin by 2°C |
| `N` | Absolute — set cabin to exactly N°C | `35:18` → when outside hits 35°C, set cabin to 18°C |
| `0:0` or `0` | Disabled — this tier is skipped | |

### Cooling Tiers

Cooling tiers fire when the outside temperature **rises to or above** the threshold. Use negative adjustments:

```
a    25:-2     ← When outside >= 25°C, lower cabin by 2°C
b    30:-3     ← When outside >= 30°C, lower cabin by 3°C more
c    35:-4     ← When outside >= 35°C, lower cabin by 4°C more
```

Deltas are cumulative. If all three fire, the cabin drops by a total of 9°C from its original setting.

### Heating Tiers

Heating tiers fire when the outside temperature **drops to or below** the threshold. Use positive adjustments:

```
d    15:+2     ← When outside <= 15°C, raise cabin by 2°C
e    10:+3     ← When outside <= 10°C, raise cabin by 3°C more
f     5:+4     ← When outside <= 5°C, raise cabin by 4°C more
```

### Absolute Targets

Instead of adjusting by a delta, you can set the cabin to an exact temperature:

```
a    35:18     ← When outside >= 35°C, set cabin to exactly 18°C
```

The direction is inferred: if the target (18) is less than the threshold (35), it's treated as cooling. If greater, it's heating.

### Disabled Tiers

Set any tier to `0:0` or just `0` to disable it. You don't need to use all 8 — only configure the ones you need.

### Example Configurations

**Summer camping (cooling only):**

```
a    25:-2
b    30:-3
c    35:-5
d    0:0
e    0:0
f    0:0
g    0:0
h    0:0
```

**Winter camping (heating only):**

```
a    0:0
b    0:0
c    0:0
d    10:+2
e    5:+3
f    0:+5
g    0:0
h    0:0
```

**All-season with gap (separate cooling and heating):**

```
a    25:-2
b    30:-4
c    35:-6
d    10:+2
e    5:+3
f    0:+5
g    0:0
h    0:0
```

**Interleaved (active temperature tracking):**

This advanced configuration places cooling and heating tiers at nearby thresholds, allowing the shortcut to track the temperature curve as it rises and then falls:

```
a    25:-1
b    27:-2
c    29:-3
d    27:+1
e    25:+2
f    23:+3
g    0:0
h    0:0
```

As the temperature rises past 25, 27, and 29°C, cooling tiers progressively lower the cabin temp. When it starts falling back through 27, 25, and 23°C, heating tiers bring it back up.

### Input Format Flexibility

You can format tier values with some flexibility:

- `25:-2` — colon separator (recommended)
- `25 -2` — space separator
- `25 : -2` — spaces around colon

A colon or space between the two numbers is required.

---

## Wi-Fi Gate

The Wi-Fi gate is an optional feature that prevents the shortcut from making unnecessary API calls when you're not in or near your vehicle.

This was designed for setups where the vehicle has its own always-on Wi-Fi source, like a Starlink unit. If your phone is connected to the Starlink network, you're likely in or near the car — so it's appropriate to check Camp Mode. If your phone is connected to a different network (like your home Wi-Fi or a coffee shop), you're clearly away from the vehicle and there's no reason to make API calls.

When enabled (`USE_WIFI_GATE` set to `1`), the shortcut checks your Wi-Fi connection on the first run of the day:

- **Connected to one of your configured SSIDs** (e.g., your Starlink network) → continues normally
- **No Wi-Fi connected** → continues normally (assumes you may be camping without Wi-Fi, or cellular only)
- **Connected to a different Wi-Fi** (like your home network) → disarms for the day

The Wi-Fi gate only runs on the first check of the day. If it detects you're on the wrong network, the shortcut disarms for the entire day — no API calls, no tier evaluation, nothing until the next day's reset. If you pass the Wi-Fi check (correct network or no Wi-Fi), the shortcut arms and subsequent runs skip the Wi-Fi check entirely, relying on Camp Mode status from the API instead.

Set `USE_WIFI_GATE` to `0` to skip this check entirely. This is fine if you don't have a vehicle-mounted Wi-Fi source or if you only want to rely on Camp Mode status.

---

## Temperature Unit

Set `TEMP_UNIT` to `"C"` for Celsius or `"F"` for Fahrenheit. This controls what unit you use for all tier thresholds and adjustments. The Tesla API always communicates in Celsius internally — the shortcut converts automatically in both directions.

When set to Fahrenheit:

- All four API temperatures (outside, cabin, min, max) are converted from Celsius to Fahrenheit immediately after being read from the API.
- Your tier thresholds and adjustments are compared in Fahrenheit.
- Clamping uses the converted min/max range.
- The display values saved to the state file (`original_temp`, `shifted_temp`) will be in Fahrenheit.
- Before sending a temperature command to the API, the target is converted back to Celsius.

When set to Celsius, no conversion happens at all — the shortcut behaves exactly as if the feature didn't exist.

Note that `prev_outside_temp_c` and `last_outside_temp_c` in the state file are always stored in Celsius regardless of your setting, since they come directly from the API before conversion. This is by design — it keeps the trend comparison consistent across unit changes.

---

## Trend Detection

The shortcut automatically tracks whether the outside temperature is rising or falling by comparing the current reading to the previous run's reading.

- **Rising temperature** → only cooling tiers can fire
- **Falling temperature** → only heating tiers can fire
- **Stable temperature** → nothing fires (waits for the next check)

This prevents unnecessary adjustments — if the temperature is already dropping on its own, the shortcut won't add more cooling.

### First Run Behavior

On the first run of the day, there's no previous temperature to compare against. The shortcut handles this based on your configuration:

- **Separated config** (cooling and heating thresholds don't overlap): tiers fire immediately based on thresholds alone. No delay.
- **Interleaved config** (cooling and heating thresholds overlap): the first run records the current temperature as a baseline and exits. The next run has trend data and can fire normally.

---

## State File

The shortcut saves its state to:

```
iCloud Drive → Shortcuts → camp-mode-state.json
```

This file resets automatically each day. You can inspect it to understand what the shortcut is doing. Here's what each field means:

| Field | Description |
|-------|-------------|
| `date` | Today's date. If stale, state resets on next run. |
| `status` | `unknown` → `armed` → `disarmed`. Stays `armed` while Camp Mode is on. |
| `temp_unit` | The temperature unit in use — `C` or `F`. |
| `temp_shifted` | Whether any cabin temperature change has been made today. |
| `original_temp` | The cabin temperature before the first adjustment of the day (in your configured unit). |
| `shifted_temp` | The cabin temperature after the most recent adjustment (in your configured unit). |
| `tier_a_applied` through `tier_h_applied` | Whether each tier has fired today. |
| `prev_outside_temp_c` | The outside temperature from the run before last (always Celsius from API). |
| `last_outside_temp_c` | The outside temperature from the most recent run (always Celsius from API). |
| `last_check_at` | Timestamp of the most recent run. |
| `last_action_at` | Timestamp of the most recent temperature change. |
| `last_exit_reason` | Why the last run exited — useful for debugging. |

### Exit Reasons

| Reason | Meaning |
|--------|---------|
| `camp_off` | Camp Mode is not active. Shortcut disarmed for the day. |
| `other_wifi_connected` | Connected to non-campsite Wi-Fi. Disarmed for the day. |
| `establishing_baseline` | First run with interleaved config. Recording temperature baseline. |
| `below_threshold` | No tier thresholds were crossed this run. |
| `no_useful_shift` | Tiers fired but the result was the same temperature (already at min/max). |
| `shift_applied` | Temperature was successfully adjusted. |
| `api_read_failed` | Couldn't read vehicle state from Tessie. Will retry next run. |
| `api_command_failed` | Couldn't send temperature command to Tessie. Will retry next run. |
| `outside_temp_missing` | Outside temperature not available from the API. Will retry next run. |
| `temperature_fields_missing` | One or more temperature fields missing from the API. Will retry next run. |

---

## First Run — Permissions Setup

**This is critical.** iOS requires you to grant permissions the first time a shortcut uses certain actions. These appear as popup dialogs with options like "Allow Once," "Always Allow," or "Don't Allow." The catch is that **each permission only appears when that specific part of the shortcut runs.** Different branches of the shortcut trigger different permissions, so a single test run won't cover them all.

If you don't grant all permissions before your camping trip, the shortcut will fail silently when it runs unattended via automation while your phone is locked.

**Note:** The exact permissions and number of prompts can vary between iOS versions and devices. The steps below cover the known ones, but Apple may present additional dialogs we haven't listed here. The fix is always the same — tap **"Always Allow"** on anything that pops up.

### Permissions that may be required

The shortcut uses several actions that typically require permission on first use. The exact prompts vary by iOS version, but these are the likely ones:

| Permission | Triggered by | When it may appear |
|-----------|-------------|-------------|
| iCloud Drive file access | Reading/writing the state file | Every run — likely on the very first run |
| Wi-Fi network info | Checking your current SSID | Only on the first run of the day, and only if `USE_WIFI_GATE` is `1` |
| Network access (GET) | Reading vehicle state from Tessie API | When the shortcut reaches the API call |
| Network access (POST) | Sending a temperature command to Tessie | Only if tiers fire and produce a new temperature — requires Camp Mode to be active with outside temp past a threshold |

### How to trigger all permissions

iOS may prompt separately for different operations — for example, reading a file, creating a new file, and overwriting an existing file could each trigger their own dialog. Similarly, the GET request (reading vehicle state) and POST request (sending a temperature command) may prompt separately. The exact number of prompts varies by iOS version.

The safest approach is to **run the shortcut through its full lifecycle manually**, approving every dialog that appears along the way. Tap **"Always Allow"** on every prompt.

**Run 1 — First run of the day (fresh state):**
Delete the state file if it exists (iCloud Drive → Shortcuts → `camp-mode-state.json`). Make sure Camp Mode is ON and you're connected to your Starlink/campsite Wi-Fi (or set `USE_WIFI_GATE` to `0`). Set at least one tier to a threshold the current outside temperature will cross (e.g., if it's 30°C outside, set tier `a` to `25:-1`).

Run the shortcut manually. This first run may trigger prompts for:
- File access (creating and writing the state file)
- Wi-Fi network info (if the Wi-Fi gate is enabled)
- Network access (reading vehicle state and sending a temperature command)

Approve every dialog. Check the state file afterward — `last_exit_reason` should be `"shift_applied"`.

**Run 2 — Subsequent run (existing state):**
Run the shortcut again immediately without deleting the state file. This exercises the "read existing file" and "overwrite existing file" path, which may trigger additional prompts that didn't appear during file creation. Approve any new dialogs.

**Run 3 (optional) — Verify silent execution:**
Run one more time. If no permission dialogs appear and the state file updates correctly, all permissions are fully granted.

If at any point a permission was accidentally denied, you can fix it by opening the shortcut in the editor, tapping the **(i)** info button, going to **Privacy**, and toggling the relevant permissions back on.

### Other settings to configure

- **Allow Running While Locked** — Open the shortcut in the editor, tap the **(i)** info button at the top, go to **Privacy**, and make sure this is enabled. Without this, automations won't run while your phone is locked.
- **Ask Before Running** — On each Time of Day automation, make sure this is turned **OFF**. Otherwise iOS will show a notification asking you to confirm, and the automation won't run if you don't tap it in time.

### Verify everything works

Do a full test cycle before your trip. After granting all permissions:

1. Turn on Camp Mode
2. Set at least one tier that the current outside temperature will trigger
3. Run the shortcut manually
4. Check the state file — `last_exit_reason` should be `"shift_applied"` and `shifted_temp` should show the new temperature

If you see `"shift_applied"`, every permission has been granted and the full pipeline works end to end.

**If the shortcut still fails silently on an automated run**, run it manually one more time — there may be a remaining permission dialog that only appears under specific conditions (like running while locked, or on a particular iOS version). Once you've approved every prompt that appears across a few manual runs, it should run unattended reliably going forward.

---

## Troubleshooting

**The shortcut doesn't seem to do anything:**
Check the state file. Look at `last_exit_reason` for why it exited. Make sure Camp Mode is actually on in the Tesla app. If `status` is `disarmed`, delete the state file and try again (it won't re-arm until the next day otherwise).

**Tiers aren't firing even though the temperature is past the threshold:**
Check `prev_outside_temp_c` and `last_outside_temp_c` in the state file. If the temperature is falling and you have cooling tiers, trend detection will block them — this is by design. For interleaved configs, the first run always records a baseline and doesn't fire. Wait for the second run.

**The shortcut disarmed immediately:**
If `last_exit_reason` is `camp_off`, your Tesla isn't in Camp Mode. If it's `other_wifi_connected`, you're on a Wi-Fi network that isn't in your SSID list. Either add the network to your config or set `USE_WIFI_GATE` to `0`.

**I changed my tier config but old tiers are still marked as applied:**
The state file persists within a day. Delete `camp-mode-state.json` from iCloud Drive → Shortcuts to force a fresh start.

---

## Notes

- All temperature values use the unit you configure (`TEMP_UNIT` — either `"C"` for Celsius or `"F"` for Fahrenheit). The Tesla API uses Celsius internally, but conversion is handled automatically.
- Cabin temperature is clamped to the Tesla's allowed range (typically 15–28°C / 59–82°F) as reported by the API.
- The shortcut does not run in the background. It only executes when triggered by your iOS automations, so there is no battery drain between runs.
- The state file lives in iCloud Drive and survives app closures, device restarts, and software updates.
- Not affiliated with Tesla or Tessie.
