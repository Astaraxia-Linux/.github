# Astaraxia

*A flexible, source-based Linux distribution built on transparency, configurability, and reproducibility.*

> **Warning:**
> Astaraxia is in early but active development. The distribution bootstraps and the package manager works - but expect rough edges, missing packages, and the occasional existential crisis.

One dev. Too many ambitions. Somehow still going.

## Table of Contents

* [Astaraxia's Software](#astaraxias-software)
* [Overview](#overview)
* [Key Features](#key-features)
* [Status](#status)
* [Installation / Bootstrapping](#installation--bootstrapping)
* [Configuration](#configuration)
* [Directory Layout](#directory-layout)
* [Goals](#goals)
* [Astral Philosophy & Inspiration](#astral-philosophy--inspiration)
* [Roadmap / TODO](#roadmap--todo)
* [Contributing](#contributing)
* [License](#license)

---

## Astaraxia's Software

* **Astral** - The source-based package manager, written entirely in POSIX shell. Minimal, transparent, auditable, hackable, and never going to be rewritten in Rust. Currently at v5.0.0.0 with parallel builds/removals, GPG signing, certificate pinning, FIM, atomic transactions, built-in service management, and a sandbox build system. ~10,000 lines of sh. Yes, really.

* **astral-env** - The declarative environment and system configuration layer. Describe your entire system - packages, services, dotfiles, hostname, timezone, file snapshots - in a `.stars` file and apply it all at once. Think NixOS-style reproducibility without the functional language headache. Written in C++20.

* **astral-recipegen** - Recipe generator for Astral. Auto-detects build systems (autotools, cmake, meson, python, make), generates v3 `.stars` recipes from a URL, converts old formats, and can even import Arch PKGBUILDs. Because writing boilerplate by hand is a crime.

* **Future tools** - Build helpers, auditing scripts, quirky CLI utilities… all manual, all inspectable, all likely to make you question your life choices.

Every tool follows the same philosophy: if you can't read it, you shouldn't be using it.

---

## Overview

Astaraxia is a Linux distribution built around a unified hybrid package model. It gives users full control over their system through transparent source builds via **Astral**, with declarative configuration management via **astral-env**.

Inspired by source-based distributions but designed to stay approachable: predictable in behavior, fully reproducible, and bootstrappable from a standard LFS base.

---

## Key Features

* **Source-based**: packages build from recipes, you see everything that happens
* **Declarative system config**: describe your system in `.stars` files, apply with one command
* **Parallel builds and removals**: `--parallel-build`, `--parallel-remove`, `--parallel-removedep`
* **Atomic transactions with rollback**: every operation is transactional, `--recover` handles interruptions
* **File snapshots**: content-addressed, zstd-compressed, deduplicated via astral-env
* **Init-system agnostic**: systemd, OpenRC, runit, s6, SysVinit - Astral handles all of them
* **Security features**: GPG signing, Web of Trust, certificate pinning, FIM, audit trail, sandbox isolation
* **Plain inspectable recipes**: `.stars` format inspired by QML and Nix - readable without a manual
* **Minimal base**: bootstrapped via Linux From Scratch, nothing hidden

---

## Status

Astaraxia is functional but young. Here's where things actually stand:

| Component | Status |
|---|---|
| LFS bootstrap (chapters 1-8) |  Done |
| Astral package manager |  Working (v5.0.0.0) |
| astral-env (declarative config) |  Working (v1.0.0.0) |
| astral-recipegen |  Working (v2.2.0) |
| Recipe index (AOHARU) |  Small but growing (~10 packages) |
| Community overlay (ASURA) |  Available, contributions welcome |
| Base system packages |  Partial |
| ISO release |  Not yet |
| Binary package support |  Planned |

---

## Installation / Bootstrapping

### Prerequisites

* x86_64 CPU
* ≥8GB RAM (compiling is hungry)
* ~25GB free disk
* Working Linux host with build tools
* Internet access
* Coffee (load-bearing dependency)

### Stage 1 - LFS Base

Follow LFS chapters 1-8 to build the initial toolchain. This gives you a minimal bootable system with nothing interesting on it yet.

### Stage 2 - Install Astral

```bash
curl -O https://raw.githubusercontent.com/Astaraxia-Linux/Astral/main/astral
chmod +x astral
sudo mv astral /usr/bin/
sudo astral --version    # initializes directories
```

Configure `/etc/astral/make.conf`:

```bash
CFLAGS="-O2 -pipe -march=native"
CXXFLAGS="$CFLAGS"
MAKEFLAGS="-j$(nproc)"
CCACHE_ENABLED="yes"
FEATURES="ccache parallel-make strip"
```

### Stage 3 - Install Core Packages

```bash
# Install base packages via Astral
sudo astral -S e2fsprogs
sudo astral -S neovim
# ... etc
```

Or better yet, use astral-env to declare the whole system at once:

### Stage 4 - Install astral-env

```bash
git clone https://github.com/Astaraxia-Linux/astral-env
cd astral-env
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
sudo cmake --install build
```

Enable it in `/etc/astral/astral.stars`:

```
$AST.core: {
    astral-env        = "enabled"
    astral-env-system = "enabled"
};
```

### Stage 5 - Declare Your System

```bash
# Create system config
sudo astral-env system init
sudo astral-env system init-user yourname

# Edit /etc/astral/env/env.stars to describe your system
# Then apply everything at once:
sudo astral-env system apply
```

From here, your entire system configuration lives in `.stars` files. Commit them to git. Fresh install → clone → apply → done.

---

## Configuration

### Astral (`/etc/astral/`)

```
/etc/astral/astral.stars     # global Astral config (feature flags, etc.)
/etc/astral/make.conf        # compiler flags, ccache, parallel settings
/etc/astral/package.mask     # masked package versions
/etc/astral/virtuals/        # virtual package providers
```

### astral-env (`/etc/astral/env/`)

```
/etc/astral/env/env.stars            # system-wide packages, services, hostname, timezone
/etc/astral/env/<username>.stars     # per-user packages, dotfiles, environment vars
/etc/astral/env/dotfiles/<username>/ # dotfile sources (symlinked to $HOME)
```

---

## Directory Layout

```
/usr/bin/astral                    # package manager
/usr/bin/astral-env                # declarative config tool
/usr/bin/astral-env-snapd          # snapshot daemon
/usr/bin/astral-recipegen          # recipe generator

/usr/src/astral/recipes/           # official package recipes (AOHARU)
/etc/astral/                       # system configuration
/var/cache/astral/src/             # cached source archives
/var/cache/astral/bin/             # cached binary packages (future)
/var/lib/astral/db/                # installed package metadata
/var/log/astral/                   # build and install logs

/astral-env/store/                 # astral-env content-addressed store
/astral-env/snapshots/             # file snapshot index
```

---

## Goals

* Fully transparent build system - every recipe is readable plain text
* Unified package management for source and (eventually) binaries
* Declarative, reproducible system configuration via astral-env
* Keep the system minimal, predictable, rollbackable, and maintainable
* Allow users to fully rebuild or inspect any component
* Avoid unnecessary abstractions - abstraction is betrayal
* Never rewrite anything in Rust (this is load-bearing)

---

## Astral Philosophy & Inspiration

| Trait | Arch/Gentoo | NixOS | Astaraxia |
|---|---|---|---|
| Minimal, hackable system | yes |  | yes |
| Predictable builds | yes |  | yes |
| Source-based control | yes |  | yes |
| Binary convenience | yes |  |  Planned |
| Rollbacks / transactional safety | yes |  | yes |
| Declarative config | yes |  | yes |
| Package recipes / ebuild-like | yes |  | yes |
| Human-readable config syntax | yes |  | yes |
| Init-system agnostic | yes |  | yes |

Astral takes the **predictability and minimalism of Gentoo/Arch**, the **rollback and reproducibility of NixOS**, and keeps everything in **plain POSIX sh and readable `.stars` files**. No functional language required. No Rust either.

---

## Roadmap / TODO

*  LFS bootstrap complete
*  Astral package manager (v5.0.0.0) - parallel builds/removals, transactions, GPG, FIM, sandbox, service management
*  astral-env (v1.0.0.0) - declarative system config, file snapshots, GC, rollback
*  astral-recipegen (v2.2.0) - auto-detect, templates, migration, PKGBUILD import
*  Recipe format specification (v3 `.stars`)
*  Growing AOHARU recipe index
*  Base system package set
*  Init system integration (systemd/openrc/runit)
*  Developer documentation
*  Binary package support (`BINPKG_ENABLED`)
*  Bootable ISO
*  Official binary repository

---

## Contributing

The codebase is open and readable (it's literally shell scripts and C++). Contributions welcome:

* **Recipes**: write a `.stars` recipe for a missing package, submit to [ASURA](https://codeberg.org/Izumi/ASURA)
* **Bug reports**: if something breaks, open an issue - there's only one maintainer and he can't test everything
* **astral-env**: C++20, cmake build system - see the astral-env repo
* **Astral core**: POSIX sh only, no bashisms, keep it stupid simple

Guidelines will get more formal as the project matures. For now: test your changes, write descriptive commit messages, don't rewrite anything in Rust.

---

## License

GPL-3.0 for Astral and astral-env. Upstream packages retain their respective licenses.

---

*"If I succeed, you'll see it here. If I fail, blame entropy."*  
- One Maniac™, still going after 100 days
