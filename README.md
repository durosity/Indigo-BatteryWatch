# BatteryWatch for Indigo

Watches your battery-powered [Indigo](https://www.indigodomo.com/) devices and adds a
reminder to Apple Reminders to replace the batteries when a battery is low or a device
stops responding. When the battery later reads as replaced, the reminder is completed
for you.

- **Plugin ID:** `com.durosity.indigo.batterywatch`
- **Repository:** <https://github.com/Durosity/Indigo-BatteryWatch>
- **Current version:** 0.0.2 (early release)

## Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Setup](#setup)
- [Settings reference](#settings-reference)
- [How it decides](#how-it-decides)
- [The reminder](#the-reminder)
- [Device states and triggers](#device-states-and-triggers)
- [Menu](#menu)
- [Troubleshooting](#troubleshooting)
- [Good to know](#good-to-know)
- [Version history](#version-history)

## Features

- Pick any battery-powered devices to watch. It uses Indigo's native battery level, so it
  works with any device or plugin that reports one.
- Raises a reminder when a battery drops to your threshold.
- Raises a reminder when a device stops responding, which catches devices whose battery
  indicator is stuck on a healthy reading.
- You choose the Reminders list, due date, due time and priority.
- Completes the reminder automatically when the battery shows it has been replaced
  (optional).
- One reminder per device at a time, so you are never flooded.
- Several Battery Monitors with different rules, for example smoke alarms at high
  priority and rarely-triggered sensors with the not-responding check switched off.
- Each monitor is an Indigo device with states you can use in triggers and control pages.

## Requirements

- **[Calendars & Reminders](https://github.com/Durosity/IndigoCalendar) v4.2.11 or later**
  (`com.durosity.indigo.calendar`), installed and enabled, with Reminders access already
  granted to it. BatteryWatch uses it to create and complete the reminders. Earlier
  versions can't report which reminder they created, so auto-complete would not work.
- No third-party Python packages, so there is nothing else to download.

## Installation

1. Install and enable Calendars & Reminders v4.2.11 or later, and confirm it has
   permission to use Reminders (see its README).
2. Install and enable BatteryWatch.

## Setup

1. **Devices > New**, Type: **BatteryWatch**, Model: **Battery Monitor**.
2. Choose the devices to watch. Only devices that report a battery level are listed.
3. Choose a **Reminders list**. The list is fetched from Calendars & Reminders, so that
   plugin must be enabled.
4. Adjust the thresholds, due date and priority if you want, then save.

A device can only be in one Battery Monitor. If some devices need different rules,
create a second monitor for them.

## Settings reference

Settings live on each Battery Monitor device.

| Setting | Default | What it does |
|---|---|---|
| Devices to watch | none | The devices this monitor looks after. |
| Remind me at or below (%) | 20 | A reminder is raised when the battery reaches this level. |
| Battery counts as replaced at or above (%) | 80 | The level that means the battery has been replaced. Must be higher than the low level. |
| Replaced level must hold for (hours) | 24 | The replaced level has to hold this long, without dropping back, before the battery counts as replaced. `0` means immediately. |
| Auto-complete reminders | on | Complete the reminder when the battery counts as replaced. |
| Not responding | on | Also raise a reminder when a device goes quiet. |
| No response for (hours) | 168 | How long a device can stay silent (168 hours is 7 days). |
| Reminders list | none | Where reminders are created. |
| Due date | on | Give the reminder a due date. |
| Due in (days) | 0 | Days from now (0 is today). |
| Due time (HH:MM) | blank | Optional 24-hour time. Blank gives a date-only reminder. |
| Priority | None | None, High (!!), Medium (!) or Low. |

The only plugin-wide setting is **Debug logging** (Plugins > BatteryWatch > Configure).

## How it decides

**Low battery.** Indigo's native battery level is at or below your threshold. Checked
every time a watched device's level changes.

**Replaced.** The level reaches the "replaced" threshold and stays there for the confirm
period (default 24 hours). The reminder is then completed (if auto-complete is on) and
the device is re-armed for next time. If the level drops back below the replaced level
first, the clock starts over.

The confirm period matters because some devices flip between a low reading and a normal
one. Without it, each flip would look like a battery change and a new low battery, and
you would get a reminder, an auto-completion, and another reminder, over and over.

**Not responding.** No sign of life for the chosen number of hours. A sign of life is any
of these, whichever is latest:

- a state change on the device,
- a successful communication with it (for example a Z-Wave wake-up),
- any update BatteryWatch saw for it.

Checked every 5 minutes. Devices that only report when something happens (a rarely used
door, say) can look silent when they are fine, so give those their own Battery Monitor
with *Not responding* switched off.

**One reminder per device at a time.** While a reminder is outstanding nothing more is
raised for that device, however the level bounces. Ignoring the reminder, or ticking it
off yourself, does not cause nagging; the device re-arms once the battery reads as
replaced.

**Start-up grace.** BatteryWatch doesn't judge *Not responding* for the first hour after
it starts, so restarting Indigo doesn't make every device look dead. Low batteries are
reported straight away.

## The reminder

> **Replace batteries: Hallway Motion Sensor**
> Battery level is 12% (reminder threshold: 20%).
> Device: Hallway Motion Sensor (Indigo ID 123456)
> Raised by BatteryWatch on 2026-10-14 09:30.
> This reminder is completed automatically once the battery reads 80% or more.

For a device that has gone quiet, the notes say how long it has been silent, when it was
last heard from, and the last battery level it reported, with a reminder that the
indicator may be wrong.

## Device states and triggers

Each Battery Monitor has these states:

| State | Meaning |
|---|---|
| `status` | Shown in Indigo: "OK", or for example "1 low, 1 not responding" |
| `watchedCount` | Number of devices in the monitor |
| `lowCount` | Devices with a low-battery reminder outstanding |
| `silentCount` | Devices with a not-responding reminder outstanding |
| `attentionList` | The names of the devices that need attention |
| `onOffState` | **On** when anything needs attention |

Use them in triggers (for example "when `lowCount` changes", or "when the monitor turns
on"), conditions and control pages.

## Menu

**Plugins > BatteryWatch > Check Batteries Now** re-checks everything and writes each
watched device's level, how long ago it was last heard from, and its status to the Indigo
log.

## Troubleshooting

**A device isn't in the list.** Only devices that expose Indigo's native battery level
are listed. A device that has never reported a level won't appear until it does.

**The Reminders list is empty or says it needs a newer Calendars & Reminders.** Install
Calendars & Reminders v4.2.11 or later and make sure it is enabled.

**"Calendars & Reminders is older than v4.2.11" in the log.** Reminders are still
created, but BatteryWatch can't tell which one it made, so it can't auto-complete it.
Update Calendars & Reminders.

**No reminder appears.** Check the Indigo log. BatteryWatch keeps the problem on record
and retries every 5 minutes, warning once an hour, so a temporary problem (the other
plugin disabled, a wrong list name) fixes itself once resolved.

**The reminder wasn't completed after I replaced the battery.** It completes only once the
battery has read at or above the replaced level for the whole confirm period (24 hours by
default). **Check Batteries Now** shows "confirming" for devices still waiting.

**A device keeps flipping between a low and a normal reading.** It gets one reminder that
stays open. Turn on Debug logging to see each level change (for example "battery level
100 -> 0") and decide whether the device's reading can be trusted.

**A device is wrongly flagged as not responding.** Raise *No response for (hours)*, or
put it in a Battery Monitor with *Not responding* switched off.

## Good to know

- What BatteryWatch has raised is remembered in
  `Preferences/Plugins/com.durosity.indigo.batterywatch/tracking.json` inside your Indigo
  folder, and survives restarts. Delete the file to make it forget everything (existing
  reminders are not removed).
- If you delete a reminder yourself, BatteryWatch notices when the battery is replaced and
  carries on quietly.
- Removing a device from a Battery Monitor, or deleting the monitor, stops tracking it. Any
  reminder already created is left in Reminders.
- This is an early release. Please open an issue if something doesn't behave as described.

## Version history

### v0.0.2

- **Fixed: repeated reminders for devices that flip between a low and a normal reading.**
  Such a device was being reminded and auto-completed over and over. The replaced level
  now has to hold for a confirm period (default 24 hours, set per Battery Monitor) before
  the reminder is completed. Debug logging now records each battery level change.

### v0.0.1

First build. Low-battery and not-responding reminders through Calendars & Reminders, with
auto-complete when the battery is replaced.
