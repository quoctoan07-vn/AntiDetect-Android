# AntiDetect-Android

![Version](https://img.shields.io/badge/version-1.0.0-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![Platform](https://img.shields.io/badge/platform-Android-brightgreen)
![Frida](https://img.shields.io/badge/Frida-compatible-orange)

A professional toolkit to hide injection footprints and emulate a clean environment on Android, defeating anti-tampering and debugging detection mechanisms that rely on `/proc` inspection, `inotify` monitoring, and syscall analysis.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [How It Works](#how-it-works)
  - [Hide Injection Footprint](#hide-injection-footprint)
  - [Clean Environment Emulation](#clean-environment-emulation)
- [Installation](#installation)
- [Usage](#usage)
  - [1. Frida Script](#1-frida-script)
  - [2. LD_PRELOAD Library](#2-ld_preload-library)
  - [3. Memory‑only dlopen](#3-memoryonly-dlopen)
  - [4. Unidbg / QEMU Integration](#4-unidbg--qemu-integration)
- [Supported Platforms](#supported-platforms)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Many Android apps and security libraries detect dynamic instrumentation, hooking frameworks, or custom ROMs by:

- Reading `/proc/self/status` → `TracerPid` field
- Using `inotify` to watch for new files (e.g., injected `.so`, Frida server)
- Checking the environment for suspicious variables (e.g., `LD_PRELOAD`, `FRIDA_SERVER`)
- Scanning `/proc/self/maps` for unknown modules or memory‑mapped files

**AntiDetect-Android** neutralises these checks by combining:

- A **Frida**‑based hook engine that masks `/proc` contents and `inotify` calls.
- An **LD_PRELOAD** library that transparently blocks or redirects sensitive syscalls.
- A technique to load native libraries entirely from memory (`memfd_create` + `dlopen`), leaving no disk footprint.
- Clean‑room emulation helpers for **Unidbg** and **QEMU** that filter out suspicious syscalls and set benign environment variables before the target library initialises.

---

## Features

- ✅ **Hide TracerPid** – Remount `/proc` or hook file operations to always return `TracerPid: 0`.
- ✅ **Neutralise inotify** – Hook `inotify_add_watch` to return `‑1` or redirect watches to a dummy file.
- ✅ **Diskless library loading** – Use `memfd_create` + `dlopen` to load an `.so` directly from memory, bypassing filesystem scans.
- ✅ **Clean environment emulation** – Pre‑defined environment profiles for Unidbg/QEMU that remove `LD_PRELOAD`, `FRIDA_*`, `XPOSED_*`, etc.
- ✅ **Syscall filtering** – Block or modify calls to `ptrace`, `inotify_init1`, `openat("/proc/…")` before they reach the kernel.
- ✅ **Pluggable architecture** – Use as a standalone Frida script, a preload library, or embed in your own analysis setup.

---

## How It Works

### Hide Injection Footprint

#### 1. Masking `TracerPid`
Anti‑debugging code reads `/proc/self/status` and looks for a non‑zero `TracerPid`.  
**Solution** – Mount a custom proc filesystem on top of the real one, using a bind mount of a pre‑crafted file where `TracerPid` is always `0`.  
*Frida alternative*: Hook `fopen`, `read`, or `openat` and fake the file content when the path matches `/proc/*/status`.

#### 2. Disabling `inotify`
Apps use `inotify` to monitor directories (e.g., `/data/app`, `/data/local/tmp`) for newly created files.  
**Solution** – Hook `inotify_add_watch` and either:
- Return `-1` (watch failed)
- Redirect the watch to a harmless dummy file, so the real directory remains unobserved.

#### 3. Memory‑only Loading (`memfd_create` + `dlopen`)
Instead of writing a `.so` to disk (which can be spotted by `inotify` or periodic scanning), we:
1. Read the shared library bytes into memory.
2. Create an anonymous file descriptor with `memfd_create`.
3. Write the library content to that fd.
4. Call `dlopen` on the `/proc/self/fd/<fd>` path.

This leaves **zero disk footprint** and evades file‑based detection.

### Clean Environment Emulation

When running a target library inside **Unidbg** or **QEMU**, it often performs environment checks.  
AntiDetect-Android provides:

- **Pre‑configured environment variables** – Strips out `LD_PRELOAD`, `FRIDA_*`, `XPOSED_*`, and sets `ANDROID_ROOT=/system`, `BOOTCLASSPATH` to expected values.
- **Syscall interceptor** – Before any `ptrace`, `inotify_init1`, or `openat("/proc/…")` call reaches the backend, our hook either returns a safe value or blocks the call entirely.
- **Late‑init override** – All patches are applied **before** `JNI_OnLoad` or the library’s constructor runs, so detection never gets a chance to trigger.

---

## Installation

### Prerequisites
- Android device/emulator with root access (for mounting `/proc` and using `LD_PRELOAD`)
- [Frida](https://frida.re) installed on host and device (for the Frida script)
- (Optional) Unidbg or QEMU‑user for emulation

<p align="center">
  <b><big>🔥 Mua code antiban external ib telegram @quoctoansieudz 🔥</big></b>
</p>
