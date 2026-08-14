# Advanced Ring Keypad V2 Blueprint for Home Assistant (Alarmo)

This blueprint synchronizes a Ring Keypad V2 with Home Assistant and Alarmo. It builds upon previous work from [ImSorryButWho](https://github.com/ImSorryButWho) but includes significantly more advanced features like environmental alarms, post-alarm warnings, customizable panic buttons, and rich mobile notifications.

## ✨ Features

- **Full Alarmo Sync:** Perfectly syncs Armed, Disarmed, Arming (Exit Delay), and Pending (Entry Delay) states.
- **Custom Alarmo Mode Mapping:** You can independently map the keypad's physical "Arm Home" and "Arm Away" buttons to specific Alarmo modes, including Home, Away, Night, Vacation, or Custom.
- **Actionable Mobile Notifications (Optional):** Receive interactive iOS/Android notifications directly synced with your alarm.
  - **Dynamic Buttons:** Notification action buttons automatically inherit your configured `keypad_arm_home_mapping` and `keypad_arm_away_mapping` choices.
  - **Persistent Resend Logic:** Intercepts `mobile_app_notification_cleared` events. If you accidentally swipe away an active alarm notification, it automatically resends to ensure you don't lose access to the quick actions.
  - **Platform Enhancements:** Leverages Android custom channels, UI colors, dynamic status icons, and custom vibration patterns, as well as iOS `thread-id`/`apns-collapse-id` grouping. _(Disclaimer: not all advanced visual features work on all phones/OS versions)._
- **Auto Force Arm on Open Sensors (Optional):** If arming fails due to open sensors, a customizable countdown timer will begin. If left unresolved, the blueprint automatically bypasses the open sensors and force arms the system. The countdown can be canceled pressing the cancel option in the notification.
- **S2 Security Collision Workaround:** Includes a customizable delay toggle (0-3 seconds) to prevent the Z-Wave race condition (#4879) that causes the keypad to flash a red network error and fail to update its status when disarming.
- **Bypasses Z-Wave JS Limits:** Uses raw `invoke_cc_api` commands to bypass Home Assistant's strict indicator filters, ensuring all special alarms actually work.
- **Environmental Alarms (Optional):** Connect your environmental sensors (Smoke, CO, Moisture, Gas, Heat). If any of these trigger, the keypad will sound the loud, permanent Fire/CO siren until disarmed. You can leave this blank if you don't have any, and the blueprint will still work perfectly.
- **Smart "Post-Alarm" Warning:** If your burglar alarm triggers while you are away, the siren will sound. When the alarm times out and re-arms itself (make sure this is enabled in Alarmo), the keypad will flash the **Medical button** as a silent warning/indicator. When you come home, you instantly know an incident occurred before you even enter your code!
- **Custom Panic Buttons:** Map any Home Assistant action to the physical Fire, Police, and Medical buttons on the keypad (Requires holding the button down for 3 seconds).
- **Local Panic Sirens:** Independent toggles allow you to choose whether holding a panic button also activates the keypad's internal siren/flashing lights, turning it into a true standalone panic station.
- **Instant Silence "Kill-Switch":** Typing your code and pressing 'Disarm' will instantly force the keypad to silence any sounding sirens, regardless of the current state of Alarmo.
- **Brute Force Protection:** Triggers the burglar alarm if someone tries to guess your PIN and fails too many times while the system is armed.

## 🙏 Credits

A huge thank you to [ImSorryButWho](https://github.com/ImSorryButWho/HomeAssistantNotes/blob/main/RingKeypadV2.md) for the original research on Z-Wave command classes and the foundational blueprint. This advanced version builds upon that foundation to fix silent alarms, synchronization race-conditions, add custom button mapping, interactable nottifications, optionally auto force arm on unresolved open sensors after a set delay.

## ⚙️ Prerequisites

1. A Ring Keypad V2 paired via **Z-Wave JS**. ⚠️ **IMPORTANT: It MUST be paired using S2 Security (S2 Access Control).** If it is paired with no security or legacy S0, the keypad buttons and sirens will not function correctly!
2. [Alarmo](https://github.com/nielsfaber/alarmo) installed and configured in Home Assistant. (If you use environmental sensors, be sure to add them to Alarmo's environmental category too).
3. Alarmo "Disarm after triggering" option **disabled**.
4. **Counter Helper (Optional):** If you wish to use the Brute Force Protection feature, you must create a Counter helper in Home Assistant (instructions below).

## 🚀 Installation

The easiest way to install this blueprint is by clicking the import button below:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://raw.githubusercontent.com/Ryckie/Ring-Keypad-V2-Advanced-Blueprint/refs/heads/main/ring_keypad_v2_advanced.yaml)

**Manual installation:**
If you prefer to install it manually:

1. In Home Assistant, go to your `config/blueprints/automation` folder.
2. Create a new file called `ring_keypad_v2_advanced.yaml` and paste the raw code from this repository.
3. Go to Developer Tools -> YAML -> Reload Automations.

## 🔒 Setting up Brute Force Protection (Optional)

To enable the feature that triggers the alarm after too many failed PIN attempts, you need to create a simple Counter Helper in Home Assistant:

1. Go to **Settings** -> **Devices & Services** -> **Helpers** or click the helpers button below.

[![Open your Home Assistant instance and show your helper entities.](https://my.home-assistant.io/badges/helpers.svg)](https://my.home-assistant.io/redirect/helpers/)

2. Click **+ Create Helper** and select **Counter**.
3. Name it something like "Keypad Failed Attempts" and leave the rest as default.
4. When setting up this blueprint, toggle "Brute Force Protection" to true, select your newly created Counter entity, and choose your maximum allowed attempts.

## 💡 How the Post-Alarm Warning Works

The Ring Keypad hardware prioritizes the Burglar siren. To ensure the post-alarm warning works, this blueprint will:

1. Wait for Alarmo's siren timer to expire.
2. Officially send the "Armed Away/Home" command to the keypad to kill the siren.
3. Wait exactly 6 seconds for the keypad to finish its voice prompt ("Away and Armed").
4. Send the raw `invoke_cc_api` command to flash the Medical light permanently until you disarm the system.

## 🛡️ Note on Panic Buttons

To prevent conflicting states, the physical Fire, Police, and Medical buttons are programmed to be ignored if your alarm system is currently in the `pending` (Entry Delay) or `triggered` (Siren Sounding) states.

### 🐛 Support / Issue Reports

If you run into a problem or want to report an issue, please open a ticket on: [GitHub Issues](https://github.com/Ryckie/Ring-Keypad-V2-Advanced-Blueprint/issues)

### ☕ If You Would Like To Thank Me 🤟

<a href="https://www.buymeacoffee.com/apradagl9g" target="_blank"><img src="https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20coffee&emoji=&slug=apradagl9g&button_colour=FFDD00&font_colour=000000&font_family=Cookie&outline_colour=000000&coffee_colour=ffffff" alt="Buy Me A Coffee" height="40"></a>
<a href="https://patreon.com/Ryckie" target="_blank"><img src="https://img.shields.io/badge/Patreon-F96854?style=for-the-badge&logo=patreon&logoColor=white" alt="Support on Patreon" height="40"></a>
<a href="https://github.com/sponsors/Ryckie" target="_blank"><img src="https://img.shields.io/badge/Sponsor-EA4AAA?style=for-the-badge&logo=githubsponsors&logoColor=white" alt="GitHub Sponsors" height="40"></a>
