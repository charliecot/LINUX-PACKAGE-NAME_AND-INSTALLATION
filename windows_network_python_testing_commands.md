# Windows Network Testing, Basic Windows Commands, and Python Network Tests

**Author:** Nzegge Charles Ngong

A practical reference for testing network connectivity, inspecting Windows network configuration, checking TCP ports, and creating simple Python network test services.

> Use active network tests only on your own devices, your own lab, or systems for which you have permission.

---

# Part 1 — Windows network testing commands

## 1. `ipconfig` — inspect IP configuration

```powershell
ipconfig
ipconfig /all
ipconfig /release
ipconfig /renew
ipconfig /flushdns
```

- `/all` displays detailed adapter information, including IPv4 address, subnet mask, default gateway, DNS servers, DHCP status, and physical (MAC) address.
- `/release` releases the DHCP-assigned IPv4 configuration.
- `/renew` requests DHCP configuration again.
- `/flushdns` clears the DNS resolver cache.

Be aware that `/release` temporarily disconnects your network until you renew the configuration.

## 2. `ping` — test basic reachability

```powershell
ping 192.168.1.1
ping 8.8.8.8
ping -n 4 8.8.8.8
ping -t 192.168.1.1
ping -l 1000 8.8.8.8
ping -w 3000 8.8.8.8
```

| Option | Meaning |
|---|---|
| `-n 4` | Send four echo requests |
| `-t` | Continue until stopped with `Ctrl+C` |
| `-l 1000` | Send a 1000-byte data buffer |
| `-w 3000` | Wait up to 3000 milliseconds for each reply |

A timeout does not prove that a device is offline. A firewall may block ICMP echo requests.

## 3. `tracert` — trace the route

```powershell
tracert google.com
tracert -h 10 google.com
tracert -w 2000 google.com
```

Each numbered hop represents a router or other Layer 3 device along the route. Asterisks can mean a router did not return the expected response; they do not automatically mean the route is broken.

## 4. `pathping` — inspect route and packet loss

```powershell
pathping google.com
```

`pathping` combines route tracing with packet-loss statistics. It can take longer than `ping`.

## 5. `nslookup` — test DNS

```powershell
nslookup google.com
nslookup google.com 8.8.8.8
nslookup -type=A google.com
nslookup -type=AAAA google.com
nslookup -type=MX gmail.com
nslookup -type=NS google.com
```

- `A`: IPv4 address
- `AAAA`: IPv6 address
- `MX`: mail server
- `NS`: authoritative name server

## 6. `arp` — inspect local IP-to-MAC mappings

```powershell
arp -a
arp -d 192.168.1.1
```

ARP maps local IPv4 addresses to link-layer (usually Ethernet or Wi-Fi) MAC addresses. Deleting an entry may require an elevated terminal.

## 7. `route` — inspect the routing table

```powershell
route print
route print -4
route print -6
```

The routing table helps Windows decide which next hop and network interface to use for a destination.

## 8. `netstat` — inspect connections and ports

```powershell
netstat
netstat -ano
netstat -ano -p TCP | findstr LISTENING
netstat -r
```

| Option | Meaning |
|---|---|
| `-a` | Show all connections and listening ports |
| `-n` | Show numeric addresses and port numbers |
| `-o` | Show the owning process ID (PID) |
| `-b` | Show the executable involved (may require administrator privileges) |
| `-p TCP` | Limit output to TCP |
| `-r` | Display routing information |

Find the process associated with a PID:

```powershell
tasklist /FI "PID eq 1234"
```

Replace `1234` with the PID shown by `netstat`.

## 9. `Test-NetConnection` — test a TCP port

This is a PowerShell cmdlet, not a CMD command.

```powershell
Test-NetConnection google.com
Test-NetConnection 192.168.1.110 -Port 8080
Test-NetConnection google.com -Port 443
```

Look for:

```text
TcpTestSucceeded : True
```

- `True`: the TCP connection succeeded.
- `False`: the TCP connection did not succeed.

A failed TCP test can result from a closed port, firewall filtering, routing problems, or other connectivity issues.

## 10. PowerShell network inspection cmdlets

```powershell
Get-NetAdapter
Get-NetIPConfiguration
Get-NetIPConfiguration -Detailed
Get-NetTCPConnection
Get-NetTCPConnection -State Listen
```

Show useful columns:

```powershell
Get-NetTCPConnection -State Listen |
    Select-Object LocalAddress, LocalPort, State, OwningProcess
```

Inspect one local port:

```powershell
Get-NetTCPConnection -LocalPort 8080
```

Inspect firewall profiles:

```powershell
Get-NetFirewallProfile
```

---

# Part 2 — Python network testing commands

Python can create a small server, test TCP connections, inspect addresses, and perform basic network diagnostics.

## 1. `python -m http.server` — start a simple HTTP server

Start an HTTP server on port `8080`, bound to your computer's LAN IPv4 address:

```powershell
python -m http.server 8080 --bind 192.168.1.110
```

What each part means:

