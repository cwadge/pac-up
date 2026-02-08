# pac-up

A comprehensive update script for Arch Linux and derivatives, designed to streamline system maintenance. It can be run as an interactive one-shot or install itself with a config file, hook directories, and more. **pac-up** handles mirror optimization, system package upgrades, AUR updates, old kernel and orphaned package cleanup, cache management, and Arch news checks. Built with flexibility and extensibility in mind, it supports colored output, interactive and non-interactive modes, custom pre/post/failure hooks, and optional logging.

## Features
- **Mirror Optimization:** Uses reflector or pacman-mirrors with GeoIP default or custom countries.

- **System Updates:** Runs pacman -Syu for core packages.

- **Dry-Run Mode:** Preview all changes without modifying the system.

- **AUR Support:** Can do AUR updates via yay or paru if installed.

- **Kernel Cleanup:** Removes outdated kernels, keeping the current and previous versions.

- **Cache Cleaning:** Manages pacman cache with paccache or basic cleanup, and can purge orphaned packages.

- **Arch News Check:** Fetches the latest Arch Linux news headlines so you know what to expect.

- **Extensible via Hooks:** Pre, post, and failure hooks in /etc/pac-up.d for customization.

- **Resource Control:** Adjustable nice and ionice for non-interactive runs.

- **Logging:** Optional logs to /var/log/pac-up.log.

## Requirements
- `Arch Linux`-based system (e.g. Arch, Manjaro, etc.)

- `Bash` (works with sh, but error handling’s weaker).

- `pacman` (core requirement).

- `Root` privileges (sudo or direct root).

- **Optional:**

  - `reflector` or `pacman-mirrors` (for mirror optimization).

  - `paccache` (for advanced cache cleaning - included in `pacman-contrib` package).

  - `paru` or `yay` (for AUR updates).

  - `curl` (for news checks).

  - `ionice` (for lower I/O priority in non-interactive mode).

## Installation
Download:

```bash
wget https://raw.githubusercontent.com/cwadge/pac-up/main/pac-up -O pac-up
```
Or clone the repo:

```bash
git clone https://github.com/cwadge/pac-up.git
cd pac-up
```

Make executable:
```bash
chmod +x pac-up
```

(Optional but recommended) Install System-Wide:

```bash
sudo mv pac-up /usr/local/sbin/pac-up
```

Or  download + install all at once:
```bash
sudo wget https://raw.githubusercontent.com/cwadge/pac-up/main/pac-up -O /usr/local/sbin/pac-up && sudo chmod 755 /usr/local/sbin/pac-up
```

## Usage
Run interactively:
```bash
sudo pac-up
```
Run non-interactively (e.g., cron):
```bash
sudo pac-up --no-interactive
```

### Options
```
--no-interactive         Run without user interaction
--no-system-update       Skip system package updates
--no-cache-clean         Skip package cache cleanup
--no-orphan-cleanup      Skip orphaned package removal
--no-kernel-cleanup      Skip old kernel removal
--no-news                Skip Arch news check
--no-hooks               Skip running hook scripts
--no-optimize-mirrors    Skip optimizing the mirror list
--no-aur-update          Skip AUR updates
--interactive            Run the script in interactive mode
--aur-update             Apply AUR updates via specified user account (requires yay/paru)
--aur-user=USERNAME      Specify user for AUR updates (overrides config/detection)
--system-update          Update system packages
--cache-clean            Perform package cache cleanup
--orphan-cleanup         Cleanup orphaned packages
--kernel-cleanup         Remove old kernels
--news                   Check the Arch news RSS feed
--hooks                  Run hook scripts
--optimize-mirrors       Optimize mirror list (reflector or pacman-mirrors)
--mirror-countries=LIST  Set countries for mirrors (e.g., "US,DE" for reflector, "United_States,Germany" for pacman-mirrors)
--mirror-count=NUM       Set number of mirrors for reflector (default: 10)
--dry-run                Show what would be done without making changes
--install                Create config and hook directories
--help                   Display this help message
```
### Configuration

Edit `/etc/pac-up.conf` (after optional `sudo pac-up --install`).

## Dry-Run Mode

Preview all changes before committing to them with `--dry-run`. This mode shows what would happen without modifying your system.

**Important:** Dry-run mode **does** download package lists (via `pacman -Sy`) to show accurate information about available updates. This is necessary to provide meaningful previews. However, no packages are installed, removed, or upgraded.

### What Gets Checked

- **Mirror Optimization:** Shows which mirror tool would be used and with what settings
- **Arch News:** Normal display (read-only operation)
- **System Updates:** Downloads package lists, then shows what would be upgraded (using `pacman -Sup`)
- **AUR Updates:** Shows available AUR updates (using `yay -Qua` or `paru -Qua`)
- **Cache Cleaning:** Shows current cache size and what would be cleaned
- **Orphan Cleanup:** Lists orphaned packages that would be removed
- **Kernel Cleanup:** Lists old kernels that would be removed
- **Hooks:** Lists which hooks would run without executing them

