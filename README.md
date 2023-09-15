# Kinesis Advantage 360 Pro ZMK Config

## How to get neo layouts?

The advantage 360 pro is programmable via zkm, so ideally we'd like to just programm the neo behaviour onto the 
keyboard and save us the setup on the operating system. This is not so easy however because a keyboard does not 
actually send letters to the PC but keys (HID codes to be exact). And there is for example no `ℓ` key. Another thing is dead key behavior:
The keyboard sends a grave accent (or rather its key) and then the letter e and then the os turns that into `è`. 
If we want the keyboard to do this we'd have to send nothing at all for the grave accent but somehow save this user 
intent in a "grave accent has been pressed" state and then send `è` when e is pressed. And again we cannot simply send 
`è` because there is no `è` key.  
It seems generally possible to program all of this onto the keyboard, but it is a lot of work and so to start we keep 
it simple and make the keyboard work with existing keyboard layouts for the operating systems.

### with keyboard layouts on os

The kinesis advantage 360 pro has fewer keys than a regular keyboard, fewer even then the previous model.
In order to be able to have full functionality we need to address this first:

#### F keys are on the digits  
On a regular keyboard they have separate keys, on the kinesis you press `fn` + `1` for `F2`.
This is fine, we'll keep the fn layer and add any non neo functionality there.

#### No `Print` key
We use the `fn` layer: `fn` + `p` = `Print`.

#### `` ` `` `=`  `<` rotation  

| Key        | US layout                                                    | Kinesis                                | we use      | reason                                                                                          |
|------------|--------------------------------------------------------------|----------------------------------------|-------------|-------------------------------------------------------------------------------------------------|                   
| `<`        | at the bottom left, to the left of `z`                       | not there                              | not there   | not on base layer anyway                                                                        |
| `` ` (~)`` | left of `1`                                                  | below `z` close to where ´>´ should be | left of `1` | easier to learn                                                                                 |
| `=`        | on the right end of the digit row, 2 to the right of `0` and | left of `1`                            | below `Z`   | has to move anyway (we could also put it on `\ ` but for now I have decided to use it as Mod 4) |

#### Modifiers  

| Layer | Keys intended     | we use                     | reason                |
|:-----:|-------------------|----------------------------|-----------------------|
|   1   | none              | none                       | -                     |
|   2   | Shift             | Shift                      | -                     |
|   3   | Caps lock         | Caps or `\ `               | easy access           |
|   4   | alt gr (Mod 4)    | Caps or `\ ` a second time | not often used anyway |
|   5   | Shift + Caps lock | Caps or `\ ` a third time  | "                      |
|   6   | Caps lock + Mod 4 | Caps or `\ ` a fourth time | "                      |

I originally wanted to use fn fom layer 3 on, but it is difficult to reach, so for start we use the existing keys.

#### Numpad
Kinesis has the numpad as a layer, you press `kp` and existing keys get new meaning.
Neo treats them as separate keys. I guess we need to create separate layers that get activated on `kp` + <regular modifier>
but I don't see myself using kp anytime soon, so I'll skip it for now.

### Flashing neo directly onto the keyboard (no os keyboard layout)

To put everything on the keyboard we need to overcome several challenges.

#### shift behaviour

The keyboard does not send `a` and `A` to the computer.
Instead, it sends `[A]` and `[A] + [shift]`.
This is no problem for letters, but it is one for the digit row:
in neo, `[shift] + [7]` means `€`, but how should the computer know that, without a dedicated neo keyboard layout?

https://www.reddit.com/r/ErgoMechKeyboards/comments/ujhp0g/why_dont_zmk_keycodes_separate_upper_and/

##### extra ´[shift]´ key

One way would be to just define a second shift key, lets call it `[shift2]`. 
With this key pressed we switch to a new layer where pressing `[7]` sends `[alt gr] + [E]` = `€`.
This works for all characters on the digit row that exist in the target layout (german).
For all others, such as `ℓ` and maybe `ẞ` on windows we need to use a trick and send unicode characters directly.

##### mod morph behaviour

we can use the same shift key and define a behaviour that send something else when `[shift] + [7]` is pressed.
Problem of unknown characters like `ℓ` remains.

##### zmk-nodefree-config

this repo lets us define 2 arbitrary unicode symbols for any key, one send when shift is not pressed, one when it is.
See Unicode for disadvantages.
https://github.com/urob/zmk-nodefree-config#zmk_unicode
https://github.com/urob/zmk-nodefree-config#international-characters

#### Unicode

Operating systems allow you to input unicode characters directly by their number.
For instance on linux you simply press `[shift] + [ctl] + [u]` and then enter the number and conform with `[enter]`.
Kinesis can send many key presses at once so this is no problem.
But, on windows unicode characters need a different activation.

https://github.com/urob/zmk-nodefree-config/issues/2

So you would need to define every key twice, once for windows, once for linux, and thus define every layer twice, 
and then somehow switch between them when you switch os. 
I'm not sure if such an implicit state (layer, os) can ve maintained with zkm.



#### Dead keys ( ` + e = è )

