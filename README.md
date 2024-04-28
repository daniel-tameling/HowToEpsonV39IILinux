# How to Install the Epson Perfection V39II under Linux

## Lessons Learned, a.k.a. Things You Might Want to Know

I managed to get the scanner working under Arch Linux. It was a lot of struggle, but in the end I managed to make it work. The tldr is that I had to use the rpm instead of the deb. (The precise version I succeeded with is 6.7.63.) If you want to know the exact steps, look at the section General Installation Procedure below.

But here are a few things I learned along the way:
  - Despite having almost the same name, the V39II is not just the V39 in different packaging. It has a different product ID (0x013f) and uses a different firmware file (esfw0282.bin). So forget all guides targeting only V39.
  - This means you have to use epsonscan2. You need the firmware, even if you want to use sane to scan. iscan doesn't work: it is too old and doesn't support the V39II.
  - Epsonscan2 outputs some debugging information, if you create a `/tmp/epson` directory. For epsonscan2, the log is under `/tmp/epson/epsonscan2/epsonscan2.log`. In this repository you can find a log for the non-working case (`epsonscan2.broken.log`) and a log for the working case `epsonscan2.working.log`. In there, look for `fork_`, after that the cases start to diverge: The subsequent `open_` fails or succeeds, respectively. The external program that is executed is `es2intif`, which is part of the non-free part.
  - If the scanner's LED blinks, it doesn't necessarily indicate the wrong firmware. It can also come if you disabled USB autosuspend for all devices. (Don't ask)
  - The package installs a file with the udev rules: `60-epsonscan2.rules`. This is the line for the V39II:
    ```
    ATTRS{idProduct}=="013f", ENV{epsonscan2_driver}="esci", ENV{firmware_file}="esfw0282.bin"
    ```
  - If you want to make sure that this is executed, you can add this at the end of the file:
    ```
    RUN+="/bin/sh -c 'echo == >> /dev/shm/udev-env.txt; env >> /dev/shm/udev-env.txt'"
    ```
    (adapted from https://unix.stackexchange.com/questions/200194/how-to-debug-an-udev-rule-in-etc-udev-rules-d)
    If you now plug the scanner in, you find the environment in `/dev/shm/udev-env.txt`. This might require reloading the rules. The easiest way to do this is a reboot after editing the file.
  - The message the epsonscan2 displayed in the non-working case was
    ```
    Unable to communicate with the scanner. Make sure the scanner is connected to the computer and turned on.
    ```
  - scanimage on the other hand printed:
    ```
    $ scanimage --format=png --output-file test.png --progress
    scanimage: open of device epsonscan2:Epson Perfection V39II:001:004:esci2:usb:ES0283:319 failed: Error during device I/O
    ```
    This doesn't necessarily mean that the device it tires to open has the wrong permission.
  - The manual for epsonscan2 is https://download.ebz.epson.net/man/linux/epsonscan2_e.html, which describes the overall process, but isn't overly useful. Most distros package epsonscan2 and you should a) install the open source part through your package manager b) copy how they deal with the non-free part, in particular where they put the files.
  - Looking at the epson webpage has two V39II support pages, one of them doesn't list Linux when you select the operating system. Don't let that fool you into thinking, it might not work. I have it working under Linux.

## General Installation Procedure

The necessary software consists of two parts: The core, which is open source, and the plugin, which is only available in binary form.

If your distro uses rpm or deb packages, you can download a package that contains both the compiled core and the plugin here: https://support.epson.net/linux/en/epsonscan2.php

If that's not the case, you still need one of those packages and extract the plugin. With the deb, the scanner wouldn't work, so I would recommend trying the rpm first. You should also try the rpm if the scanner fails to after you installed the deb on a distro with a deb package manager.

You can compile the core yourself. I expect that this part is available through the package manager you are using. If you want a newer version or want to compile it for yourself, look at how the corresponding package for your distro is build and follow that. The source code is also available at https://support.epson.net/linux/en/epsonscan2.php, at a link at the bottom called "the source file". If you need further help with the compilation, the manual https://download.ebz.epson.net/man/linux/epsonscan2_e.html describes how to do it, or check out the Arch Linux PKGBUILD at https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=epsonscan2. Arch also applies three patches, which you might also do.

The non-free plugin just needs to be extracted and moved to the correct location. If you have downloaded the rpm, then do
```
bsdtar xf epsonscan2-bundle-6.7.63.0.x86_64.rpm.tar.gz
cd epsonscan2-bundle-6.7.63.0.x86_64.rpm/plugins
bsdtar xf epsonscan2-non-free-plugin-1.0.0.6-1.x86_64.rpm
```
This gives you a `usr` directory in the current folder. You need to move the files in that directory to the corresponding place in your `/usr` tree. If your distro packages the non-free part as well, follow what it does. You can also look at the Arch Linux PKGBUILD at https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=epsonscan2-non-free-plugin. They use the deb that didn't work for me, but it contains the same files and they need to end up in the same locations. Note although that the deb is organized differently than the rpm.


