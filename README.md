<div align="center">
  <img src="https://avatars.githubusercontent.com/u/246477946?s=400&u=1cc036e46c741ab71c8eb189ff673d8410232af5&v=4" alt="Astaraxia Logo" width="120"/>
  
  # Astaraxia
  ***VERSION:*** AZURE
  
  *A flexible, source-based Linux distribution built on transparency, configurability, and reproducibility.*
</div>

> [!WARNING]
> Astaraxia is in early but active development. The distribution bootstraps and the package manager works, but expect rough edges, missing packages, and the occasional existential crisis.

One dev. Too many ambitions. Somehow still going.

## Table of Contents

* [Astaraxia's Software](#astaraxias-software)
* [Overview](#overview)
* [Versions](#versions)
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

* **Astral** - The source-based package manager, written entirely in POSIX shell. Minimal, transparent, auditable, hackable, and never going to be rewritten in Rust. Currently at V5.3.1.5 with parallel builds/removals, GPG signing, certificate pinning, FIM, atomic transactions, built-in service management, a sandbox build system, and Horizon (the bootstrap system) built right in. About 12,000 lines of sh. Yes, really. And it Means ***Astaraxia's Redevelopment in Advance**

* **astral-env** - The declarative environment and system configuration layer. Describe your entire system packages, services, dotfiles, hostname, timezone, file snapshots in a `.stars` file and apply it all at once. Think NixOS-style reproducibility without the functional language headache. Also handles binary package installation. Written in C++20. And it also Means ***Astaraxia's Redevelopment in Advance for the Enviroment**

* **astral-recipegen** - Recipe generator for Astral. Auto-detects build systems (autotools, cmake, meson, python, make), generates v3 `.stars` recipes from a URL, converts old formats, and can import Arch PKGBUILDs. Because writing boilerplate by hand is a crime.

* **Future tools** - Build helpers, auditing scripts, quirky CLI utilities... all manual, all inspectable, all likely to make you question your life choices.

Every tool follows the same philosophy: if you can't read it, you shouldn't be using it.

---

## Overview

Astaraxia is a Linux distribution built around a unified hybrid package model. It gives users full control over their system through transparent source builds via **Astral**, with declarative configuration management via **astral-env**.

Inspired by source-based distributions but designed to stay approachable: predictable in behavior, fully reproducible, and bootstrappable from scratch using Horizon.

---

## Versions
Codenames represent major shifts within the Astaraxia ecosystem.

| Version | Astral version | Features |
|---|---|---|
| Yara | V0.0.0.1 - V5.0.0.0 | Self-Explainatory |
| AZURE | V5.0.0.0+ | Astral-env and more |

If you're wondering why "AZURE" is in full caps, its an acryonm.
It means:

*A next Zone for a Unified Rapid Execution*

---

## Key Features

* **Source-based**: packages build from recipes, you see everything that happens
* **Declarative system config**: describe your system in `.stars` files, apply with one command
* **Horizon bootstrap**: 3-stage LFS-style installer built into Astral (`astral -h`)
* **Parallel builds and removals**: `--parallel-build`, `--parallel-remove`, `--parallel-removedep`
* **Atomic transactions with rollback**: every operation is transactional, `--recover` handles interruptions
* **File snapshots**: content-addressed, zstd-compressed, deduplicated via astral-env
* **Init-system agnostic**: systemd, OpenRC, runit, s6, SysVinit -- Astral handles all of them
* **Security features**: GPG signing, Web of Trust, certificate pinning, FIM, audit trail, sandbox isolation
* **Plain inspectable recipes**: `.stars` format inspired by QML and Nix -- readable without a manual
* **Minimal base**: bootstrapped via Linux From Scratch, nothing hidden

---

## Status

Astaraxia is functional but young. Here is where things actually stand:

| Component | Status |
|---|---|
| LFS bootstrap (chapters 1-8) | Done |
| Horizon (3-stage bootstrap) | Working |
| Astral package manager | Working (V5.3.1.5) |
| astral-env (declarative config) | Working (v1.1.0.0) |
| astral-recipegen | Working (v2.2.0) |
| Recipe index (AOHARU) | Small but growing (84 packages) |
| Community overlay (ASURA) | Available, contributions welcome |
| Base system packages | Partial |
| Binary package support | Planned (via astral-env) |
| ISO release | Not yet |

---

## Installation / Bootstrapping

### Prerequisites

* x86_64 CPU
* 8GB+ RAM (compiling is hungry)
* ~25GB free disk
* Working Linux host with build tools (gcc, g++, make, bison, gawk)
* Internet access
* Coffee (load-bearing dependency)

### Stage 0 -- Prepare the disk

```bash
export LFS=/mnt/lfs
sudo mkdir -pv $LFS
sudo mount /dev/sdXY $LFS   # replace with your actual partition
```

### Stages 1-3 -- Bootstrap via Horizon

Horizon is built into Astral. Install Astral on your host first:

```bash
curl -O https://raw.githubusercontent.com/Astaraxia-Linux/Astral/main/astral
chmod +x astral
sudo mv astral /usr/bin/
```

Then initialize and run the bootstrap:

```bash
# Download the stage scripts from the Horizon repo
sudo astral -h --init

# Stage 1: temporary cross-compilation toolchain (~2-4 hours)
#          builds into $LFS/tools, based on LFS chapter 5
sudo astral -h --stage 1

# Stage 2: chroot preparation (~3-5 hours)
#          builds additional tools, based on LFS chapters 6-7
sudo astral -h --stage 2

# Chroot into the new system
sudo astral --chroot $LFS

# Stage 3: final system (~6-10 hours)
#          builds the complete bootable system, based on LFS chapter 8
astral -h --stage 3
```

Check progress at any time:

```bash
astral -h --status
astral -h --info 1    # detailed info about a specific stage
```

### After the Bootstrap -- Install Astral on the New System

```bash
# Inside the chroot
curl -O https://raw.githubusercontent.com/Astaraxia-Linux/Astral/main/astral
chmod +x astral
mv astral /usr/bin/
astral --version
```

Configure `/etc/astral/make.conf`:

```bash
CFLAGS="-O2 -pipe -march=native"
CXXFLAGS="$CFLAGS"
MAKEFLAGS="-j$(nproc)"
CCACHE_ENABLED="yes"
FEATURES="ccache parallel-make strip"
```

### Optional -- Install astral-env

For declarative system configuration:

```bash
git clone https://github.com/Astaraxia-Linux/astral-env
cd astral-env
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
sudo cmake --install build
```

Enable in `/etc/astral/astral.stars`:

```
$AST.core: {
    astral-env        = "enabled"
    astral-env-system = "enabled"
};
```

Then declare your whole system:

```bash
sudo astral-env system init
sudo astral-env system init-user yourname

# Edit /etc/astral/env/env.stars, then:
sudo astral-env system apply
```

From here your entire system configuration lives in `.stars` files. Commit them to git. Fresh install, clone, apply, done.

---

## Configuration

### Astral (`/etc/astral/`)

```
/etc/astral/astral.stars       global Astral config (feature flags, enabled tools)
/etc/astral/make.conf          compiler flags, ccache, parallel settings
/etc/astral/package.mask       masked package versions
/etc/astral/virtuals/          virtual package providers
```

### astral-env (`/etc/astral/env/`)

```
/etc/astral/env/env.stars                system-wide packages, services, hostname, timezone
/etc/astral/env/<username>.stars         per-user packages, dotfiles, environment vars
/etc/astral/env/dotfiles/<username>/     dotfile sources (symlinked to $HOME)
```

---

## Directory Layout

```
/usr/bin/astral                      package manager + Horizon bootstrap
/usr/bin/astral-env                  declarative config tool
/usr/bin/astral-env-snapd            snapshot daemon
/usr/bin/astral-recipegen            recipe generator

/usr/src/astral/recipes/             official package recipes (AOHARU)
/usr/src/astral/horizon/             Horizon stage scripts (downloaded on init)
/etc/astral/                         system configuration
/var/cache/astral/src/               cached source archives
/var/lib/astral/db/                  installed package metadata
/var/lib/astral/horizon/             Horizon build state and stage markers
/var/log/astral/                     build, install, and horizon logs

/astral-env/store/                   astral-env content-addressed store
/astral-env/snapshots/               file snapshot index
```

---

## Goals

* Fully transparent build system -- every recipe is readable plain text
* Unified package management for source builds, with binary support coming via astral-env
* Declarative, reproducible system configuration via astral-env
* Keep the system minimal, predictable, rollbackable, and maintainable
* Allow users to fully rebuild or inspect any component
* Avoid unnecessary abstractions -- abstraction is betrayal
* Never rewrite anything in Rust (this is load-bearing)

---

## Astral Philosophy & Inspiration

| Trait | Arch/Gentoo | NixOS | Astaraxia |
|---|---|---|---|
| Minimal, hackable system | yes | partial | yes |
| Predictable builds | yes | yes | yes |
| Source-based control | partial | partial | yes |
| Binary convenience | partial | yes | planned (astral-env) |
| Rollbacks / transactional safety | partial | yes | yes |
| Declarative config | partial | yes | yes |
| Package recipes / ebuild-like | yes | partial | yes |
| Human-readable config syntax | yes | no | yes |
| Init-system agnostic | partial | partial | yes |

Astral takes the predictability and minimalism of Gentoo/Arch, the rollback and reproducibility of NixOS, and keeps everything in plain POSIX sh and readable `.stars` files. No functional language required. No Rust either.

---

## Roadmap / TODO

* LFS bootstrap complete
* Horizon 3-stage bootstrap system (built into Astral)
* Astral package manager v5.3.1.5 -- parallel builds/removals, transactions, GPG, FIM, sandbox, service management
* astral-env v1.1.0.0 -- declarative system config, file snapshots, GC, rollback
* astral-recipegen v2.2.0 -- auto-detect, templates, migration, PKGBUILD import
* Recipe format specification (v3 `.stars`)
* Growing AOHARU recipe index (in progress)
* Base system package set (in progress)
* Developer documentation (in progress)
* Binary package support via astral-env (in progress)
* Bootable ISO (planned)
* Official binary repository (planned)

---

## Contributing

The codebase is open and readable (it's literally shell scripts and C++). Contributions are welcome:

* **Recipes**: write a `.stars` recipe for a missing package and submit to [ASURA](https://codeberg.org/Izumi/ASURA)
* **Bug reports**: if something breaks, open an issue -- there is only one maintainer and he cannot test everything
* **astral-env**: C++20, cmake build -- see the astral-env repo
* **Astral core**: POSIX sh only, no bashisms, keep it stupid simple

Guidelines will get more formal as the project matures. For now: test your changes, write descriptive commit messages, do not rewrite anything in Rust.

---

## License

GPL-3.0 for Astral and astral-env. Upstream packages retain their respective licenses.

---

*"If I succeed, you'll see it here. If I fail, blame entropy."*
-- One Maniac, still going after 102 days (2025 - 11 - 26)
