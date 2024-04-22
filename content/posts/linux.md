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
13. [Steam](https://cn.linux-console.net/?p=14000#:~:text=%E5%A6%82%E4%BD%95%E5%9C%A8%20Arch%20Linux%20%E4%B8%8A%E5%AE%89%E8%A3%85%E5%92%8C%E4%BD%BF%E7%94%A8%20Steam%201%20%E4%BF%AE%E6%94%B9%20Pacman,%E7%BB%93%E8%AE%BA%20%E6%9C%AC%E6%8C%87%E5%8D%97%E8%A7%A3%E9%87%8A%E4%BA%86%20Arch%20Linux%20%E7%B3%BB%E7%BB%9F%E4%B8%AD%E6%B8%B8%E6%88%8F%E7%9A%84%20Steam%20%E5%8F%91%E8%A1%8C%E6%B8%A0%E9%81%93%E7%9A%84%E4%BD%BF%E7%94%A8%E3%80%82%20)

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

## HiDPI

Don't use Hyprland monitor scale, though you can rescale xwayland apps with `force_zero_scaling=true` and set `QT` or `GTK` SCALE env, it will still cause some issues like when using satty to annotation pictures, the blur function will resolved to be weird.

## LazyVim

- MarkdownPreview Error: Cannot find module 'tslib': <https://github.com/iamcco/markdown-preview.nvim/issues/188#issuecomment-841356921> 
   
    ```z ~/.local/share/LazyVim/lazy/markdown-preview.nvim/app && npm install```

## Clash verge conflict with ssh

Add this `- DST-PORT,22,DIRECT` to clash profile.
