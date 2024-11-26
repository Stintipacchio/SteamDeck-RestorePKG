# SteamDeck-RestorePKG
Automates the restoration of all installed packages after a SteamOS update, including AUR packages.

## How It Works

### **pkglist-command**
This command generates a list of all currently installed packages, separating repository and AUR packages. It creates two text files in the current directory: `pkglist-repo.txt` and `pkglist-aur.txt`.

Commands:
```bash
pacman -Qqen > pkglist-repo.txt
pacman -Qqem > pkglist-aur.txt
```

- `pacman -Qqen > pkglist-repo.txt`:
  - This command generates a list of all packages installed from official repositories (`-Qqen`), and saves it to `pkglist-repo.txt`. The options are:
    - `-Qq`: Provides a simple list of package names without additional details.
    - `-e`: Only includes packages explicitly installed by the user.
    - `-n`: Limits the list to official repository packages.

- `pacman -Qqem > pkglist-aur.txt`:
  - This command generates a list of AUR (Arch User Repository) packages (`-Qqem`), and saves it to `pkglist-aur.txt`. The options are:
    - `-Qq`: Provides a simple list of package names without additional details.
    - `-e`: Only includes packages explicitly installed by the user.
    - `-m`: Limits the list to AUR packages.

### **restore-command**
This script restores the system to its pre-update state by reinstalling all packages and dependencies, including AUR packages, while handling keyring issues and repository updates.

Script:
```bash
#!/bin/sh

# Disable SteamOS read-only mode
sudo steamos-readonly disable
```
- `sudo steamos-readonly disable`:
  - SteamOS is configured to be read-only by default to protect the integrity of system files. This command disables the read-only mode so that modifications can be made.

```bash
# Update the system
sudo pacman -Syu --noconfirm
```
- `sudo pacman -Syu --noconfirm`:
  - Updates the system (`-Syu`) to ensure all packages are up-to-date before proceeding. The `--noconfirm` flag skips user prompts to proceed automatically.

```bash
# Reset and refresh pacman keyring
sudo rm -rf /etc/pacman.d/gnupg/ && sudo pacman-key --init && sudo pacman-key --populate
```
- `sudo rm -rf /etc/pacman.d/gnupg/`:
  - Removes the existing GPG keys used by pacman. This ensures that any corrupted or old keys are cleared before reinitialization.
- `sudo pacman-key --init`:
  - Initializes a new keyring for the pacman package manager.
- `sudo pacman-key --populate`:
  - Populates the keyring with the default set of keys required to validate packages.

```bash
# Reinstall repository packages
sudo pacman -S --needed - < pkglist-repo.txt --overwrite "*" --noconfirm
```
- `sudo pacman -S --needed - < pkglist-repo.txt --overwrite "*" --noconfirm`:
  - Installs all the packages listed in `pkglist-repo.txt`. The options are:
    - `--needed`: Skips reinstalling packages that are already installed.
    - `--overwrite "*"`: Overwrites any conflicting files to ensure the installation proceeds smoothly.
    - `--noconfirm`: Bypasses prompts, running automatically.

```bash
# Ensure Git and base-devel are installed
sudo pacman -S --needed git base-devel --noconfirm
```
- `sudo pacman -S --needed git base-devel --noconfirm`:
  - Installs essential packages (`git` and `base-devel`) required for building AUR packages. The `base-devel` group is necessary for compiling software from source.

```bash
# Clean up Yay cache and clone the Yay AUR helper
sudo rm -r /home/deck/.cache/yay
sudo rm -r ./yay-bin
```
- `sudo rm -r /home/deck/.cache/yay`:
  - Removes the cache directory used by Yay, an AUR helper, to ensure no stale or outdated files are used during package restoration.
- `sudo rm -r ./yay-bin`:
  - Deletes any existing local copy of the Yay AUR helper source directory to ensure a clean reinstallation.

```bash
sudo pacman -S --needed git base-devel && \
git clone https://aur.archlinux.org/yay-bin.git && \
cd yay-bin && \
git checkout cb857e898d7081a60cf8742d26247fd6a3c5443c && \
yes | makepkg -si
```
- `git clone https://aur.archlinux.org/yay-bin.git`:
  - Clones the Yay AUR helper repository from the AUR.
- `cd yay-bin`:
  - Changes to the `yay-bin` directory to begin the build process.
- `git checkout cb857e898d7081a60cf8742d26247fd6a3c5443c`:
  - Checks out a specific commit of the `yay-bin` repository to ensure a stable version is used.
- `yes | makepkg -si`:
  - Builds and installs the Yay package, automatically confirming any prompts (`yes`).

```bash
# Install AUR packages from the saved list
for x in $(< pkglist-aur.txt); do yay -S $x --noconfirm; done
```
- `for x in $(< pkglist-aur.txt); do yay -S $x --noconfirm; done`:
  - Loops through each package listed in `pkglist-aur.txt` and installs it using Yay (`-S`). The `--noconfirm` flag automatically confirms each installation.


**Note**: Do not run the script as the root user. Running it as root can cause permission issues and unintended behavior.

Execute the restore script:
- Right-click on the script file and select **Run in Konsole**
- Or run it directly in the terminal:
  ```bash
  ./restore-command.sh
  ```

### 3. Re-enable SteamOS Read-Only Mode
If you want to return to the default SteamOS read-only state, run:
```bash
sudo steamos-readonly enable
```

## Key Features of the Updated Script
- Automatically refreshes and reinitializes the pacman keyring.
- Resolves potential conflicts with files using `--overwrite`.
- Handles AUR package restoration using the Yay AUR helper, including cleaning up old caches.
- Automates system updates as part of the restoration process.

