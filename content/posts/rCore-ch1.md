+++
title = 'rCore Learning Ch1'
date = 2024-03-24T09:00:00+08:00
summary = "Hello world on bare riscv64gc machine"
tags = ["Rust", "Programming", "OS"]
+++

> Reference: [rCore-Tutorial-Book 3rd Edition](https://rcore-os.cn/rCore-Tutorial-Book-v3/index.html)

> Repo url: <https://github.com/zooeywm/rcore-learning>

# Target

# On Install

I didn't install QEMU 7.0.0 described in the post, I install it by 

``` fish
sudo pacman -S qemu-system-riscv
```

When I run `make run` in the `rCore-Tutorial-v3` `os` directory, I've encountered this problem: 

``` text
qemu-system-riscv64: -device virtio-gpu-device: 'virtio-gpu-device' is not a valid device model name
```

After a lot of inquiries, I found out it was because I didn't install [qemu-hw-display-virtio-gpu](https://archlinux.org/packages/extra/x86_64/qemu-hw-display-virtio-gpu/), after install it, we can run the program successfully.
