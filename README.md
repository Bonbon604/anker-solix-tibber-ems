# Anker Solarbank 3 - Smart Multi-Strategy EMS (Tibber 15-Min Synchronized)

An advanced, highly optimized Energy Management System (EMS) Blueprint for Home Assistant designed specifically for the **Anker Solarbank 3 (E2700 Pro)**. This automation synchronizes perfectly with **Tibber's 15-minute electricity price slots** to grid-charge your battery with maximum financial or logistical efficiency.

It eliminates relay chatter by strictly operating on 15-minute intervals and supports both time-bound targets and strict electricity price thresholds.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Bonbon604/anker-solix-tibber-ems/blob/main/anker_solix_tibber_ems.yaml)

---

## Requirements

Before importing this blueprint, ensure you have the following integrations fully configured and running in your Home Assistant instance:
1. **[Anker Solix Integration](https://github.com/thomluther/ha-anker-solix/):** To read battery SoC, capacity, and to control the operation mode (`backup`/`smartmeter`) and AC input limits.
2. **[Tibber Integration](https://www.home-assistant.io/integrations/tibber/):** To fetch the 15-minute synchronized dynamic electricity prices.

---

## Key Features

* **15-Minute Synchronization:** Fully aligned with Tibber's 15-minute market intervals. Eliminates rapid on/off switching and minimizes relay wear.
* **Dual-Optimization Strategies:**
    * **Target Completion Time:** Guarantees your battery reaches the desired SoC by a specific deadline using either distributed slots or a single consecutive block.
    * **Price Cap / Threshold:** Bypasses deadlines entirely, charging the battery only when the market price drops below your specified EUR/kWh threshold.
* **Flexible Charging Profiles (for Target Time):**
    * **Dynamic / Split:** Automatically picks the absolute cheapest 15-minute intervals before your deadline, even if they are scattered.
    * **Continuous / Block:** Computes a moving window to find the cheapest *consecutive* block of slots, ensuring a single, uninterrupted charging cycle.
* **Smart Dynamic Notifications:** Send rich push notifications to your phone about specific events. The messages automatically append context-aware warnings if the system is left in *Run Always* mode versus *Run Once*.
* **Automatic Midnight-Crossing & 48h Lookup:** Automatically queries prices up to 2 days ahead, allowing seamless execution when target times cross into the next calendar day (e.g., setting a target for 06:00 AM on the previous evening).
* **Time-Deficit Panic Mode (Target Time Only):** If time is running out and the target completion time is approaching, the system overrides price sorting to force-charge the battery, ensuring your target SoC is guaranteed.
* **Offline Fallback Protection:** If Tibber's API is completely unreachable and the local cache is empty, the system automatically falls back to a time-deficit safe mode to ensure you never wake up with an empty battery.
* **Run Once or Run Always:** Supports self-deactivation (`Run Once`) immediately upon reaching the target SoC or passing the target completion time.

---

## Technical Details & Variables

The automation evaluates the system state every 15 minutes (`:00`, `:15`, `:30`, `:45`) and computes the exact number of required slots based on your hardware configuration:

$$\text{Slots Required} = \left\lceil \frac{\text{Needed Wh}}{\text{AC Limit W} \times 0.25\,\text{h}} \right\rceil$$

### Lookback Allowance
The system incorporates a **900-second (15-minute) lookback window** when filtering price arrays. This mathematical buffer ensures that if the automation executes slightly late or is triggered manually mid-slot (e.g., at 17:08 instead of 17:00), the currently active slot is correctly captured and evaluated instead of being discarded as past data.

---

## Configuration Inputs

### Hardware & Entities
| Input Field | Type | Description |
| :--- | :--- | :--- |
| **Solarbank Usage Mode Select** | `select` | The `select` entity controlling your Anker Solarbank's operation mode. |
| **Default Smart Mode Option** | `options` | The normal everyday mode (e.g., `smartmeter`) to restore when grid charging ends. |
| **Solarbank Backup Option Switch** | `switch` | The AC grid-charging toggle switch entity. |
| **Solarbank AC Input Limit** | `number/sensor` | Entity reflecting the grid charging power limit in Watts (W). |
| **Solarbank Battery Capacity** | `select` | Sensor providing total system capacity in Wh. |
| **Battery State of Charge (SoC)** | `sensor` | The current battery percentage sensor. |

### Optimization & Strategy Settings
| Input Field | Type | Description |
| :--- | :--- | :--- |
| **Optimization Strategy** | `options` | Choose between `Target Completion Time` or `Price Cap / Threshold`. |
| **Charging Profile** | `options` | Choose `Dynamic` (scattered cheap slots) or `Continuous` (uninterrupted block). Only used in Target Time mode. |
| **Price Cap Threshold** | `number` | Price limit in €/kWh. Only used in Price Cap mode. |
| **Ignore Target Time in Price Cap Mode** | `boolean` | If enabled, *Run Once* will NOT turn off the automation at the target time while in Price Cap mode. It will keep running across days until Target SoC is fully reached. |
| **Target SoC (%)** | `slider` | The desired battery percentage to achieve. |
| **Target Completion Time** | `time` | The deadline time by which the Target SoC must be met. *Note: UI remains visible but is ignored during Price Cap mode if the ignore toggle is enabled.* |
| **Run Mode** | `options` | `Run Always` to keep active, or `Run Once` to turn the automation off after target completion or expiration. |

### Notifications (Optional)
| Input Field | Type | Description |
| :--- | :--- | :--- |
| **Notification Device** | `device` | Select the mobile device to receive event push notifications. Leave empty to disable notifications. |
| **Notification Events** | `select (multiple)` | Multi-select drop-down to pick triggers: `Grid Charging Started`, `Target SoC Reached`, or `Target Completion Time Passed`. |

---

## Intelligent Notification Text Behavior

Instead of spamming duplicate alerts, notifications are merged into the core process events. The automation checks the `Run Mode` configuration on the fly and alters the messages accordingly:

* **On Target SoC Reached:**
  * *Run Once:* `"Target SoC reached successfully! Battery is at 60%. Automation will now self-deactivate."`
  * *Run Always:* `"Target SoC reached successfully! Battery is at 60%. ⚠️ Note: Automation is in ALWAYS RUN mode and remains active for future slots."`
* **On Target Completion Time Passed:**
  * *Run Once:* `"Target completion time passed. Battery is at 48% (Target: 60%). Automation will now self-deactivate."`
  * *Run Always:* `"Target completion time passed. Battery is at 48% (Target: 60%). ⚠️ Note: System remains active in ALWAYS RUN mode."`

---

## Installation & Setup

### Method 1: Direct Import (Recommended)
1. Click the **Import Blueprint** button badge below to add this automation directly to your Home Assistant:

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https://github.com/Bonbon604/anker-solix-tibber-ems/blob/main/anker_solix_tibber_ems.yaml)

2. Follow the on-screen prompts in your Home Assistant UI to review and approve the import.

### Method 2: Manual Installation
1. Manually create a new file in your `/config/blueprints/automation/` directory named `anker_solix_tibber_ems.yaml`.
2. Paste the full blueprint YAML code into the file and save.
3. Reload your Blueprints or restart Home Assistant.

---

## Hardware Limitations & Safety Disclaimers

*This EMS blueprint relies heavily on cloud-to-cloud synchronization between the Tibber API and the Anker Solix API. The Anker Solarbank 3 relies on internal relays to switch into Backup/Grid-Charging mode. While this blueprint introduces a strict 15-minute interval synchronization to minimize relay chatter, users must monitor the system for hardware switching frequency to prevent premature hardware wear. Ensure that your AC charging power limits are calibrated accurately to reflect your hardware variant's thermal and electrical tolerances.*

---

## Credits & Inspiration

*Inspired by [Adcompro/ha-anker-solix-tibber-blueprint](https://github.com/Adcompro/ha-anker-solix-tibber-blueprint)*
