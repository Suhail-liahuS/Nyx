# ❄️ Nyx — Framework 13 AMD (AI 300 / Strix Point) • Niri on NixOS 25.11

A declarative, modular, and opinionated **NixOS 25.11** configuration built for the **Framework 13 AMD (AI 300 / Strix Point)** laptop.

Nyx ships a **Desktop Router**: instantly switch between a modern **Noctalia** desktop and a retro-futuristic **Waybar (Aurora)** setup while keeping a *single*, unified backend for theming, keybinds, and shell tooling.

---

## License

This project is licensed under the Apache License 2.0.

## Contents

* [Architecture](#architecture)
* [Dual-profile system](#dual-profile-system)
* [Features & tooling](#features-tooling)
* [System & LatencyFleX profiles](#system-latencyflex-profiles)
* [Install & configuration](#install-configuration)
* [Keybindings (Niri)](#keybindings-niri)
* [Core package set](#core-package-set)
* [Performance & monitoring](#performance-monitoring)
* [Credits](#credits)

---

<a id="architecture"></a>

## 🏗 Architecture

* **Basis:** NixOS 25.11 + Home Manager (flakes)
* **Compositor:** [Niri](https://github.com/YaLTeR/niri) — scrollable tiling Wayland compositor
* **Theme:** Catppuccin Mocha via **Stylix**
* **Shells:** Fish (primary) + Bash (fallback), unified aliases
* **Kernel:** `linuxPackages_latest` (RDNA 3.5 graphics & NPU support)

---

<a id="dual-profile-system"></a>

## 🌗 Dual-profile system

Nyx provides two distinct desktop “personalities,” switchable in `home-ashy.nix`:

| Feature       | **Noctalia profile**               | **Waybar (Aurora) profile**    |
| ------------- | ---------------------------------- | ------------------------------ |
| Aesthetic     | Modern, Material You, widget-heavy | Cyberpunk/Aurora, text-forward |
| Panel / bar   | Noctalia Shell (Quickshell)        | Waybar (Aurora config)         |
| Launcher      | Fuzzel                             | Wofi                           |
| Terminal      | Kitty                              | Kitty                          |
| Notifications | Mako                               | Mako                           |

### Switch profiles

Edit `home-ashy.nix`:

```nix
my.desktop.panel = "noctalia";  # or "waybar"
```

Then rebuild:

```bash
sudo nixos-rebuild switch --flake .#nyx
```

---

<a id="features-tooling"></a>

## 🧰 Features & tooling

### 🖱️ Hardware & input

* **G502 Lightspeed:** `g502-manager` CLI for hardware profile mapping (no GUI bloat)
* **DualSense:** kernel-level support via `game-devices-udev-rules`
* **Bluetooth:** minimal BlueZ build (stripped of OBEX/Mesh extras)

### ⚡ Performance & latency

* **ZRAM:** multiple compressed-swap profiles, selectable via flake outputs
* **LatencyFleX:** optional Vulkan implicit layer for vendor-agnostic latency reduction
* **Sysctl:** tuned `vm.swappiness`, dirty writeback, and cache pressure

### 🔒 Kernel hardening (defaults)

* `kernel.randomize_va_space=2` enforces full ASLR.
* `kernel.kptr_restrict=2` hides kernel pointers from unprivileged users.
* `kernel.dmesg_restrict=1` restricts kernel logs to privileged users.
  *Tradeoff:* diagnostics/perf tooling may be less informative; temporarily lower these only when debugging.

### ⏱️ Secure time sync

* Chrony is enabled with NTS (Network Time Security) against a default
  set of NTS-capable servers (Cloudflare + Netnod).
* Override servers via `my.security.timeSync.ntsServers = [ "time.example.net" ... ];`
  in `configuration.nix` or a host overlay.
* Verify status at runtime: `chronyc sources -v` (look for `NTS`/`PNTS`)
  and `chronyc tracking`.

### 🔌 USBGuard (declarative policy)

* USBGuard is enabled with a declarative ruleset at `etc/usbguard/rules.conf`
  (deployed to `/etc/usbguard/rules.conf`).

* Generate an initial policy from currently attached devices:

  ```bash
  scripts/usbguard-generate-policy.sh > /tmp/usbguard.rules
  # review/edit, then replace etc/usbguard/rules.conf with the approved rules
  sudo nixos-rebuild switch --flake .#nyx
  ```

* Keep a root shell/TTY/SSH session open when testing to avoid lockout.

* Refine rules before enabling on untrusted ports; `usbguard list-devices`
  helps inspect current devices.

### 🛡️ Systemd hardening

* `DefaultNoNewPrivileges=yes` is set globally via systemd manager
  defaults.
* Override for a specific unit only when necessary:

  ```nix
  systemd.services.my-service.serviceConfig.NoNewPrivileges = lib.mkForce false;
  ```

  Use sparingly—most services should run with `NoNewPrivileges=true`.

### 🔑 Privilege escalation

* `doas` is the supported escalation path; `sudo` is disabled.
* Only the admin user (`ashy` by default) may use `doas`; no persistence
  tokens are issued by default.
* Root login is disabled (including over SSH). For recovery, use a
  console/TTY or boot into a rescue environment and edit
  `configuration.nix` if needed.

### 🎮 NVIDIA support (install-driven)

* Enable via install answers (`nvidia.enable = true`) with mode:

  * `desktop` (single GPU): uses `videoDrivers = [ "nvidia" ]`
  * `laptop-offload`: PRIME offload (`nvidia-offload <app>`), `videoDrivers = [ "modesetting" "nvidia" ]`
  * `laptop-sync`: PRIME sync (iGPU + dGPU sync)
* Open kernel module is enabled by default for Turing+; set
  `nvidia.open = false` in answers to force the proprietary module.
* For hybrid modes provide bus IDs (from `lspci -D`):

  ```bash
  lspci -D | grep -iE 'vga|3d'
  # format: PCI:bus:device:function (e.g., PCI:0:2:0, PCI:1:0:0)
  ```

  In answers: `nvidia.nvidiaBusId = "PCI:1:0:0";` plus either
  `intelBusId = "PCI:0:2:0"` or `amdgpuBusId = "PCI:0:0:0"` (exactly one).
* Wayland: Niri/Wayland works with the open module; for the proprietary
  module or older GPUs, verify compositor support and fall back to X11
  if necessary.

See `docs/HARDENING.md` for consolidated hardening details.

### 🤖 AI & development

* **Local AI:** `aichat`, `rocm-smi`, Python data stack
* **Web apps:** isolated Brave instances (Wayland-native)
* **Chat swapper:** `aiswap` for fast model/persona switching

---

<a id="system-latencyflex-profiles"></a>

## 🗜️ System & LatencyFleX profiles

[content unchanged…]

---

<a id="install-configuration"></a>

## ⚙️ Install & configuration

[content unchanged…]

---

<a id="keybindings-niri"></a>

## ⌨️ Keybindings (Niri)

[content unchanged…]

---

<a id="core-package-set"></a>

## 📦 Core package set

[content unchanged…]

---

<a id="performance-monitoring"></a>

## ⚡ Performance & monitoring

[content unchanged…]

---

## 🎨 Credits

<a id="credits"></a>

* **Aurora Theme:** based on [Aurora Dotfiles by flickowoa](https://github.com/flickowoa/dotfiles/tree/aurora), ported/adapted to Nix and Niri workspaces
* **Catppuccin:** system palette by [Catppuccin](https://github.com/catppuccin/catppuccin)
* **Niri:** compositor by [YaLTeR](https://github.com/YaLTeR/niri)
