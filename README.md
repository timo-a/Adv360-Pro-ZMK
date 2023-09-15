# ADV360-PRO-ZMK

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

There is a GUI for editing the keymap. It is available at https://kinesiscorporation.github.io/Adv360-Pro-GUI. This repository is also compatible with certain other web based ZMK keymap editors.

Certain ZMK features require knowing the exact key positions in the matrix. They can be found in both image and text format [here](assets/key-positions.md)

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

* Either Podman or Docker is required, Podman is preferred if both are present.
* Make is also required

#### Windows specific

* If compiling on Windows use WSL2 and Docker [Docker Setup Guide](https://docs.docker.com/desktop/windows/wsl/).
* Install make using `sudo apt-get install make`.
* The repository can be cloned directly into the WSL2 instance or accessed through the C: mount point WSL provides by default (`/mnt/c/path-to-repo`).

### Build firmware

1. Execute `make`.
2. Check the `firmware` directory for the latest firmware build.

### Cleanup

The built docker container and compiled firmware files can be deleted with `make clean`. This might be necessary if you updated your fork from V2.0 to V3.0 and are encountering build failures

## Flashing firmware

Follow the programming instruction on page 8 of the [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) to flash the firmware.

### Overview

1. Extract the firmwares from the downloaded archive.
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

### Upgrading from V2 to V3

If you are upgrading from V2 to V3, and if the flashing didn't work as expected (i.e. if you are unable to pair the keyboard via Bluetooth), then consider [resetting](https://kinesis-ergo.com/support/kb360pro/#firmware-updates) both halves of the keyboard to its native state. Make sure to use the `settings-reset.uf2` file from 
the V3 branch of this repository. After doing this, proceed with the flashing instructions above.

## Changelog

The changelog for both the config repo and the underlying ZMK fork that the config repo builds against can be found [here](CHANGELOG.md)

## Note

By default this config repository references [a customised version of ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.2) with Advantage 360 Pro specific functionality and changes over [base ZMK](https://github.com/zmkfirmware/zmk). The Kinesis fork is regularly updated to bring the latest updates and changes from base ZMK however will not always be completely up to date.

Whilst the Advantage 360 Pro is compatible with base ZMK some of the more advanced features will not work, and Kinesis cannot provide customer service for usage of base ZMK. Likewise the ZMK community cannot provide support for either the Kinesis keymap editor, nor any usage of the Kinesis custom fork.

## Other support

Further support resources can be found on Kinesis.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals
