# Reproducible Nix Toolbox

Declare and reproduce your complete development environment — **tools** and **configurations** — across Linux and macOS systems.

## Prerequisites

### Set up Nix

**Skip this step if you're using a multi-user installation.** For single-user installations, create the `/nix` directory first:

```bash
sudo mkdir -p /nix
sudo chown -R $(whoami) /nix
```

Alternatively, if you're setting up for a different user, run this as root:

```bash
chown -R nix-user /nix
```

### Install Nix

You'll need Nix installed. Visit [nixos.org/download](https://nixos.org/download.html) to download and install it.

## Getting Started

### Clone the repository to your home directory

```bash
git clone <repo-url> $HOME
```

### Enable experimental features

Flakes are still an experimental feature in Nix, though they're widely used. For now, enable them using an environment variable instead of modifying the Nix configuration file directly (you can do that later with GNU Stow):

```bash
export NIX_CONFIG="experimental-features = nix-command flakes"
```

### Install the profile

Now install the profile:

```bash
nix profile add ~/dotfiles/nix/.config/nix
```

You should now have your first profile installed. Verify it by running:

```bash
nix profile list
```

You should see an entry named `nix-1`.

### Verify the installation

```bash
which git
which stow
git --version
```

You should see paths like `/home/user/.nix-profile/bin/git`. If you had previously installed these packages locally, Nix will now take precedence since it manages your `PATH` by default.

## Managing Your Toolbox

Since your configuration files are stored in this Git repository, using Stow allows you to symlink them to your system while keeping them version-controlled. This example focuses on Nix configuration, but you can apply the same approach to manage configuration files for any other tool or application.

First, create the `~/.config` directory if it doesn't exist:

```bash
mkdir -p ~/.config
```

Then run Stow to create symlinks. From within the dotfiles directory:

```bash
stow nix
```

Or specify the dotfiles path explicitly:

```bash
stow nix -d ~/dotfiles
```

To undo the symlinks later:

```bash
stow -D nix
```

Verify the symlink was created:

```bash
ls ~/.config
```

### Update your packages to the latest versions

First, update the `flake.lock` file to fetch the latest package versions:

```bash
nix flake update --flake ~/.config/nix
```

Then upgrade your profile:

```bash
nix profile upgrade nix-1
```

**Note:** The `flake.lock` file tracks specific versions of your packages and allows you to manage different generations. Always update it before upgrading your profile.

To see all available profiles:

```bash
nix profile list
```

### Roll back if something breaks

If an update causes issues, roll back to the previous generation:

```bash
nix profile rollback
```

To switch to a specific generation:

```bash
nix-env --switch-generation 2
```

To view your generation history:

```bash
nix profile history
```

**Note:** If you roll back to the first generation, your `nix-1` profile will temporarily stop working, but your packages are always safe in `/nix/store`.

### Add or remove tools

Edit `flake.nix` and modify the `paths` list:

```nix
paths = with pkgs; [
  git
  stow
  uv
  fnm
  neovim
  tmux
  ripgrep
  fzf
  # Add more packages here
];
```

Then upgrade your profile to apply the changes:

```bash
nix profile upgrade nix-1
```

**Note**: To find more packages, browse the official Nix package search: https://search.nixos.org/packages?channel=unstable&. The unstable channel provides the latest package versions. You can also use the stable channel for more conservative updates, but this requires changing the channel reference in your `flake.nix` file.

## Delete or disable your Nix profile

### Temporarily disable Nix

If you want to use locally installed packages instead of those provided by Nix, rename your profile:

```bash
mv ~/.nix-profile ~/.nix-profile-disabled
```

You can verify that Nix packages are no longer available:

```bash
which <package-name>
```

### Remove the profile permanently

To remove just the profile while keeping the binaries:

```bash
nix profile remove nix-1
```

Check how much space your Nix packages are using:

```bash
du -sh /nix
```

### Clean up everything

To completely remove the profile, its history, and all binaries:

```bash
nix profile remove nix-1
nix profile wipe-history
nix-collect-garbage
```

Verify that the binaries have been removed:

```bash
du -sh /nix
```

## Summary

This approach lets you list all your tools in one place `flake.nix`. With Stow managing your dotfiles, you get a reproducible environment, simple updates, and complete version control of **both** your tools and their configurations.

### Going further

You can extend this setup to version control configuration files for any tool. For example, you could keep your Neovim config files with the binary, manage your `.bashrc`, or organize settings for any other tool. Everything stays in one reproducible place.
