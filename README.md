# zen-scripts

GPC scripts for the Cronus Zen. Load a `.gpc` file into a Zen Studio slot,
compile, and program it to the device.

## bo7/aim_jiggle.gpc

Call of Duty: Black Ops 7 script with three independent features, four
tuning profiles, primary/secondary weapon tracking, and an on-device OLED
config menu. PlayStation button names are used throughout (L1 = aim, R1 =
shoot, Triangle = swap weapon, Circle hold = knife).

### Weapon slots

The script keeps track of whether you are holding your primary or secondary
by watching Triangle. A release within 250 ms counts as a swap and flips the
slot. Held longer, it was an armor plate and nothing changes. Triangle always
reaches the game untouched.

Each slot owns its own profile and its own rapid fire state, so one tap of
Triangle swaps both. The idle screen shows both slots with a chevron on the
one in hand:

```
>PRI  MP AR  R:OFF
 SEC  SUB    R:ON
```

Holding Circle pulls out the knife. The script marks that after 250 ms and
shows KNIFE OUT on the top row. Tapping Triangle then puts the knife away and
returns you to the same weapon, so the slot does not flip. Mods keep running
with the knife out.

The slot and knife state are not saved. The script boots on primary with the
knife away. It also drifts when the game swaps for you: respawns, picking a
gun up off the ground, or tapping Triangle while carrying only one weapon.
When the chevron disagrees with your hands, hold D-pad Left and press Up to
force primary with the knife away.

### Profiles

Four profiles with fixed names hold their own copy of every tuning setting
(menu pages 2 through 11). Each weapon slot points at one profile. Rapid fire
is per slot. TAP MS is global because it depends on your frame rate, not the
weapon.

| Profile | OLED name | Preset | Intended for |
|---|---|---|---|
| SMG | SUB | Pull 15, fade to 50% over 600 ms, backoff 60, FIG8 jitter 4 x 0 | Close range, aim assist does less work |
| Warzone AR | WZ AR | Pull 30, fade to 60% over 1500 ms, backoff 75, OVAL jitter 10 x 4, lap 100 ms | Long range, sustained fire |
| Multiplayer AR | MP AR | Pull 25, fade to 50% over 1000 ms, backoff 70, FIG8 jitter 8 x 0 | Mid range, the general defaults |
| Misc | MISC | Same as MP AR | Whatever you want |

The presets are starting points, not measurements. Tune each one and it is
saved independently.

| Action | Input |
|---|---|
| Next profile for the current slot | Hold D-pad Up, press Right |
| Previous profile for the current slot | Hold D-pad Up, press Left |
| Feedback | Rumble pulses once per profile number (SUB = 1, MISC = 4). Idle OLED updates |

The first menu page, PROFILE, also cycles the current slot's profile with Up
or Down and reloads the other pages with that profile's values right away.
The menu edits whichever slot you were on when you opened it, and whatever
profile is showing when you press X is the one your edits are saved into.

Fresh defaults: primary on MP AR, secondary on SUB, rapid fire off on both.

### Aim-assist jitter

Runs whenever L1 is held, firing or not. The right stick traces a small
continuous pattern so it never sits still. Call of Duty only applies rotational
aim assist while the stick is moving, so this keeps it engaged while you track
a target.

- PATTERN picks the shape. FIG8 is a sideways infinity that crosses the center
  twice per lap and looks like a natural wander. OVAL is an ellipse that never
  crosses the center, so the stick is always deflected. That is the strongest
  aim-assist signal but reads as a small circle.
- X JITTER and Y JITTER bound the pattern's half-width and half-height.
  Y JITTER 0 turns either shape into a smooth horizontal sweep.
- JITTER MS is the time for one full lap of the pattern.
- LS JITTER adds a smooth left/right micro-strafe to the left stick while ADS,
  but only while the left stick is idle, so it never disturbs deliberate
  movement. This is experimental. 0 disables it.

The jitter cannot cancel the game's idle sway. Sway has a phase the script
cannot see, so any fixed pattern adds motion as often as it subtracts. Reduce
sway with attachments and perks. The jitter's only job is aim assist.

**Lower the in-game right-stick deadzone.** The jitter values are small on
purpose. The default deadzone in Call of Duty swallows stick movement of
roughly 10 units, so with a stock deadzone the game may never see the jitter
at all. Drop the right-stick minimum deadzone in the controller settings and
verify that the crosshair visibly wobbles while ADS.

A symmetric wobble nets to zero, so the jitter does not counter horizontal
recoil. X CENTER does that.

### Anti-recoil

Runs while L1 and R1 are held together. Pulls the right stick down by PULL
DOWN and sideways by X CENTER, then scales both by two factors:

- **Fade.** Full strength on the first shots, ramping linearly down to FADE TO
  percent after FADE MS of sustained fire. Release the trigger and it resets.
  Most recoil curves are front-loaded, so a constant pull over-corrects late in
  the magazine. Set FADE TO to 100 for a constant pull.
