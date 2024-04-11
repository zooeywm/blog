+++
title = 'Linux OS'
date = 2024-04-03T14:40:15+08:00
summary = "My Linux Configuration"
tags = ["OS"]
+++

As for the distribution, I choose archlinux.

# Installation and Post-Installation

follow archlinux's official website: <https://wiki.archlinux.org/title/installation_guide>

My only one disk devided into two parts: one 2G [UEFI](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface) partion and the remains are root [LVM2](https://wiki.archlinux.org/title/Install_Arch_Linux_on_LVM) partion with [GPT](https://wiki.archlinux.org/title/Partitioning#GUID_Partition_Table). The LVM partion is devided into one 20G [SWAP](https://wiki.archlinux.org/title/Swap) volume and the remains are /root volume with [EXT4](https://wiki.archlinux.org/title/Ext4) fs.

For network management, I choose [NetworkManager](https://wiki.archlinux.org/title/NetworkManager)

# System maintenance

1. Regularly run `systemctl --failed` and fix.
2. tty font size too small - <https://wiki.archlinux.org/title/HiDPI#Linux_console_(tty)>
3. [Periodically backup system](https://wiki.archlinux.org/title/Cron)
4. systemd unit modify: <https://wiki.archlinux.org/title/Systemd#Editing_provided_units>
5. Display server - [Wayland](https://wiki.archlinux.org/title/Wayland) with [Hyprland](https://wiki.archlinux.org/title/Hyprland)

   - Issue:
     ```
     terminate called after throwing an instance of 'std::runtime_error'
     what():  wlr_backend_autocreate() failed!
     pcilib: Error reading /sys/bus/pci/devices/0000:00:08.3/label: Operation not permitted
     terminate called recursively
     Aborted (core dumped)
     ```
   - fix: `pacman -S polkit`

6. System bar - Waybar
7. IM - fcitx
8. pulseaudio `systemctl --user enable pipewire` with pamixer
9. Resolve hyprland cannot resume after hibernate: add `resume` after `udev` in `/etc/mkinitcpio.conf`'s `HOOKS`
10. clash-meta authorization - `sudo /usr/bin/setcap 'cap_net_admin,cap_net_bind_service=+ep' /usr/bin/clash-meta`
11. Fix flickering on screen - <https://wiki.archlinux.org/title/AMDGPU#Screen_artifacts_and_frequency_problem> Set the udev.
12. obs - screen-record: `pipewire-v4l2`, virtual camera: `v4l2loopback-dkms`.

## Pipewire

PipeWire is a new low-level multimedia framework. can offer capture and playback for both audio and video for PulseAudio,JACK,ALSA and GStreamer-based applitcations.

- PulseAudio: General sound server, also offers easy network streaming using Avahi(A zeroconf multicast DNS/DNS-SD service discovery)
- JACK: Audio Connect Kit, A sound server daemon for audio and MIDI data
- ALSA: Advanced Linux Sound Architecture, provides kernel driven sound card drivers, which replaces the OSS
- GStreamer: A pipeline-based multimedia framework, can offer meadia-handling for audio playback, audio and video playback, recording, streaming and editing, and cross-platform.

The daemon based on pipewire, can be configured to be both audio and video capture server.

It doesn't rely on audio or video user group, instead, it uses polkit-like security policy, asking Flatpak or Wayland for permission to record screen or audio.

It supports bluetooth by default. Just need to install bluez and bluez-util

> Pipewire can be used as audio server, similar to PulseAudio and JACK, it aims to replace both.

Pirewire doesn't implements connection logic, So we need Session Manager - WirePlumber

## Polkit

An App-level tookit for defining and handling the policy that allows unprivileged processes to speak to privileged processes.

## Fcitx

Fix fcitx on wayland with config: <https://fcitx-im.org/wiki/Using_Fcitx_5_on_Wayland#TL.3BDR_Do_we_still_need_XMODIFIERS.2C_GTK_IM_MODULE_and_QT_IM_MODULE.3F>