should work as long as the german layout knows them, otherwise there will be a problem.

https://en.wikipedia.org/wiki/Combining_character

#### Layers

should we do anything different?
Maybe fn key as Mod 3+ and long press for f-behavior?

### numpad

über kp taste einfach x mal drücken und einmal zum aufheben?


### locale generator

https://github.com/joelspadin/zmk-locale-generator

This fits nowhere else. with the locale generator you can generate your own header to use in the keymap.
E.g. in german layout bottom left is `[Y]` not `[Z]`. 
but in the config you would write `[Z]` because that is the name of the key.
with locale generator you can use `de_Y` instead of `Z`.


## Modifying the keymap

[The ZMK documentation](https://zmk.dev/docs) covers both basic and advanced functionality and has a table of OS compatibility for keycodes. Please note that the RGB Underglow, Backlight and Power Management sections are not relevant to the Advantage 360 Pro's custom ZMK fork. For more information see [this note](#note)

* If you would like to continue using GitHub we recommend using Nick Coutsos’s keymap editor: https://nickcoutsos.github.io/keymap-editor/.
* If you would prefer to leave GitHub and firmware flashing behind you can perform a one-time firmware update to gain access to Clique. Get started here: https://kinesis-ergo.com/360p-clique-upgrade/.

Certain ZMK features (e.g. combos) require knowing the exact key positions in the matrix. They can be found in both image and text format [here](assets/key-positions.md)

## Building the Firmware with GitHub Actions

### Setup

1. Fork this repo.
2. Enable GitHub Actions on your fork.

### Build firmware

1. Push a commit to trigger the build.
2. Download the artifact.

## Building the Firmware in a local container

### Setup

#### Software

* Either Podman or Docker is required, Podman is chosen if both are installed.
* Make is also required

#### Windows specific

* If compiling on Windows use WSL2 and Docker [Docker Setup Guide](https://docs.docker.com/desktop/windows/wsl/).
* Install make using `sudo apt-get install make` inside the WSL2 instance.
* The repository can be cloned directly into the WSL2 instance or accessed through the C: mount point WSL provides by default (`/mnt/c/path-to-repo`).

#### macOS specific

On macOS [brew](https://brew.sh) can be used to install the required components.

* docker
* [colima](https://github.com/abiosoft/colima) can be used as the docker engine

```shell
brew install docker colima
colima start
```
> Note: On Apple Silicon (ARM based) systems you need to make sure to start colima with the correct architecture for the container being used.
> ```
> colima start --arch x86_64
> ```

#### Ubuntu/Debian specific

```shell
sudo apt-get install docker make
```

### Building the firmware

1. Execute `make` to build firmware for both halves or `make left` to only build firmware for the left hand side.
2. Check the `firmware` directory for the latest firmware build. The first part of the filename is the timestamp when the firmware was built.

### Cleanup

The built docker container and compiled firmware files can be deleted with `make clean`. This might be necessary if you updated your fork from V2.0 to V3.0 and are encountering build failures.

Creating the docker container takes some time. Therefore `make clean_firmware` can be used to only clean firmware without removing the docker container. Similarly `make clean_image` can be used to remove the docker container without removing compiled firmware files.

## Flashing firmware

Follow the programming instruction on page 8 of the [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) to flash the firmware.

### Overview

1. Extract the firmwares from the archive downloaded from the GitHub build job (If using the cloud builder) or the firmware folder (If building locally).
1. Connect the left side keyboard to USB.
1. Press Mod+macro1 to put the left side into bootloader mode; it should attach to your computer as a USB drive.
1. Copy `left.uf2` to the USB drive and it will disconnect.
1. Power off both keyboards (by unplugging them and making sure the switches are off).
1. Turn on the left side keyboard with the switch.
1. Connect the right side keyboard to USB to power it on.
1. Press Mod+macro3 to put the right side into bootloader mode to attach it as a USB drive.
1. Copy `right.uf2` to the mounted drive.
1. Unplug the right side keyboard and turn it back on.
1. Enjoy!

> Note: There are also physical reset buttons on both keyboards which can be used to enter and exit the bootloader mode. Their location is described in section 2.7 on page 9 in the [User Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf) and use is described in section 5.9 on page 14. 

> Note: Some operating systems wont always treat the drive as ejected after the settings-reset file is flashed or may throw a spurious error, this doesn't mean that the flashing process has failed.

### Upgrading from V2 to V3

If you encounter a git conflict when updating your repository to V3.0 please follow the instructions on how to resolve it [here](UPGRADE.md).

Updating from V2.0 based firmwares to V3.0 based firmwares can be a rather complex process. There are reset files for every major firmware revision as well as documentation on the update process available [here](https://kinesis-ergo.com/support/kb360pro/#firmware-updates).

## Versioning

Starting on 11/15/2023 the Advantage 360 Pro will now automatically record the compilation date, branch and Git commit hash in a macro that can be accessed with Mod+V. This will type out the following string: YYYYMMDD-XXXX-YYYYYY, where XXXX is the first 4 characters of the Git branch and YYYYYY is the Git commit hash. In addition to this the builds compiled by GitHub actions are now timestamped and also record the commit hash in the filename. 

## N-Key Rollover

By default this keyboard has NKRO enabled, however for compatibility reasons the higher ranges are not enabled. If you want to use F13-F24 or the INTL1-9 keys with NKRO enabled you can change `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=n` to `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L65)

## Battery reporting

By default reporting the battery level over BLE is disabled as this can cause some computers to spontaneously wake up repeatedly. If you'd like to enable this functionality change `CONFIG_BT_BAS=n` to  `CONFIG_BT_BAS=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L58).

## Modifier indicator color

The color of the CAPS/NUM/SCROLL LOCK indicator LEDs may be configured by specifying a hexadecimal RGB color code. For example, `CONFIG_ZMK_RGB_UNDERGLOW_MOD_COLOR=0xFF0000` would give red indicator colors. In order to set the indicator color on both modules, ensure that both [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig) and [adv360_right_defconfig](/config/boards/arm/adv360/adv360_right_defconfig) have been updated.

## Layer colors

A total of 32 layers are supported by ZMK, with the highest currently active layer displayed using the layer LEDs on each of the left and right modules. All possible colors are listed below; for the first 8 layers the same color is displayed on both modules. After that, only the right module color will cycle through until "rolling over", which will cause the left module color to change as well (and this then repeats). To avoid confusion, the black/off LED color is only used for layer 0.

| Layer # | L/R | Layer # | L/R | Layer # | L/R | Layer # | L/R |
| ---: | :---: | ---: | :---: | ---: | :---: | ---: | :---: |
| 0 | <img valign='middle' src='assets/swatches/000000.svg'/> <img valign='middle' src='assets/swatches/000000.svg'/> | 8 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 16 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 24 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 1 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 9 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 17 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 25 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |
| 2 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 10 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 18 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 26 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> |
| 3 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 11 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 19 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 27 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> |
| 4 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 12 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 20 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 28 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> |
| 5 | <img valign='middle' src='assets/swatches/FF00FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 13 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 21 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 29 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> |
| 6 | <img valign='middle' src='assets/swatches/00FFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 14 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 22 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 30 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 7 | <img valign='middle' src='assets/swatches/FFFF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 15 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 23 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 31 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |

## Changelog

The changelog for both the config repo and the underlying ZMK fork that the config repo builds against can be found [here](CHANGELOG.md).

## Beta testing

The Advantage 360 Pro is always getting updates and refinements. If you are willing to beta test you can follow [this guide from ZMK](https://zmk.dev/docs/features/beta-testing#testing-features) on how to change where your config repo points to. The `west.yml` file that is mentioned is located in config/. [This link](config/west.yml) can take you to the file. Typically you will only need to change the `revision: ` to match the beta branch. There is currently no beta branch available for testing.

Feedback on beta branches should be submitted as a GitHub issue on the base ZMK repository as opposed to this config repository.

In the event of a major update the beta branch may not be compatible with the current mainline version of the config repository. If this is the case it will be detailed here along with instructions on how to update.

## Note

By default this config repository references [a customised version of ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.5) with Advantage 360 Pro specific functionality and changes over [base ZMK](https://github.com/zmkfirmware/zmk). The Kinesis fork is regularly updated to bring the latest updates and changes from base ZMK however will not always be completely up to date, some features such as new keycodes will not be immediately available on the 360 Pro after they are implemented in base ZMK.

Whilst the Advantage 360 Pro is compatible with base ZMK (The pull request to merge it can be seen [here](https://github.com/zmkfirmware/zmk/pull/1454) if you want to see how to implement it) some of the more advanced features (the indicator RGB leds) will not work, and Kinesis cannot provide customer service for usage of base ZMK. Likewise the ZMK community cannot provide support for either the Kinesis keymap editor, nor any usage of the Kinesis custom fork.

## Other support

Further support resources can be found on Kinesis.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals

In the event of a hardware issue it may be necessary to open a support ticket directly with Kinesis as opposed to a GitHub issue in this repository.
* https://kinesis-ergo.com/support/kb360pro/#ticket

