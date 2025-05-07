# pac-up

A comprehensive update script for Arch Linux and derivatives, designed to streamline system maintenance. It can be run as an interactive one-shot or install itself with a config file, hook directories, and more. **pac-up** handles mirror optimization, system package upgrades, AUR updates, old kernel and orphaned package cleanup, cache management, and Arch news checks. Built with flexibility and extensibility in mind, it supports colored output, interactive and non-interactive modes, custom pre/post/failure hooks, and optional logging.

## Features
- **Mirror Optimization:** Uses reflector or pacman-mirrors with GeoIP default or custom countries.

- **System Updates:** Runs pacman -Syu for core packages.

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

  - `paccache` (for advanced cache cleaning).

  - `yay` or `paru` (for AUR updates).

  - `curl` (for news checks).

  - `ionice` (for I/O priority in non-interactive mode).

## Installation
Download:

```bash
curl -sL https://raw.githubusercontent.com/cwadge/pac-up/main/pac-up -o pac-up
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
 --optimize-mirrors       Optimize mirror list (reflector or pacman-mirrors)
 --mirror-countries=LIST  Set countries for mirrors (e.g., "US,DE" for reflector, "United_States,Germany" for pacman-mirrors)
 --mirror-count=NUM       Set number of mirrors for reflector (default: 10)
 --aur-update             Apply AUR updates via specified user account (requires yay/paru)
 --aur-user=USERNAME      Specify user for AUR updates (overrides config/detection)
 --install                Create config and hook directories
 --help                   Display this help message
```
### Configuration

Edit `/etc/pac-up.conf` (after optional `sudo pac-up --install`).

## Example Run
Here’s what it looks like in action, optimizing mirrors and cleaning up, then running a post-hook script too:
```
$ sudo pac-up --optimize-mirrors --mirror-count=10 --mirror-countries=US --no-interactive --aur-update
[INFO] Loading configuration from /etc/pac-up.conf
[INFO] Setting CPU priority (nice level: 10)
[INFO] Setting I/O priority (class: 2, level: 7)
[INFO] Running pre hooks
 ________________________________ 
/ Exporting global variables...  \ 
\________________________________/ 
 ________________________________ 
/ Optimizing mirror list...      \ 
\________________________________/ 
[INFO] Optimizing mirror list with reflector (countries: US, count: 10)...
________________________________ 
/ Checking Arch news...          \ 
\________________________________/ 
[INFO] Fetching latest Arch news...
 1. Valkey to replace Redis in the [extra] Repository
 2. Cleaning up old repositories
 3. Glibc 2.41 corrupting Discord installation
 4. Critical rsync security release 3.4.0
 5. Providing a license for package sources
 ________________________________ 
/ Updating system packages...    \ 
\________________________________/ 
[INFO] Updating package database and system...
:: Synchronizing package databases...
 core                                     117.1 KiB   505 KiB/s 00:00 [#######################################] 100%
 extra                                      7.8 MiB  13.8 MiB/s 00:01 [#######################################] 100%
:: Starting full system upgrade...
 there is nothing to do
 ________________________________ 
/ Updating AUR packages...       \ 
\________________________________/ 
[WARNING] AUR updates are not supported in non-interactive mode, skipping...
 ________________________________ 
/ Cleaning package cache...      \ 
\________________________________/ 
[INFO] Keeping last 3 versions of cached packages...
==> no candidate packages found for pruning
 ________________________________ 
/ Removing orphaned packages...  \ 
\________________________________/ 
[INFO] No orphaned packages found.
 ________________________________ 
/ Cleaning old kernels...        \ 
\________________________________/ 
[INFO] No old kernels to remove.
 ________________________________ 
/ Syncing buffers to disk...     \ 
\________________________________/ 
[INFO] Running post hooks
[INFO] Running hook: yt-dlp-update.sh
Latest version: stable@2025.04.30 from yt-dlp/yt-dlp
yt-dlp is up to date (stable@2025.04.30 from yt-dlp/yt-dlp)
 ________________________________ 
/           Finished.            \ 
\________________________________/
```
Normally output is color-coded, if the terminal supports it.

## Hooks

If you want to extend pac-up’s functionality, you can add custom scripts to:
- `/etc/pac-up.d/pre.d/` (before updates).

- `/etc/pac-up.d/post.d/` (after updates).

- `/etc/pac-up.d/fail.d/` (on failure).

Scripts with `critical` in the name (e.g., `00-critical-check.sh`) halt execution if they fail.

Run `sudo pac-up --install` to create these directories (with a sample hook in `pre.d`).

## License

GNU General Public License v2.0 or later ([LICENSE](https://www.gnu.org/licenses/old-licenses/gpl-2.0.en.html)) - contributions must remain open-source.

## Contributing

You know the drill: fork, open an issue, or submit a pull request.

---

_A tool for maintaining Arch-based systems by Chris Wadge_
