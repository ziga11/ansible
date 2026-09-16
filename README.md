# Fedora Hyprland Dev Workstation — Ansible Setup

Provisions a fresh Fedora box with:

- **Hyprland** (via the `solopasha/hyprland` COPR — not yet in official Fedora repos on most releases) + hyprpaper/hyprlock/hypridle/waybar/dunst
- **Kitty** terminal
- **Zsh + Oh My Zsh** (unattended install, autosuggestions + syntax-highlighting plugins)
- **Tmux** with **TPM**, **resurrect**, and **continuum** (auto-save every 15 min, auto-restore on start)
- **Kanata** (built from source via `cargo`, since there's no Fedora package — includes the `input` group + udev rule it needs to touch `/dev/uinput`)
- **Neovim** with **kickstart.nvim** cloned into `~/.config/nvim` (existing non-git configs get backed up first)
- **Brave** browser (official RPM repo)
- **grim + slurp + wl-clipboard** for Wayland screenshots
- **PHP + extensions + Composer + the Laravel installer**

## Requirements

On the machine you're running Ansible *from* (can be the same Fedora box):

```bash
sudo dnf install ansible
ansible-galaxy collection install -r requirements.yml
```

## Usage

```bash
ansible-playbook -K -i inventory.ini playbook.yml
```

`-K` prompts for your sudo password (needed for `dnf`/`copr`/udev steps).

Run it as your normal user — the playbook uses `become` only on the specific
tasks that need root, and installs everything else (oh-my-zsh, tmux plugins,
kickstart.nvim, kanata via cargo) into your own home directory.

### Run just one part

Everything is tagged, so you can install piecemeal:

```bash
ansible-playbook -K playbook.yml --tags hyprland,kitty
ansible-playbook -K playbook.yml --tags tmux
ansible-playbook -K playbook.yml --tags kanata
ansible-playbook -K playbook.yml --tags neovim
ansible-playbook -K playbook.yml --tags brave
ansible-playbook -K playbook.yml --tags php
```

Available tags: `base`, `hyprland`, `kitty`, `zsh`, `tmux`, `kanata`,
`neovim`, `brave`, `grim`, `php`.

### Target a different user

By default it installs for `$USER` (whoever runs the playbook). Override with:

```bash
ansible-playbook -K playbook.yml -e target_user=someoneelse
```

## Post-install notes

- **Log out and back in** (or reboot) after the first run — the `input` group
  membership needed by kanata won't apply to your current session otherwise.
- **kanata** is installed but *not* auto-started, since it ships with only a
  placeholder `~/.config/kanata/kanata.kbd`. Once you drop in your real
  config, enable it with:
  ```bash
  systemctl --user daemon-reload
  systemctl --user enable --now kanata.service
  ```
- **tmux**: prefix + `I` inside tmux will re-fetch/update plugins later;
  the playbook already runs the headless installer once for you.
- Since you mentioned you'll be adding your own dotfiles/repo links for the
  actual configs later — the files this playbook drops (`.tmux.conf`,
  `kanata.kbd`, kickstart's `nvim` clone) are meant as safe, working
  defaults you can freely overwrite. Nothing here fights a `git clone`
  of your own dotfiles over the top.

## Structure

```
.
├── inventory.ini
├── playbook.yml
├── requirements.yml
└── README.md
```
