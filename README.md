# Ansible Raspi VLC Media Player

## Summary:

A [VLC media player](https://www.videolan.org/) that runs on minimal operating system while headless (*i.e. no graphical desktop environment such as GNOME, XFCE, KDE, and MATE being used*) on Raspberry Pi.

How is this possible? Using [X.Org](https://x.org/wiki/) display server along with [Openbox](http://openbox.org/wiki/Main_Page) window manager. Of course, the Raspberry Pi has to be connected to an external display via HDMI.

What was the inspiration behind this? I was not able to find a free, convenient, open-source alternative that would allow me to run videos on the TV to be used as a digital signage, 24/7. Other options were either cost-prohibitive, lacks local storage option (*i.e. cloud-only service*), or didn't provide the flexibility of how I would like to play the videos.

Raspberry Pi also has CEC support, which works out of the box, as I was able to have my TV automatically turn on. As a bonus, one can remotely control the video playlist via the web GUI accessible only on their local network.

VLC media player is the most versatile open-source multimedia player there is. Its popularity has prevailed for this reason.

I took the opportunity to use Ansible as I'd be able to automatically reconfigure the Pi or set up multiple Pis simultaneously. Also, we can avoid dealing with configuration drift headaches which inevitably comes with configuring the Pi manually. However, the Ansible playbook isn't fully idempotent due to the [imported role](https://github.com/bashtheshell/headless-raspi-config-ansible) which I created. The nature of the `raspi-config` commands has made it difficult to implement.


## Intended Users:

This is for a technical audience. Specifically, the users must have the following:

- A basic familiarity with Ansible or a scripting language, managing the command-line shell.
- Possesses a third-generation Raspberry Pi or newer.
- Has some previous experience, interacting with VLC media player, especially in a GUI desktop environment.


## Setup:

Before you can proceed, you would need to meet some requirements.

#### Requirements:

- Ansible (preferably at least core `v2.17.0`)
- Raspberry Pi OS Lite using Debian `v12` (`bookworm`) or newer (64-bit is strongly preferred)
- Raspberry Pi 3 or newer (strongly preferred) connected to the network
- Keyboard connected to Raspberry Pi
- An external display (preferably a TV with audio support) connected to Raspberry Pi using HDMI cable
- A USB with the exact filesystem label `VIDEOS`, containing video files in format that is [playable by VLC](https://www.videolan.org/vlc/features.html)


#### Steps:

1. Ensure you have the Raspberry Pi configured using Raspberry Pi OS image loaded on the microSD card. It's recommended you use Raspberry Pi Imager to format your microSD as you would be able to preconfigure a username, password, wi-fi configuration (if needed), and authorized key for your SSH user.

2. Replace the `vars:` values as desired at the top of the `main.yml` Ansible playbook (only `headless_raspi_config_pi_hostname` and `vlc_web_password`).

3. First, install the required module from the Ansible collection or else, Step 4 would fail to run:

    ```shell
    ansible-galaxy collection install -r requirements.yml
    ```

4. Once the Pi is booted, you'd proceed to run the `main.yml` playbook in the same directory as this README using one of the following commands that suits you below:

    ```shell
    # Use the 'Host' name in your `~/.ssh/config` file (highly-recommended)
    ansible-playbook -i "my_pi_host_name," main.yml

    # Use your Pi's IP address using custom SSH port 2222, SSH's username, and SSH private key
    ansible-playbook -i "192.168.50.10:2222," -u my_pi_username --private-key ~/.ssh/id_ed25519 main.yml

    # Run using your Pi's IP address using custom SSH port 2222, SSH's username and its password (provide at the prompt)
    # NOTE: This is not a highly-recommended security practice and most SSH servers have disabled the interactive 
    # password authentication by default.
    ansible-playbook -i "192.168.50.10," -u my_pi_username --ask-pass main.yml
    ```

4. The pi will reboot at least twice. Please wait until the video is automatically loaded after the second reboot unless you didn't have an acceptable USB storage device connected to the Pi.

5. If the videos didn't play at this point, please review the playbook output to see which task didn't run as expected. You can attempt to re-run the command in Step 4 with the correct USB device connected this time, and the videos should load.

**NOTE:** On Step 4, you may have noticed in the output that some tasks have a warning message that were considered 'OK' (highlighted in green) by Ansible. You can safely disregard them as the task would have halted the playbook and failed the task execution with a red highlight instead.


## Features:

#### Remote Control:

Instead of using a keyboard, you can use a universal multifunctional wireless remote control similar to the [WeChip remote control](https://www.amazon.com/Wireless-Keyboard-W1-Multifunctional-Projector/dp/B0787Z1C2G) demonstrated below:

![WeChip W1 universal remote control](./README/WeChip_W1_universal_remote_control.jpg)

As you may have gathered from the long VLC command in the [./templates/bash_profile.j2](./templates/bash_profile.j2), the VLC `--global-key-*` *Hot keys* option, enabling the users to set up key bindings, associating the keyboard keys with individual controls (*e.g. pausing, exiting the full-screen, fast-forwarding the playback, and etc.*).

Here are the WeChip remote button assignments (*which are fundamentally keyboard keys or key combinations*) that I configured:
- `--global-key-stop="Browser Home"` - **Home** button
- `--global-key-loop="Browser Back"` - **Return** button
- `--global-key-random="Menu"` - **Menu** button
- `--global-key-next="Up"` - **Up** directional button
- `--global-key-prev="Down"` - **Down** directional button
- `--global-key-jump+short="Right"` - **Right** directional button
- `--global-key-jump-short="Left"` - **Left** directional button
- `--global-key-play-pause="Enter"` - **OK** button
- `--global-key-rate-normal="Volume Down"` - **Volume-** button
- `--global-key-rate-faster-fine="Volume Up"` - **Volume+** button

For good documentations on configuring keys or buttons, please have a look at ArchWiki's [Xbindkeys article](https://wiki.archlinux.org/title/Xbindkeys#Identifying_keycodes) and [VLC's GUI hotkey doc](https://wiki.videolan.org/QtHotkeys/). `playerctl` command tool may also come in handy if you are attempting to set up a different generic remote control.


#### Web Access:

You can also remotely control the VLC media player on the Raspberry Pi via its HTTP web interface, which is made possible with [Lua module](https://wiki.videolan.org/documentation:modules/http_intf/).

The `--http-host 0.0.0.0` option enable the VLC client to start receiving HTTP requests from the external hosts on its LAN. The HTTP port is configured with `--http-port=8080` option.

To access the web, you'd need to access it using the following URL: `http://192.168.50.10:8080`. Of course, replace the Pi's IP address with yours. At this time, there isn't a secured HTTPS option available. Upon logging in, you would only need to provide the password while leaving the username blank.

Please note that there isn't a native way to view the video stream from the web as the project maintenance for the HTTP web interface has been abandoned for quite some time.


## Pinned Versions:

Here's the list of software and packages along with their respective versions that were tested at the time of this writing so that one can attempt to reproduce if a future version introduces breaking changes:

**Date last checked:** `06/12/2024`

**Pi model tested on:** `Raspberry Pi 4 Model B Rev 1.5`

**Packages:**
- `vlc` = `1:3.0.20-0+rpt6+deb12u1`
- `xorg` = `1:7.7+23`
- `openbox` = `3.6.1-10+rpt1`
- `pulseaudio` = `16.1+dfsg1-2+rpt1`

**System packages:**
- `python3` (system) = `3.11.2-1+b1`
- `linux-headers-rpi-2712` = `1:6.6.31-1+rpt1`
- `linux-headers-rpi-v8` = `1:6.6.31-1+rpt1`
- `linux-image-rpi-2712` = `1:6.6.31-1+rpt1`
- `linux-image-rpi-v8` = `1:6.6.31-1+rpt1`
- `raspi-firmware` = `1:1.20240529-1`

**System:**
- `uname -srvm` = `Linux 6.6.31+rpt-rpi-v8 #1 SMP PREEMPT Debian 1:6.6.31-1+rpt1 (2024-05-29) aarch64`
- `cat /etc/debian_version` = `12.5`

**Image:**
- `2024-03-15-raspios-bookworm-arm64-lite.img`

**Ansible:**
```shell
> ansible --version
ansible [core 2.17.0]
    <-- lines intentionally trimmed here -->
    python version = 3.12.3 (main, Apr  9 2024, 08:09:14) [Clang 15.0.0 (clang-1500.3.9.4)]
    jinja version = 3.1.4
    <-- lines intentionally trimmed here -->
```

## Issue:

As always, please feel free submit [an issue](https://github.com/bashtheshell/ansible_raspi_vlc_media_player/issues) if you have questions or need some assistance.
