# MapleStory Classic World on macOS

If Finally the Controller Works receives inputs but MapleStory Classic World
has missing or incorrect controls, the launcher's SDL2 backend may need an
explicit mapping for the app's virtual gamepad.

This mapping was tested on macOS 27 with a Switch 2 Pro Controller, the app's
CoreHID virtual gamepad, and the MapleStory launcher's bundled SDL 2.30.12.
SDL initially reported the virtual device as an unmapped joystick; after
adding the mapping it recognized a GameController, and the player confirmed
the game's controls worked after a fresh launch.

## Check the virtual gamepad first

1. Keep the release app in `/Applications`, launch it, and connect the
   controller before starting the game.
2. Check the app's live log for `system-wide virtual gamepad created` for the
   connected slot. A Bluetooth connection or moving inputs in the app's
   dashboard alone does not establish that macOS created the virtual device.
   If creation is refused, resolve that first; an SDL mapping cannot create
   a virtual gamepad.
3. Log out of the game, then quit **both MapleStory Classic World and
   MapleStory Launcher completely** before editing the bottle configuration.

## Add the SDL2 mapping

The tested launcher's configuration is:

```text
$HOME/Library/Application Support/MapleStoryNA/Bottles/maplestory/cxbottle.conf
```

`$HOME` means your home directory. If your installation uses a different
bottle, locate its `cxbottle.conf` rather than creating a new file at this
path.

Make a backup before editing. In Terminal, for the path above:

```sh
maple_config="$HOME/Library/Application Support/MapleStoryNA/Bottles/maplestory/cxbottle.conf"
cp -p "$maple_config" "$maple_config.ftcw-backup-$(date +%Y%m%d-%H%M%S)"
```

Open the original file in a plain-text editor. Under its **existing**
`[EnvironmentVariables]` section, add the following as one physical line,
leaving all other settings intact:

```ini
"SDL_GAMECONTROLLERCONFIG" = "03003c347e0500006920000000000000,Finally Switch 2 Pro,a:b0,b:b1,x:b2,y:b3,back:b4,guide:b5,start:b6,leftstick:b7,rightstick:b8,leftshoulder:b9,rightshoulder:b10,leftx:a0,lefty:a1,rightx:a2,righty:a5,lefttrigger:a3,righttrigger:a4,dpup:h0.1,dpright:h0.2,dpdown:h0.4,dpleft:h0.8,"
```

If `SDL_GAMECONTROLLERCONFIG` already exists, do not add a duplicate key or
discard its other mappings. The value supports newline-separated mapping
rows: retain the other rows and add this device's row, or replace only the
row for this exact GUID. Preserve the configuration's existing string and
newline escaping when editing a value that contains multiple rows. The
one-line example above is the tested case with no previous SDL mapping.

Save the file, keep Finally the Controller Works running with the controller
connected, and relaunch MapleStory Launcher and the game. Enable native
controller support in **Game Options → Controller**. Restart both processes
again after any further mapping change; SDL reads the environment at startup.

## Expected layout

This preserves the game's controller layout shown in its key-binding screen.
SDL's `a`, `b`, `x`, and `y` names describe standard controller positions;
the Nintendo labels on those positions differ.

| Physical control | Game action |
| --- | --- |
| Left stick | Character movement |
| Right stick | Cursor |
| B (bottom face button) | Jump |
| X (top face button) | Attack |
| Y (left face button) | OK |
| A (right face button) | Cancel |
| L shoulder | Sit |
| R shoulder | Pick up |
| Minus | Shortcut |
| Plus | Menu |
| D-pad | The game's configured menu/quick-slot actions |
| ZL / ZR | Left/right quick-slot modifiers |

Individual actions still follow the game's key-binding settings.

## Scope and reusable mapping

The SDL2 mapping is also provided in
[`gamecontrollerdb-macos.txt`](gamecontrollerdb-macos.txt) for applications
that load an external SDL controller database. That file includes SDL's
macOS platform tag. Simply placing it beside a game does not make the game
load it; the MapleStory steps above use the environment variable instead.

The observed device GUID is `03003c347e0500006920000000000000`. Its raw SDL
axes were `a0` = left X, `a1` = left Y, `a2` = right X, `a3` = left trigger,
`a4` = right trigger, and `a5` = right Y. In particular, mapping right Y
from `a3` would mix it up with the left trigger.

This applies to the **Finally virtual Pro Controller 2**, rather than every
device with Nintendo's vendor/product IDs. The GUID depends on the SDL
backend and device identity, including its name and version; axis indices
depend on the virtual HID descriptor and backend. Recheck the GUID and raw
input layout after changes to those components. Physical USB controllers,
other controller models, SDL3, and other macOS versions have not been
validated by this test.

## Undo

Quit the game and launcher completely. Remove only the mapping entry added
above; if it was merged into an existing value, remove only this GUID's
row. Relaunch both processes. Alternatively, restore the backup, but doing
so also discards any other bottle-configuration changes made since that
backup.

SDL references:
[mapping format](https://wiki.libsdl.org/SDL2/SDL_GameControllerAddMapping),
[environment variable](https://wiki.libsdl.org/SDL2/SDL_HINT_GAMECONTROLLERCONFIG),
and [database platform filtering](https://wiki.libsdl.org/SDL2/SDL_GameControllerAddMappingsFromRW).