| Part | Meaning |
|---|---|
| `python` | Run the Python interpreter |
| `-m` | Run a Python module as a script |
| `http.server` | Python's built-in simple HTTP server module |
| `8080` | TCP port on which the server listens |
| `--bind` | Choose the local address on which the server listens |
| `192.168.1.110` | Your computer's selected local IPv4 address |

The server shares files from the current working directory. Do not run it in a directory containing private files if other devices can reach it.

### Test it from another terminal on the same computer

Open a second PowerShell or Git Bash terminal:

```powershell
Test-NetConnection 192.168.1.110 -Port 8080
```

You can also open this address in a browser:

```text
http://192.168.1.110:8080/
```

### Bind to all IPv4 interfaces

```powershell
python -m http.server 8080 --bind 0.0.0.0
```

This listens on all available IPv4 interfaces. Other devices may be able to reach it if the network and firewall allow it.

### Bind to loopback only

```powershell
python -m http.server 8080 --bind 127.0.0.1
```

This makes the server accessible only through the local loopback interface, normally using:

```text
http://127.0.0.1:8080/
```

Use the specific LAN address when you want to test access through that interface.

**Security note:** `http.server` is intended for development and testing, not production hosting. It does not provide production-grade security or access control.

Stop the server with `Ctrl+C`.

## 2. Test a TCP port with Python's `socket` module

Create `tcp_test.py`:

```python
import socket

target_ip = "192.168.1.110"
target_port = 8080

# Create an IPv4 TCP socket.
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as client:
    # Do not wait indefinitely.
    client.settimeout(3)

    try:
        # Attempt a TCP connection.
        client.connect((target_ip, target_port))
        print(f"TCP connection succeeded: {target_ip}:{target_port}")

    except socket.timeout:
        print("Connection timed out.")

    except ConnectionRefusedError:
        print("Connection refused: no service accepted the connection.")

    except OSError as error:
        print(f"Network or socket error: {error}")
```

Run it:

```powershell
python tcp_test.py
```

| Expression | Meaning |
|---|---|
| `socket.AF_INET` | IPv4 address family |
| `socket.SOCK_STREAM` | TCP socket type |
| `settimeout(3)` | Set a three-second socket timeout |
| `connect((ip, port))` | Attempt a TCP connection |
| `ConnectionRefusedError` | The connection was actively refused |
| `socket.timeout` | The attempt timed out |

A successful connection proves that a TCP connection was established. It does not prove which application or service is running on that port.

## 3. Test ping using Python's `subprocess` module

Create `ping_test.py`:

```python
import platform
import subprocess

target = "192.168.1.1"

# Windows uses -n; common Unix-like systems use -c.
count_option = "-n" if platform.system() == "Windows" else "-c"

result = subprocess.run(
    ["ping", count_option, "4", target],
    capture_output=True,
    text=True,
    timeout=15
)

print(result.stdout)

if result.returncode == 0:
    print("Ping command completed successfully.")
else:
    print("Ping command reported a failure.")
```

Run:

```powershell
python ping_test.py
```

This calls the operating system's `ping` command from Python. `returncode == 0` means the command reported success; inspect the output for packet-loss details.

## 4. Resolve a domain name using Python

Create `dns_test.py`:

```python
import socket

domain = "google.com"

try:
    ip_address = socket.gethostbyname(domain)
    print(f"{domain} resolves to {ip_address}")

except socket.gaierror as error:
    print(f"DNS lookup failed: {error}")
```

Run:

```powershell
python dns_test.py
```

`socket.gethostbyname()` performs a basic IPv4 hostname lookup using the system's configured name-resolution facilities.

## 5. Display your hostname and selected local IP

```python
import socket

hostname = socket.gethostname()
print("Hostname:", hostname)

try:
    # Ask the OS which local address it would use for this route.
    # This UDP connect does not send an application message.
    with socket.socket(socket.AF_INET, socket.SOCK_DGRAM) as sock:
        sock.connect(("8.8.8.8", 80))
        local_ip = sock.getsockname()[0]

    print("Selected local IPv4 address:", local_ip)

except OSError as error:
    print("Could not determine local IPv4 address:", error)
```

The UDP `connect()` here does not establish a TCP connection or send a UDP application message. It asks the operating system to select a route and local address.

---

# Part 3 — Basic Windows commands

## 1. Navigation

