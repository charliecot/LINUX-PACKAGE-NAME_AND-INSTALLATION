# Linux Ubuntu Command Reference
## Package Installation, System Administration, Networking, Security and Development

**Platform:** Ubuntu / Debian / WSL Ubuntu  
**Purpose:** A practical beginner-to-advanced command and package reference.

> **Important:** Linux command names and package names are not always identical. For example, `ifconfig` is provided by the `net-tools` package. Package availability can vary by Ubuntu release and enabled repositories. Check uncertain packages with `apt-cache policy PACKAGE` or `apt-file search`.

---

# Table of Contents

1. [Understanding Package Management](#1-understanding-package-management)
2. [APT: Basic to Advanced](#2-apt-basic-to-advanced)
3. [DPKG: Managing `.deb` Packages](#3-dpkg-managing-deb-packages)
4. [Snap Packages](#4-snap-packages)
5. [Python Package Management](#5-python-package-management)
6. [Networking Commands and Packages](#6-networking-commands-and-packages)
7. [Operating-System Information](#7-operating-system-information)
8. [System Resource Monitoring](#8-system-resource-monitoring)
9. [File and Directory Management](#9-file-and-directory-management)
10. [Permissions and Ownership](#10-permissions-and-ownership)
11. [Process Management](#11-process-management)
12. [Users and Groups](#12-users-and-groups)
13. [Storage and Filesystems](#13-storage-and-filesystems)
14. [Archives and Compression](#14-archives-and-compression)
15. [Services and systemd](#15-services-and-systemd)
16. [Security and Firewall](#16-security-and-firewall)
17. [Text Processing and Data Utilities](#17-text-processing-and-data-utilities)
18. [Environment and Shell Commands](#18-environment-and-shell-commands)
19. [Scheduling and Automation](#19-scheduling-and-automation)
20. [Development Tools](#20-development-tools)
21. [External Repositories and Package Downloads](#21-external-repositories-and-package-downloads)
22. [Downloading and Transferring Files](#22-downloading-and-transferring-files)
23. [Package Name Quick Reference](#23-package-name-quick-reference)
24. [Troubleshooting](#24-troubleshooting)
25. [WSL-Specific Commands](#25-wsl-specific-commands)
26. [Shell Operators and Redirection](#26-shell-operators-and-redirection)
27. [Learning Roadmap](#27-learning-roadmap)
28. [How to Discover Commands Independently](#28-how-to-discover-commands-independently)

---

# 1. Understanding Package Management

A package manager installs, upgrades, removes and tracks software on a Linux system.

| Tool | Purpose |
|---|---|
| `apt` | Main package-management interface on Ubuntu/Debian |
| `apt-get` | Traditional, script-friendly APT interface |
| `apt-cache` | Search and inspect package metadata |
| `dpkg` | Low-level management of `.deb` packages |
| `snap` | Installs Snap applications |
| `pip` | Installs Python packages |
| `uv` | Python project and package manager |
| `npm` | Installs Node.js packages |
| `gem` | Installs Ruby packages |
| `cargo` | Installs Rust packages |

These tools manage different software ecosystems. For example, `apt install python3-requests` installs an Ubuntu-managed Python package, while `python -m pip install requests` installs a Python package into the active Python environment.

## 1.1 Command name versus package name

Example:

```bash
ifconfig
```

The Ubuntu package that provides it is:

```bash
sudo apt install net-tools
```

Another example:

```bash
dig example.com
```

The command is `dig`; the package is commonly `dnsutils` on Ubuntu.

To discover a package:

```bash
apt search PACKAGE
apt show PACKAGE
apt-cache policy PACKAGE
```

---

# 2. APT: Basic to Advanced

APT stands for Advanced Package Tool. It is the standard package manager for Ubuntu and Debian.

## 2.1 Update package indexes

```bash
sudo apt update
```

Downloads current package metadata from configured repositories. It does not upgrade installed applications.

## 2.2 Upgrade installed packages

```bash
sudo apt upgrade
```

Upgrades installed packages without normally removing packages to resolve dependency changes.

```bash
sudo apt full-upgrade
```

Performs a more comprehensive upgrade and may install or remove packages to resolve dependencies.

```bash
sudo apt-get dist-upgrade
```

Traditional `apt-get` command for a full dependency-aware upgrade.

## 2.3 Install packages

```bash
sudo apt install PACKAGE
sudo apt install curl
sudo apt install curl wget git vim
sudo apt install -y curl
sudo apt install --no-install-recommends nginx
```

- `install`: install a package.
- Multiple package names: install them together.
- `-y`: automatically answer yes to confirmation prompts.
- `--no-install-recommends`: do not install recommended packages.

Use `-y` carefully because it bypasses confirmation.

## 2.4 Remove packages

```bash
sudo apt remove PACKAGE
sudo apt purge PACKAGE
sudo apt autoremove
sudo apt clean
sudo apt autoclean
```

| Command | Meaning |
|---|---|
| `remove` | Remove package but generally retain package-managed configuration |
| `purge` | Remove package and package-managed configuration |
| `autoremove` | Remove automatically installed dependencies no longer needed |
| `clean` | Remove downloaded package files from the local cache |
| `autoclean` | Remove obsolete package files from the cache |

## 2.5 Search and inspect packages

```bash
apt search PACKAGE
apt show PACKAGE
apt-cache show PACKAGE
apt-cache policy PACKAGE
apt list --installed
apt list --installed | grep python
```

- `apt search`: search package names and descriptions.
- `apt show`: show package metadata.
- `apt-cache policy`: show candidate and repository versions.
- `apt list --installed`: list installed packages.

## 2.6 Advanced APT commands

```bash
apt download PACKAGE
sudo apt reinstall PACKAGE
sudo apt-get check
sudo apt-get -f install
apt-cache depends PACKAGE
apt-cache rdepends PACKAGE
apt-cache madison PACKAGE
```

| Command | Purpose |
|---|---|
| `apt download` | Download a package without installing it |
| `apt reinstall` | Reinstall a package |
| `apt-get check` | Check dependency problems |
| `apt-get -f install` | Attempt to repair broken dependencies |
| `apt-cache depends` | Show dependencies |
| `apt-cache rdepends` | Show reverse dependencies |
| `apt-cache madison` | Display available package versions from configured sources |

## 2.7 Hold and unhold packages

Prevent normal APT upgrades from upgrading a package:

```bash
sudo apt-mark hold nginx
```

Remove the hold:

```bash
sudo apt-mark unhold nginx
```

Show held packages:

```bash
apt-mark showhold
```

Show manually installed packages:

```bash
apt-mark showmanual
```

---

# 3. DPKG: Managing `.deb` Packages

`dpkg` is a lower-level package-management tool. APT normally uses it to install `.deb` packages.

## 3.1 Install a local `.deb` file

```bash
sudo dpkg -i application.deb
```

If dependencies are missing, try:

```bash
sudo apt --fix-broken install
```

For a local package file, APT can also resolve dependencies:

```bash
sudo apt install ./application.deb
```

## 3.2 Inspect packages

```bash
dpkg -l
dpkg -l | grep curl
dpkg -s curl
dpkg -L curl
dpkg -S /usr/bin/curl
sudo dpkg --audit
```

| Command | Purpose |
|---|---|
| `dpkg -l` | List packages |
| `dpkg -s PACKAGE` | Show package status |
| `dpkg -L PACKAGE` | List files installed by a package |
| `dpkg -S PATH` | Find which installed package owns a file |
| `dpkg --audit` | Check for partially installed or inconsistent packages |

## 3.3 Remove packages

```bash
sudo dpkg -r PACKAGE
sudo dpkg -P PACKAGE
```

- `-r`: remove the package.
- `-P`: purge the package, including package-managed configuration files.

Prefer APT for normal installation and removal because it handles dependencies.

---

# 4. Snap Packages

Snap is another Linux application distribution and packaging system.

```bash
snap version
snap find APPLICATION
sudo snap install APPLICATION
snap list
snap info APPLICATION
sudo snap refresh
sudo snap refresh APPLICATION
sudo snap remove APPLICATION
snap services
snap connections
```

Example:

```bash
sudo snap install code --classic
```

**WSL note:** Snap and systemd support depend on WSL configuration. Some Snap applications may not work in a default WSL installation.

---

# 5. Python Package Management

## 5.1 Install Python and pip through APT

```bash
sudo apt update
sudo apt install python3 python3-pip python3-venv python3-dev
```

Check versions:

```bash
python3 --version
python3 -m pip --version
```

## 5.2 Install Python packages using pip

```bash
python3 -m pip install requests
python3 -m pip install flask django requests
python3 -m pip install flask==3.1.0
python3 -m pip install --upgrade requests
python3 -m pip uninstall requests
python3 -m pip list
python3 -m pip show requests
```

Save dependencies:

```bash
python3 -m pip freeze > requirements.txt
```

Install from a requirements file:

```bash
python3 -m pip install -r requirements.txt
```

## 5.3 Create and activate a virtual environment

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Deactivate:

```bash
deactivate
```

A virtual environment isolates project dependencies from the system Python installation.

## 5.4 UV commands

Install UV using its official installer:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Start a project:

```bash
uv init my_project
cd my_project
```

Manage dependencies:

```bash
uv add requests flask
uv remove requests
uv sync
uv tree
uv run main.py
uv lock
```

UV's installation method and availability as an Ubuntu package can vary. Check `apt-cache policy uv` before assuming the APT package is available or current.

---

# 6. Networking Commands and Packages

Useful for learning TCP/IP, DNS, ARP, routing, packet capture and network administration.

## 6.1 Network interface and IP commands

Install common networking utilities:

```bash
sudo apt install iproute2 iputils-ping net-tools
```

| Command | Package | Purpose |
|---|---|---|
| `ip` | `iproute2` | Network configuration and inspection |
| `ip addr` / `ip a` | `iproute2` | Display IP addresses |
| `ip link` | `iproute2` | Display network interfaces |
| `ip route` | `iproute2` | Display routing table |
| `ip neigh` | `iproute2` | Display neighbor cache |
| `ip -s link` | `iproute2` | Interface statistics |
| `ss` | `iproute2` | Socket inspection |
| `ifconfig` | `net-tools` | Legacy interface configuration |
| `netstat` | `net-tools` | Legacy network statistics |
| `arp` | `net-tools` | Legacy ARP utility |
| `hostname -I` | `hostname` | Display assigned IP addresses |

Examples:

```bash
ip addr
ip link show
ip route
ip neigh
ip -s link
ip addr show eth0
```

## 6.2 Connectivity and route diagnostics

| Command | Package | Purpose |
|---|---|---|
| `ping` | `iputils-ping` | Test connectivity |
| `traceroute` | `traceroute` | Trace network route |
| `tracepath` | `iputils-tracepath` | Trace path and discover path MTU |
| `mtr` | `mtr-tiny` or `mtr` | Continuous route diagnostics |
| `curl` | `curl` | Make HTTP and other supported requests |
| `wget` | `wget` | Download files |

Examples:

```bash
ping -c 4 8.8.8.8
ping -i 2 8.8.8.8
ping -W 3 8.8.8.8
traceroute example.com
tracepath example.com
mtr example.com
curl -I https://example.com
curl -v https://example.com
```

## 6.3 DNS commands

Install:

```bash
sudo apt install dnsutils bind9-host
```

| Command | Common Ubuntu package | Purpose |
|---|---|---|
| `dig` | `dnsutils` | Detailed DNS lookup |
| `nslookup` | `dnsutils` | DNS query |
| `host` | `bind9-host` | Simple DNS lookup |

Examples:

```bash
dig example.com
dig A example.com
dig AAAA example.com
dig MX example.com
dig NS example.com
dig TXT example.com
dig CNAME www.example.com
dig +short example.com
dig @8.8.8.8 example.com
nslookup example.com
host example.com
```

DNS record meanings:

- `A`: IPv4 address.
- `AAAA`: IPv6 address.
- `MX`: Mail exchanger.
- `NS`: Authoritative name server.
- `TXT`: Text record.
- `CNAME`: Canonical name alias.

## 6.4 Ports and socket inspection

Install:

```bash
sudo apt install iproute2 net-tools lsof psmisc
```

| Command | Purpose |
|---|---|
| `ss -tuln` | Display listening TCP/UDP sockets |
| `ss -tulpn` | Show listening sockets and process information where permitted |
| `ss -tan` | Display TCP sockets |
| `ss -uan` | Display UDP sockets |
| `ss -s` | Socket statistics |
| `netstat -tuln` | Legacy socket listing |
| `lsof -i` | List network-related open files |
| `lsof -i :8000` | Inspect port 8000 |
| `fuser -n tcp 8000` | Identify processes using TCP port 8000 |

Example:

```bash
sudo ss -tulpn
```

## 6.5 Packet capture and analysis

Install:

```bash
sudo apt install tcpdump tshark wireshark
```

Examples:

```bash
sudo tcpdump -i any
sudo tcpdump -i eth0
sudo tcpdump -i any port 80
sudo tcpdump -i any host 8.8.8.8
sudo tcpdump -i any -w capture.pcap
tcpdump -r capture.pcap
tshark -D
tshark -r capture.pcap
```

- `-i`: choose an interface.
- `-w`: write packets to a capture file.
- `-r`: read a saved capture.
- `-D`: list capture interfaces in TShark.

Capture only traffic you own or are authorized to inspect. Packet captures can contain sensitive information.

## 6.6 Network scanning

Install:

```bash
sudo apt install nmap
```

Examples for authorized systems:

```bash
nmap 192.168.1.1
nmap -sn 192.168.1.0/24
nmap -sV 192.168.1.1
nmap -p 80,443 192.168.1.1
```

- `-sn`: host discovery without a port scan.
- `-sV`: service/version detection.
- `-p`: select ports.

Only scan systems and networks you own or have permission to assess.

## 6.7 Additional networking packages

| Command | Package | Installation |
|---|---|---|
| `whois` | `whois` | `sudo apt install whois` |
| `nc` | `netcat-openbsd` | `sudo apt install netcat-openbsd` |
| `socat` | `socat` | `sudo apt install socat` |
| `arping` | `iputils-arping` | `sudo apt install iputils-arping` |
| `ethtool` | `ethtool` | `sudo apt install ethtool` |
| `nmcli` | `network-manager` | `sudo apt install network-manager` |
| `iw` | `iw` | `sudo apt install iw` |
| `iwconfig` | `wireless-tools` | `sudo apt install wireless-tools` |
| `iperf3` | `iperf3` | `sudo apt install iperf3` |
| `rsync` | `rsync` | `sudo apt install rsync` |

---

# 7. Operating-System Information

## 7.1 Distribution and kernel information

```bash
cat /etc/os-release
uname -a
uname -r
uname -m
hostname
hostnamectl
```

| Command | Purpose |
|---|---|
| `cat /etc/os-release` | Distribution details |
| `uname -a` | Kernel and system information |
| `uname -r` | Kernel version |
| `uname -m` | Machine architecture |
| `hostname` | System hostname |
| `hostnamectl` | Hostname and system information where supported |

## 7.2 Date, time and uptime

```bash
date
cal
uptime
timedatectl
```

`cal` may be provided by `ncal`:

```bash
sudo apt install ncal
```

## 7.3 Hardware information

```bash
lscpu
free -h
lsmem
lsusb
lspci
sudo lshw -short
```

Packages:

| Command | Package |
|---|---|
| `lscpu` | `util-linux` |
| `free` | `procps` |
| `lsmem` | `util-linux` |
| `lsusb` | `usbutils` |
| `lspci` | `pciutils` |
| `lshw` | `lshw` |
| `hwinfo` | `hwinfo` |

Install:

```bash
sudo apt install util-linux procps usbutils pciutils lshw hwinfo
```

## 7.4 Shutdown and restart

```bash
sudo shutdown now
sudo shutdown -h +10
sudo shutdown -c
sudo reboot
sudo poweroff
```

These commands behave differently in WSL. Use Windows-side `wsl --shutdown` or `wsl --terminate` to stop WSL distributions.

---

# 8. System Resource Monitoring

Install:

```bash
sudo apt install htop btop sysstat
```

| Command | Package | Purpose |
|---|---|---|
| `top` | `procps` | Live process monitoring |
| `htop` | `htop` | Interactive process monitoring |
| `btop` | `btop` | Resource dashboard |
| `free` | `procps` | RAM usage |
| `vmstat` | `procps` | Memory and process statistics |
| `iostat` | `sysstat` | CPU and disk statistics |
| `mpstat` | `sysstat` | Processor statistics |
| `sar` | `sysstat` | System activity reporting |
| `uptime` | `procps` | Uptime and load averages |
| `watch` | `procps` | Repeat a command periodically |

Examples:

```bash
top
htop
btop
free -h
watch -n 2 free -h
vmstat 2
iostat -xz 2
```

Some `sysstat` historical reporting features require service configuration.

---

# 9. File and Directory Management

## 9.1 Navigation

```bash
pwd
ls
ls -l
ls -a
ls -lh
ls -la
cd DIRECTORY
cd ..
cd ~
cd -
tree -L 3
```

Package for `tree`:

```bash
sudo apt install tree
```

## 9.2 Create files and directories

```bash
touch file.txt
mkdir project
mkdir -p project/src/utils
mkdir -v project
```

- `touch`: create an empty file or update timestamps.
- `mkdir`: create a directory.
- `-p`: create missing parent directories.
- `-v`: display created directories.

## 9.3 Copy, move and delete

```bash
cp file.txt backup.txt
cp -r folder backup/
cp -v file.txt backup/
mv old.txt new.txt
mv file.txt folder/
rm file.txt
rm -i file.txt
rm -r folder/
rmdir empty_folder/
```

**Danger:** `rm -rf PATH` recursively deletes without interactive confirmation. Check the path carefully before using it.

## 9.4 Find files and directories

Package:

```bash
sudo apt install findutils
```

Examples:

```bash
find . -name "*.py"
find . -type f
find . -type d
find . -iname "readme*"
find . -size +10M
find . -mtime -7
find . -type f -perm /111
```

Install `plocate`:

```bash
sudo apt install plocate
sudo updatedb
locate filename
```

`locate` searches a database, which may need updating before newly created files appear.

## 9.5 Inspect file content and metadata

| Command | Purpose |
|---|---|
| `cat FILE` | Display file |
| `less FILE` | Navigate through content |
| `more FILE` | Page through content |
| `head FILE` | First ten lines |
| `head -n 20 FILE` | First twenty lines |
| `tail FILE` | Last ten lines |
| `tail -n 20 FILE` | Last twenty lines |
| `tail -f FILE` | Follow appended content |
| `nl FILE` | Number lines |
| `wc FILE` | Count lines, words and bytes |
| `wc -l FILE` | Count lines |
| `wc -w FILE` | Count words |
| `wc -c FILE` | Count bytes |
| `wc -m FILE` | Count characters |
| `file FILE` | Identify file type |
| `stat FILE` | Display metadata |

Most of these are supplied by `coreutils`, `less`, or `bsdextrautils`.

## 9.6 Search inside files

Package:

```bash
sudo apt install grep
```

Examples:

```bash
grep "error" app.log
grep -i "error" app.log
grep -n "error" app.log
grep -r "password" .
grep -v "success" app.log
grep -E "error|warning" app.log
```

- `-i`: ignore case.
- `-n`: show line numbers.
- `-r`: recursive search.
- `-v`: invert match.
- `-E`: extended regular expressions.

---

# 10. Permissions and Ownership

## 10.1 Read permissions

```bash
ls -l
```

Example:

```text
-rwxr-xr-- 1 username groupname 1024 script.py
```

Permission groups:

```text
- rwx r-x r--
  |   |   |
  |   |   +--- Others
  |   +------- Group
  +----------- Owner
```

## 10.2 Numeric permissions

| Number | Permission |
|---|---|
| `0` | None |
| `1` | Execute |
| `2` | Write |
| `3` | Write + execute |
| `4` | Read |
| `5` | Read + execute |
| `6` | Read + write |
| `7` | Read + write + execute |

Examples:

```bash
chmod 644 file.txt
chmod 600 private.txt
chmod 700 private_folder
chmod 755 script.sh
```

## 10.3 Symbolic permissions

```bash
chmod u+x script.sh
chmod g-w file.txt
chmod o-r file.txt
chmod a+r file.txt
```

- `u`: owner.
- `g`: group.
- `o`: others.
- `a`: all.

## 10.4 Ownership

```bash
sudo chown username file.txt
sudo chown username:groupname file.txt
sudo chown -R username:groupname directory/
sudo chgrp groupname file.txt
```

## 10.5 Advanced permissions and ACLs

Install:

```bash
sudo apt install acl
```

Commands:

```bash
getfacl file.txt
setfacl -m u:username:rw file.txt
setfacl -x u:username file.txt
```

Special permission bits:

```bash
chmod u+s FILE
chmod g+s DIRECTORY
chmod +t DIRECTORY
```

- SUID: execute a program with the file owner's effective identity, subject to system rules.
- SGID: group inheritance behavior; on directories, new entries normally inherit the directory group.
- Sticky bit: restricts who can remove or rename entries in a shared directory.

Use special permission bits cautiously.

---

# 11. Process Management

A process is an instance of a running program.

## 11.1 Inspect processes

```bash
ps
ps aux
ps -ef
ps aux | grep python
pgrep python
pidof python3
```

Common packages:

- `ps`, `pgrep`, `pidof`: `procps`.
- `pstree`: `psmisc`.

## 11.2 Stop processes

```bash
kill PID
kill -TERM PID
kill -9 PID
killall python3
pkill -f script.py
```

Prefer graceful termination (`SIGTERM`) before forceful termination (`SIGKILL`).

## 11.3 Process priority

```bash
nice -n 10 python3 script.py
renice 5 -p PID
```

## 11.4 Background and foreground jobs

```bash
jobs
bg
fg
nohup python3 script.py &
```

- `jobs`: list jobs in the current shell.
- `bg`: resume a suspended job in the background.
- `fg`: bring a job to the foreground.
- `nohup`: ignore hangup signals; useful for some detached terminal tasks.

---

# 12. Users and Groups

## 12.1 User information

```bash
whoami
id
who
w
users
groups
getent passwd
getent group
last
lastb
logname
```

`last` and `lastb` depend on login-accounting records and the relevant utilities being installed/configured.

## 12.2 Create and manage users

```bash
sudo adduser username
sudo useradd username
sudo passwd username
sudo usermod -aG sudo username
sudo deluser username
sudo deluser --remove-home username
```

- `adduser`: higher-level interactive account creation on Ubuntu.
- `useradd`: lower-level account creation utility.
- `usermod -aG`: append a user to supplementary groups.
- `deluser`: remove a user.

## 12.3 Group management

```bash
sudo groupadd developers
sudo groupdel developers
sudo usermod -aG developers username
getent group developers
```

---

# 13. Storage and Filesystems

Install common utilities:

```bash
sudo apt install util-linux parted e2fsprogs
```

| Command | Purpose |
|---|---|
| `df -h` | Filesystem disk usage |
| `df -T` | Filesystem types |
| `du -h` | File and directory sizes |
| `du -sh DIRECTORY` | Total directory size |
| `du -ah DIRECTORY` | All file sizes |
| `lsblk` | Block device information |
| `lsblk -f` | Filesystem details |
| `blkid` | Filesystem UUIDs and types |
| `mount` | Display or mount filesystems |
| `findmnt` | Display mount information |
| `umount` | Unmount a filesystem |
| `fdisk -l` | Display partition information |
| `parted -l` | Display partition information |

Examples:

```bash
df -h
du -sh ~/Python-sessions
lsblk -f
findmnt
```

**WSL note:** Windows drives are commonly accessible through `/mnt/c/`, `/mnt/d/`, and similar mount points. Physical partitioning commands may not apply to a normal WSL filesystem environment.

---

# 14. Archives and Compression

Install:

```bash
sudo apt install tar gzip bzip2 xz-utils zip unzip 7zip zstd lz4
```

## 14.1 ZIP

```bash
zip archive.zip file.txt
zip -r archive.zip directory/
unzip archive.zip
unzip archive.zip -d destination/
unzip -l archive.zip
```

## 14.2 TAR

```bash
tar -cvf archive.tar directory/
tar -xvf archive.tar
tar -czvf archive.tar.gz directory/
tar -xzvf archive.tar.gz
tar -tvf archive.tar
```

- `-c`: create.
- `-x`: extract.
- `-v`: verbose.
- `-f`: archive filename.
- `-z`: gzip compression.

## 14.3 GZIP and other compression tools

```bash
gzip file.txt
gunzip file.txt.gz
gzip -k file.txt
bzip2 file.txt
xz file.txt
zstd file.txt
lz4 file.txt
```

Check each tool's manual for decompression and output options.

---

# 15. Services and systemd

A service is a background program that provides functionality to the system or other applications. Examples include SSH, Nginx, Redis and PostgreSQL.

## 15.1 Check and manage services

```bash
systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl reload nginx
sudo systemctl enable nginx
sudo systemctl disable nginx
sudo systemctl is-active nginx
sudo systemctl is-enabled nginx
systemctl list-units --type=service
systemctl list-unit-files --type=service
```

## 15.2 Journal logs

```bash
journalctl
journalctl -b
journalctl -b -1
journalctl -u ssh
journalctl -u nginx -f
journalctl -p err
journalctl --since "1 hour ago"
journalctl --disk-usage
```

- `-b`: current boot.
- `-u`: select a unit/service.
- `-f`: follow new log entries.
- `-p err`: error priority.
- `--since`: filter by time.

**WSL note:** systemd may need to be enabled in WSL. Check support with:

```bash
ps -p 1 -o comm=
```

If the output is `systemd`, it is running as PID 1.

---

# 16. Security and Firewall

## 16.1 UFW firewall

Install:

```bash
sudo apt install ufw
```

Commands:

```bash
sudo ufw status
sudo ufw status verbose
sudo ufw enable
sudo ufw disable
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 8000/tcp
sudo ufw deny 8080/tcp
sudo ufw delete allow 8000/tcp
sudo ufw status numbered
```

Before enabling a firewall on a remote server, ensure that the necessary SSH access is allowed. WSL firewall behavior also depends on the Windows/WSL networking configuration.

## 16.2 SSH

Install client and server:

```bash
sudo apt install openssh-client openssh-server
```

Connect:

```bash
ssh username@192.168.1.10
ssh -p 2222 username@192.168.1.10
```

Generate a key:

```bash
ssh-keygen -t ed25519
```

Copy a public key to an authorized server:

```bash
ssh-copy-id username@192.168.1.10
```

Transfer files:

```bash
scp file.txt username@192.168.1.10:/home/username/
scp -r directory/ username@192.168.1.10:/home/username/
```

## 16.3 Hashes and integrity

Package: `coreutils` (normally installed).

```bash
sha256sum file.txt
sha512sum file.txt
md5sum file.txt
sha256sum -c checksums.txt
```

For modern security-sensitive integrity checking, prefer SHA-256 or stronger. MD5 and SHA-1 are not appropriate for modern collision-resistant security designs.

## 16.4 Password and authentication utilities

```bash
sudo apt install passwd whois
passwd
sudo passwd username
sudo chage -l username
sudo chage -M 90 username
```

If `faillock` is available and configured:

```bash
sudo faillock
```

It displays authentication failure records managed by the relevant PAM configuration.

---

# 17. Text Processing and Data Utilities

Useful for log analysis, shell scripting, data extraction and automation.

## 17.1 Install utilities

```bash
sudo apt install gawk sed grep coreutils
```

## 17.2 grep

```bash
grep "error" app.log
grep -i "error" app.log
grep -n "error" app.log
grep -v "success" app.log
grep -r "password" .
grep -E "error|warning" app.log
grep -oE "[0-9]{1,3}(\.[0-9]{1,3}){3}" app.log
```

## 17.3 sed

```bash
sed -n '1,10p' file.txt
sed 's/old/new/' file.txt
sed 's/old/new/g' file.txt
sed -i 's/old/new/g' file.txt
sed '5d' file.txt
sed -n '/error/p' app.log
```

`-i` edits the file in place. Make a backup first when the original data matters.

## 17.4 awk

```bash
awk '{print $1}' file.txt
awk '{print $2}' file.txt
awk -F: '{print $1}' /etc/passwd
awk 'BEGIN { OFS="," } { print $1,$2 }' file.txt
awk '$3 > 100 { print $1,$3 }' data.txt
awk 'END { print NR }' file.txt
```

- `-F`: input field separator.
- `OFS`: output field separator.
- `NR`: current record number.
- `BEGIN`: run before input processing.
- `END`: run after input processing.

## 17.5 sort and uniq

```bash
sort file.txt
sort -r file.txt
sort -n numbers.txt
sort -u file.txt
uniq file.txt
sort file.txt | uniq -c
```

`uniq` normally detects adjacent repeated lines, so sort first when counting duplicates throughout a file.

## 17.6 cut, tr and paste

```bash
cut -d: -f1 /etc/passwd
cut -c1-5 file.txt
echo "hello" | tr 'a-z' 'A-Z'
echo "123-456" | tr -d '-'
paste file1.txt file2.txt
```

---

# 18. Environment and Shell Commands

## 18.1 Environment variables

```bash
env
printenv
echo "$HOME"
echo "$PATH"
echo "$USER"
echo "$SHELL"
echo "$PWD"
```

Create a shell variable:

```bash
NAME="Nzegge"
```

Export it to child processes:

```bash
export NAME="Nzegge"
```

Remove it:

```bash
unset NAME
```

Append a directory to PATH for the current shell:

```bash
export PATH="$PATH:$HOME/.local/bin"
```

Display PATH entries on separate lines:

```bash
echo "$PATH" | tr ':' '\n'
```

## 18.2 Shell identification

```bash
echo "$SHELL"
ps -p $$ -o comm=
bash --version
zsh --version
```

## 18.3 Shell history

```bash
history
history 20
history -c
history -w
history | grep apt
cat ~/.bash_history
```

`history -c` clears the current shell's in-memory history list. `history -w` writes the current list to the history file.

## 18.4 Discover commands

```bash
which python3
type ls
command -v python3
whereis python3
whatis ls
apropos network
man ls
info coreutils
```

`type` is especially useful because commands such as `cd` are shell builtins rather than external executable files.

---

# 19. Scheduling and Automation

## 19.1 Cron

Install:

```bash
sudo apt install cron
```

Manage your crontab:

```bash
crontab -e
crontab -l
crontab -r
```

Cron format:

```text
* * * * * command
│ │ │ │ │
│ │ │ │ └── Day of week
│ │ │ └──── Month
│ │ └────── Day of month
│ └──────── Hour
└────────── Minute
```

Examples:

Every minute:

```cron
* * * * * /usr/bin/python3 /home/username/script.py
```

Every day at 8 AM:

```cron
0 8 * * * /usr/bin/python3 /home/username/script.py
```

Every Sunday at 10 AM:

```cron
0 10 * * 0 /home/username/backup.sh
```

Use absolute paths in cron jobs and redirect output to logs when appropriate.

## 19.2 One-time scheduling with at

Install:

```bash
sudo apt install at
```

Commands:

```bash
at 10:30
atq
atrm JOB_ID
```

`at` schedules a one-time task; `cron` schedules recurring tasks.

---

# 20. Development Tools

## 20.1 Git

Install:

```bash
sudo apt install git
```

Commands:

```bash
git --version
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
git clone REPOSITORY_URL
git config --list
```

## 20.2 Compilers and build tools

```bash
sudo apt install build-essential cmake ninja-build
```

Check:

```bash
gcc --version
g++ --version
make --version
cmake --version
ninja --version
```

## 20.3 Node.js and npm

```bash
sudo apt install nodejs npm
node --version
npm --version
npm install
npm install express
npm install -g PACKAGE
npm uninstall express
```

For current Node.js development, compare Ubuntu's repository version with the official Node.js distribution or a trusted version manager.

## 20.4 Other development utilities

```bash
sudo apt install vim nano tmux screen jq yq tree file
```

| Tool | Package | Purpose |
|---|---|---|
| `vim` | `vim` | Terminal editor |
| `nano` | `nano` | Simple terminal editor |
| `tmux` | `tmux` | Terminal multiplexer |
| `screen` | `screen` | Terminal session manager |
| `jq` | `jq` | JSON processing |
| `yq` | `yq` | YAML processing; check Ubuntu package implementation |
| `tree` | `tree` | Directory visualization |
| `file` | `file` | Identify file types |

Example:

```bash
echo '{"name":"Nzegge","age":25}' | jq .
```

---

# 21. External Repositories and Package Downloads

## 21.1 PPA repositories

Install repository-management utilities:

```bash
sudo apt install software-properties-common
```

Add a PPA:

```bash
sudo add-apt-repository ppa:OWNER/ARCHIVE
sudo apt update
sudo apt install PACKAGE
```

Remove a PPA:

```bash
sudo add-apt-repository --remove ppa:OWNER/ARCHIVE
```

Only add repositories from trusted maintainers.

## 21.2 Install a downloaded `.deb`

Download:

```bash
wget https://example.com/application.deb
```

Install with dependency handling:

```bash
sudo apt install ./application.deb
```

Alternatively:

```bash
sudo dpkg -i application.deb
```

## 21.3 Repository signing keys

Modern APT repository configuration commonly uses:

- `/etc/apt/sources.list.d/` for repository definitions.
- `/etc/apt/keyrings/` for repository-specific keys.
- `signed-by=` to associate a repository with its signing key.

Create the keyring directory:

```bash
sudo mkdir -p /etc/apt/keyrings
```

A generic source entry looks like:

```text
deb [signed-by=/etc/apt/keyrings/example.gpg] https://repository.example.com/ubuntu stable main
```

Use the repository owner's official instructions for the actual key URL, fingerprint, distribution codename and repository entry. Never blindly trust third-party signing keys.

---

# 22. Downloading and Transferring Files

Install:

```bash
sudo apt install curl wget rsync
```

## 22.1 wget

```bash
wget https://example.com/file.zip
wget -O backup.zip https://example.com/file.zip
wget -c https://example.com/file.zip
wget -r https://example.com/
```

- `-O`: specify output filename.
- `-c`: continue an interrupted download.
- `-r`: recursive download; use carefully.

## 22.2 curl

```bash
curl -O https://example.com/file.zip
curl -o downloaded.zip https://example.com/file.zip
curl -L https://example.com/file.zip -o file.zip
curl -I https://example.com
curl -v https://example.com
```

GET request:

```bash
curl https://example.com
```

POST request:

```bash
curl -X POST https://example.com/api/
```

JSON request:

```bash
curl -X POST https://example.com/api/ \
  -H "Content-Type: application/json" \
  -d '{"username":"nzegge"}'
```

## 22.3 rsync

```bash
rsync -av source/ destination/
rsync -av --progress source/ destination/
rsync -av project/ username@192.168.1.10:/home/username/project/
```

The trailing slash in `source/` means to copy the contents of the directory.

---

# 23. Package Name Quick Reference

The following table maps common commands to their Ubuntu package names. Some tools are already installed as part of the base system.

## 23.1 Networking

| Command | Package |
|---|---|
| `ip` | `iproute2` |
| `ss` | `iproute2` |
| `ifconfig` | `net-tools` |
| `netstat` | `net-tools` |
| `arp` | `net-tools` |
| `ping` | `iputils-ping` |
| `traceroute` | `traceroute` |
| `tracepath` | `iputils-tracepath` |
| `dig` | `dnsutils` |
| `nslookup` | `dnsutils` |
| `host` | `bind9-host` |
| `whois` | `whois` |
| `tcpdump` | `tcpdump` |
| `tshark` | `tshark` |
| `wireshark` | `wireshark` |
| `nmap` | `nmap` |
| `nc` | `netcat-openbsd` |
| `socat` | `socat` |
| `mtr` | `mtr-tiny` or `mtr` |
| `arping` | `iputils-arping` |
| `ethtool` | `ethtool` |
| `nmcli` | `network-manager` |
| `iw` | `iw` |
| `iwconfig` | `wireless-tools` |
| `curl` | `curl` |
| `wget` | `wget` |
| `iperf3` | `iperf3` |

## 23.2 System administration

| Command | Package |
|---|---|
| `htop` | `htop` |
| `btop` | `btop` |
| `top` | `procps` |
| `ps` | `procps` |
| `kill` | `procps` |
| `pgrep` | `procps` |
| `free` | `procps` |
| `vmstat` | `procps` |
| `watch` | `procps` |
| `iostat` | `sysstat` |
| `sar` | `sysstat` |
| `lscpu` | `util-linux` |
| `lsblk` | `util-linux` |
| `mount` | `util-linux` |
| `findmnt` | `util-linux` |
| `lsof` | `lsof` |
| `fuser` | `psmisc` |
| `pstree` | `psmisc` |
| `systemctl` | `systemd` |
| `journalctl` | `systemd` |
| `cron` | `cron` |
| `at` | `at` |

## 23.3 File and text utilities

| Command | Package |
|---|---|
| `tree` | `tree` |
| `locate` | `plocate` |
| `updatedb` | `plocate` |
| `find` | `findutils` |
| `xargs` | `findutils` |
| `grep` | `grep` |
| `sed` | `sed` |
| `awk` | `gawk` |
| `sort` | `coreutils` |
| `uniq` | `coreutils` |
| `wc` | `coreutils` |
| `cut` | `coreutils` |
| `tr` | `coreutils` |
| `paste` | `coreutils` |
| `stat` | `coreutils` |
| `file` | `file` |
| `hexdump` | `bsdextrautils` |
| `xxd` | `xxd` |
| `strings` | `binutils` |
| `readelf` | `binutils` |
| `objdump` | `binutils` |

## 23.4 Archives and compression

| Command | Package |
|---|---|
| `zip` | `zip` |
| `unzip` | `unzip` |
| `tar` | `tar` |
| `gzip` / `gunzip` | `gzip` |
| `bzip2` | `bzip2` |
| `xz` | `xz-utils` |
| `7z` | `7zip` (availability varies) |
| `zstd` | `zstd` |
| `lz4` | `lz4` |
| `rar` | `rar` (repository availability varies) |
| `unrar` | `unrar` (repository availability varies) |

## 23.5 Security and forensics

| Command | Package |
|---|---|
| `openssl` | `openssl` |
| `gpg` | `gpg` |
| `ssh` | `openssh-client` |
| `sshd` | `openssh-server` |
| `ssh-keygen` | `openssh-client` |
| `ufw` | `ufw` |
| `fail2ban-client` | `fail2ban` |
| `strace` | `strace` |
| `ltrace` | `ltrace` |
| `inotifywait` | `inotify-tools` |
| `auditctl` | `auditd` |
| `ausearch` | `auditd` |
| `aureport` | `auditd` |
| `chkrootkit` | `chkrootkit` |
| `rkhunter` | `rkhunter` |
| `clamscan` | `clamav` |
| `clamd` | `clamav-daemon` |
| `yara` | `yara` |
| `binwalk` | `binwalk` |
| `foremost` | `foremost` |
| `bulk_extractor` | `bulk-extractor` |

## 23.6 Python

| Command / Tool | Package |
|---|---|
| `python3` | `python3` |
| `pip3` | `python3-pip` |
| `venv` | `python3-venv` |
| Python headers | `python3-dev` |
| `python` alias | `python-is-python3` |
| `virtualenv` | `virtualenv` |
| `ipython3` | `ipython3` |
| `pytest` | `python3-pytest` |
| `black` | `black` |
| `ruff` | `ruff` |
| `mypy` | `mypy` |
| `httpie` | `httpie` |
| Flask | `python3-flask` |
| Django | `python3-django` |

For project development, prefer a virtual environment and install dependencies within it. Ubuntu package names and versions can differ from PyPI package names and versions.

## 23.7 Development and compilers

| Command / Tool | Package |
|---|---|
| `gcc` | `gcc` |
| `g++` | `g++` |
| `make` | `make` |
| `cmake` | `cmake` |
| `ninja` | `ninja-build` |
| `gdb` | `gdb` |
| `git` | `git` |
| `gh` | `gh` |
| `node` | `nodejs` |
| `npm` | `npm` |
| `ruby` | `ruby` |
| `gem` | `ruby-rubygems` |
| `cargo` | `cargo` |
| `rustc` | `rustc` |
| `java` / `javac` | `default-jdk` |
| `php` | `php-cli` |

## 23.8 Databases

| Database / Tool | Package |
|---|---|
| PostgreSQL server | `postgresql` |
| PostgreSQL client | `postgresql-client` |
| SQLite | `sqlite3` |
| SQLite development headers | `libsqlite3-dev` |
| MariaDB server | `mariadb-server` |
| MariaDB client | `mariadb-client` |
| Redis server | `redis-server` |
| Redis CLI tools | `redis-tools` |

---

# 24. Troubleshooting

## 24.1 Fix broken package dependencies

```bash
sudo apt update
sudo apt --fix-broken install
sudo dpkg --configure -a
sudo apt-get check
sudo apt clean
sudo apt autoremove
```

## 24.2 Diagnose a missing command

Suppose you execute:

```bash
last
```

And receive:

```text
Command 'last' not found
```

Procedure:

**Step 1: Search for the likely package.**

```bash
apt search wtmpdb
```

**Step 2: Inspect the package.**

```bash
apt show wtmpdb
```

**Step 3: Install it.**

```bash
sudo apt install wtmpdb
```

**Step 4: Check command availability.**

```bash
command -v last
```

**Step 5: Read the manual.**

```bash
man last
```

For this particular utility, package availability and login-accounting behavior depend on the Ubuntu release and configuration. Installing a package does not guarantee that historical login records exist.

## 24.3 Discover which package provides a command

Install:

```bash
sudo apt install apt-file
```

Update its database:

```bash
sudo apt-file update
```

Search for a command:

```bash
apt-file search bin/ifconfig
```

This is especially useful when the command name and package name are different.

---

# 25. WSL-Specific Commands

The following commands are run from Windows PowerShell or CMD, not from the Ubuntu shell.

| Command | Purpose |
|---|---|
| `wsl --list --verbose` | List distributions |
| `wsl -l -v` | Short form |
| `wsl --status` | Display WSL status |
| `wsl --version` | Display WSL version |
| `wsl --update` | Update WSL |
| `wsl --shutdown` | Stop all running distributions |
| `wsl --terminate Ubuntu` | Stop a distribution |
| `wsl --distribution Ubuntu` | Start Ubuntu |
| `wsl --set-default Ubuntu` | Set default distribution |
| `wsl --list --online` | List available distributions |
| `wsl --export Ubuntu backup.tar` | Export a distribution |
| `wsl --import UbuntuTest C:\\WSL\\UbuntuTest backup.tar` | Import a distribution |
| `wsl --unregister UbuntuTest` | Remove a distribution and its data |

From Ubuntu, access Windows drives:

```bash
cd /mnt/c
cd /mnt/e
```

Launch Windows programs:

```bash
explorer.exe .
notepad.exe file.txt
```

Inspect WSL networking:

```bash
ip addr
ip route
cat /etc/resolv.conf
```

---

# 26. Shell Operators and Redirection

These are shell operators, not standalone commands.

| Operator | Meaning | Example |
|---|---|---|
| `&` | Run in background | `python3 app.py &` |
| `&&` | Run next if previous succeeds | `sudo apt update && sudo apt upgrade` |
| `||` | Run next if previous fails | `command || echo "Failed"` |
| `;` | Run commands sequentially | `pwd; ls; whoami` |
| `\|` | Pipe output to another command | `ps aux \| grep python` |
| `>` | Redirect stdout and overwrite | `ls > files.txt` |
| `>>` | Append stdout | `date >> log.txt` |
| `<` | Read stdin from file | `sort < names.txt` |
| `2>` | Redirect stderr | `command 2> error.log` |
| `2>&1` | Redirect stderr to stdout | `command > output.log 2>&1` |
| `&>` | Redirect stdout and stderr in Bash | `command &> output.log` |
| `$( )` | Command substitution | `echo "$(date)"` |
| `*` | Wildcard | `ls *.py` |
| `?` | Match one character | `ls file?.txt` |
| `\\` | Escape special characters or continue a command | `echo \\$HOME` |

## 26.1 Standard input and output

Linux uses three standard streams:

| Stream | File descriptor | Purpose |
|---|---:|---|
| stdin | 0 | Input |
| stdout | 1 | Normal output |
| stderr | 2 | Error output |

Example:

```bash
python3 app.py > output.log 2> errors.log
```

This separates normal output and error output.

---

# 27. Learning Roadmap

Learn these topics progressively instead of memorizing hundreds of commands at once.

1. **Basic Linux:** `pwd`, `ls`, `cd`, `mkdir`, `touch`, `cp`, `mv`, `rm`, `cat`, `less`.
2. **Package management:** `apt`, `apt-get`, `dpkg`, `snap`, `pip`, `uv`.
3. **File management:** `find`, `locate`, `grep`, `tree`, `stat`, `file`, `du`, `df`.
4. **Permissions and users:** `chmod`, `chown`, `chgrp`, `getfacl`, `setfacl`, `useradd`, `usermod`.
5. **Networking:** `ip`, `ping`, `ss`, `dig`, `nslookup`, `traceroute`, `tcpdump`.
6. **Process management:** `ps`, `top`, `htop`, `kill`, `nice`, `renice`, `lsof`.
7. **Linux administration:** `systemctl`, `journalctl`, `cron`, `ufw`, `ssh`.
8. **Cybersecurity:** `nmap`, `strace`, `tshark`, `openssl`, `gpg`, `inotifywait`.
9. **Automation:** Bash scripting, cron, pipes, redirection, `awk`, `sed`, `xargs`.

---

# 28. How to Discover Commands Independently

When you encounter a new command, investigate it instead of memorizing it blindly.

## 28.1 Manual pages

```bash
man ls
```

## 28.2 Help options

```bash
ls --help
```

## 28.3 Short descriptions

```bash
whatis ls
```

## 28.4 Search manual descriptions

```bash
apropos network
```

## 28.5 Discover command location and type

```bash
which python3
type cd
command -v python3
whereis python3
```

## 28.6 Inspect package contents

```bash
dpkg -L curl
```

## 28.7 Discover the package that provides a file

```bash
apt-file search bin/COMMAND_NAME
```

## 28.8 Understand command structure

Example:

```bash
ping -c 4 google.com
```

- `ping`: command.
- `-c`: option.
- `4`: option value.
- `google.com`: positional argument.

For every new command, ask:

1. What does it do?
2. Which package provides it?
3. What arguments does it accept?
4. What options does it support?
5. What does it display or return?
6. Does it require root privileges?
7. Can it modify or delete data?
8. How can it be combined with other commands?

---

# 29. Practical Installation Collections

These are optional collections of commonly useful packages. You do not need to install all of them at once.

## 29.1 Basic utilities

```bash
sudo apt update

sudo apt install -y \
  curl wget git vim nano tree \
  zip unzip tar gzip \
  file jq plocate \
  htop btop tmux screen
```

## 29.2 Networking and cybersecurity

```bash
sudo apt install -y \
  iproute2 iputils-ping net-tools \
  dnsutils traceroute tcpdump \
  nmap netcat-openbsd whois \
  mtr-tiny lsof strace ltrace \
  wireshark tshark
```

## 29.3 Python development

```bash
sudo apt install -y \
  python3 python3-pip python3-venv \
  python3-dev build-essential \
  libssl-dev libffi-dev
```

## 29.4 System administration

```bash
sudo apt install -y \
  ufw openssh-client openssh-server \
  acl sysstat ncdu \
  rsync cron
```

---

# 30. Final Best Practices

- Run `sudo apt update` before installing packages when repository metadata may be outdated.
- Prefer APT over manually downloading `.deb` packages when a package is available in a trusted repository.
- Use official and trusted repositories.
- Do not add repositories or signing keys without checking their source.
- Use Python virtual environments for Python project dependencies.
- Avoid running unknown scripts with `sudo`.
- Understand commands before using destructive options such as `rm -rf`.
- Prefer `ip` and `ss` over legacy `ifconfig` and `netstat` for modern network administration.
- Prefer graceful process termination before `kill -9`.
- Use `man COMMAND` and `COMMAND --help` to learn unfamiliar tools.
- Verify package names with `apt-cache policy` or `apt-file`.
- Remember that WSL does not behave exactly like a conventional Linux installation. Services, firewall behavior, systemd, login accounting and hardware access may differ.

**The most important skill is not memorizing commands. It is understanding how to discover a command, identify its package, inspect its options, use it safely and combine it with other Linux tools.**