- **Backoff.** Reads your raw right-stick deflection. Below 35 percent of
  BACKOFF the anti-recoil is at full strength. At BACKOFF and above it is off.
  Linear in between. Flick or track hard and the script gets out of your way
  instead of fighting you.

### Rapid fire (semi-auto weapons)

Toggled on and off in game. While on, holding R1 pulses the trigger so a
semi-auto fires as fast as the game allows. Works for hip fire and ADS, and
runs alongside the aim mods when both are active.

| Action | Input |
|---|---|
| Toggle rapid fire for the current slot | Hold D-pad Right, press Down |
| Feedback | Two rumble pulses = ON, one pulse = OFF. Idle OLED shows R:ON / R:OFF per slot |

The script does not try to know each weapon's fire rate cap. Taps that land
during a weapon's cooldown are ignored by the game, so tapping faster than the
cap costs nothing and the effective rate settles just under the cap. One TAP MS
value therefore works for every semi-auto.

### Config menu

| Action | Input |
|---|---|
| Enter config | Hold D-pad Left, press Down |
| Resync to primary, knife away | Hold D-pad Left, press Up (in game) |
| Change page | D-pad Left / Right (wraps) |
| Adjust value | D-pad Up / Down |
| Reset the current profile to its preset | Square |
| Save and exit | X |

Config-mode buttons are blocked from reaching the game.

Defaults below are the MP AR preset. Pages 2 through 11 are per profile.

| Page | Setting | Range | Step | MP AR preset | What it does |
|---|---|---|---|---|---|
| 1 | PROFILE | SUB / WZ AR / MP AR / MISC | cycle | MP AR | Which profile the following pages edit |
| 2 | PULL DOWN | 0 to 50 | 1 | 25 | Vertical anti-recoil strength |
| 3 | X CENTER | -30 to +30 | 1 | 3 | Horizontal anti-recoil bias, fights left/right drift |
| 4 | FADE TO % | 0 to 100 | 10 | 50 | Anti-recoil strength after FADE MS of fire. 100 = no fade |
| 5 | FADE MS | 100 to 2000 | 100 | 1000 | Time to ramp from full strength to FADE TO % |
| 6 | BACKOFF | 30 to 100 | 5 | 70 | Stick deflection at which anti-recoil is fully off |
| 7 | PATTERN | FIG8 / OVAL | toggle | FIG8 | Jitter shape |
| 8 | X JITTER | 0 to 20 | 1 | 8 | Pattern half-width |
| 9 | Y JITTER | 0 to 20 | 1 | 0 | Pattern half-height. 0 = horizontal sweep |
| 10 | JITTER MS | 20 to 500 | 10 | 80 | Time for one full lap of the pattern |
| 11 | LS JITTER | 0 to 20 | 1 | 0 | Left-stick strafe while ADS and idle. 0 = off |
| 12 | RAPID | ON / OFF | toggle | OFF | Rapid fire on or off (per weapon slot) |
| 13 | TAP MS | 10 to 100 | 5 | 30 | Rapid fire hold and release time per tap (global) |

### Settings persist

All four profiles, each slot's profile and rapid fire state, and TAP MS are
saved when you exit the menu with X, switch profiles, toggle rapid fire, or
reset with Square. They survive power cycles. Which slot you are holding is
not saved.

The Zen only gives a script 16 persistent slots, so every setting is stored as
a step index and bit-packed into a stream across the slots. A full profile
takes 47 bits and the whole save takes 207 of the 240 available. Bit widths
are derived from the settings tables, so a new setting packs automatically. A
magic number at the front of the stream guards the save: when the tables or
profile layout change in a future version, the script bumps the magic and
falls back to the presets instead of reading stale bits.

### Choosing TAP MS

Each tap holds R1 for TAP MS then releases it for TAP MS, so one full tap takes
twice the value. The game samples the controller once per frame, about 17 ms at
60 fps and 8 ms at 120 fps, and a press or release shorter than a frame can be
missed. That sets the floor. Human tapping speed sets the ceiling of what is
worth configuring.

| TAP MS | Taps per minute | Comparable to |
|---|---|---|
| 100 | 300 | A quick human (5 taps/s) |
| 60 to 85 | 360 to 480 | A fast human sustained over a magazine |
| 40 to 50 | 600 to 720 | An elite human burst |
| 30 | 1000 | Default. Above any human rate, safe at 60 fps |
| 20 | 1500 | Works at 120 fps, marginal at 60 fps |
| 10 | 3000 | Below one 60 fps frame, taps get dropped |

Calibrate once, since the right value depends on your frame rate rather than
the weapon:

1. In the firing range with a semi-auto, turn rapid fire on and hold R1 for
   10 seconds. Note the ammo used.
2. Lower TAP MS by 5 and repeat.
3. Stop at the lowest value where ammo used per 10 seconds stops climbing or
   starts dropping.

If a specific weapon misbehaves at that value, raise TAP MS for it.