| Command | Purpose |
|---|---|
| `cd` | Change directory |
| `cd ..` | Move to parent directory |
| `cd \` | Move to the drive root |
| `cd /d E:\Projects` | Change drive and directory in CMD |
| `pwd` | Display current directory in PowerShell/Git Bash |
| `dir` | List directory contents |
| `ls` | List directory contents in PowerShell/Git Bash |
| `tree` | Display folder structure |
| `tree /f` | Display folders and files |

Example:

```powershell
cd E:\ETHICAL-HACKING-PART-2
tree /f
```

## 2. Create files and directories

```powershell
mkdir NETWORK_TESTING
New-Item test.py -ItemType File
```

CMD alternative:

```cmd
type nul > test.py
```

Git Bash alternative:

```bash
touch test.py
```

## 3. Copy, move, and rename

```powershell
Copy-Item notes.txt backup.txt
Copy-Item source backup -Recurse
Move-Item notes.txt E:\Backup
Rename-Item old_name.txt new_name.txt
```

## 4. Delete files and directories

```powershell
Remove-Item test.py
Remove-Item NETWORK_TESTING
Remove-Item NETWORK_TESTING -Recurse
```

Be careful with recursive deletion: it can remove many files and folders.

## 5. Read text files

```powershell
Get-Content notes.txt
Get-Content notes.txt -TotalCount 10
Get-Content notes.txt -Tail 10
Get-Content application.log -Wait
```

`-Wait` keeps monitoring a file as new lines are added.

## 6. Find executables and commands

```powershell
where.exe python
where.exe git
where.exe pip
```

PowerShell alternatives:

```powershell
Get-Command python
Get-Command ping
Get-Command ipconfig
```

## 7. Process management

```powershell
tasklist
tasklist | findstr python
tasklist /FI "PID eq 1234"
taskkill /PID 1234
taskkill /F /PID 1234
```

PowerShell alternatives:

```powershell
Get-Process
Get-Process python
Stop-Process -Name python
```

## 8. Basic system information

| Command | Purpose |
|---|---|
| `hostname` | Display computer name |
| `whoami` | Display current user |
| `systeminfo` | Display detailed Windows system information |
| `ver` | Display Windows version |
| `Get-Date` | Display date and time in PowerShell |
| `Get-ComputerInfo` | Display computer information |

---

# Part 4 — Discover command help

## CMD

```cmd
help
ping /?
ipconfig /?
```

## PowerShell

```powershell
Get-Command
Get-Command *Network*
Get-Help Get-NetTCPConnection
Get-Help Get-NetTCPConnection -Examples
```

If detailed help is not installed locally, PowerShell may ask you to update its help files.

---

# Part 5 — A practical troubleshooting sequence

If a website is not reachable, investigate in this order:

1. Check your IP configuration:

   ```powershell
   ipconfig /all
   ```

2. Test your default gateway (replace this with the actual gateway shown by `ipconfig`):

   ```powershell
   ping 192.168.1.1
   ```

3. Test a public IP:

   ```powershell
   ping 8.8.8.8
   ```

4. Test DNS:

   ```powershell
   nslookup google.com
   ```

5. Trace the route:

   ```powershell
   tracert google.com
   ```

6. Test HTTPS over TCP:

   ```powershell
   Test-NetConnection google.com -Port 443
   ```

7. Inspect local listening ports:

   ```powershell
   Get-NetTCPConnection -State Listen
   ```

This sequence helps separate local configuration, router reachability, internet connectivity, DNS resolution, routing, and TCP port problems.

---

# Part 6 — Scapy practice using a local test server

Use Python's HTTP server to create a controlled test for your own Scapy port scanner.

## Terminal 1 — Start the server

```powershell
python -m http.server 8080 --bind 192.168.1.110
```

Keep this terminal open. The server must remain running during the scan.

## Terminal 2 — Confirm the TCP port

```powershell
Test-NetConnection 192.168.1.110 -Port 8080
```

Expected when the connection succeeds:

```text
TcpTestSucceeded : True
```

## Terminal 3 — Run your Scapy SYN scan

```bash
python3 port_scaning.py -t 192.168.1.110 -i "Intel(R) Wi-Fi 6 AX201 160MHz" -p 8080 -s syn
```

Expected classification while the server is listening:

```text
Port 8080 is open
```

Stop the server with `Ctrl+C`, then test again. The port should normally be reported as closed if no other service is listening and the local network stack returns a TCP reset.

If results differ, investigate the actual response packet, listening address, firewall, interface, and scanner classification logic.

---

# Part 7 — Important distinctions

| Test | What it tells you | What it does not prove |
|---|---|---|
| `ping` | Whether an ICMP response arrives | That a TCP service is open |
| `nslookup` | Whether DNS can resolve a name | That the website is reachable |
| `Test-NetConnection -Port` | Whether a TCP connection succeeds | The exact application behind the port |
| `netstat` | Local connection and socket information | That a remote host is reachable |
| `python -m http.server` | Creates a simple HTTP test service | Production-ready web hosting |
| Scapy SYN scan | Infers TCP port state from probe responses | The exact service or application version |

## Recommended learning order

1. `ipconfig /all`
2. `ping`
3. `arp -a`
4. `tracert`
5. `nslookup`
6. `netstat -ano`
7. `Get-NetTCPConnection`
8. `Test-NetConnection`
9. `python -m http.server`
10. Python `socket`
11. Scapy `IP()`, `TCP()`, `sr1()`, and response inspection

**Core idea:** Use Windows commands to establish a baseline, Python to build small test clients and servers, and Scapy to study and inspect the packets exchanged in your authorized lab.
