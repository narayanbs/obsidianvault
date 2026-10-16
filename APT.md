Here is a practical `apt` command cheat sheet for Ubuntu 26.04, especially useful since you're using both GNOME and Niri. These commands help you inspect packages, troubleshoot issues, maintain security, and understand where software comes from.

Most commands below are read-only. I'll mark the ones that change your system.

## 1. Inspect packages and their sources

These are some of the most useful commands to learn first.

| Command                    | What it does                                                  |                                       |
| -------------------------- | ------------------------------------------------------------- | ------------------------------------- |
| `apt policy kitty`         | Checks Kitty's version and repository                         |                                       |
| `apt show kitty`           | Shows package details, description, version, and dependencies |                                       |
| `apt-cache depends kitty`  | Shows declared dependencies                                   |                                       |
| `apt-cache rdepends kitty` | Shows packages that depend on Kitty                           |                                       |
| `apt list --installed`     | Lists installed packages                                      |                                       |
| `apt list --installed \\   | grep -i kitty`                                                | Searches installed packages for Kitty |
| `apt search terminal`      | Searches available packages by keyword                        |                                       |
| `apt-cache madison niri`   | Shows available package versions from configured sources      |                                       |
| `dpkg -L kitty`            | Lists files installed by the Kitty package                    |                                       |
| `dpkg -S /usr/bin/kitty`   | Identifies which package owns that file                       |                                       |

For example, if you want to investigate an application before installing it, use:

bash
```
apt show package-name
apt policy package-name
apt-cache depends package-name
```

Replace `package-name` with the actual package name.

## 2. Security and updates

Check available upgrades

bash
```
apt list --upgradable
```

Lists installed packages for which newer versions are available from your configured repositories.

Refresh package lists

bash
```
sudo apt update
```

Downloads current package metadata. It does not install upgrades.

Install available upgrades

bash
```
sudo apt upgrade
```

Installs available package updates. Review the proposed changes before confirming.

Inspect repository configuration

bash
```
apt policy
```

Shows package-source priorities and helps you understand which repositories `apt` uses.

Additional useful commands:

| Command | Purpose |
| --- | --- |
| `apt list --upgradable 2>/dev/null \\| grep -i security` | Quick search for upgrade entries containing “security”; not a complete security audit |
| `apt-get -s upgrade` | Simulates an upgrade without applying changes |
| `apt-get check` | Checks package dependencies for broken states |
| `sudo apt-get install --only-upgrade kitty` | Upgrades Kitty only, if an upgrade is available |
| `sudo apt install --fix-broken` | Attempts to repair broken package dependencies |

Security tip: Don't assume every update is a security update or that every security fix is labelled in the package name. Ubuntu's security notices and package changelogs are better sources for identifying security fixes.

## 3. Install, remove, and repair software

These commands modify your system when indicated.

| Command | What it does |
| --- | --- |
| `sudo apt install kitty` | Installs Kitty |
| `sudo apt remove kitty` | Removes Kitty but generally leaves system configuration files |
| `sudo apt purge kitty` | Removes Kitty and its package-managed system configuration files |
| `sudo apt autoremove` | Removes packages that `apt` considers no longer needed |
| `sudo apt install --reinstall kitty` | Reinstalls Kitty |
| `sudo apt download kitty` | Downloads the package file into your current directory |
| `apt-get -s remove kitty` | Simulates removing Kitty without doing it |

Be particularly careful with `autoremove`, `purge`, and dependency repairs. Review the package list before accepting, especially on a system with GNOME and Niri installed.

## 4. Understand what installed a package

This is helpful when troubleshooting or investigating software you don't remember installing.

bash
```
apt policy niri
apt-cache depends niri
apt-cache rdepends niri
dpkg -L niri
dpkg -s niri
```

These answer different questions:

- `apt policy` — where does the package come from, and which version is selected?
- `apt-cache depends` — what does it declare that it needs?
- `apt-cache rdepends` — what other packages declare a dependency on it?
- `dpkg -L` — what files did the package install?
- `dpkg -s` — what is the installed package's status and metadata?

To investigate the history of package operations, check:

bash
```
less /var/log/apt/history.log
```

For older logs, Ubuntu may have rotated or compressed the history files.

## 5. Find and manage PPAs

Since your Niri installation uses a PPA, these commands are particularly relevant.

| Command | Purpose |
| --- | --- |
| `grep -R -n 'ppa.launchpadcontent.net' /etc/apt/sources.list.d/` | Searches traditional repository entries for Launchpad PPAs |
| `ls -l /etc/apt/sources.list.d/` | Lists repository configuration files |
| `apt policy niri` | Checks which configured repository supplies Niri |
| `sudo apt update` | Refreshes metadata from configured repositories |

Ubuntu also supports deb822 `.sources` files, so searching only for `deb ` lines may miss some repositories.

Before removing a PPA, remember that removing the repository does not necessarily downgrade or remove software already installed from it. Tools such as `ppa-purge`, where available and appropriate, can help revert packages to Ubuntu versions.

## 6. Clean up package downloads

| Command | What it does |
| --- | --- |
| `apt clean` | Removes downloaded package files from the local cache |
| `apt autoclean` | Removes cached package files that can no longer be downloaded |
| `du -sh /var/cache/apt/archives` | Shows the approximate size of the package archive cache |
| `apt-get -s autoremove` | Previews packages that autoremove would consider removing |

Cleaning the package cache does not uninstall applications.

## 7. A daily maintenance routine

For a normal Ubuntu desktop, this is a reasonable routine:

bash
```

# 1. Refresh package metadata
sudo apt update

# 2. Check available upgrades
apt list --upgradable

# 3. Check whether package dependencies are healthy
sudo apt-get check

# 4. Review the proposed upgrade changes
apt-get -s upgrade
```

If everything looks reasonable, run:

bash
```
sudo apt upgrade
```

You don't need to run every diagnostic command every day. The goal is to understand your package sources, review upgrades, and apply security fixes regularly.

## 8. Five commands I'd memorize first

apt policy PACKAGE

 Copy

Version and repository source

apt show PACKAGE

 Copy

Package details

apt-cache depends PACKAGE

 Copy

Dependencies

apt list --upgradable

 Copy

Available upgrades

dpkg -L PACKAGE

 Copy

Files installed by a package

One final distinction worth remembering: `apt` manages packages, but it doesn't fully audit the security of an application, its runtime permissions, or the PPA maintainer. For Niri, package-source verification and sensible session configuration remain important alongside normal updates.
