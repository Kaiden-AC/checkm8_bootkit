# checkm8_bootkit

Little utility to boot **iBoot** on some **checkm8**-able platforms. Now also can decrypt KBAGs and demote

It doesn't require any modifications to **ipwndfu**/**gaster**/etc. shellcodes since it utilizes **ipwndfu**'s [custom protocol](https://github.com/axi0mX/ipwndfu/blob/master/src/usb_0xA1_2_armv7.S)

You can run it on iOS as well (if you are lucky)

## SoC support

* **S5L8940X** - Apple A5
* **S5L8942X** - Apple A5 (32nm)
* **S5L8945X** - Apple A5X
* **S5L8947X** - Apple A5 (single-core)
* **S5L8950X** - Apple A6
* **S5L8955X** - Apple A6X
* **S5L8747X** - Haywire SoC
* **S7002** - Apple S1
* **T8002** - Apple S1P/S2/T1
* **T8004** - Apple S3

## Usage

```
➜  checkm8_bootkit git:(master) ✗ build/checkm8_bootkit
usage: build/checkm8_bootkit VERB [args]

where VERB is one of the following:
        boot <bootloader>
        kbag <kbag>
        demote
        batch <input> <output>

for batch KBAG processing, you must input a JSON in the following format:
        [
                {
                        "kbag": "KBAG",
                        "metadata_1": "METADATA",
                        ...
                        "metadata_n": "METADATA"
                },
                ...
        ]

in return you'll get the same structure, but with "key" appended to each entry

supported platforms:
        s5l8747x, s5l8940x, s5l8942x, s5l8945x, s5l8947x, s5l8950x, s5l8955x, s7002, t8002, t8004
```

* `bootloader` must be a path to raw unpacked **iBoot** image (usually you'd want to load **iBSS**)
* `kbag` must be a hex string

Set **LIBBOOTKIT_DEBUG** environment variable to 1 to enable verbose logging

## A6 (iPhone 5) with a6meowing

An A6 pwned with [a6meowing](https://github.com/LukeZGD/a6meowing) works with the `kbag` and `batch` VERBs out of the box.

Note that a6meowing patches the ROM to relocate the DFU download address to **0x10020000** (instead of ipwndfu's 0x10000000), so code that reads or writes the download buffer — like the KBAG decryption path — must use that address. This tree therefore defaults `s5l8950x` to `loadaddr = 0x10020000`.

Set the **LIBBOOTKIT_LOADADDR** environment variable to override the load address (dec, hex, or octal) for your specific payload setup; for example, an ipwndfu-pwned A6 needs `LIBBOOTKIT_LOADADDR=0x10000000`.

The **demote** VERB does not depend on the load address and works unchanged.

## Building

Requirements:

* [lilirecovery](https://github.com/NyanSatan/lilirecovery)
    * My little libirecovery fork
    * Included as a Git module

* [vmacho](https://github.com/Siguza/misc/blob/master/vmacho.c)
    * Only needed if you want to rebuild the payloads

Then just use `make`:

```
➜  checkm8_bootkit git:(full) ✗ make      
        building checkm8_bootkit for Mac
        building checkm8_bootkit for iOS
%%%%% done building
```

Add `WITH_ARMV7=1` to the invocation to build armv7 iOS version as well (broken since Xcode 16)
