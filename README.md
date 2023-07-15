# SteamDeck-RestorePKG
Restores all packages after SteamOS update

## How it works

#### pkglist-command

Creates 2 txt files (pkglist-repo.txt && pkglist-aur.txt) in current folder with all the packages currently installed AUR included.

using:

```bash
pacman -Qqen > pkglist-repo.txt
```

and

```bash
pacman -Qqem > pkglist-aur.txt
```

#### restore-command

Disable SteamOS read-only setting.

```bash
sudo steamos-readonly disable
```

Updates all the keys needed to install the packages.

```bash
sudo pacman-key --init
sudo pacman-key --populate
sudo pacman-key --refresh-keys
```

Installs all the packages.

```bash
sudo pacman -S --needed - < pkglist-repo.txt

for x in $(< pkglist-aur.txt); do yay -S $x; done
```

## How to use
Before the update run "pkglist-command" in a folder inside home.

After the update run "restore-command".

If you want re-eanble SteamOS read-only setting use in terminal:

```bash
sudo steamos-readonly enable
```
