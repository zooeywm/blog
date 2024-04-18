+++
title = 'Rclone'
date = 2024-04-14T19:55:32+08:00
summary = "Use of rclone"
tags = ["Misc", "Sync", "Linux"]
+++

> https://rclone.org/docs/

Follow the directives to finish Google Drive config: https://rclone.org/drive/#making-your-own-client-id

# Config OneDrive Organization

Simply use `webdav` with sharepoint: <https://rclone.org/webdav>, note that the password need to be encrypted.

# Usage

Having used `Mount`, `Sync`, `Copy`, and `BiSync` modes, I finally choose to use `BiSync` at most chances. Because the `Mount` is based on FUSE, which is erased every time the front or daemon was killed, which is not what I want. `Copy` cannot delete files. `Sync` is a one-way operation. So I choose the two-way operation `BiSync`.