### Usage

Basic dry-run:
```bash
sudo pac-up --dry-run
```

Dry-run for specific operations:
```bash
# Preview system update only
sudo pac-up --dry-run --no-kernel-cleanup --no-orphan-cleanup

# Check what kernels would be cleaned
sudo pac-up --dry-run --no-system-update --no-cache-clean
```

### Hook Scripts

When dry-run mode is active, hook scripts are **not executed**. Instead, pac-up displays which hooks would be run:

```bash
[INFO] Running pre hooks
[DRY-RUN] Would run hook: 00-example
[DRY-RUN] Would run hook: 50-custom-backup
```

### Example Output

```bash
$ sudo pac-up --dry-run

╔════════════════════════════════════════════════════════════════╗
║                         DRY-RUN MODE                           ║
║            No changes will be made to the system               ║
╚════════════════════════════════════════════════════════════════╝

[INFO] Running pre hooks
[DRY-RUN] Would run hook: 00-example

[DRY-RUN] Downloading latest package lists...
[DRY-RUN] Checking for available updates...
[DRY-RUN] Packages that would be upgraded (5 total):
https://mirror.example.com/archlinux/core/os/x86_64/linux-6.7.2-1-x86_64.pkg.tar.zst
https://mirror.example.com/archlinux/extra/os/x86_64/firefox-122.0-1-x86_64.pkg.tar.zst
...

[DRY-RUN] Current package cache size: 2.3G
[DRY-RUN] Would keep last 3 versions of cached packages (paccache -r)

[DRY-RUN] No orphaned packages found

[DRY-RUN] No old kernels to remove

[INFO] Running post hooks
[DRY-RUN] Would run hook: yt-dlp-update.sh
```

## Example Run
Here's what it looks like in action, including running a post-hook script:
```
[INFO] Loading configuration from /etc/pac-up.conf
╔════════════════════════════════════════════════════════════════╗
║                 Running pre-update scripts...                  ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Running pre hooks
╔════════════════════════════════════════════════════════════════╗
║                 Exporting global variables...                  ║
╚════════════════════════════════════════════════════════════════╝
╔════════════════════════════════════════════════════════════════╗
║                      Checking Arch news...                     ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Fetching latest Arch news...
 1. linux-firmware >= 20250613.12fe085f-5 upgrade requires manual intervention
 2. Plasma 6.4.0 will need manual intervention if you are on X11
 3. Transition to the new WoW64 wine and wine-staging
 4. Valkey to replace Redis in the [extra] Repository
 5. Cleaning up old repositories
╔════════════════════════════════════════════════════════════════╗
║                   Updating system packages...                  ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Updating package database and system...
:: Synchronizing package databases...
 core is up to date
 extra is up to date
 multilib is up to date
:: Starting full system upgrade...
 there is nothing to do
╔════════════════════════════════════════════════════════════════╗
║                    Cleaning package cache...                   ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Keeping last 3 versions of cached packages...
==> no candidate packages found for pruning
╔════════════════════════════════════════════════════════════════╗
║                  Removing orphaned packages...                 ║
╚════════════════════════════════════════════════════════════════╝
[INFO] No orphaned packages found.
╔════════════════════════════════════════════════════════════════╗
║                     Cleaning old kernels...                    ║
╚════════════════════════════════════════════════════════════════╝
[INFO] No old kernels to remove.
╔════════════════════════════════════════════════════════════════╗
║                   Syncing buffers to disk...                   ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Disk sync completed successfully.
╔════════════════════════════════════════════════════════════════╗
║                 Running post-update scripts...                 ║
╚════════════════════════════════════════════════════════════════╝
[INFO] Running post hooks
[INFO] Running hook: yt-dlp-update.sh
Latest version: stable@2025.06.30 from yt-dlp/yt-dlp
yt-dlp is up to date (stable@2025.06.30 from yt-dlp/yt-dlp)
╔════════════════════════════════════════════════════════════════╗
║                           Finished.                            ║
╚════════════════════════════════════════════════════════════════╝
```
(Normally output is color-coded, if the terminal supports it.)

## Hooks

If you want to extend pac-up’s functionality, you can add custom scripts to:
- `/etc/pac-up.d/pre.d/` (before updates).

- `/etc/pac-up.d/post.d/` (after updates).

- `/etc/pac-up.d/fail.d/` (on failure).

NOTE: Scripts with `critical` in the name (e.g., `00-critical-check.sh`) halt execution if they fail.

Run `sudo pac-up --install` to create these directories (with a sample hook in `pre.d`).

## License

GNU General Public License v2.0 or later ([LICENSE](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)) - contributions must remain open-source.

## Contributing

You know the drill: fork, open an issue, or submit a pull request.

---

_A tool for maintaining Arch-based systems by Chris Wadge_
