+++
title = 'Develop Windows App With Rust On Linux'
date = 2024-04-11T23:49:41+08:00
summary = "Cross build win app with rust"
tags = ["Rust", "Cross Building", "Windows"]
+++

# Key points

## Windows Metadata

Windows Runtime(WinRT) Apis are described in binary metadata files with extension `.winmd`. [Languages projections](https://github.com/microsoft/win32metadata/) are dependent on metadata files.

- AI
- AppilicationModel
- Data
- Devices
- Foundation
- Gaming
- Globalization
- Graphics
- Management.Setup
- Management
- Media
- Networking
- Perception
- Security
- Services
- Storage
- System
- UI
- UI.Xaml - Excluded from rust bind
- Web

## Rust Bindgen

`rust-bindgen` automatically parse C/C++ header files, and generated corresponding Rust binding code, which can reduce working on writing bind codes, improve develop efficiency.
Usually used with build.rs, generate bind code before build source code.

### [windows](https://crates.io/crates/windows)

Provides bindings for Windows API, like C-style APIS, COM(Component Object Model) and WinRT APIs, provides the most comprehensive API coverage for Windows. And provide more safe programming model.

### [windows-sys](https://crates.io/crates/windows-sys)

Provides raw bindings for C-style Windows APIs, lack support for COM and WinRT APIS, for faster compile.

# Dependency

[cargo-xwin](https://github.com/rust-cross/cargo-xwin)
