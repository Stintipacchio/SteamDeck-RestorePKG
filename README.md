# SteamDeck-RestorePKG
Restores all packages after SteamOS update

#### pkglist-command

creates 2 txt files (pkglist-repo.txt && pkglist-aur.txt) in current folder with all the packages currently installed AUR included.

#### restore-command

Disable SteamOS read-only setting.
Updates all the keys needed to install the packages.
Installs all the packages.

#### How to use
Before the update run "pkglist-command" in a folder inside home.
After the update run "restore-command".
