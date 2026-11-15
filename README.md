# Lightwell

----
> Lightwell is a project built for a Garmin hackathon held in Calgary in November 2026.   

----


A Garmin Connect IQ watch app for people who work at a desk. It counts the daylight you get with the watch's solar charging sensor, nudges you outside while the sun is still up, and keeps a small log of coffee, water and tense spells.

Built for the fenix 7 Pro at the Tech Thursday x Garmin Hackathon in Calgary on November 14, 2026. It has run in the simulator only.

## Quick start

You need:

| What | Notes |
|---|---|
| Connect IQ SDK 9.3.0 | Install it with the Connect IQ SDK Manager and set it as current. With 9.1.0 the simulator refused to start the app ("requires a newer SDK"). |
| fenix 7 Pro device files | Download `fenix7pro` in the SDK Manager. |
| VS Code with the Monkey C extension | Also needs a Java runtime. |
| A developer key | In VS Code run "Monkey C: Generate a Developer Key". Keep it out of the repo. |

Run it:

1. Open this folder in VS Code.
2. Press Ctrl+F5 and pick `fenix7pro`. The simulator opens with the app.
3. The first launch shows the setup questions. Answer them and the main screen appears.

Build from a terminal instead (Git Bash):

```bash
SDK="/path/to/connectiq-sdk-win-9.3.0"
java -jar "$SDK/bin/monkeybrains.jar" -o bin/Lightwell.prg -f monkey.jungle -y /path/to/developer_key -d fenix7pro_sim -w
```

Use `-d fenix7pro_sim` for the simulator and `-d fenix7pro` for a watch. `bin/` is build output and is not committed.

## Controls

| Button | Does |
|---|---|
| UP / DOWN | Previous / next page |
| Hold UP | Settings |
| START | Steps the demo light level. On the drinks page it opens the drink log, and on the settings page it opens the focused button. |
| BACK | Session menu: keep going, reset today, or end the session |
| Tap | On the drinks page, left adds a coffee and right adds a water |

## Pages

| # | Page | Shows |
|---|---|---|
| 1 | Well | Minutes of light today against the goal. The well fills from the bottom and turns green when the goal is met. |
| 2 | Sun finder | Where the sun is from where you stand, its height, and the sunset time. Turns with the compass. |
| 3 | Drinks | Coffee and water today, logged by hand, against your limit and goal. |
| 4 | Stress | Minutes today spent tense while sitting still, and the current spell. |
| 5 | Week | Six finished days and today against the goal line. |
| 6 | Month | Thirty days as dots: goal met, some light, none. |
| 7 | Sensors | What each sensor reads right now. |
| 8 | Settings | Current settings, with buttons for the settings menu and the setup questions. |

## Setup and settings

The first launch asks six questions, one per screen. Each answer is a setting you can change later (hold UP).

| Question | Setting |
|---|---|
| How do you get light? | Wording of the nudges |
| Daylight on a workday? | Daily goal in minutes |
| How should I nudge you? | Buzz and tone, soft buzz, or screen only |
| Coffee a day? | Daily cup limit |
| Water a day? | Daily glass goal |
| Tense and sitting still? | Stress nudge after 15 or 30 minutes, or off |

Settings also has an accessibility option that switches the nudges to wheelchair wording.

## Notifications

Each one is a full-screen card. START closes it; BACK means later.

| Card | When |
|---|---|
| Sit nudge | About an hour of sitting while the sun is up and the well is not full. Says how long you sat and how much sun is left. |
| Well is full | The daily goal is reached. |
| Midday check-in | Once a day, past the middle of the daylight with the well under half. |
| Tense and still | A tense spell reaches the length you chose. |
| Coffee limit | A logged coffee reaches your daily limit. |

Nothing buzzes or pops up by itself between 10 PM and 7 AM. While the app is closed, a background service reads the solar sensor every 5 minutes, the shortest interval Connect IQ allows.

## Demo helpers

These exist so the app can be shown in the simulator in a few minutes.

- **START** cycles a forced light level: sensor, 0, 15, 60, 100. Forced levels run 60 times faster and the screen says DEMO. Level 0 stands for an hour at a desk and fires the sit nudge.
- **Settings > Preview nudge** fires the sit nudge.
- **Settings > Demo: new user** wipes all stored data and restarts the setup questions.
- **Settings > Demo: month later** loads a month of sample history. The week page is then labelled as demo data.
- **Settings > Demo: tense spell** adds 20 tense minutes and fires that card.
- **Settings > Demo: midday check** fires the midday card.
- **Location** is pinned to the University of Calgary (51.0775, -114.1331), because the simulator has no sky to fix on. Set `PINNED` to `false` in `source/LightwellModel.mc` to use the GPS fix.

## Sensors

| Sensor | Used for |
|---|---|
| Solar sensor | The light reading that fills the well |
| GPS | Sunrise, sunset, and the sun's direction and height (pinned in the demo build) |
| Compass | Turning the sun finder to match the way you face |
| Accelerometer and steps | Telling time spent moving from time spent sitting |
| Heart rate | Heart rate when you stepped into light against now; a stand-in for stress when there is no stress score |
| Barometer | A pressure drop over 3 hours |
| Stress score | Counting tense minutes while sitting still |

## Project layout

| File | Role |
|---|---|
| `manifest.xml` | App id, target devices, permissions |
| `source/LightwellApp.mc` | Entry point; picks setup or the main view; registers the background service |
| `source/LightwellModel.mc` | All state and sensor reading, once a second |
| `source/LightwellView.mc` | Draws the eight pages |
| `source/LightwellDelegate.mc` | Buttons and taps on the main view |
| `source/SetupView.mc`, `source/SetupDelegate.mc` | The setup questions |
| `source/SettingsDelegate.mc`, `source/StyledMenu.mc` | Settings, drink log and session menus |
| `source/CardView.mc`, `source/CardDelegate.mc` | The full-screen notification cards |
| `source/LightwellService.mc` | Background service |
| `source/Day.mc` | Day rollover, streak, and settings shared with the background service |
| `source/DemoData.mc` | The sample month |
| `source/Palette.mc` | Colours taken from the logo |
| `resources/` | Strings, launcher icon and logo |
| `docs/` | Hackathon planning notes; parts are out of date |

## Limits

- It has run in the simulator only.
- `solarIntensity` is a charge efficiency from 0 to 100, not lux. The cut-off of 5 for "brighter than a room" (`LIGHT_MIN` in `source/Day.mc`) has not been calibrated on a watch.
- In the background one reading stands for the 5 minutes since the last one.
- A sleeve covers the sensor. The app can ask you to uncover the watch but cannot see through fabric.
- The stress page has not been checked against a real stress reading.
