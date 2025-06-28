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

- In the nixos config file the first line `{config, pkgs, ...}:` denotes that the configuration file is actually a function that takes at least two arguments config and pkgs and returns a set of option definitions in the second {}, which have the format `name = value` where name is the name of the option.

- The sets can be nested and the dots are actually shorthand for defining a set containing another set.

```zsh
{ config, pkgs, ... }:

{ services.httpd.enable = true;
  services.httpd.adminAddr = "alice@example.org";
  services.httpd.virtualHosts.localhost.documentRoot = "/webroot";
}

# this can also be written without shorthand

{ config, pkgs, ... }:

{ services = {
    httpd = {
      enable = true;
      adminAddr = "alice@example.org";
      virtualHosts = {
        localhost = {
          documentRoot = "/webroot";
        };
      };
    };
  };
}
```

- The long form may be more convenient when many options share the same prefix.
- Nixos checks the option definitions for correctness and the value for the correct type.
  - types in nix are strings, booleans, integers, sets (denoted by {} and seperated by `;`), lists ( denoted by [] and seperated by whitespace) or packages (most of the packages needed as a value are part of nix package collection which can be accessed through the function argument pkgs like `services.postgresql.package = pkgs.postgresql_14;`)

- If we have many repetitions then we can use abstractions like : 

```zsh
{
  services.httpd.virtualHosts =
    { "blog.example.org" = {
        documentRoot = "/webroot/blog.example.org";
        adminAddr = "alice@example.org";
        forceSSL = true;
        enableACME = true;
      };
      "wiki.example.org" = {
        documentRoot = "/webroot/wiki.example.org";
        adminAddr = "alice@example.org";
        forceSSL = true;
        enableACME = true;
      };
    };
}

# can write as 

let
  commonConfig =
    { adminAddr = "alice@example.org";
      forceSSL = true;
      enableACME = true;
    };
in
{
  services.httpd.virtualHosts =
    { "blog.example.org" = (commonConfig // { documentRoot = "/webroot/blog.example.org"; });
      "wiki.example.org" = (commonConfig // { documentRoot = "/webroot/wiki.example.org"; });
    };
}

```

- Here `let commonConfig = ...` defines a variable and the `//` operator merges two attribute sets to extend the commonconfig set.

- Functions are also available if we want to generate many things with identical config except one parameter or so : 

```nix

{
  services.httpd.virtualHosts =
    let
      makeVirtualHost = webroot:
        { documentRoot = webroot;
          adminAddr = "alice@example.org";
          forceSSL = true;
          enableACME = true;
        };
    in
      { "example.org" = (makeVirtualHost "/webroot/example.org");
        "example.com" = (makeVirtualHost "/webroot/example.com");
        "example.gov" = (makeVirtualHost "/webroot/example.gov");
        "example.nl" = (makeVirtualHost "/webroot/example.nl");
      };
}

```

- Modularity is possible with this import option where we need to have a list of other module files where config is defined : 

```nix
{ config, pkgs, ... }:

{ imports = [ ./vpn.nix ./kde.nix ];
  services.httpd.enable = true;
  environment.systemPackages = [ pkgs.emacs ];
  # ...
}
```

- Here both configuration.nix and kde.nix can define the option `environment.systemPackages` and when multiple modules define an option, nixos tries to merge the definitions, maybe concatenate the list of packages. If nixos is unable to merge properly then we get an error `The unique option "services.httpd.adminAddr" is defined multiple times, in "/etc/nixos/httpd.nix" and "/etc/nixos/configuration.nix".`
  - In this situation we can force one definition to take precedence over another like `services.httpd.adminAddr = pkgs.lib.mkForce "bob@example.org";`

- When we have multiple modules we might want to access config variables in other modules, and the `config` function argument allows this as it contains the complete merged system configuration. So it is both the input and the output of the configuration files and this is possible without infinite recursion as nix is a "lazy" language and only computes values when they are needed. Here is an example of this : 

```nix
{ config, pkgs, ... }:

{ environment.systemPackages =
    if config.services.xserver.enable then
      [ pkgs.firefox
        pkgs.thunderbird
      ]
    else
      [ ];
}
```

- If we want to find out the final value of a configuration option we can use the command `nixos-option`.

```zsh
nixos-option services.xserver.enable
# true

nixos-option boot.kernelModules
# [ "tun" "ipv6" "loop" ... ]
```

