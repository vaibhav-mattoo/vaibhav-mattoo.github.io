---
title: Configuring NixOS
description: How to change your NixOS cnofiguration
layout: post
categories: [OS, NixOS]
tags: [Operating systems, Configuration]
date: 2025-06-27 08:20 -0700
---

- Current configuration of the machine held in the file /etc/nixos/configuration.nix
- After making changes in this file we run the command:
```zsh
sudo nixos-rebuild switch
```
    - This builds the new configuration and makes it the default configuration for booting in the grub menu and restarts all the user services (by reload, not starting and stopping each service) to get the configuration in the running system.
- There are other options with nixos-rebuild :

```zsh
# to build and switch without making default boot
# so reboot returns current config

sudo nixos-rebuild boot

# to just build the config to check compilation

sudo nixos-rebuild build
```
- To keep the installation upto date we use a nixos channel.
    - A channel is basically a snapshot of the nixpkgs repo (which has all the package definitions and NixOS modules) and this snapshot is verified (passes nixos CICD).
- Channel types are:
    - **Stable channel**: These only get conservative bug fixes and package upgrades, and have a guarantee of no breaking changes till the next stable branch of linux has been released. New stable channel is released every 6 months.
    - **Unstable channel**: Corresponds to nixos's main development branch and may see big changes between channel updates. Is updated on a rolling basis every few days (not as frequent as arch).
    - **Small channels**: can be nixos-25.05-small or nixos-unstable-small and is identical to the stable and unstable branch but has fewer binary packages, so they get faster updates than regular channels, but may require more packages to be built from source and is mostly for servers.
- Here are some nix-channel commands:
```zsh
# to see which nix channels you're subscribed to
nix-channel --list

# to switch to a different nix channel
nix-channel --add https://channels.nixos.org/channel-name nixos

# to upgrade nixos to the latest version in the chosen channel
nic-channel switch --upgrade
```
- Channels are set per user so running `nix-channel --add` as a non root user or without sudo will not affect configuration in /etc/nixos/configuration.nix.
- To keep nixos up to date automatically, in configuration.nix add:

```nix
  system.autoUpgrade.enable = true;
  system.autoUpgrade.allowReboot = true;
```
- These lines enable a periodically executed systemd service called nixos-upgrade.service.
```nix
# to specify a channel to autoUpgrade
{
  system.autoUpgrade.channel = "https://channels.nixos.org/nixos-25.05";
}
```