- NixOS has two styles of package management:

  - **Declarative** : where you declare packages in configuration.nix, need root and ensures consistent set of binaries
  - **Ad hoc**: Installing, upgrading and uninstalling packages using nix-env command, can so this as a non sudo user, allows mixing packages from different nixpkgs versions.

  - For declarative package management we can specify the packages in `environment.systemPackages`
  - To get a list of available packages we have the command : 

  ```zsh
  nix-env -qaP 'package_name' --description

  # The above command is very slow since it evaluates the entire NixPkgs package set (over 60,000 packages)\

  # A faster way to do this is to use targetted attribute paths like:
   nix-env -qaP -A nixpkgs.package_name


  ```

  - This command will return an output like `nixos.firefox   firefox-23.0   Mozilla Firefox - the browser, reloaded` here the forst column is the attribute name, where the nixos prefix tells use we can get the package from the nixos channel, while putting in declarative configuration use pkgs prefix.
  - To uninstall a package we need to remove it from environment.systemPackages and run nixos-rebuild switch.

  - To configure the nixpkgs repo itself we can set the `nixpkgs.config` option, like:
  
  ```nix
  {
    nixpkgs.config = {
      allowUnfree = true;
    };
  }
  ```

  - Nixpkgs currently doesnt have a way to query available package configuration options but some packages have options to change optional functionality or other aspects.

  - We can also change or disable dependencies like if Emacs has a depedency on GTK2 by default and we want to build it using gtk3 then we can do:

  ```nix
    environment.systemPackages = [ (pkgs.emacs.override { gtk = pkgs.gtk3; }) ];
  ```

- Here the `override` function ammends the arguments to the function that produces Emacs.

- To change the package even more we have the `overrideAttrs` function which would allow changing the source code of the package, like : 
```nix
{
  environment.systemPackages = [
    (pkgs.emacs.overrideAttrs (oldAttrs: {
      name = "emacs-25.0-pre";
      src = /path/to/my/emacs/tree;
    }))
  ];
}
```

- Here overrideAttrs takes the nix derivation produces by pkgs.emacs and produces a new derivation of the package where the original name and src attributes have been replaced.
- These overrides are not global and do not affect the original package, and other packages continue to depend on the original package instead of the custom package, which means oyu would have two instances of the package. If we want everything to depend on the custom package only we can apply a global oevrride : 

```nix
{
  nixpkgs.config.packageOverrides = pkgs:
    { emacs = pkgs.emacs.override { gtk = pkgs.gtk3; };
    };
}
```

- If we have a package which is not available in nixos, we can add the package from outside the nixpkgs tree.

```nix

# in configuration.nix
{
  environment.systemPackages = [ (import ./my-hello.nix) ];
}

# then in a file called my-hello.nix

with import <nixpkgs> {}; # bring all of Nixpkgs into scope

stdenv.mkDerivation rec {
  name = "hello-2.8";
  src = fetchurl {
    url = "mirror://gnu/hello/${name}.tar.gz";
    hash = "sha256-5rd/gffPfa761Kn1tl3myunD8TuM+66oy1O7XqVGDXM=";
  };
}

```

- We can test the package like :

```zsh
nix-build my-hello.nix

./result/bin/hello

# returns "Hello, world!"
```

- We can also use pre built executables, but most of them will not work on nix except flatpaks and appimages.
- To add support for appimahes we need to add these to configuration.nix

```nix
  programs.appimage.enable = true;
  programs.appimage.binfmt = true;
```

- Then when we need to run the appimage we can do `appimage-run foo.appimage`


### Ad Hoc Package management

- We can install and uninstall packages from the command line using nix-env. If we envoke this as root then the package is installed in the nix profile `/nix/var/nix/profiles/default` but if we dont use sudo then then package will be in `/nix/var/nix/profiles/per-user/username/profile` and will not be visible to other users.

- The -A flag specifies the package by the attribute name; without it the package is installed bu matching against its package name which is slow as requires matching against all nix packages, and may be ambiguous if multiple matching packages.
- Since packages come from the nixos channel we upgrade packages by updating to the latest version of the nixos channel by :

```zsh
nix-channel --update nixos
nix-env -iA package_name
```

- Other packages in the profile are not affected, which is what seperates the ad hoc style from declarative style, where running nixos-rebuild switch causes all packages to be updated to their current versions in the nixos channel. We can upgrade all packages for which there is a current version in ad hoc by :

```zsh
nix-env -u '*'
```

- To uninstall a package in ad hoc method we need to use the -e flag:

```zsh
nix-env -e thunderbird
```

- To rollback an undesirable nix-env action we have:

```zsh
nix-env --rollback
```


