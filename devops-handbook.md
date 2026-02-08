# The Practical DevOps Handbook
### A VPS-First Guide to Running Real Systems

*Written for developers who build things and need to ship them without the magic going away in production.*

---

## 1. What DevOops Actually Is (and what it is NOT)

### Why DevOps Exists

Let me tell you what used to happen before DevOps became a thing.

You'd have developers - let's call them Dev - who wrote code on their MacBooks. Everything worked beautifully in development. Tests passed. The demo looked great. Then they'd throw the code over a metaphorical wall to Operations - the Ops team - who were responsible for keeping servers alive.

Ops people didn't write the code. They didn't understand the application's quirks. They just knew that at 3am when the server went down, *they* got the call. So naturally, Ops became extremely conservative. "Don't change anything" was the mantra. They controlled the servers with an iron fist.

Dev wanted to ship features fast. Ops wanted stability and sleep. These goals were fundamentally at odds.

The result? Dev would say "works on my machine" and Ops would say "well it doesn't work in production." Each blamed the other. Deployments happened once a month if you were lucky, involved a dozen manual steps, and everyone held their breath.

**DevOps is the philosophy that says: the people who build the software should also be responsible for running it in production.**

This changes everything. Suddenly:
- If you write code that crashes at 3am, YOU get paged
- If you write code that's hard to deploy, YOU feel that pain
- If you don't understand how the server works, that's YOUR problem

This alignment of incentives creates better software and better infrastructure.

### What DevOps Is NOT

DevOps is not:
- A job title (though recruiters ruined this)
- A separate team (that defeats the entire point)
- Just CI/CD pipelines
- Just Docker and Kubernetes
- A certification you get in a weekend

DevOps is a *culture shift* where developers gain operational responsibilities and operational concerns influence how software is built.

### The Mental Model Shift

**Before DevOps thinking:**
- "I write code, someone else deploys it"
- "Production is scary and mysterious"
- "Logs? That's an ops problem"
- "Why would I need to SSH into a server?"

**After DevOps thinking:**
- "I own this service end-to-end"
- "I know exactly how my code runs in production"
- "I instrument my code with logging and metrics from day one"
- "I can debug production issues because I understand the stack"

### What Actually Happens Under the Hood

DevOps isn't magic. It's a combination of:

1. **Cultural practices**: On-call rotations, blameless postmortems, shared responsibility
2. **Technical practices**: Infrastructure as code, automated deployments, monitoring
3. **Tools**: Git, CI/CD systems, containers, configuration management

The tools are just tools. The culture is what matters. You can have all the Kubernetes in the world and still have a toxic "throw it over the wall" culture.

### Common Beginner Mistakes

**Mistake 1: Thinking DevOps = tools**
Buying a bunch of tools doesn't make you DevOps. I've seen teams spend months setting up Kubernetes for a simple web app that could run on a single VPS. That's not DevOps, that's cargo culting.

**Mistake 2: Creating a "DevOps team"**
This recreates the same wall, just with a different name. DevOps should be embedded in product teams.

**Mistake 3: Ignoring the fundamentals**
You can't debug a container if you don't understand Linux processes. You can't fix networking issues if you don't understand how TCP works. The abstraction layers fail, and then you need the fundamentals.

**Mistake 4: Automation without understanding**
Automating a process you don't understand just means you break things faster. Always do it manually first, understand it deeply, then automate.

### Hands-On Practice Tasks

These assume you have a basic Ubuntu VPS. If you don't have one yet, get a $5/month VPS from DigitalOcean, Linode, or Hetzner.

**Task 1: SSH into your VPS**
```bash
ssh root@your-vps-ip
```
Just sit with that. You're now in a real Linux server somewhere in a datacenter. Everything you do here affects a real system.

**Task 2: Cause a problem and fix it**
```bash
# Create a simple Node.js app
echo 'console.log("Hello from VPS")' > app.js
node app.js

# Now try to run it in the background
node app.js &

# Close your SSH session and reconnect
# Is the process still running? (Spoiler: no)
ps aux | grep node
```

You just discovered why we need process managers. The shell killed your process when you disconnected.

**Task 3: Make it survive**
```bash
# Install PM2
npm install -g pm2

# Run with PM2
pm2 start app.js

# Disconnect and reconnect
# Check if it's still running
pm2 list
```

Now it survives. You just learned the difference between a foreground process and a properly daemonized service.

### Mini Project: Deploy a Simple Web App End-to-End

**Goal:** Deploy a basic Express.js app to your VPS and access it from your browser.

**Steps:**
1. Create a simple Express app on your VPS
2. Get it running with PM2
3. Access it via your VPS's IP address and port
4. When someone asks "where is your app deployed?" you can give them a real URL

**The Code:**
```bash
# On your VPS
mkdir ~/my-first-deploy
cd ~/my-first-deploy

# Create a simple Express app
cat > app.js << 'EOF'
const express = require('express');
const app = express();
const PORT = 3000;

app.get('/', (req, res) => {
  res.json({
    message: 'Hello from my VPS!',
    timestamp: new Date().toISOString()
  });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# Install dependencies
npm init -y
npm install express

# Run it
pm2 start app.js --name my-app
pm2 save  # Save the process list
pm2 startup  # Make it start on boot
```

Now visit `http://your-vps-ip:3000` in your browser.

**What you just did:**
- Deployed code to a real server
- Made it survive reboots
- Made it accessible over the internet

This is the foundation. Everything else is elaborating on this pattern.

---

## 2. Linux for DevOps (Deep, Practical)

### Why Linux?

Windows runs most desktop computers. macOS runs developer laptops. But Linux runs the internet.

When you visit a website, when you use an app, when you stream a video - somewhere in that chain is a Linux server doing the actual work. Understanding Linux isn't optional for DevOps. It's the foundation.

### The Filesystem Hierarchy (Why It's Designed This Way)

When you run `ls /` on a Linux system, you see directories like `/bin`, `/etc`, `/home`, `/var`. This isn't random. There's a philosophy here.

```
/
├── bin/     # Essential binaries (commands you need to boot and fix the system)
├── boot/    # Bootloader files, kernel
├── dev/     # Device files (everything is a file in Linux)
├── etc/     # System configuration files
├── home/    # User home directories
├── lib/     # Shared libraries (like .dll files on Windows)
├── opt/     # Optional software (third-party apps)
├── proc/    # Virtual filesystem, kernel and process info
├── root/    # Home directory for the root user
├── sbin/    # System binaries (admin commands)
├── tmp/     # Temporary files (cleared on boot)
├── usr/     # User programs and data
│   ├── bin/     # Non-essential user commands
│   ├── local/   # Locally installed software
│   └── share/   # Architecture-independent data
└── var/     # Variable data (logs, databases, caches)
    ├── log/     # Log files
    ├── www/     # Web server files (common convention)
    └── lib/     # Variable state information
```

**The Philosophy:**

`/bin` vs `/usr/bin`: `/bin` contains commands needed to boot the system and repair it in single-user mode. `/usr/bin` contains everything else. This matters if `/usr` is on a separate partition that failed to mount.

`/etc`: "Editable Text Configuration". All system config goes here. No binaries, just configuration. This makes backups simple - back up `/etc` and you've got your configuration.

`/var`: "Variable". Anything that changes during operation goes here. Logs grow, databases change, caches fill. If you run out of disk space, check `/var` first.

`/tmp`: Temporary files that don't need to survive a reboot. Programs can write here without permission. Some systems mount this as a tmpfs (RAM disk).

`/proc` and `/sys`: These aren't real directories. They're virtual filesystems that the kernel uses to expose information. Want to know a process's memory usage? Read `/proc/[pid]/status`. Want to change kernel parameters? Write to `/proc/sys/`.

### What Actually Happens Under the Hood

When you run a command like `ls`, here's what happens:

1. Shell searches `$PATH` for an executable named `ls`
2. Finds `/usr/bin/ls` (you can verify: `which ls`)
3. Kernel loads the binary into memory
4. Creates a new process
5. Process makes syscalls to read directory entries
6. Writes output to stdout (file descriptor 1)
7. Process exits, kernel cleans up

The key insight: **Everything is a file.** That's not metaphorical. Devices are files in `/dev`. Processes expose info as files in `/proc`. Sockets are files. Even your network connection is represented as a file.

This means you can use the same tools (`cat`, `echo`, `grep`) to interact with everything.

### Users, Groups, and Permissions (Real Security Implications)

Linux is a multi-user system from the 1970s. Even though your VPS might only have you using it, the system still thinks in terms of multiple users who shouldn't mess with each other's stuff.

**The Basics:**
```bash
ls -l /var/log/syslog
-rw-r----- 1 syslog adm 147123 Feb  7 10:30 /var/log/syslog
```

Let's decode this:
- `-`: Regular file (d=directory, l=symlink)
- `rw-r-----`: Permissions (more on this in a sec)
- `1`: Number of hard links
- `syslog`: Owner (user)
- `adm`: Group
- `147123`: Size in bytes
- `Feb 7 10:30`: Last modified
- `/var/log/syslog`: Filename

**Permissions: rwxrwxrwx**

Three groups of three:
1. `rw-`: Owner permissions (read, write, no execute)
2. `r--`: Group permissions (read only)
3. `---`: Other permissions (nothing)

Each position means:
- `r` (4): Read permission
- `w` (2): Write permission
- `x` (1): Execute permission

**Why This Matters in DevOps:**

When you deploy an app, you don't run it as root. You create a dedicated user:

```bash
# Create a user for your app
useradd --system --shell /bin/bash --home /opt/myapp myapp

# Set ownership
chown -R myapp:myapp /opt/myapp

# Now your app can only touch its own files
```

If your app gets compromised, the attacker only has the privileges of that user. They can't install system packages, modify other apps, or access sensitive files. This is **defense in depth**.

**Common Permission Patterns:**

```bash
# Web server files: readable by web server, writable by deploy user
chown -R deploy:www-data /var/www/myapp
chmod -R 755 /var/www/myapp  # rwxr-xr-x

# Configuration files: readable by app, writable only by root
chown root:myapp /etc/myapp/config.json
chmod 640 /etc/myapp/config.json  # rw-r-----

# Log files: writable by app, readable by admin group
chown myapp:adm /var/log/myapp.log
chmod 640 /var/log/myapp.log
```

**The Sticky Bit and SUID:**

You'll sometimes see permissions like `rwsr-xr-x` or `rwxrwxrwt`. These are special:

- **SUID (s in user position)**: Execute as the file's owner, not the current user
  ```bash
  ls -l /usr/bin/passwd
  -rwsr-xr-x 1 root root 68208 passwd
  ```
  Regular users can run `passwd` to change their password, but it runs as root (needed to modify `/etc/shadow`).

- **Sticky bit (t in other position)**: On directories, only file owners can delete their files
  ```bash
  ls -ld /tmp
  drwxrwxrwt 20 root root 4096 /tmp
  ```
  Anyone can create files in `/tmp`, but you can't delete someone else's files.

### Processes, Signals, and systemd

**What is a Process?**

A process is a running program. When you execute `node app.js`, the kernel creates a process for that Node.js instance.

Every process has:
- **PID**: Process ID (unique number)
- **PPID**: Parent process ID (who spawned this?)
- **User**: Who owns this process
- **Memory**: RAM allocated to it
- **File descriptors**: Open files, sockets, etc.

```bash
# See all processes
ps aux

# See process tree
pstree

# See what a specific process is doing
top -p [PID]

# See open files for a process
lsof -p [PID]
```

**The Process Lifecycle:**

```
                    fork()
   Parent Process ---------> Child Process

                             exec()
                             ------> Replace child with new program

                             exit()
                             ------> Process terminates

                             (zombie state)

   Parent calls wait()
   -----------------> Child fully cleaned up
```

When you run a command in bash, bash forks itself (creates a copy), then that copy execs the command (replaces itself with the new program).

**Signals: How to Talk to Processes**

Signals are how you communicate with processes. They're like taps on the shoulder, except some taps are gentle and some are "I'm killing you right now."

Common signals:
- `SIGTERM` (15): "Please shut down gracefully" - default for `kill`
- `SIGKILL` (9): "Die immediately, no cleanup" - use as last resort
- `SIGHUP` (1): "Reload your config" - used by many daemons
- `SIGINT` (2): Ctrl+C in terminal
- `SIGUSR1/SIGUSR2`: Application-defined behavior

```bash
# Graceful shutdown (app can cleanup)
kill [PID]
kill -SIGTERM [PID]
kill -15 [PID]

# Force kill (immediate, no cleanup)
kill -9 [PID]
kill -SIGKILL [PID]

# Reload config
kill -SIGHUP [PID]
```

**Why This Matters:**

When you deploy a new version of your app, you want to:
1. Send SIGTERM to the old process
2. Wait for it to finish handling existing requests
3. Start the new process

If you use SIGKILL, you'll drop active connections. Users will see errors.

**systemd: The Modern Init System**

When your Linux system boots, the kernel starts one process: PID 1. This is the init system. On modern Ubuntu (16.04+), that's systemd.

systemd is responsible for:
- Starting services at boot
- Restarting crashed services
- Managing dependencies between services
- Collecting logs

**A Simple systemd Service:**

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Node.js Application
After=network.target

[Service]
Type=simple
User=myapp
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/node /opt/myapp/app.js
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Let's break this down:

**[Unit]**: Metadata and dependencies
- `After=network.target`: Don't start until network is up

**[Service]**: How to run it
- `Type=simple`: Process doesn't daemonize itself
- `User=myapp`: Run as this user (not root!)
- `ExecStart`: Command to run
- `Restart=on-failure`: Restart if it crashes
- `StandardOutput=journal`: Send logs to systemd journal

**[Install]**: When to run it
- `WantedBy=multi-user.target`: Start in normal multi-user mode

**Using systemd:**

```bash
# Enable service (start on boot)
systemctl enable myapp

# Start service now
systemctl start myapp

# Check status
systemctl status myapp

# View logs
journalctl -u myapp -f  # -f follows like tail

# Restart after code change
systemctl restart myapp

# Stop service
systemctl stop myapp
```

**PM2 vs systemd:**

PM2 is easier to start with:
```bash
pm2 start app.js
pm2 logs
pm2 restart app
```

But systemd is more powerful:
- Better integration with the OS
- Stricter security controls (capabilities, namespaces)
- All logs go to one place (journald)
- Service dependencies

For production, I prefer systemd. For quick experiments, PM2 is fine.

### Logs and Observability

**Where Do Logs Live?**

- systemd services: `journalctl -u servicename`
- Traditional syslog: `/var/log/syslog`
- Application logs: Wherever you configured (often `/var/log/appname/`)
- Web server logs:
  - Nginx: `/var/log/nginx/access.log` and `/var/log/nginx/error.log`
  - Apache: `/var/log/apache2/`

**The Mental Model:**

Logs are a stream of events. They tell you what happened, in what order. When something breaks, logs are your time machine.

**Good Logging Practice:**

```javascript
// Bad
console.log('error');

// Better
console.log('Failed to connect to database');

// Best
console.log(JSON.stringify({
  level: 'error',
  message: 'Failed to connect to database',
  error: err.message,
  host: dbConfig.host,
  timestamp: new Date().toISOString(),
  requestId: req.id
}));
```

Structured logs (JSON) are grep-able and parseable. You can search for `"level":"error"` or filter by `requestId`.

**Watching Logs:**

```bash
# Follow a log file
tail -f /var/log/myapp/app.log

# Follow systemd logs
journalctl -u myapp -f

# Search logs
grep "ERROR" /var/log/myapp/app.log

# Count error frequency
grep "ERROR" /var/log/myapp/app.log | wc -l

# Show errors in the last hour
journalctl -u myapp --since "1 hour ago" | grep ERROR
```

### Networking Basics in Linux

**Network Interfaces:**

Your server has network interfaces. Think of them like physical network cards, though they might be virtual.

```bash
# See all network interfaces
ip addr show

# Common interfaces:
# lo    - loopback (127.0.0.1, talking to yourself)
# eth0  - first Ethernet interface
# wlan0 - wireless
```

**Listening on Ports:**

When your app does `app.listen(3000)`, it's telling the kernel "I want to handle TCP connections on port 3000."

```bash
# See what's listening on what ports
netstat -tlnp
# or on newer systems
ss -tlnp

# Example output:
# tcp   0   0 0.0.0.0:3000   0.0.0.0:*   LISTEN   12345/node
#       │   │    │              │                  │
#       │   │    │              │                  └─ Process
#       │   │    │              └─ Remote address (any)
#       │   │    └─ Local address (any IP) and port
#       │   └─ Receive queue
#       └─ Send queue
```

`0.0.0.0:3000` means "listen on port 3000 on all interfaces." If you see `127.0.0.1:3000`, it only accepts connections from localhost.

**Testing Connectivity:**

```bash
# Is the port open?
nc -zv your-server.com 3000

# Can I connect to it?
curl http://your-server.com:3000

# Is DNS resolving?
dig your-server.com

# Can I reach the server at all?
ping your-server.com

# Trace the route packets take
traceroute your-server.com
```

**Firewall Basics:**

Your VPS likely has a firewall. On Ubuntu, that's usually `ufw` (Uncomplicated Firewall):

```bash
# Check firewall status
ufw status

# Allow SSH (don't lock yourself out!)
ufw allow 22/tcp

# Allow HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow a specific port
ufw allow 3000/tcp

# Enable firewall
ufw enable

# See numbered rules (for deletion)
ufw status numbered

# Delete a rule
ufw delete [number]
```

**The cardinal rule: ALWAYS allow SSH before enabling the firewall, or you'll lock yourself out.**

### What Actually Happens Under the Hood

When you run `app.listen(3000)`, here's the syscall chain:

1. `socket()` - Create a socket
2. `bind()` - Bind socket to 0.0.0.0:3000
3. `listen()` - Start listening for connections
4. `accept()` - Wait for and accept incoming connections (blocking call)

When a request comes in:
1. Packet arrives at network interface
2. Kernel checks firewall rules
3. Kernel checks if anything is listening on that port
4. Creates a new socket for this specific connection
5. Wakes up your process
6. Your process calls `accept()` and gets the connection

All of this is hidden behind `app.listen()`, but when things break, you need to know where in this chain the problem is.

### Common Beginner Mistakes

**Mistake 1: Running everything as root**
```bash
# Bad
sudo node app.js

# Good
# Create a dedicated user and use systemd
```
Running as root means if your app is compromised, the attacker owns the entire server.

**Mistake 2: Ignoring file permissions**
```bash
# This will cause subtle bugs
chmod 777 -R /var/www  # Never do this
```
World-writable directories are a security nightmare. Be explicit about who needs what access.

**Mistake 3: Not checking what's already using a port**
```bash
# Your app fails to start with "port already in use"
# Check what's using it:
lsof -i :3000
# Kill it or use a different port
```

**Mistake 4: Forgetting about the firewall**
"It works on localhost but not from my browser!" - Check `ufw status`.

**Mistake 5: Not understanding stdout vs files for logs**
systemd can capture stdout/stderr. But if your app writes to a file, systemd won't see it. Be consistent.

### Hands-On Practice Tasks

**Task 1: Explore the filesystem**
```bash
# Find the biggest directories in /var
du -h /var | sort -h | tail -20

# What's in /proc for your shell?
ls -l /proc/$$/
cat /proc/$$/status
cat /proc/$$/cmdline

# Find all config files modified in the last 7 days
find /etc -type f -mtime -7 -ls
```

**Task 2: Practice with users and permissions**
```bash
# Create a test user
useradd -m testuser

# Create a file as root
echo "secret" > /root/secret.txt
chmod 600 /root/secret.txt

# Try to read it as testuser
su - testuser
cat /root/secret.txt  # Permission denied

# Exit back to root and create a shared file
exit
echo "shared" > /tmp/shared.txt
chown root:testuser /tmp/shared.txt
chmod 640 /tmp/shared.txt

# Now testuser can read it
su - testuser
cat /tmp/shared.txt  # Works!
```

**Task 3: Master systemd**
```bash
# List all running services
systemctl list-units --type=service --state=running

# Check SSH service
systemctl status ssh

# See recent logs
journalctl -u ssh -n 50

# See logs from last boot
journalctl -b
```

**Task 4: Network debugging**
```bash
# What ports are listening?
ss -tlnp

# Install a test HTTP server
python3 -m http.server 8000 &

# Check if it's listening
ss -tlnp | grep 8000

# Access it
curl localhost:8000

# Kill it
killall python3
```

### Mini Project: Create a Multi-User Environment

**Goal:** Set up a real production-like environment for an app with proper user isolation.

```bash
# 1. Create a dedicated user for your app
useradd --system --shell /bin/bash --home /opt/webapp webapp

# 2. Create the directory structure
mkdir -p /opt/webapp/{app,logs,config}
chown -R webapp:webapp /opt/webapp

# 3. Switch to that user
su - webapp

# 4. Create a simple app
cat > /opt/webapp/app/server.js << 'EOF'
const http = require('http');
const fs = require('fs');

const PORT = 8080;

const server = http.createServer((req, res) => {
  const log = `${new Date().toISOString()} ${req.method} ${req.url}\n`;
  fs.appendFileSync('/opt/webapp/logs/access.log', log);

  res.writeHead(200);
  res.end(`Hello! I'm running as user: ${process.env.USER}\n`);
});

server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
EOF

# 5. Exit back to root
exit

# 6. Create systemd service
cat > /etc/systemd/system/webapp.service << 'EOF'
[Unit]
Description=Web Application
After=network.target

[Service]
Type=simple
User=webapp
Group=webapp
WorkingDirectory=/opt/webapp/app
ExecStart=/usr/bin/node server.js
Restart=always
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

# 7. Start it
systemctl daemon-reload
systemctl enable webapp
systemctl start webapp

# 8. Check status
systemctl status webapp
journalctl -u webapp -f
```

Now you have:
- An app running as a non-root user
- Automatic restart on crashes
- Automatic start on boot
- Centralized logging
- Proper file ownership

This is how real systems are set up.

---

## 3. How the Internet Reaches Your Server

### DNS Resolution (Step-by-Step Flow)

When you type `example.com` into your browser, your computer doesn't know where that is. It needs to translate that human-readable name into an IP address. This is DNS resolution.

**The Complete Flow:**

```
You type: https://api.example.com

1. Browser checks its cache
   "Have I looked up api.example.com recently?"

2. OS checks /etc/hosts
   (Local override file)

3. OS asks configured DNS resolver
   (Usually ISP's DNS or 8.8.8.8)

4. Recursive resolver checks its cache

5. If not cached, resolver asks root nameservers
   "Who handles .com domains?"
   Root: "Ask the .com nameservers at [IP list]"

6. Resolver asks .com nameservers
   "Who handles example.com?"
   .com: "Ask ns1.example.com at [IP]"

7. Resolver asks example.com's nameserver
   "What's the IP for api.example.com?"
   Nameserver: "It's 1.2.3.4"

8. Resolver caches this answer (TTL: time to live)
   Returns to your OS

9. OS caches it, returns to browser

10. Browser caches it, connects to 1.2.3.4
```

**The Key Insight:**

DNS is cached at every level. When you update a DNS record, it doesn't propagate instantly. It has to wait for caches to expire based on TTL (Time To Live).

```bash
# Check DNS resolution
dig example.com

# The output shows the full chain
# ;; ANSWER SECTION:
# example.com.  3600  IN  A  93.184.216.34
#               └─ TTL in seconds (1 hour)

# Check a specific record type
dig example.com A      # IPv4 address
dig example.com AAAA   # IPv6 address
dig example.com MX     # Mail server
dig example.com TXT    # Text records

# See the full resolution path
dig +trace example.com
```

**DNS Record Types:**

- **A**: Maps domain to IPv4 address
- **AAAA**: Maps domain to IPv6 address
- **CNAME**: Alias to another domain
- **MX**: Mail server for this domain
- **TXT**: Arbitrary text (used for verification, SPF, etc.)
- **NS**: Nameservers for this domain

**Setting Up DNS for Your VPS:**

When you buy a domain, you need to point it to your VPS:

```
1. Go to your domain registrar (Namecheap, GoDaddy, etc.)
2. Find DNS settings
3. Create an A record:
   Host: @              (means root domain)
   Value: your-vps-ip
   TTL: 3600           (1 hour)

4. Create another A record for www:
   Host: www
   Value: your-vps-ip
   TTL: 3600

5. Wait for propagation (up to 48 hours, usually minutes)

6. Test:
   dig yourdomain.com
```

### TCP vs UDP

Your app communicates over the network using protocols. The two fundamental ones are TCP and UDP.

**TCP (Transmission Control Protocol): The Reliable One**

TCP is like a phone call. You establish a connection, both sides know about each other, and messages arrive in order.

```
Client                          Server
  |                               |
  |--------- SYN ------------->   |  "I want to connect"
  |                               |
  | <------- SYN-ACK ----------   |  "OK, I acknowledge"
  |                               |
  |--------- ACK ------------->   |  "Great, we're connected"
  |                               |
  |===== DATA EXCHANGE =====|
  |                               |
  |--------- FIN ------------->   |  "I'm done"
  |                               |
  | <------- FIN-ACK ----------   |  "OK, closing"
```

**Features:**
- Connection-oriented (handshake required)
- Guaranteed delivery (packets are retransmitted if lost)
- Ordered delivery (packets arrive in sequence)
- Flow control (sender doesn't overwhelm receiver)
- Error checking

**Use cases:** HTTP, HTTPS, SSH, database connections - anything where you can't afford to lose data.

**UDP (User Datagram Protocol): The Fast One**

UDP is like sending postcards. You throw them in the mail and hope they arrive. No guarantees.

```
Client                          Server
  |                               |
  |--------- DATA ------------->  |  "Here's some data!"
  |--------- DATA ------------->  |  "Here's some more!"
  |                               |
  (No acknowledgment, no connection)
```

**Features:**
- Connectionless (no handshake)
- No guaranteed delivery
- No ordered delivery
- Faster (less overhead)
- Simple

**Use cases:** DNS, video streaming, gaming, VoIP - anything where speed matters more than perfect delivery.

**Why DNS uses UDP:**

DNS queries are tiny. A UDP packet can fit the entire query and response. TCP's handshake would triple the latency for no benefit. But if a DNS response is too large (>512 bytes traditionally, now often >1232), it falls back to TCP.

### What Happens When You Type a URL in a Browser

Let's trace the entire journey for `https://api.example.com/users`:

**Step 1: DNS Resolution**
```
Browser: "What's the IP for api.example.com?"
DNS: "It's 1.2.3.4"
```

**Step 2: TCP Connection**
```
Browser: "I want to connect to 1.2.3.4 on port 443"
(SYN, SYN-ACK, ACK - three-way handshake)
TCP connection established
```

**Step 3: TLS Handshake (because HTTPS)**
```
Client: "I want to speak HTTPS. Here are the encryption methods I support."
Server: "Let's use TLS 1.3 with AES. Here's my certificate to prove I'm api.example.com."
Client: Verifies certificate, generates session key
Both: Encrypted connection established
```

**Step 4: HTTP Request**
```
GET /users HTTP/1.1
Host: api.example.com
User-Agent: Mozilla/5.0...
Accept: application/json
(encrypted over TLS)
```

**Step 5: Server Processing**
```
Nginx receives request on port 443
Decrypts TLS
Looks at Host header: "api.example.com"
Checks configuration: "This should go to Node.js on localhost:3000"
Proxies request to Node.js app
```

**Step 6: Application Response**
```
Node.js app:
- Checks route: GET /users
- Queries database
- Returns JSON

Response goes back through:
Node.js -> Nginx -> TLS encryption -> TCP -> Internet -> Your browser
```

**Step 7: Browser Rendering**
```
Browser receives encrypted response
Decrypts it
Parses JSON
Renders in DevTools or on page
```

**Total time: Usually 100-500ms**
- DNS: 20-100ms (cached afterward)
- TCP handshake: 1 round trip (~20-50ms)
- TLS handshake: 1-2 round trips (~20-100ms)
- HTTP request/response: 1 round trip + processing (~50-250ms)

### Ports, Sockets, and Firewalls

**What is a Port?**

An IP address identifies a server. A port identifies a specific service on that server.

Think of it like an apartment building:
- IP address = building address
- Port = apartment number

Common ports:
- 22: SSH
- 80: HTTP
- 443: HTTPS
- 3000: Node.js convention
- 5432: PostgreSQL
- 6379: Redis
- 27017: MongoDB

Ports 0-1023 are "privileged" - only root can listen on them. That's why you need sudo to run a web server on port 80.

**What is a Socket?**

A socket is an endpoint of communication. It's a combination of IP + port + protocol.

```
Socket = (IP address, Port, Protocol)

Example:
192.168.1.100:3000 TCP
```

When your app calls `listen(3000)`, it creates a socket and tells the kernel "I'm ready to accept connections on this socket."

**Connection Tracking:**

When a client connects, the kernel creates a unique socket for that connection:

```
Server listening: 0.0.0.0:3000
Client 1 connects: 192.168.1.50:54321 -> server:3000
Client 2 connects: 192.168.1.51:54322 -> server:3000

Three separate sockets:
1. Listening socket (server)
2. Connection socket for client 1
3. Connection socket for client 2
```

**Firewalls:**

A firewall is a filter that decides which network packets to allow.

```
Packet arrives at server
    |
    v
Firewall checks rules:
1. Is this an established connection? -> ALLOW
2. Is this new connection to port 22? -> ALLOW (SSH)
3. Is this new connection to port 80? -> ALLOW (HTTP)
4. Is this new connection to port 443? -> ALLOW (HTTPS)
5. Is this new connection to port 3000? -> DENY
    |
    v
Drop packet or send to application
```

**UFW (Uncomplicated Firewall) Deep Dive:**

```bash
# Default policy: deny incoming, allow outgoing
ufw default deny incoming
ufw default allow outgoing

# Allow SSH (ALWAYS DO THIS FIRST)
ufw allow 22/tcp

# Allow HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Enable
ufw enable

# Check what's allowed
ufw status verbose

# Allow from specific IP
ufw allow from 1.2.3.4 to any port 22

# Allow port range
ufw allow 6000:6100/tcp

# Delete a rule
ufw status numbered
ufw delete [number]
```

**iptables: The Real Firewall**

`ufw` is actually a friendly wrapper around `iptables`, the real Linux firewall.

```bash
# See actual rules
iptables -L -n -v

# This shows the raw rules ufw created
# It's complex, which is why ufw exists
```

You don't need to master iptables right away, but know it exists. When ufw can't do something, iptables can.

### What Actually Happens Under the Hood

When a packet arrives at your VPS:

```
1. Network interface receives packet

2. Kernel checks iptables (firewall)
   - PREROUTING chain (modify before routing decision)
   - Routing decision (is this for me or should I forward it?)
   - INPUT chain (final check before delivery)

3. If allowed, kernel checks:
   - Is this an existing connection? -> Send to socket
   - Is this a new connection?
     - Is something listening on this port?
       - Yes: Create new socket, wake up process
       - No: Send RST (connection reset)

4. Application reads from socket

5. Application writes response to socket

6. Kernel sends response:
   - OUTPUT chain (firewall check)
   - POSTROUTING chain (NAT if needed)
   - Network interface sends packet
```

All of this happens in milliseconds.

### Common Beginner Mistakes

**Mistake 1: Forgetting to open firewall ports**
```bash
# App is running, but can't connect from outside
# Check firewall
ufw status

# If port isn't allowed
ufw allow [port]/tcp
```

**Mistake 2: Binding to 127.0.0.1 instead of 0.0.0.0**
```javascript
// This only accepts connections from localhost
app.listen(3000, '127.0.0.1');

// This accepts connections from anywhere
app.listen(3000, '0.0.0.0');
// Or just
app.listen(3000);  // Defaults to 0.0.0.0
```

**Mistake 3: Using HTTP when you meant HTTPS**
```bash
# Wrong
curl http://example.com

# Right
curl https://example.com
```
They're different ports (80 vs 443). Check which one is actually running.

**Mistake 4: Not understanding DNS caching**
"I changed the DNS but it still points to the old IP!"

DNS is cached everywhere. Lower TTL before making changes:
```bash
# Before making a change
# Set TTL to 300 (5 minutes)
# Wait for old TTL to expire
# Make the change
# After it's working, raise TTL back to 3600
```

**Mistake 5: Hardcoding localhost**
```javascript
// Bad - breaks when you deploy
const DB_HOST = 'localhost';

// Good - use environment variables
const DB_HOST = process.env.DB_HOST || 'localhost';
```

### Hands-On Practice Tasks

**Task 1: Trace a DNS lookup**
```bash
# Use dig with +trace to see the full chain
dig +trace google.com

# You'll see:
# - Root servers
# - .com servers
# - google.com's nameservers
# - Final answer
```

**Task 2: Watch a TCP connection**
```bash
# Install tcpdump
apt install tcpdump

# Capture HTTP traffic (run in one terminal)
tcpdump -i any -n port 80

# In another terminal, make a request
curl http://example.com

# Watch the SYN, SYN-ACK, ACK, then data
```

**Task 3: Test port connectivity**
```bash
# From your local machine

# Test if port is open
nc -zv your-vps-ip 80

# Test if SSH is working
nc -zv your-vps-ip 22

# Test if something is listening but firewalled
nc -zv your-vps-ip 3000  # Probably times out if firewalled
```

**Task 4: Understand socket states**
```bash
# On your VPS, run a web server
python3 -m http.server 8000 &

# Watch the socket
watch -n 1 'ss -tn | grep 8000'

# From another terminal, connect
curl localhost:8000

# Watch the socket states change:
# LISTEN -> SYN-SENT -> ESTABLISHED -> FIN-WAIT -> CLOSED
```

### Mini Project: Troubleshoot a Connectivity Issue

**Scenario:** You deployed an app but can't connect to it from your browser.

**Systematic Debugging:**

```bash
# Step 1: Is the app running?
ps aux | grep node
systemctl status myapp

# Step 2: Is it listening on the right port?
ss -tlnp | grep 3000

# If it says 127.0.0.1:3000, it's only listening locally
# Fix: Change app to listen on 0.0.0.0

# Step 3: Can you connect locally?
curl localhost:3000

# If this works, the app is fine

# Step 4: Is the firewall blocking?
ufw status | grep 3000

# If not listed, add it
ufw allow 3000/tcp

# Step 5: Can you connect from your machine?
curl http://your-vps-ip:3000

# If this fails, check VPS provider firewall (security groups)

# Step 6: Is DNS resolving correctly?
dig yourdomain.com

# Does it point to your VPS IP?

# Step 7: Try connecting via IP directly
curl http://your-vps-ip:3000

# If IP works but domain doesn't, it's DNS
# If neither works, it's firewall or app binding
```

This systematic approach will solve 95% of connectivity issues.

---

## 4. Running Applications on a Server

### Why Apps Run on Ports

Your server has one IP address but needs to run multiple services: a web app, a database, maybe an API. How does the kernel know which packets go to which service?

Ports.

When you run `app.listen(3000)`, you're telling the kernel: "Any TCP packets arriving at port 3000, send them to me."

**The Mental Model:**

```
Internet packet arrives at your server
    |
    v
Kernel looks at destination port:
    - Port 22? -> SSH server
    - Port 80? -> Nginx
    - Port 3000? -> Your Node.js app
    - Port 5432? -> PostgreSQL
    |
    v
Packet delivered to the right process
```

Multiple apps can coexist because each listens on a different port.

**Port Conflicts:**

You can't have two processes listening on the same port:

```bash
# Terminal 1
node app.js  # Listens on 3000

# Terminal 2
node app.js  # Error: EADDRINUSE (address already in use)
```

The kernel enforces this. One port, one listener.

**Finding What's Using a Port:**

```bash
# Method 1: lsof
lsof -i :3000

# Method 2: netstat
netstat -tlnp | grep 3000

# Method 3: ss (modern replacement)
ss -tlnp | grep 3000

# Kill the process using the port
kill [PID]
```

### Background Processes

When you run `node app.js` in your SSH session, it runs in the foreground. Close your terminal, the process dies.

**Why?**

Your shell (bash) creates a session. Processes in that session receive a SIGHUP (hangup) when the session ends. By default, this kills them.

**Solutions:**

**Option 1: nohup (No Hang Up)**
```bash
nohup node app.js &

# & puts it in background
# nohup makes it ignore SIGHUP
# Output goes to nohup.out
```

This is crude but works for quick tests.

**Option 2: screen/tmux**
```bash
# Start screen
screen

# Run your app
node app.js

# Detach: Ctrl+A, then D
# Reattach later
screen -r
```

This is good for interactive debugging, not for production.

**Option 3: PM2**
```bash
pm2 start app.js

# PM2 daemonizes the process
# You can disconnect, it keeps running
```

**Option 4: systemd** (Best for production)
```bash
# Create a service file
systemctl start myapp

# systemd manages it independently of any session
```

### PM2 vs systemd vs Containers

Let's compare these approaches properly.

**PM2:**

```bash
# Start
pm2 start app.js --name myapp

# Features
pm2 logs myapp       # View logs
pm2 restart myapp    # Restart
pm2 stop myapp       # Stop
pm2 delete myapp     # Remove
pm2 monit            # Live monitoring
pm2 startup          # Generate startup script
pm2 save             # Save process list
```

**Pros:**
- Easy to use
- Great for Node.js apps
- Built-in load balancing (cluster mode)
- Nice CLI

**Cons:**
- One more thing to manage
- Not OS-native
- Less control than systemd

**systemd:**

```bash
# Create /etc/systemd/system/myapp.service
[Unit]
Description=My Application

[Service]
ExecStart=/usr/bin/node /opt/myapp/app.js
User=myapp
Restart=always

[Install]
WantedBy=multi-user.target

# Then
systemctl start myapp
```

**Pros:**
- OS-native (it's going to be there anyway)
- Powerful (can limit CPU, memory, set capabilities)
- All logs go to journald (centralized)
- Better security options

**Cons:**
- More verbose configuration
- Requires root to modify service files

**Containers (Docker):**

```bash
# Dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
CMD ["node", "app.js"]

# Run
docker build -t myapp .
docker run -d -p 3000:3000 myapp
```

**Pros:**
- Consistent environment (works same everywhere)
- Isolation (dependencies don't conflict)
- Easy to ship

**Cons:**
- Added complexity
- Overhead (disk, memory)
- Overkill for simple apps

**My Recommendation:**

- Learning/quick tests: PM2
- Production single-server: systemd
- Production multi-server or complex dependencies: Docker + orchestration

### Environment Variables and Secrets

Your app needs configuration that changes between environments:

```
Development:
- Database: localhost
- API keys: test keys
- Debug: enabled

Production:
- Database: production-db.internal
- API keys: real keys
- Debug: disabled
```

**Don't hardcode this.**

**Use Environment Variables:**

```javascript
// Bad
const dbHost = 'localhost';
const apiKey = 'sk_test_12345';

// Good
const dbHost = process.env.DB_HOST || 'localhost';
const apiKey = process.env.API_KEY;

if (!apiKey) {
  throw new Error('API_KEY environment variable required');
}
```

**Setting Environment Variables:**

**Method 1: .env file (development)**
```bash
# .env file (NEVER commit this to git)
DB_HOST=localhost
DB_USER=admin
DB_PASS=secretpassword
API_KEY=sk_test_12345

# Load it with dotenv
npm install dotenv
```

```javascript
require('dotenv').config();
console.log(process.env.DB_HOST);  // localhost
```

**Method 2: systemd service (production)**
```ini
[Service]
Environment="DB_HOST=prod-db.internal"
Environment="DB_USER=webapp"
EnvironmentFile=/etc/myapp/secrets.env
ExecStart=/usr/bin/node /opt/myapp/app.js
```

**Method 3: Export in shell**
```bash
export DB_HOST=localhost
node app.js
```

**Secrets Management:**

**Bad:**
```bash
# Committing secrets to git
git add .env
git commit -m "Added config"
# Now your API keys are in git history forever
```

**Good:**
```bash
# .gitignore
.env
secrets/
*.key

# Keep secrets in a separate file
sudo cat > /etc/myapp/secrets.env << 'EOF'
DB_PASSWORD=actuallySecure123
API_KEY=sk_live_realkey
EOF

# Restrict permissions
sudo chmod 600 /etc/myapp/secrets.env
sudo chown myapp:myapp /etc/myapp/secrets.env
```

**Even Better:**

Use a secrets manager (Vault, AWS Secrets Manager) but that's beyond a single VPS setup.

**Environment Variable Best Practices:**

1. **Never commit secrets to git**
2. **Use .env for local development**
3. **Use systemd EnvironmentFile for production**
4. **Validate required vars on startup**
5. **Use sensible defaults for non-secrets**

```javascript
// Validate on startup
const requiredEnvVars = ['DB_HOST', 'DB_PASSWORD', 'API_KEY'];
for (const envVar of requiredEnvVars) {
  if (!process.env[envVar]) {
    console.error(`Missing required environment variable: ${envVar}`);
    process.exit(1);
  }
}
```

### What Actually Happens Under the Hood

When you run a Node.js app with systemd:

```
1. systemd reads /etc/systemd/system/myapp.service
2. Sets up the environment (User, WorkingDirectory, Environment vars)
3. Forks a new process
4. Sets the user/group for that process
5. Executes the ExecStart command
6. Monitors the process
7. If it crashes, waits RestartSec seconds
8. Restarts it (if Restart=always)
9. Logs stdout/stderr to journald
```

When the process is running:

```
Node.js process:
1. Loads environment variables
2. Requires modules
3. Calls listen(3000)
4. Kernel creates socket, binds to port 3000
5. Process enters event loop
6. Waits for connections (blocking on accept())
7. Connection arrives, kernel wakes process
8. Process handles request
9. Back to waiting
```

### Common Beginner Mistakes

**Mistake 1: Not using a process manager**
```bash
# This dies when SSH disconnects
node app.js
```

Use PM2 or systemd.

**Mistake 2: Running as root**
```bash
# Bad
sudo node app.js
```

If your app is compromised, attacker has root. Create a dedicated user.

**Mistake 3: Not handling process signals**
```javascript
// Bad - abrupt shutdown drops connections

// Good - graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, closing server gracefully');
  server.close(() => {
    console.log('Server closed');
    // Close database connections
    db.disconnect();
    process.exit(0);
  });
});
```

**Mistake 4: Committing .env to git**
```bash
# Check what would be committed
git status

# If .env is there, remove it
git rm --cached .env

# Add to .gitignore
echo ".env" >> .gitignore
```

**Mistake 5: Not logging errors**
```javascript
// Bad
app.listen(3000);

// Good
const server = app.listen(3000, () => {
  console.log('Server started on port 3000');
});

server.on('error', (err) => {
  if (err.code === 'EADDRINUSE') {
    console.error('Port 3000 is already in use');
    process.exit(1);
  }
  console.error('Server error:', err);
});
```

### Hands-On Practice Tasks

**Task 1: Process management**
```bash
# Start an app without a process manager
node app.js &
jobs

# Note the PID
echo $!

# Disconnect SSH and reconnect
# Is it still running?
ps aux | grep node

# Now try with PM2
pm2 start app.js
# Disconnect and reconnect
pm2 list  # Still there
```

**Task 2: Environment variables**
```bash
# Create a test app
cat > env-test.js << 'EOF'
console.log('DB_HOST:', process.env.DB_HOST);
console.log('DB_PASSWORD:', process.env.DB_PASSWORD);
EOF

# Run with inline env vars
DB_HOST=localhost DB_PASSWORD=secret node env-test.js

# Create .env file
cat > .env << 'EOF'
DB_HOST=localhost
DB_PASSWORD=secret
EOF

# Use dotenv
npm install dotenv

cat > env-test2.js << 'EOF'
require('dotenv').config();
console.log('DB_HOST:', process.env.DB_HOST);
console.log('DB_PASSWORD:', process.env.DB_PASSWORD);
EOF

node env-test2.js
```

**Task 3: systemd service**
```bash
# Create a simple app
cat > /opt/test-app.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  res.end('Running via systemd!\n');
});
server.listen(8080, () => console.log('Server started'));

process.on('SIGTERM', () => {
  console.log('SIGTERM received');
  server.close(() => process.exit(0));
});
EOF

# Create service file
cat > /etc/systemd/system/test-app.service << 'EOF'
[Unit]
Description=Test Application

[Service]
ExecStart=/usr/bin/node /opt/test-app.js
Restart=always
StandardOutput=journal

[Install]
WantedBy=multi-user.target
EOF

# Start it
systemctl daemon-reload
systemctl start test-app
systemctl status test-app

# View logs
journalctl -u test-app -f

# Test graceful shutdown
systemctl stop test-app
# Check logs - you should see "SIGTERM received"
```

### Mini Project: Multi-App Server

**Goal:** Run three different apps on one VPS, each managed properly.

```bash
# App 1: API server (port 3000)
# App 2: Admin dashboard (port 3001)
# App 3: Background worker (no port)

# Directory structure
mkdir -p /opt/{api,dashboard,worker}

# API app
cat > /opt/api/server.js << 'EOF'
const express = require('express');
const app = express();

app.get('/api/status', (req, res) => {
  res.json({ status: 'ok', service: 'api' });
});

app.listen(3000, () => console.log('API running on 3000'));
EOF

# Dashboard app
cat > /opt/dashboard/server.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Admin Dashboard</h1>');
});

app.listen(3001, () => console.log('Dashboard running on 3001'));
EOF

# Worker app
cat > /opt/worker/worker.js << 'EOF'
setInterval(() => {
  console.log(`[${new Date().toISOString()}] Processing job...`);
}, 5000);
EOF

# Install dependencies for each
cd /opt/api && npm install express
cd /opt/dashboard && npm install express

# Create systemd services for each
cat > /etc/systemd/system/api.service << 'EOF'
[Unit]
Description=API Service

[Service]
WorkingDirectory=/opt/api
ExecStart=/usr/bin/node server.js
Restart=always
StandardOutput=journal

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/system/dashboard.service << 'EOF'
[Unit]
Description=Dashboard Service

[Service]
WorkingDirectory=/opt/dashboard
ExecStart=/usr/bin/node server.js
Restart=always
StandardOutput=journal

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/system/worker.service << 'EOF'
[Unit]
Description=Background Worker

[Service]
WorkingDirectory=/opt/worker
ExecStart=/usr/bin/node worker.js
Restart=always
StandardOutput=journal

[Install]
WantedBy=multi-user.target
EOF

# Start everything
systemctl daemon-reload
systemctl enable --now api dashboard worker

# Check status
systemctl status api dashboard worker

# Test
curl localhost:3000/api/status
curl localhost:3001

# View all logs
journalctl -u api -u dashboard -u worker -f
```

Now you have three independent services running on one server, each properly managed and logged.

---

## 5. Reverse Proxies (Nginx Explained Properly)

### Why Reverse Proxies Exist

Right now, your app listens on port 3000. Users have to visit `http://yoursite.com:3000`. That's awkward.

You want:
- `https://yoursite.com` -> your app
- No port number in the URL
- HTTPS (port 443)
- Maybe multiple apps on the same server with different domains

You can't run your Node.js app on port 80/443 directly because:
1. Ports < 1024 require root privileges
2. You can only have one process per port
3. Your app doesn't know how to handle HTTPS certificates
4. You want to serve static files efficiently

**Enter the reverse proxy.**

**The Mental Model:**

```
Internet
    |
    v
Nginx (port 80/443)
    |
    ├─> Domain: api.example.com -> localhost:3000 (Node.js API)
    ├─> Domain: admin.example.com -> localhost:3001 (Admin dashboard)
    └─> Domain: static.example.com -> /var/www/static (Static files)
```

Nginx sits in front, accepts all requests on port 80/443, and routes them based on the domain name to the appropriate backend.

**Why "Reverse"?**

A forward proxy (like a VPN) sits between clients and the internet, acting on behalf of clients.

A reverse proxy sits between the internet and your servers, acting on behalf of servers.

```
Forward Proxy:
Client -> Proxy -> Internet

Reverse Proxy:
Client -> Proxy -> Your servers
```

### How Nginx Routes Traffic by Domain

Every HTTP request includes a `Host` header:

```
GET / HTTP/1.1
Host: api.example.com
```

Nginx reads this header and decides where to send the request.

**Basic Nginx Configuration:**

```nginx
# /etc/nginx/sites-available/myapp
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Let's break this down:

**`server { ... }`**: Defines a virtual host

**`listen 80`**: Listen on port 80 (HTTP)

**`server_name api.example.com`**: Only handle requests for this domain

**`location / { ... }`**: For all paths starting with /

**`proxy_pass http://localhost:3000`**: Forward to this backend

**Proxy headers**: These are critical. Let me explain each.

### Proxy Headers and Why They Matter

When Nginx forwards a request, it's making a new HTTP request to your backend. Without special headers, your app sees the request as coming from Nginx (127.0.0.1), not the original client.

**Critical Headers:**

```nginx
# Preserve the original Host header
proxy_set_header Host $host;

# Tell the backend the real client IP
proxy_set_header X-Real-IP $remote_addr;

# Full chain of proxies (if multiple proxies)
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

# Tell the backend the original protocol (http or https)
proxy_set_header X-Forwarded-Proto $scheme;

# For WebSockets
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection 'upgrade';
proxy_cache_bypass $http_upgrade;
```

**Why This Matters:**

Without these headers:
- Your app logs show all requests from 127.0.0.1
- Rate limiting doesn't work (everyone has the same IP)
- Redirect URLs break (app generates http:// URLs even when user used https://)

**In Your Application:**

```javascript
// Express.js
app.set('trust proxy', true);

// Now req.ip gives you the real client IP
app.get('/', (req, res) => {
  console.log('Client IP:', req.ip);  // Real IP, not 127.0.0.1
  console.log('Protocol:', req.protocol);  // https if X-Forwarded-Proto was https
});
```

### Virtual Hosts: Hosting Multiple Apps on One VPS

Here's the power of Nginx: multiple domains, one server.

**Scenario:**

You have:
- `mysite.com` - Main website (static files)
- `api.mysite.com` - REST API (Node.js on port 3000)
- `admin.mysite.com` - Admin panel (Node.js on port 3001)

One VPS, one IP, three sites.

**Nginx Configuration:**

```nginx
# /etc/nginx/sites-available/mysite
server {
    listen 80;
    server_name mysite.com www.mysite.com;
    root /var/www/mysite;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}

# /etc/nginx/sites-available/api
server {
    listen 80;
    server_name api.mysite.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# /etc/nginx/sites-available/admin
server {
    listen 80;
    server_name admin.mysite.com;

    # Restrict access to specific IPs
    allow 1.2.3.4;  # Your office IP
    deny all;

    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

**Enable These Sites:**

```bash
# Create symlinks to enable
ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/admin /etc/nginx/sites-enabled/

# Test configuration
nginx -t

# Reload
systemctl reload nginx
```

**How It Works:**

```
Request arrives: api.mysite.com
    |
    v
Nginx looks at Host header: "api.mysite.com"
    |
    v
Finds matching server block
    |
    v
Proxies to localhost:3000
    |
    v
Your Node.js app handles it
```

All three domains point to the same IP in DNS, but Nginx routes them differently based on the Host header.

### What Actually Happens Under the Hood

**Full Request Flow:**

```
1. Client makes request:
   GET https://api.mysite.com/users

2. DNS resolves api.mysite.com to your VPS IP

3. TCP connection to your VPS port 443

4. TLS handshake (Nginx handles this)

5. Client sends HTTP request:
   GET /users HTTP/1.1
   Host: api.mysite.com

6. Nginx receives request on port 443

7. Nginx decrypts TLS

8. Nginx checks server_name directives
   Finds match: api.mysite.com

9. Nginx creates new HTTP request:
   GET /users HTTP/1.1
   Host: api.mysite.com
   X-Real-IP: [client IP]
   X-Forwarded-For: [client IP]
   X-Forwarded-Proto: https

10. Nginx sends to localhost:3000 (plain HTTP)

11. Node.js app receives request

12. App processes, sends response

13. Nginx receives response

14. Nginx encrypts with TLS

15. Nginx sends to client
```

The key insight: **Backend communication is plain HTTP**. Nginx handles all the TLS complexity.

### Common Beginner Mistakes

**Mistake 1: Not reloading Nginx after config changes**
```bash
# Changed config but it's not working?
nginx -t  # Test config first
systemctl reload nginx  # Reload if test passes
```

**Mistake 2: Forgetting proxy headers**
```nginx
# Bad - app sees all requests from 127.0.0.1
location / {
    proxy_pass http://localhost:3000;
}

# Good
location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

**Mistake 3: Trailing slashes in proxy_pass**
```nginx
# These behave differently!

location /api/ {
    proxy_pass http://localhost:3000/;
}
# Request to /api/users -> proxied to /users

location /api/ {
    proxy_pass http://localhost:3000;
}
# Request to /api/users -> proxied to /api/users
```

**Mistake 4: Not checking Nginx error logs**
```bash
# Always check both
tail -f /var/log/nginx/access.log
tail -f /var/log/nginx/error.log
```

**Mistake 5: DNS not pointing to VPS**
```bash
# Nginx config looks good but site doesn't work
# Check DNS
dig api.mysite.com
# Does it point to your VPS IP?
```

### Hands-On Practice Tasks

**Task 1: Install and configure Nginx**
```bash
# Install
apt update
apt install nginx

# Check status
systemctl status nginx

# Visit your VPS IP in browser
# You should see "Welcome to nginx"

# Default config location
ls /etc/nginx/
ls /etc/nginx/sites-available/
ls /etc/nginx/sites-enabled/
```

**Task 2: Create a simple reverse proxy**
```bash
# Start a Node.js app on port 3000
node -e "require('http').createServer((req,res)=>res.end('Hello from Node\n')).listen(3000)"

# Test it locally
curl localhost:3000

# Create Nginx config
cat > /etc/nginx/sites-available/test << 'EOF'
server {
    listen 80;
    server_name _;  # Matches any domain

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
    }
}
EOF

# Enable it
ln -s /etc/nginx/sites-available/test /etc/nginx/sites-enabled/
rm /etc/nginx/sites-enabled/default  # Remove default

# Test and reload
nginx -t
systemctl reload nginx

# Visit your VPS IP in browser
# Should see "Hello from Node"
```

**Task 3: Multiple virtual hosts**
```bash
# Create two simple apps
node -e "require('http').createServer((req,res)=>res.end('App 1\n')).listen(3000)" &
node -e "require('http').createServer((req,res)=>res.end('App 2\n')).listen(3001)" &

# Create two Nginx configs
cat > /etc/nginx/sites-available/app1 << 'EOF'
server {
    listen 80;
    server_name app1.test;
    location / {
        proxy_pass http://localhost:3000;
    }
}
EOF

cat > /etc/nginx/sites-available/app2 << 'EOF'
server {
    listen 80;
    server_name app2.test;
    location / {
        proxy_pass http://localhost:3001;
    }
}
EOF

# Enable both
ln -s /etc/nginx/sites-available/app1 /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/app2 /etc/nginx/sites-enabled/

# Reload
nginx -t && systemctl reload nginx

# Test (since you don't have real DNS)
curl -H "Host: app1.test" http://your-vps-ip
curl -H "Host: app2.test" http://your-vps-ip

# Different responses based on Host header!
```

**Task 4: Debug a broken config**
```bash
# Create an intentionally broken config
cat > /etc/nginx/sites-available/broken << 'EOF'
server {
    listen 80;
    server_name broken.test;
    location / {
        proxy_pass http://localhost:9999;  # Nothing listening here
    }
}
EOF

ln -s /etc/nginx/sites-available/broken /etc/nginx/sites-enabled/
nginx -t && systemctl reload nginx

# Try to access it
curl -H "Host: broken.test" http://your-vps-ip

# Check Nginx error log
tail /var/log/nginx/error.log

# You'll see connection refused errors
# This teaches you how to debug proxy issues
```

### Mini Project: Multi-App Production Setup

**Goal:** Host multiple apps on one VPS with proper Nginx configuration.

**Apps:**
1. Main website (static files)
2. API (Node.js)
3. Admin dashboard (Node.js, IP-restricted)

```bash
# Create directory structure
mkdir -p /var/www/{main,api,admin}

# Main website (static)
cat > /var/www/main/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><title>My Site</title></head>
<body>
    <h1>Welcome to My Site</h1>
    <p>API: <a href="http://api.example.com">api.example.com</a></p>
</body>
</html>
EOF

# API app
cat > /var/www/api/server.js << 'EOF'
const express = require('express');
const app = express();

app.get('/api/data', (req, res) => {
    res.json({
        message: 'Data from API',
        clientIp: req.ip,
        protocol: req.protocol
    });
});

app.listen(3000, () => console.log('API on 3000'));
EOF

# Admin app
cat > /var/www/admin/server.js << 'EOF'
const express = require('express');
const app = express();

app.get('/', (req, res) => {
    res.send('<h1>Admin Dashboard</h1><p>Restricted access</p>');
});

app.listen(3001, () => console.log('Admin on 3001'));
EOF

# Install dependencies
cd /var/www/api && npm install express
cd /var/www/admin && npm install express

# Create systemd services
cat > /etc/systemd/system/api.service << 'EOF'
[Unit]
Description=API Service

[Service]
WorkingDirectory=/var/www/api
ExecStart=/usr/bin/node server.js
Restart=always

[Install]
WantedBy=multi-user.target
EOF

cat > /etc/systemd/system/admin.service << 'EOF'
[Unit]
Description=Admin Service

[Service]
WorkingDirectory=/var/www/admin
ExecStart=/usr/bin/node server.js
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Start services
systemctl daemon-reload
systemctl enable --now api admin

# Nginx configurations
cat > /etc/nginx/sites-available/main << 'EOF'
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/main;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Cache static assets
    location ~* \.(jpg|jpeg|png|gif|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
EOF

cat > /etc/nginx/sites-available/api << 'EOF'
server {
    listen 80;
    server_name api.example.com;

    # Rate limiting (100 requests per minute)
    limit_req_zone $binary_remote_addr zone=api_limit:10m rate=100r/m;

    location / {
        limit_req zone=api_limit burst=20;

        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # CORS headers
        add_header Access-Control-Allow-Origin *;
    }
}
EOF

cat > /etc/nginx/sites-available/admin << 'EOF'
server {
    listen 80;
    server_name admin.example.com;

    # IP whitelist (replace with your IP)
    allow 1.2.3.4;
    deny all;

    location / {
        proxy_pass http://localhost:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
EOF

# Enable all sites
ln -s /etc/nginx/sites-available/main /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/api /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/admin /etc/nginx/sites-enabled/

# Remove default
rm -f /etc/nginx/sites-enabled/default

# Test and reload
nginx -t
systemctl reload nginx

# Test with curl
curl -H "Host: example.com" http://your-vps-ip
curl -H "Host: api.example.com" http://your-vps-ip/api/data
curl -H "Host: admin.example.com" http://your-vps-ip
```

You now have a production-like multi-app setup with proper routing, rate limiting, IP restrictions, and static file serving.

---

## 6. HTTPS and Certificates

### How HTTPS Actually Works (The Handshake)

HTTP sends everything in plain text. Anyone between you and the server can read it. Passwords, API keys, private data - all visible.

HTTPS adds a layer of encryption using TLS (Transport Layer Security, formerly SSL).

**The TLS Handshake:**

```
Client                                     Server
  |                                           |
  |-------- ClientHello -------------------> |
  |  "I want to speak TLS 1.3"               |
  |  "I support these encryption algorithms"  |
  |                                           |
  | <------- ServerHello ------------------- |
  |         "Let's use TLS 1.3"              |
  |         "Let's use AES-256-GCM"          |
  |         "Here's my certificate"          |
  |                                           |
  | (Client verifies certificate)            |
  |                                           |
  |-------- KeyExchange -------------------> |
  |  (Encrypted with server's public key)    |
  |                                           |
  | <------- Finished ---------------------- |
  |  "Handshake complete"                    |
  |                                           |
  |======= Encrypted Communication =========|
```

**Step-by-Step:**

1. **Client Hello**: "I want HTTPS, here are my supported encryption methods"
2. **Server Hello**: "Let's use TLS 1.3 with AES-256. Here's my certificate to prove I'm really example.com"
3. **Certificate Verification**: Client checks:
   - Is this certificate signed by a trusted authority?
   - Does it match the domain name?
   - Is it still valid (not expired)?
4. **Key Exchange**: Both sides agree on encryption keys
5. **Encrypted Communication**: All data is encrypted with those keys

**What's in a Certificate?**

```
Certificate:
    Version: 3
    Serial Number: 12:34:56:78
    Issuer: Let's Encrypt Authority X3
    Validity:
        Not Before: Jan 1 2026
        Not After: Apr 1 2026
    Subject: example.com
    Subject Public Key: [PUBLIC KEY]
    Extensions:
        Subject Alternative Names:
            - example.com
            - www.example.com
    Signature: [SIGNATURE FROM LET'S ENCRYPT]
```

The certificate contains:
- **Domain name(s)**: What domains this covers
- **Public key**: Used to encrypt messages to the server
- **Signature**: Cryptographic proof from a Certificate Authority (CA)
- **Validity period**: Start and end dates

### Why Certificates Are Trusted

Your browser ships with a list of trusted Certificate Authorities (CAs). These are organizations like Let's Encrypt, DigiCert, GlobalSign.

**The Trust Chain:**

```
Root CA (built into browser)
    |
    └─> Intermediate CA
            |
            └─> Your Certificate
```

When you get a certificate:
1. A CA verifies you control the domain
2. CA signs your certificate with their private key
3. Anyone can verify the signature with CA's public key (built into browsers)
4. If signature is valid, certificate is trusted

**Let's Encrypt made this free and automated.**

Before Let's Encrypt, certificates cost $50-200/year and involved manual processes. Let's Encrypt certificates are:
- Free
- Valid for 90 days (forces automation)
- Issued automatically via ACME protocol

### Let's Encrypt and Certbot Internals

**Certbot** is the official Let's Encrypt client. Here's what it does:

**The ACME Protocol:**

```
1. You run: certbot --nginx -d example.com

2. Certbot asks Let's Encrypt:
   "I want a certificate for example.com"

3. Let's Encrypt responds:
   "Prove you control example.com by completing a challenge"

4. Challenge (HTTP-01):
   "Place this file at http://example.com/.well-known/acme-challenge/[random-token]"

5. Certbot:
   - Creates the file in your web root
   - Tells Let's Encrypt "ready"

6. Let's Encrypt:
   - Tries to fetch the file from example.com
   - If it matches, you proved control

7. Let's Encrypt issues certificate

8. Certbot:
   - Downloads certificate and private key
   - Installs in /etc/letsencrypt/
   - Modifies Nginx config to use HTTPS
   - Reloads Nginx
```

**File Locations:**

```
/etc/letsencrypt/
    live/
        example.com/
            cert.pem           # Your certificate
            chain.pem          # Intermediate certificates
            fullchain.pem      # cert.pem + chain.pem (use this)
            privkey.pem        # Private key (keep secret!)
    archive/                   # Old certificates
    renewal/                   # Renewal configuration
        example.com.conf
```

**Installation:**

```bash
# Install Certbot
apt update
apt install certbot python3-certbot-nginx

# Get a certificate and auto-configure Nginx
certbot --nginx -d example.com -d www.example.com

# Certbot will:
# 1. Modify your Nginx config
# 2. Set up HTTPS on port 443
# 3. Redirect HTTP to HTTPS
# 4. Set up auto-renewal
```

**After Certbot runs, your Nginx config looks like:**

```nginx
server {
    listen 80;
    server_name example.com www.example.com;
    return 301 https://$server_name$request_uri;  # Redirect to HTTPS
}

server {
    listen 443 ssl;
    server_name example.com www.example.com;

    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-Proto https;
    }
}
```

### Auto-Renewal Logic

Let's Encrypt certificates expire after 90 days. Certbot sets up automatic renewal.

**How Auto-Renewal Works:**

```bash
# Certbot creates a systemd timer
systemctl list-timers | grep certbot

# The timer runs twice daily
# It checks if certificates are <30 days from expiry
# If so, it renews them

# See the renewal configuration
cat /etc/letsencrypt/renewal/example.com.conf
```

**Test Renewal:**

```bash
# Dry run (doesn't actually renew)
certbot renew --dry-run

# Manual renewal (if needed)
certbot renew
```

**Renewal Process:**

```
1. certbot renew runs (via systemd timer)
2. For each certificate:
   - Is it <30 days from expiry?
   - If yes, request renewal from Let's Encrypt
   - Same challenge process
   - Download new certificate
   - Replace old files in /etc/letsencrypt/live/
3. Reload Nginx (seamless, no downtime)
```

**Hooks:**

You can run commands before/after renewal:

```bash
# /etc/letsencrypt/renewal/example.com.conf
[renewalparams]
pre_hook = echo "About to renew"
post_hook = systemctl reload nginx
```

### What Actually Happens Under the Hood

**When a browser connects to your HTTPS site:**

```
1. DNS resolution: example.com -> your VPS IP

2. TCP connection to port 443

3. TLS Handshake:
   Client: ClientHello
   Server: ServerHello + Certificate
   Client: Verifies certificate
       - Check signature (is it signed by trusted CA?)
       - Check domain name (does CN/SAN match?)
       - Check dates (is it valid now?)
   Both: Exchange keys

4. Encrypted tunnel established

5. HTTP request sent through tunnel:
   GET / HTTP/1.1
   Host: example.com
   (encrypted)

6. Nginx receives encrypted request
   Decrypts using private key
   Proxies to backend (plain HTTP on localhost)

7. Backend responds

8. Nginx encrypts response
   Sends back to client

9. Client decrypts, displays page
```

**The Critical Files:**

- `fullchain.pem`: Public certificate (safe to share)
- `privkey.pem`: Private key (NEVER share, NEVER commit to git)

If someone gets your private key, they can:
- Decrypt past traffic (if they recorded it)
- Impersonate your server

Protect it:
```bash
chmod 600 /etc/letsencrypt/live/example.com/privkey.pem
chown root:root /etc/letsencrypt/live/example.com/privkey.pem
```

### Common Beginner Mistakes

**Mistake 1: Forgetting to open port 443**
```bash
# HTTPS won't work if firewall blocks it
ufw allow 443/tcp
```

**Mistake 2: Wrong DNS before running Certbot**
```bash
# Certbot needs to verify domain ownership
# DNS must point to your VPS BEFORE running certbot
dig example.com  # Should show your VPS IP
```

**Mistake 3: Not setting up auto-renewal**
```bash
# Check if timer is active
systemctl status certbot.timer

# If not, enable it
systemctl enable certbot.timer
```

**Mistake 4: Committing private keys to git**
```bash
# NEVER do this
# Add to .gitignore
/etc/letsencrypt/
*.pem
*.key
```

**Mistake 5: Mixed content warnings**
```html
<!-- Bad: Loading HTTP resource on HTTPS page -->
<script src="http://example.com/app.js"></script>

<!-- Good: Use HTTPS or protocol-relative -->
<script src="https://example.com/app.js"></script>
<script src="//example.com/app.js"></script>
```

### Hands-On Practice Tasks

**Task 1: Inspect a certificate**
```bash
# Check any site's certificate
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -text

# Check expiry date
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -dates

# Check what domains it covers
echo | openssl s_client -connect google.com:443 2>/dev/null | openssl x509 -noout -text | grep "DNS:"
```

**Task 2: Set up Let's Encrypt**
```bash
# Prerequisites:
# 1. Domain pointing to your VPS
# 2. Nginx running
# 3. Port 80 open

# Install Certbot
apt install certbot python3-certbot-nginx

# Get certificate
certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Certbot asks for email (for renewal notifications)
# Asks to agree to ToS
# Asks about HTTP to HTTPS redirect (choose Yes)

# Test your site
curl https://yourdomain.com

# Check certificate
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com
```

**Task 3: Test auto-renewal**
```bash
# Check renewal configuration
cat /etc/letsencrypt/renewal/yourdomain.com.conf

# Test renewal (doesn't actually renew)
certbot renew --dry-run

# Check systemd timer
systemctl list-timers | grep certbot

# See when it last ran
journalctl -u certbot -n 50
```

**Task 4: Understand the TLS handshake**
```bash
# See the full TLS handshake
openssl s_client -connect google.com:443 -showcerts

# You'll see:
# - Supported protocols and ciphers
# - Certificate chain
# - Session information

# Test specific TLS version
openssl s_client -connect google.com:443 -tls1_3
```

### Mini Project: Secure Multi-Domain Setup

**Goal:** Set up HTTPS for multiple domains/subdomains on one VPS.

**Scenario:**
- `mysite.com` - Main site
- `api.mysite.com` - API
- `admin.mysite.com` - Admin panel

```bash
# Prerequisite: All domains in DNS pointing to your VPS

# Set up apps (from previous section)
# Assuming you have Nginx configs at:
# /etc/nginx/sites-available/mysite
# /etc/nginx/sites-available/api
# /etc/nginx/sites-available/admin

# Install Certbot
apt install certbot python3-certbot-nginx

# Get certificates for all domains in one command
certbot --nginx \
  -d mysite.com \
  -d www.mysite.com \
  -d api.mysite.com \
  -d admin.mysite.com

# Certbot will:
# 1. Verify all domains
# 2. Get a single certificate covering all
# 3. Update all Nginx configs
# 4. Set up redirects

# Check your Nginx config now
cat /etc/nginx/sites-available/mysite

# You'll see ssl_certificate lines added

# Test all sites
curl https://mysite.com
curl https://www.mysite.com
curl https://api.mysite.com/api/data
curl https://admin.mysite.com

# Check certificate details
echo | openssl s_client -connect mysite.com:443 2>/dev/null | openssl x509 -noout -text | grep DNS:
# Should show all four domains

# Set up monitoring for expiry
cat > /etc/cron.daily/check-cert-expiry << 'EOF'
#!/bin/bash
DOMAIN="mysite.com"
EXPIRY=$(echo | openssl s_client -connect $DOMAIN:443 -servername $DOMAIN 2>/dev/null | openssl x509 -noout -enddate | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
NOW_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $NOW_EPOCH) / 86400 ))

if [ $DAYS_LEFT -lt 7 ]; then
    echo "WARNING: Certificate expires in $DAYS_LEFT days!" | mail -s "Cert Expiry Warning" you@email.com
fi
EOF

chmod +x /etc/cron.daily/check-cert-expiry
```

Now you have:
- HTTPS on all domains
- Automatic renewal
- HTTP to HTTPS redirects
- Certificate expiry monitoring

---

## 7. Git for Production Systems

### Git Beyond Just Pushing Code

You know `git add`, `git commit`, `git push`. But in production, Git is more than version control - it's your deployment mechanism, your audit trail, and your rollback strategy.

**The Production Mindset:**

Development Git:
- Experiment freely
- Commit messages like "fix stuff"
- Force push to your branch
- Branches are temporary

Production Git:
- Every commit is traceable
- Commit messages are documentation
- Never force push to main
- Branches are deployment points

### Branching for Deployment

**A Common Strategy:**

```
main          (production code, always deployable)
    |
    ├── develop      (integration branch, deployed to staging)
    |       |
    |       ├── feature/user-auth
    |       ├── feature/api-v2
    |       └── bugfix/login-error
    |
    └── hotfix/security-patch  (urgent fixes, merged to main)
```

**The Flow:**

```bash
# Feature development
git checkout -b feature/new-api develop
# ... work work work ...
git commit -m "Add new API endpoint"
git push origin feature/new-api

# Merge to develop
git checkout develop
git merge feature/new-api
git push origin develop

# Deploy to staging
# (trigger CI/CD or manual deploy)

# After testing, merge to main
git checkout main
git merge develop
git push origin main

# Tag the release
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin v1.2.0

# Deploy to production
# (CI/CD deploys tagged versions)
```

**Hotfix Process:**

```bash
# Critical bug in production
git checkout -b hotfix/security-patch main

# Fix it
git commit -m "Fix XSS vulnerability in user input"

# Merge to main
git checkout main
git merge hotfix/security-patch
git tag -a v1.2.1 -m "Hotfix: XSS vulnerability"
git push origin main --tags

# Also merge to develop so fix isn't lost
git checkout develop
git merge hotfix/security-patch
git push origin develop

# Deploy main to production
```

### Tags, Releases, and Rollbacks

**Tags are immutable pointers to commits.** Use them to mark releases.

```bash
# List tags
git tag

# Create annotated tag (recommended)
git tag -a v1.0.0 -m "First production release"

# Push tags
git push origin v1.0.0
# Or all tags
git push origin --tags

# Check out a specific version
git checkout v1.0.0

# See what changed between versions
git diff v1.0.0 v1.1.0

# See commit log between versions
git log v1.0.0..v1.1.0
```

**Semantic Versioning:**

```
v1.2.3
│ │ └─ Patch: Bug fixes, no new features
│ └─── Minor: New features, backward compatible
└───── Major: Breaking changes
```

**Rollback Strategy:**

When production breaks, you need to roll back fast.

**Option 1: Deploy previous tag**
```bash
# On your VPS
cd /opt/myapp

# Check current version
git describe --tags

# See available tags
git tag

# Check out previous version
git checkout v1.2.0

# Restart app
systemctl restart myapp

# You're now running the old version
```

**Option 2: Revert commit**
```bash
# This broke production
git log --oneline -5
# abc123 Deploy new feature
# def456 Update dependencies

# Create a new commit that undoes abc123
git revert abc123

# Push
git push origin main

# Deploy (this creates new history, doesn't rewrite)
```

**Option 3: Reset (dangerous, only for dire situations)**
```bash
# This rewrites history - use only if you must
git reset --hard v1.2.0
git push origin main --force

# This is dangerous because:
# - Rewrites history
# - Can lose commits
# - Causes issues for other devs
```

### Git Hooks and Webhooks

**Git Hooks**: Scripts that run locally during git operations

```bash
# Location
ls .git/hooks/
# pre-commit, pre-push, post-merge, etc.

# Example: pre-commit hook to run tests
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
echo "Running tests before commit..."
npm test
if [ $? -ne 0 ]; then
    echo "Tests failed, commit aborted"
    exit 1
fi
EOF

chmod +x .git/hooks/pre-commit

# Now git commit will fail if tests fail
```

**Common Hooks:**

- `pre-commit`: Run linters, formatters
- `pre-push`: Run tests
- `post-merge`: Install dependencies
- `post-checkout`: Clean up build artifacts

**Webhooks**: HTTP callbacks that notify external services when you push

**Example: Auto-deploy on push**

```bash
# Set up webhook on GitHub/GitLab
# Points to: https://yourserver.com/webhook

# On your server, create webhook endpoint
cat > /opt/deploy/webhook-handler.js << 'EOF'
const express = require('express');
const { exec } = require('child_process');
const crypto = require('crypto');

const app = express();
app.use(express.json());

const SECRET = process.env.WEBHOOK_SECRET;

app.post('/webhook', (req, res) => {
    // Verify signature (GitHub sends this)
    const signature = req.headers['x-hub-signature-256'];
    const hash = 'sha256=' + crypto
        .createHmac('sha256', SECRET)
        .update(JSON.stringify(req.body))
        .digest('hex');

    if (signature !== hash) {
        return res.status(401).send('Invalid signature');
    }

    // Only deploy on push to main
    if (req.body.ref === 'refs/heads/main') {
        console.log('Push to main detected, deploying...');

        exec('/opt/deploy/deploy.sh', (error, stdout, stderr) => {
            if (error) {
                console.error('Deploy failed:', error);
                return res.status(500).send('Deploy failed');
            }
            console.log('Deploy successful');
            res.send('Deployed');
        });
    } else {
        res.send('Not main branch, ignoring');
    }
});

app.listen(3002, () => console.log('Webhook handler on 3002'));
EOF

# Deploy script
cat > /opt/deploy/deploy.sh << 'EOF'
#!/bin/bash
set -e  # Exit on error

cd /opt/myapp
git pull origin main
npm install --production
systemctl restart myapp

echo "Deploy completed at $(date)" >> /var/log/deploy.log
EOF

chmod +x /opt/deploy/deploy.sh
```

Now when you `git push origin main`, GitHub sends a webhook, your server receives it, and auto-deploys.

### What Actually Happens Under the Hood

**When you git push:**

```
1. Git bundles your new commits
2. Connects to remote server (SSH or HTTPS)
3. Sends commit objects, trees, and blobs
4. Remote server validates
5. Remote updates refs (branches/tags)
6. If webhook configured, remote sends HTTP POST to your URL
7. Your webhook handler receives notification
8. Runs deploy script
```

**Git Storage:**

Git stores everything in `.git/`:

```
.git/
    objects/      # All commits, files, trees (content-addressed storage)
    refs/
        heads/    # Branches
        tags/     # Tags
        remotes/  # Remote branches
    HEAD          # Points to current branch
    config        # Repository config
    hooks/        # Hook scripts
```

Every commit, file, and tree is a SHA-1 hash. Git is essentially a content-addressed database.

```bash
# See what a commit really is
git cat-file -p HEAD

# Output:
# tree abc123...
# parent def456...
# author Name <email>
# committer Name <email>
#
# Commit message
```

### Common Beginner Mistakes

**Mistake 1: Deploying uncommitted changes**
```bash
# On VPS, you edit files directly
vim /opt/myapp/config.js

# This works, but:
# - Not in version control
# - Next git pull overwrites it
# - No one else knows about the change

# Right way:
# Make changes in dev, commit, push, deploy
```

**Mistake 2: Not using tags**
```bash
# Without tags, you can't easily say "deploy v1.2.0"
# You have to use commit hashes (error-prone)

# Use tags for releases
git tag -a v1.2.0 -m "Release notes"
```

**Mistake 3: Force pushing to main**
```bash
# Never do this
git push origin main --force

# This can lose other people's commits
# In production, this can cause chaos
```

**Mistake 4: Large files in Git**
```bash
# Don't commit build artifacts
git add dist/
git add node_modules/
git add *.log

# Use .gitignore
cat > .gitignore << 'EOF'
node_modules/
dist/
*.log
.env
EOF
```

**Mistake 5: Not having a rollback plan**
```bash
# Deployment breaks production
# You panic and try to fix forward
# Takes an hour

# Better:
# Tag releases, test rollback procedure
# Can rollback in 30 seconds
```

### Hands-On Practice Tasks

**Task 1: Set up a deployment branch strategy**
```bash
# Initialize a test repo
mkdir test-deploy
cd test-deploy
git init

# Create develop branch
git checkout -b develop

# Make some commits
echo "v1" > app.js
git add app.js
git commit -m "Initial version"

# Create a feature branch
git checkout -b feature/new-stuff
echo "v2" > app.js
git commit -am "Add new feature"

# Merge to develop
git checkout develop
git merge feature/new-stuff

# Tag and merge to main
git checkout -b main
git merge develop
git tag -a v1.0.0 -m "First release"

# See the graph
git log --oneline --graph --all
```

**Task 2: Practice rollback**
```bash
# Make a breaking change
echo "broken" > app.js
git commit -am "This breaks everything"
git tag -a v1.1.0 -m "Broken release"

# Oh no, production is broken!
# Rollback to previous tag
git checkout v1.0.0

# Check the file
cat app.js  # Back to v2

# Or revert the commit
git checkout main
git revert HEAD
cat app.js  # Fixed, but history preserved
```

**Task 3: Set up a pre-commit hook**
```bash
# In your project
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash

# Check for console.log in staged files
if git diff --cached --name-only | xargs grep -n "console.log" 2>/dev/null; then
    echo "Error: Found console.log statements"
    echo "Please remove them before committing"
    exit 1
fi

echo "Pre-commit checks passed"
EOF

chmod +x .git/hooks/pre-commit

# Test it
echo 'console.log("test")' > test.js
git add test.js
git commit -m "Test"  # Will fail!

# Remove console.log and try again
echo 'const x = 1;' > test.js
git add test.js
git commit -m "Test"  # Passes
```

**Task 4: Understand git internals**
```bash
# Create a commit
echo "test" > file.txt
git add file.txt
git commit -m "Test commit"

# Get the commit hash
COMMIT=$(git rev-parse HEAD)

# Look inside
git cat-file -p $COMMIT

# Get the tree hash from output
TREE=$(git cat-file -p $COMMIT | grep tree | awk '{print $2}')

# Look inside the tree
git cat-file -p $TREE

# Get the blob hash
BLOB=$(git cat-file -p $TREE | awk '{print $3}')

# Look inside the blob
git cat-file -p $BLOB  # This is your file content!
```

### Mini Project: Automated Deployment System

**Goal:** Set up a complete git-based deployment with automatic deploys, tagging, and rollback capability.

```bash
# Set up the repository structure
mkdir ~/deploy-demo
cd ~/deploy-demo
git init

# Create .gitignore
cat > .gitignore << 'EOF'
node_modules/
.env
*.log
EOF

# Create a simple app
cat > app.js << 'EOF'
const http = require('http');
const fs = require('fs');

const VERSION = process.env.VERSION || 'unknown';

const server = http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`App version: ${VERSION}\n`);
});

server.listen(3000, () => {
    console.log(`Version ${VERSION} running on port 3000`);
});

process.on('SIGTERM', () => {
    console.log('SIGTERM received, shutting down');
    server.close(() => process.exit(0));
});
EOF

# Initial commit
git add .
git commit -m "Initial commit"

# Tag as v1.0.0
git tag -a v1.0.0 -m "Version 1.0.0"

# Create deployment script
cat > deploy.sh << 'EOF'
#!/bin/bash
set -e

DEPLOY_DIR="/opt/myapp"
REPO_URL="."  # Use local repo for demo

# Parse arguments
VERSION=${1:-main}

echo "Deploying version: $VERSION"

# Create deploy directory if needed
mkdir -p $DEPLOY_DIR

# If first deploy, clone
if [ ! -d "$DEPLOY_DIR/.git" ]; then
    git clone $REPO_URL $DEPLOY_DIR
fi

cd $DEPLOY_DIR

# Fetch latest
git fetch --all --tags

# Checkout requested version
git checkout $VERSION

# Record deployment
echo "Deployed $VERSION at $(date)" >> /var/log/deploy.log

# Set version in environment
CURRENT_VERSION=$(git describe --tags)
export VERSION=$CURRENT_VERSION

# Restart service
systemctl restart myapp || echo "Service not set up yet"

echo "✓ Deployed $CURRENT_VERSION successfully"
EOF

chmod +x deploy.sh

# Create systemd service
cat > myapp.service << 'EOF'
[Unit]
Description=My Application
After=network.target

[Service]
Type=simple
WorkingDirectory=/opt/myapp
Environment="VERSION=unknown"
ExecStart=/usr/bin/node app.js
Restart=always

[Install]
WantedBy=multi-user.target
EOF

# Create rollback script
cat > rollback.sh << 'EOF'
#!/bin/bash
DEPLOY_DIR="/opt/myapp"

cd $DEPLOY_DIR

# Get current and previous tags
CURRENT=$(git describe --tags)
PREVIOUS=$(git describe --tags $CURRENT^ --abbrev=0)

echo "Current version: $CURRENT"
echo "Rolling back to: $PREVIOUS"

read -p "Continue? (y/n) " -n 1 -r
echo
if [[ $REPLY =~ ^[Yy]$ ]]; then
    git checkout $PREVIOUS
    systemctl restart myapp
    echo "✓ Rolled back to $PREVIOUS"
fi
EOF

chmod +x rollback.sh

# Commit everything
git add .
git commit -m "Add deployment scripts"
git tag -a v1.1.0 -m "Add deployment automation"

# Test deployment
./deploy.sh v1.0.0

# Check it works
curl localhost:3000

# Deploy newer version
./deploy.sh v1.1.0
curl localhost:3000

# Test rollback
./rollback.sh
```

You now have:
- Version-controlled deployments
- Easy rollback capability
- Deployment audit trail
- Tagged releases

---

## 8. CI/CD from First Principles

### What CI/CD Really Automates

CI/CD is automation for your deployment pipeline. Let's break down what it actually does.

**Continuous Integration (CI):**
Every time you push code, automatically:
1. Run tests
2. Run linters
3. Build the application
4. Report success/failure

**Continuous Deployment (CD):**
If tests pass, automatically:
1. Deploy to staging
2. Run integration tests
3. Deploy to production (or wait for approval)

**Why This Matters:**

Without CI/CD:
```
Developer pushes code
→ Another dev pulls and finds it breaks tests
→ Takes 2 hours to debug
→ Blocks everyone else
→ Frustration
```

With CI/CD:
```
Developer pushes code
→ CI runs tests automatically in 2 minutes
→ Tests fail, developer gets notification
→ Developer fixes before anyone else is affected
→ Push again, tests pass
→ Auto-deploys to staging
→ Everyone happy
```

### Build vs Deploy Difference

**Build**: Transform source code into something runnable

```bash
# Example builds:
npm run build          # React app: JSX → optimized JS
go build              # Go: source → binary
docker build          # Dockerfile → container image
tsc                   # TypeScript → JavaScript
```

**Deploy**: Put the built artifact somewhere it can run

```bash
# Example deploys:
scp dist/* user@server:/var/www/
docker push myimage:latest
kubectl apply -f deployment.yaml
git pull && systemctl restart app
```

**The Pipeline:**

```
Code → Build → Test → Package → Deploy

Example:
1. Push to GitHub
2. GitHub Actions triggers
3. Checks out code
4. Installs dependencies (npm install)
5. Runs tests (npm test)
6. Builds (npm run build)
7. Creates Docker image
8. Pushes to registry
9. SSHs to VPS
10. Pulls new image
11. Restarts container
```

### GitHub Actions Deep Dive

GitHub Actions runs workflows in response to events (push, PR, schedule, etc.).

**Anatomy of a Workflow:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

# When to run
on:
  push:
    branches: [ main ]

# Jobs to run
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup Node
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test

  deploy:
    needs: test  # Only run if test succeeds
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy to VPS
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        run: |
          echo "$SSH_PRIVATE_KEY" > key.pem
          chmod 600 key.pem
          ssh -i key.pem -o StrictHostKeyChecking=no user@your-vps.com '
            cd /opt/myapp &&
            git pull origin main &&
            npm install --production &&
            systemctl restart myapp
          '
```

**Let's break this down:**

**`on: push: branches: [main]`**: Run when code is pushed to main

**`runs-on: ubuntu-latest`**: GitHub provides a clean Ubuntu VM for each run

**`uses: actions/checkout@v3`**: Clones your repo into the VM

**`run: npm test`**: Executes the command

**`needs: test`**: deploy job waits for test job to succeed

**`secrets.SSH_PRIVATE_KEY`**: Stored securely in GitHub settings

**The Full Flow:**

```
1. You push to main

2. GitHub detects push event

3. Spins up Ubuntu VM

4. Runs test job:
   - Checkout code
   - Install Node
   - Install dependencies
   - Run tests

5. If tests pass, runs deploy job:
   - Checkout code again (new job = new VM)
   - SSH to your VPS
   - Pull latest code
   - Install deps
   - Restart service

6. If any step fails, job stops and you get notified

7. VM is destroyed (nothing persists between runs)
```

### Secrets in CI/CD

**Never put credentials in your code or workflow files.**

**Bad:**
```yaml
- name: Deploy
  run: |
    ssh user@server.com password123 'deploy'
```

**Good:**
```yaml
- name: Deploy
  env:
    SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
  run: |
    echo "$SSH_KEY" > key.pem
    ssh -i key.pem user@server.com 'deploy'
```

**Setting Up Secrets:**

```bash
# On your VPS, create a deploy key
ssh-keygen -t ed25519 -f ~/.ssh/deploy_key -N ""

# Add public key to authorized_keys
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys

# Copy private key
cat ~/.ssh/deploy_key
# Copy this output

# In GitHub:
# Settings → Secrets → New repository secret
# Name: SSH_PRIVATE_KEY
# Value: [paste private key]
```

**Common Secrets:**

- SSH keys
- API tokens
- Database passwords
- Docker registry credentials
- Cloud provider credentials

**Environment Variables in CI:**

```yaml
jobs:
  deploy:
    env:
      # Available to all steps in this job
      NODE_ENV: production
      DATABASE_URL: ${{ secrets.DATABASE_URL }}

    steps:
      - name: Deploy
        env:
          # Available only to this step
          API_KEY: ${{ secrets.API_KEY }}
        run: |
          echo "Deploying to $NODE_ENV"
          # API_KEY is available here
```

### Deploying to a VPS Safely

**The Safe Deployment Pattern:**

```yaml
name: Deploy

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Deploy to VPS
        env:
          SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          HOST: ${{ secrets.VPS_HOST }}
          USER: ${{ secrets.VPS_USER }}
        run: |
          # Setup SSH
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key

          # Deploy via SSH
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no $USER@$HOST << 'ENDSSH'
            set -e  # Exit on error

            # Pull latest code
            cd /opt/myapp
            git pull origin main

            # Install dependencies
            npm ci --production

            # Run migrations (if any)
            npm run migrate

            # Health check
            curl -f http://localhost:3000/health || exit 1

            # Restart service
            systemctl restart myapp

            # Wait for it to start
            sleep 5

            # Verify it's running
            systemctl is-active myapp || exit 1

            echo "✓ Deploy successful"
ENDSSH
```

**Key Safety Features:**

1. **`set -e`**: Exit on any error (don't continue if something fails)
2. **Health check**: Make sure app responds before restarting
3. **Verification**: Check service is actually running after restart
4. **`npm ci`**: Clean install (reproducible, faster than `npm install`)

**Zero-Downtime Deployment:**

```bash
# Use a process manager that supports reload
pm2 reload myapp

# Or with systemd and multiple instances:
# Start new instance
systemctl start myapp@2

# Health check new instance
curl localhost:3001/health

# Stop old instance
systemctl stop myapp@1
```

### What Actually Happens Under the Hood

**When GitHub Actions runs:**

```
1. Event triggers workflow (push, PR, schedule, etc.)

2. GitHub queues job

3. GitHub allocates runner (VM):
   - Fresh Ubuntu VM
   - Pre-installed tools (git, docker, Node, Python, etc.)
   - Isolated (nothing persists between runs)

4. Runner executes steps sequentially:
   - Each step is a new shell process
   - Environment variables persist within job
   - Failures stop execution unless continue-on-error: true

5. Artifacts:
   - Actions can upload artifacts (build outputs, logs, etc.)
   - Downloaded by later jobs or kept for 90 days

6. After job completes:
   - VM is destroyed
   - Logs are saved
   - Status reported (success/failure)

7. Notifications sent (if configured)
```

**Workflow Execution Model:**

```yaml
jobs:
  job1:
    runs-on: ubuntu-latest
    steps: [...]

  job2:
    runs-on: ubuntu-latest
    needs: job1  # Wait for job1
    steps: [...]

  job3:
    runs-on: ubuntu-latest
    needs: [job1, job2]  # Wait for both
    steps: [...]
```

Jobs run in parallel by default. Use `needs` to create dependencies.

### Common Beginner Mistakes

**Mistake 1: Not testing locally first**
```yaml
# Don't iterate by pushing and waiting for CI
# Test locally:
# - Run tests: npm test
# - Run build: npm run build
# - Test deploy script on your VPS manually

# Then put it in CI
```

**Mistake 2: Secrets in logs**
```yaml
# Bad - secrets will appear in logs
- run: echo "API key is $API_KEY"

# Good - GitHub auto-masks secrets, but still don't print them
- run: curl -H "Authorization: $API_KEY" api.com
```

**Mistake 3: No rollback strategy**
```yaml
# Add rollback capability
- name: Deploy
  id: deploy
  run: ./deploy.sh

- name: Rollback on failure
  if: failure() && steps.deploy.outcome == 'failure'
  run: ./rollback.sh
```

**Mistake 4: Deploying on every commit**
```yaml
# Bad - deploys even if tests fail
on:
  push:
    branches: [ main ]
jobs:
  deploy:
    steps:
      - run: ./deploy.sh

# Good - only deploy if tests pass
jobs:
  test:
    steps:
      - run: npm test
  deploy:
    needs: test  # Only runs if test succeeds
    steps:
      - run: ./deploy.sh
```

**Mistake 5: Not handling dependencies properly**
```yaml
# Bad - Might use different versions between runs
- run: npm install

# Good - Reproducible installs
- run: npm ci  # Uses package-lock.json exactly
```

### Hands-On Practice Tasks

**Task 1: Create a simple GitHub Actions workflow**
```bash
# In your repo
mkdir -p .github/workflows

cat > .github/workflows/test.yml << 'EOF'
name: Run Tests

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'

    - name: Install dependencies
      run: npm ci

    - name: Run linter
      run: npm run lint

    - name: Run tests
      run: npm test

    - name: Build
      run: npm run build

    - name: Upload build artifacts
      uses: actions/upload-artifact@v3
      with:
        name: build
        path: dist/
EOF

git add .github/workflows/test.yml
git commit -m "Add CI workflow"
git push

# Check GitHub Actions tab to see it run
```

**Task 2: Set up deployment workflow**
```bash
# First, set up SSH key (on your VPS)
ssh-keygen -t ed25519 -f ~/.ssh/github_deploy -N ""
cat ~/.ssh/github_deploy.pub >> ~/.ssh/authorized_keys

# Copy private key
cat ~/.ssh/github_deploy

# Add to GitHub Secrets:
# SSH_PRIVATE_KEY: [private key]
# VPS_HOST: your-vps-ip
# VPS_USER: your-username

# Create deploy workflow
cat > .github/workflows/deploy.yml << 'EOF'
name: Deploy to VPS

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Deploy to VPS
      env:
        SSH_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
        HOST: ${{ secrets.VPS_HOST }}
        USER: ${{ secrets.VPS_USER }}
      run: |
        mkdir -p ~/.ssh
        echo "$SSH_KEY" > ~/.ssh/key
        chmod 600 ~/.ssh/key

        ssh -i ~/.ssh/key -o StrictHostKeyChecking=no $USER@$HOST '
          cd /opt/myapp &&
          git pull origin main &&
          npm ci --production &&
          systemctl restart myapp &&
          echo "Deployed at $(date)"
        '
EOF

git add .github/workflows/deploy.yml
git commit -m "Add deploy workflow"
git push

# Watch it deploy automatically!
```

**Task 3: Add deployment notifications**
```bash
# Get a webhook URL (Slack, Discord, etc.)
# Add to secrets as SLACK_WEBHOOK

cat > .github/workflows/deploy-with-notifications.yml << 'EOF'
name: Deploy with Notifications

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v3

    - name: Notify deployment started
      run: |
        curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
          -H 'Content-Type: application/json' \
          -d '{"text":"🚀 Deployment started for commit ${{ github.sha }}"}'

    - name: Deploy
      id: deploy
      run: ./deploy.sh

    - name: Notify deployment success
      if: success()
      run: |
        curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
          -H 'Content-Type: application/json' \
          -d '{"text":"✅ Deployment successful!"}'

    - name: Notify deployment failure
      if: failure()
      run: |
        curl -X POST ${{ secrets.SLACK_WEBHOOK }} \
          -H 'Content-Type: application/json' \
          -d '{"text":"❌ Deployment failed!"}'
EOF
```

### Mini Project: Complete CI/CD Pipeline

**Goal:** Build a complete CI/CD pipeline that tests, builds, deploys to staging, runs smoke tests, and deploys to production with approval.

```yaml
# .github/workflows/full-pipeline.yml
name: Full CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  # Job 1: Run tests
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run unit tests
        run: npm test

      - name: Run security audit
        run: npm audit --production

  # Job 2: Build application
  build:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist/
          retention-days: 7

  # Job 3: Deploy to staging
  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - uses: actions/checkout@v3

      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/

      - name: Deploy to staging
        env:
          SSH_KEY: ${{ secrets.STAGING_SSH_KEY }}
          HOST: ${{ secrets.STAGING_HOST }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/key
          chmod 600 ~/.ssh/key

          # Copy files to staging
          rsync -avz -e "ssh -i ~/.ssh/key -o StrictHostKeyChecking=no" \
            dist/ deploy@$HOST:/opt/staging/

          # Restart staging
          ssh -i ~/.ssh/key -o StrictHostKeyChecking=no deploy@$HOST \
            'systemctl restart myapp-staging'

  # Job 4: Run smoke tests on staging
  smoke-test:
    needs: deploy-staging
    runs-on: ubuntu-latest
    steps:
      - name: Health check
        run: |
          for i in {1..30}; do
            if curl -f https://staging.example.com/health; then
              echo "Health check passed"
              exit 0
            fi
            echo "Waiting for staging..."
            sleep 2
          done
          echo "Health check failed"
          exit 1

      - name: Smoke tests
        run: |
          curl -f https://staging.example.com/api/status
          # Add more smoke tests here

  # Job 5: Deploy to production (requires approval)
  deploy-production:
    needs: smoke-test
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://example.com
    # Only run on pushes to main (not PRs)
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3

      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist
          path: dist/

      - name: Tag release
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          VERSION=$(date +%Y%m%d.%H%M%S)
          git tag -a v$VERSION -m "Release $VERSION"
          git push origin v$VERSION

      - name: Deploy to production
        env:
          SSH_KEY: ${{ secrets.PRODUCTION_SSH_KEY }}
          HOST: ${{ secrets.PRODUCTION_HOST }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/key
          chmod 600 ~/.ssh/key

          # Blue-green deployment pattern
          ssh -i ~/.ssh/key -o StrictHostKeyChecking=no deploy@$HOST << 'ENDSSH'
            set -e

            # Deploy to blue environment (inactive)
            rsync -avz dist/ /opt/production-blue/

            # Health check blue
            systemctl restart myapp-blue
            sleep 5
            curl -f http://localhost:3001/health || exit 1

            # Switch traffic to blue
            ln -sfn /opt/production-blue /opt/production-active
            systemctl reload nginx

            # Old green is now backup
            echo "✓ Production deployed successfully"
ENDSSH

      - name: Notify deployment
        if: always()
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
        run: |
          if [ "${{ job.status }}" == "success" ]; then
            curl -X POST $SLACK_WEBHOOK -d '{"text":"✅ Production deployment successful"}'
          else
            curl -X POST $SLACK_WEBHOOK -d '{"text":"❌ Production deployment failed"}'
          fi
```

This pipeline:
1. ✅ Runs tests and lint
2. ✅ Builds application
3. ✅ Deploys to staging automatically
4. ✅ Runs smoke tests
5. ✅ Deploys to production (with approval gate)
6. ✅ Uses blue-green deployment for zero downtime
7. ✅ Tags releases
8. ✅ Sends notifications

---

## 9. Containers & Docker (Deep Mental Model)

### Why Containers Exist

**The Problem:**

```
Your laptop: Ubuntu 22.04, Node 18, Python 3.10
Staging server: Ubuntu 20.04, Node 16, Python 3.8
Production server: Ubuntu 22.04, Node 20, Python 3.11
```

Your app works on your laptop. Breaks in staging. Works in production but with subtle bugs.

"Works on my machine" is a real problem.

**The Old Solution:** Virtual Machines

Run a complete OS inside another OS. Heavy, slow to start, wastes resources.

**The New Solution:** Containers

```
Container = Isolated process + filesystem

Not a full OS, just:
- Your app
- Its dependencies
- A minimal filesystem
- Process isolation
```

**The Mental Model:**

Think of a container as a **lightweight, isolated Unix process** with its own:
- Filesystem (can't see host files)
- Network (can have its own IP)
- Process list (can't see host processes)

But it shares the kernel with the host. That's why it's lightweight.

```
Traditional: App → OS → Hypervisor → Hardware
Container: App → Container Runtime → OS → Hardware
```

### Images vs Containers vs Layers

**Image**: A template. Like a class in OOP.

**Container**: A running instance. Like an object in OOP.

```bash
# Image is the blueprint
docker build -t myapp .

# Container is the running instance
docker run myapp

# You can run multiple containers from one image
docker run --name app1 myapp
docker run --name app2 myapp
docker run --name app3 myapp
```

**Layers**: Images are built in layers. Each Dockerfile instruction creates a layer.

```dockerfile
FROM node:18              # Layer 1: Base image
WORKDIR /app              # Layer 2: Set working directory
COPY package*.json ./     # Layer 3: Copy dependency files
RUN npm install           # Layer 4: Install dependencies
COPY . .                  # Layer 5: Copy application code
CMD ["node", "app.js"]    # Layer 6: Default command
```

**Why Layers Matter:**

Layers are cached. If you change your code but not dependencies, only the code layer rebuilds.

```
Rebuild without code changes:
✓ Layer 1: Cached (base image unchanged)
✓ Layer 2: Cached (workdir unchanged)
✓ Layer 3: Cached (package.json unchanged)
✓ Layer 4: Cached (dependencies unchanged)
✗ Layer 5: Rebuilt (code changed)
✓ Layer 6: Cached (command unchanged)

Result: Fast rebuilds (seconds instead of minutes)
```

**Bad Order:**
```dockerfile
FROM node:18
WORKDIR /app
COPY . .                  # Copy everything first
RUN npm install           # Install dependencies
CMD ["node", "app.js"]

# Every code change invalidates npm install layer
# Slow!
```

**Good Order:**
```dockerfile
FROM node:18
WORKDIR /app
COPY package*.json ./     # Copy dependencies first
RUN npm install           # This layer is cached if package.json unchanged
COPY . .                  # Copy code last
CMD ["node", "app.js"]

# Code changes don't invalidate npm install
# Fast!
```

### Dockerfile Best Practices

**A Production-Ready Node.js Dockerfile:**

```dockerfile
# Use specific version (not "latest")
FROM node:18-alpine AS builder

# Add metadata
LABEL maintainer="you@example.com"
LABEL version="1.0"

# Create app directory
WORKDIR /app

# Copy dependency manifests
COPY package*.json ./

# Install dependencies (including devDependencies for build)
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# Production stage (multi-stage build)
FROM node:18-alpine

# Run as non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

WORKDIR /app

# Copy only production dependencies
COPY package*.json ./
RUN npm ci --production --ignore-scripts && \
    npm cache clean --force

# Copy built application from builder
COPY --from=builder /app/dist ./dist

# Change ownership to nodejs user
RUN chown -R nodejs:nodejs /app

# Switch to nodejs user
USER nodejs

# Expose port (documentation only)
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => process.exit(r.statusCode === 200 ? 0 : 1))"

# Start application
CMD ["node", "dist/app.js"]
```

**Let's break down the best practices:**

**1. Use specific versions**
```dockerfile
# Bad
FROM node:latest

# Good
FROM node:18-alpine
```

**2. Multi-stage builds** (reduces final image size)
```dockerfile
# Build stage: has dev dependencies
FROM node:18 AS builder
RUN npm install
RUN npm run build

# Production stage: only runtime dependencies
FROM node:18-alpine
COPY --from=builder /app/dist ./dist
```

**3. Use alpine for smaller images**
```
node:18        ~ 900MB
node:18-slim   ~ 180MB
node:18-alpine ~  120MB
```

**4. Run as non-root user**
```dockerfile
# Security: Don't run as root
USER nodejs
```

**5. Health checks**
```dockerfile
HEALTHCHECK --interval=30s CMD curl -f http://localhost:3000/health || exit 1
```

**6. .dockerignore** (like .gitignore)
```
# .dockerignore
node_modules
npm-debug.log
.git
.env
dist
*.md
Dockerfile
```

### Volumes and Persistence

**The Problem:**

Containers are ephemeral. When you stop a container, data inside is lost.

```bash
# Run a container
docker run --name db postgres

# Write data to database
# Stop container
docker stop db

# Start again
docker start db

# Data is still there

# BUT remove container
docker rm db

# Run new container
docker run --name db postgres

# Data is gone!
```

**Solution: Volumes**

Volumes are directories that persist outside the container.

```bash
# Create a volume
docker volume create mydata

# Run container with volume
docker run -v mydata:/var/lib/postgresql/data postgres

# Data now persists even if container is removed
```

**Types of Storage:**

**1. Named Volumes** (managed by Docker)
```bash
docker run -v mydata:/app/data myapp

# Docker stores this in /var/lib/docker/volumes/mydata
```

**2. Bind Mounts** (mount host directory)
```bash
docker run -v /home/user/data:/app/data myapp

# Mounts host directory into container
# Useful for development
```

**3. tmpfs Mounts** (in-memory, fast but not persistent)
```bash
docker run --tmpfs /app/cache myapp

# Stored in RAM, gone when container stops
```

**Development Pattern:**

```bash
# Mount your source code for live reloading
docker run \
  -v $(pwd):/app \
  -v /app/node_modules \
  -p 3000:3000 \
  myapp

# Your code changes are immediately reflected in container
# But node_modules stays in container (faster)
```

### Networking Between Containers

**The Problem:**

Your app needs to talk to a database. Both run in containers.

**Solution: Docker Networks**

```bash
# Create a network
docker network create mynetwork

# Run database on network
docker run --name db --network mynetwork postgres

# Run app on same network
docker run --name app --network mynetwork myapp

# App can now reach database at hostname "db"
# connection string: postgres://db:5432/mydb
```

**How it works:**

Docker provides DNS resolution within a network. Containers can reach each other by name.

```
Container "app" tries to connect to "db:5432"
    |
    v
Docker DNS resolves "db" to container's IP
    |
    v
Connection established
```

**Port Mapping:**

```bash
# Run without port mapping
docker run --name app myapp

# App is listening on 3000 inside container
# But NOT accessible from host

# Run with port mapping
docker run -p 8080:3000 --name app myapp

# Host port 8080 -> Container port 3000
# Access via http://localhost:8080
```

**The Flow:**

```
Browser → http://localhost:8080
    |
    v
Host OS sees request on port 8080
    |
    v
Docker forwards to container port 3000
    |
    v
App receives request
```

### What Actually Happens Under the Hood

**When you `docker run`:**

```
1. Docker daemon checks if image exists locally
   - If not, pulls from registry

2. Creates a new container:
   - Allocates a writable layer on top of image layers
   - Creates network interface (virtual)
   - Assigns IP address
   - Sets up iptables rules for port mapping

3. Starts the process:
   - Runs in isolated namespaces:
     * PID namespace (can't see host processes)
     * Network namespace (own network stack)
     * Mount namespace (own filesystem)
     * UTS namespace (own hostname)
   - Applies cgroups (limits CPU, memory)

4. Process runs

5. On stop:
   - Sends SIGTERM
   - Waits 10 seconds
   - Sends SIGKILL if still running
   - Container stops but data in writable layer persists

6. On rm:
   - Removes writable layer
   - Container is gone
```

**Namespaces and cgroups:**

Docker uses Linux kernel features:

- **Namespaces**: Isolation (what you can see)
- **cgroups**: Resource limits (what you can use)

```bash
# Inside container
ps aux  # Only sees container processes

# On host
ps aux | grep docker  # Sees container processes

# They're the same processes, just isolated views
```

### Common Beginner Mistakes

**Mistake 1: Running containers as root**
```dockerfile
# Bad - runs as root inside container
FROM node:18
CMD ["node", "app.js"]

# Good - create and use non-root user
FROM node:18
USER node
CMD ["node", "app.js"]
```

**Mistake 2: Large images**
```dockerfile
# Bad - includes dev dependencies and build tools
FROM node:18
COPY . .
RUN npm install
CMD ["node", "app.js"]

# Good - multi-stage build, only production deps
FROM node:18-alpine
COPY package*.json ./
RUN npm ci --production
COPY dist ./dist
CMD ["node", "dist/app.js"]
```

**Mistake 3: Not using .dockerignore**
```
# Without .dockerignore, COPY . . includes:
node_modules (huge!)
.git (unnecessary)
.env (dangerous!)
```

**Mistake 4: Storing secrets in images**
```dockerfile
# Bad - secret is baked into image
ENV API_KEY=sk_live_12345

# Good - pass at runtime
docker run -e API_KEY=$API_KEY myapp
```

**Mistake 5: Not handling signals**
```javascript
// Bad - container takes 10 seconds to stop
app.listen(3000);

// Good - graceful shutdown
const server = app.listen(3000);

process.on('SIGTERM', () => {
  server.close(() => {
    console.log('Graceful shutdown');
    process.exit(0);
  });
});
```

### Hands-On Practice Tasks

**Task 1: Build your first Docker image**
```bash
# Create a simple app
mkdir docker-test
cd docker-test

cat > app.js << 'EOF'
const http = require('http');
const server = http.createServer((req, res) => {
  res.end('Hello from Docker!\n');
});
server.listen(3000, () => console.log('Running on 3000'));
EOF

# Create Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
COPY app.js .
EXPOSE 3000
CMD ["node", "app.js"]
EOF

# Build image
docker build -t myapp .

# Run container
docker run -p 3000:3000 myapp

# Test
curl localhost:3000

# See running containers
docker ps

# Stop container
docker stop [container-id]
```

**Task 2: Understand layers**
```bash
# Build with layer visibility
docker build -t myapp . --progress=plain

# See image layers
docker history myapp

# Check image size
docker images myapp

# Now modify app.js and rebuild
echo "// comment" >> app.js
docker build -t myapp .

# Notice: Early layers are cached, only code layer rebuilds
```

**Task 3: Use volumes**
```bash
# Run without volume - data lost
docker run --name db -e POSTGRES_PASSWORD=secret postgres
docker exec -it db psql -U postgres -c "CREATE DATABASE test;"
docker rm -f db
docker run --name db -e POSTGRES_PASSWORD=secret postgres
docker exec -it db psql -U postgres -l  # test DB is gone

# Run with volume - data persists
docker volume create pgdata
docker run --name db -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres
docker exec -it db psql -U postgres -c "CREATE DATABASE test;"
docker rm -f db
docker run --name db -v pgdata:/var/lib/postgresql/data -e POSTGRES_PASSWORD=secret postgres
docker exec -it db psql -U postgres -l  # test DB still there!
```

**Task 4: Container networking**
```bash
# Create network
docker network create mynet

# Run database
docker run -d --name db --network mynet -e POSTGRES_PASSWORD=secret postgres

# Run app that connects to database
cat > dbtest.js << 'EOF'
const { Client } = require('pg');
const client = new Client({
  host: 'db',  // Container name!
  user: 'postgres',
  password: 'secret'
});
client.connect()
  .then(() => console.log('Connected to database!'))
  .catch(err => console.error('Failed:', err));
EOF

cat > Dockerfile << 'EOF'
FROM node:18-alpine
WORKDIR /app
RUN npm install pg
COPY dbtest.js .
CMD ["node", "dbtest.js"]
EOF

docker build -t dbtest .
docker run --network mynet dbtest

# See "Connected to database!"
```

### Mini Project: Containerize a Full Application

**Goal:** Containerize a Node.js app with PostgreSQL database.

```bash
mkdir container-project
cd container-project

# Create app
cat > app.js << 'EOF'
const express = require('express');
const { Pool } = require('pg');

const app = express();
const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  user: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME || 'myapp'
});

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.get('/users', async (req, res) => {
  try {
    const result = await pool.query('SELECT * FROM users');
    res.json(result.rows);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.get('/init', async (req, res) => {
  try {
    await pool.query('CREATE TABLE IF NOT EXISTS users (id SERIAL PRIMARY KEY, name TEXT)');
    await pool.query("INSERT INTO users (name) VALUES ('Alice'), ('Bob')");
    res.json({ message: 'Database initialized' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server on port ${PORT}`));

process.on('SIGTERM', () => {
  pool.end(() => {
    console.log('Graceful shutdown');
    process.exit(0);
  });
});
EOF

# Create package.json
cat > package.json << 'EOF'
{
  "name": "myapp",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.0",
    "pg": "^8.11.0"
  }
}
EOF

# Create optimized Dockerfile
cat > Dockerfile << 'EOF'
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .

FROM node:18-alpine
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
WORKDIR /app
COPY --from=builder /app .
USER nodejs
EXPOSE 3000
HEALTHCHECK CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1
CMD ["node", "app.js"]
EOF

# Create .dockerignore
cat > .dockerignore << 'EOF'
node_modules
.git
.env
*.md
Dockerfile
EOF

# Build image
docker build -t myapp:1.0 .

# Create network
docker network create mynet

# Run database
docker volume create pgdata
docker run -d \
  --name db \
  --network mynet \
  -v pgdata:/var/lib/postgresql/data \
  -e POSTGRES_PASSWORD=secret \
  -e POSTGRES_DB=myapp \
  postgres:15-alpine

# Wait for database to start
sleep 5

# Run app
docker run -d \
  --name app \
  --network mynet \
  -p 3000:3000 \
  -e DB_HOST=db \
  -e DB_USER=postgres \
  -e DB_PASSWORD=secret \
  -e DB_NAME=myapp \
  myapp:1.0

# Wait for app to start
sleep 2

# Test
curl http://localhost:3000/health
curl http://localhost:3000/init
curl http://localhost:3000/users

# Check logs
docker logs app
docker logs db

# Cleanup
docker stop app db
docker rm app db
docker network rm mynet
# docker volume rm pgdata  # Uncomment to delete data
```

You've now:
- Built an optimized multi-stage image
- Set up container networking
- Used volumes for persistence
- Ran a multi-container application
- Implemented health checks
- Used proper security (non-root user)

---

## 10. Docker Compose (Production Mindset)

### Multi-Service Apps

Real applications have multiple services:
- Web app
- API
- Database
- Cache (Redis)
- Background worker

Running each with `docker run` is tedious:

```bash
docker network create mynet
docker run -d --name db --network mynet postgres
docker run -d --name redis --network mynet redis
docker run -d --name api --network mynet -p 3000:3000 myapi
docker run -d --name worker --network mynet myworker
docker run -d --name web --network mynet -p 80:80 myweb
```

**Docker Compose** defines all this in one file.

### Service Dependencies

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  # Database
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # Cache
  redis:
    image: redis:7-alpine
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
    restart: unless-stopped

  # API service
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: postgres
      DB_PASSWORD: secret
      DB_NAME: myapp
      REDIS_HOST: redis
      NODE_ENV: production
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M

  # Background worker
  worker:
    build:
      context: ./worker
      dockerfile: Dockerfile
    environment:
      DB_HOST: db
      DB_USER: postgres
      DB_PASSWORD: secret
      REDIS_HOST: redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped

  # Web frontend
  web:
    build:
      context: ./web
      dockerfile: Dockerfile
    ports:
      - "80:80"
    depends_on:
      - api
    restart: unless-stopped

volumes:
  pgdata:

networks:
  default:
    name: myapp-network
```

**Let's break this down:**

**Services**: Each service is a container

**depends_on**: Start order and health dependencies
```yaml
depends_on:
  db:
    condition: service_healthy  # Wait for db to be healthy
```

**healthcheck**: How to check if service is ready
```yaml
healthcheck:
  test: ["CMD", "pg_isready", "-U", "postgres"]
  interval: 5s      # Check every 5 seconds
  timeout: 3s       # Timeout after 3 seconds
  retries: 5        # Retry 5 times before marking unhealthy
```

**restart**: Restart policy
```yaml
restart: unless-stopped  # Always restart unless manually stopped
# Other options:
# - no (never restart)
# - always (always restart)
# - on-failure (restart only if exit code != 0)
```

**deploy**: Resource limits (Docker Compose v3+ with Swarm, or use Docker Compose v2.x syntax)
```yaml
deploy:
  resources:
    limits:
      cpus: '0.5'     # Max 50% of one CPU
      memory: 512M     # Max 512MB RAM
```

**volumes**: Named volumes for persistence

**networks**: Custom network (all services on same network by default)

### Commands

```bash
# Start all services
docker compose up -d

# See status
docker compose ps

# View logs
docker compose logs
docker compose logs -f api  # Follow logs for api service

# Stop all services
docker compose stop

# Stop and remove containers
docker compose down

# Rebuild and restart
docker compose up -d --build

# Scale a service
docker compose up -d --scale worker=3

# Execute command in service
docker compose exec api sh
docker compose exec db psql -U postgres
```

### Environment Separation

**Production vs Development:**

**docker-compose.yml** (base config):
```yaml
version: '3.8'
services:
  api:
    build: ./api
    environment:
      DB_HOST: db
```

**docker-compose.dev.yml** (development overrides):
```yaml
version: '3.8'
services:
  api:
    build:
      context: ./api
      target: development  # Multi-stage build target
    volumes:
      - ./api:/app  # Mount code for live reload
      - /app/node_modules  # But don't override node_modules
    environment:
      NODE_ENV: development
      DEBUG: '*'
```

**docker-compose.prod.yml** (production overrides):
```yaml
version: '3.8'
services:
  api:
    image: myregistry.com/api:${VERSION}
    environment:
      NODE_ENV: production
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
```

**Usage:**

```bash
# Development
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### Common Anti-Patterns

**Anti-Pattern 1: Not using volumes for databases**
```yaml
# Bad - data lost when container removed
services:
  db:
    image: postgres

# Good - data persists
services:
  db:
    image: postgres
    volumes:
      - pgdata:/var/lib/postgresql/data
volumes:
  pgdata:
```

**Anti-Pattern 2: Hardcoding secrets**
```yaml
# Bad
environment:
  DB_PASSWORD: secret123

# Good - use environment variables or secrets
environment:
  DB_PASSWORD: ${DB_PASSWORD}

# Run with:
# DB_PASSWORD=secret123 docker compose up
```

**Anti-Pattern 3: Not using health checks**
```yaml
# Bad - dependent services might start before database is ready
services:
  api:
    depends_on:
      - db

# Good - wait for healthy state
services:
  api:
    depends_on:
      db:
        condition: service_healthy
  db:
    healthcheck:
      test: ["CMD", "pg_isready"]
```

**Anti-Pattern 4: Exposing unnecessary ports**
```yaml
# Bad - database exposed to host
services:
  db:
    ports:
      - "5432:5432"  # Now accessible from outside!

# Good - only expose what's needed
# API and web services can reach db on internal network
# No ports needed
services:
  db:
    # No ports section - only accessible within Docker network
```

**Anti-Pattern 5: Not setting resource limits**
```yaml
# Bad - one service can starve others
services:
  api:
    # No limits

# Good
services:
  api:
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
```

### What Actually Happens Under the Hood

**When you `docker compose up`:**

```
1. docker compose reads docker-compose.yml

2. Creates network (if specified, or default)

3. Creates volumes (if they don't exist)

4. Builds images (if build: specified)

5. Creates containers in dependency order:
   - Starts db
   - Waits for db health check
   - Starts redis
   - Waits for redis health check
   - Starts api (depends on db and redis)
   - Starts worker (depends on db and redis)
   - Starts web (depends on api)

6. Attaches to logs (if not -d)

7. Monitors containers and restarts based on restart policy
```

**Networking:**

All services are on the same network and can reach each other by service name:

```
api service connects to "db:5432"
    |
    v
Docker DNS resolves "db" to db container IP
    |
    v
Connection established
```

### Hands-On Practice Tasks

**Task 1: Create a simple compose file**
```bash
mkdir compose-test
cd compose-test

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    volumes:
      - ./html:/usr/share/nginx/html
    restart: unless-stopped

volumes: {}
EOF

# Create HTML file
mkdir html
echo "<h1>Hello from Docker Compose</h1>" > html/index.html

# Start
docker compose up -d

# Test
curl localhost:8080

# View logs
docker compose logs

# Stop
docker compose down
```

**Task 2: Multi-service app with dependencies**
```bash
mkdir multi-service
cd multi-service

# Create docker-compose.yml
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_PASSWORD: secret
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 2s
      timeout: 3s
      retries: 5

  api:
    image: node:18-alpine
    working_dir: /app
    command: sh -c "npm install && node server.js"
    volumes:
      - ./api:/app
    ports:
      - "3000:3000"
    environment:
      DB_HOST: db
      DB_USER: postgres
      DB_PASSWORD: secret
    depends_on:
      db:
        condition: service_healthy
EOF

# Create API
mkdir api
cat > api/package.json << 'EOF'
{
  "dependencies": {
    "express": "^4.18.0",
    "pg": "^8.11.0"
  }
}
EOF

cat > api/server.js << 'EOF'
const express = require('express');
const { Pool } = require('pg');
const app = express();

const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD
});

app.get('/', async (req, res) => {
  const result = await pool.query('SELECT NOW()');
  res.json({ time: result.rows[0].now });
});

app.listen(3000, () => console.log('API ready'));
EOF

# Start everything
docker compose up

# In another terminal, test
curl localhost:3000

# See logs
docker compose logs db
docker compose logs api

# Stop
docker compose down
```

**Task 3: Development vs production configs**
```bash
# Base config
cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  app:
    build: .
    ports:
      - "3000:3000"
EOF

# Dev overrides
cat > docker-compose.dev.yml << 'EOF'
version: '3.8'
services:
  app:
    volumes:
      - .:/app  # Mount source for hot reload
    environment:
      - DEBUG=*
EOF

# Prod overrides
cat > docker-compose.prod.yml << 'EOF'
version: '3.8'
services:
  app:
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
EOF

# Run dev
docker compose -f docker-compose.yml -f docker-compose.dev.yml up

# Run prod
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### Mini Project: Complete Stack with Docker Compose

**Goal:** Deploy a complete application stack (web, API, database, cache, worker).

```yaml
# docker-compose.yml
version: '3.8'

services:
  # PostgreSQL Database
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-myapp}
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?Database password required}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped
    networks:
      - backend

  # Redis Cache
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped
    networks:
      - backend

  # API Service
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
      args:
        NODE_ENV: production
    environment:
      NODE_ENV: production
      PORT: 3000
      DATABASE_URL: postgresql://${POSTGRES_USER:-postgres}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB:-myapp}
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G
    networks:
      - backend
      - frontend

  # Background Worker
  worker:
    build:
      context: ./worker
      dockerfile: Dockerfile
    environment:
      NODE_ENV: production
      DATABASE_URL: postgresql://${POSTGRES_USER:-postgres}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB:-myapp}
      REDIS_URL: redis://redis:6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
    restart: unless-stopped
    deploy:
      replicas: 2
      resources:
        limits:
          cpus: '0.5'
          memory: 512M
    networks:
      - backend

  # Web Frontend (Nginx)
  web:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./web/dist:/usr/share/nginx/html:ro
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      api:
        condition: service_healthy
    restart: unless-stopped
    networks:
      - frontend

volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

**Environment file (.env):**
```bash
# .env (don't commit this!)
POSTGRES_DB=myapp
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_secure_password_here
```

**Usage:**
```bash
# Start everything
docker compose up -d

# Check status
docker compose ps

# View logs
docker compose logs -f

# Scale workers
docker compose up -d --scale worker=5

# Stop everything
docker compose down

# Stop and remove volumes (deletes data!)
docker compose down -v
```

This gives you a production-ready stack with:
- Database with persistence
- Cache with persistence
- API with health checks and resource limits
- Scalable background workers
- Frontend with Nginx
- Proper networking (frontend/backend separation)
- Environment-based configuration
- Restart policies

---

## 11. Infrastructure as Code

### Why Manual Servers Don't Scale

You've been SSH-ing into your VPS and running commands. This works for one server, but what about:

- 3 web servers
- 2 database servers
- 1 cache server
- 1 load balancer

Suddenly you're SSH-ing into 7 servers, running the same commands 7 times. You make a typo on server 3. Configuration drift happens - each server is slightly different.

**The problem:**
- Manual = slow, error-prone, not reproducible
- "What did I install last month?" (no documentation)
- Can't easily replicate setup
- Disaster recovery means rebuilding from memory

**Infrastructure as Code (IaC) solution:**

Define your infrastructure in code files. Run a tool that creates/updates everything to match the code.

```
Code → Tool → Real Infrastructure

Servers don't exist → Run tool → Servers created
Change code → Run tool → Servers updated
Delete code → Run tool → Servers destroyed
```

### Terraform Fundamentals

**Terraform** is the most popular IaC tool. It works with any cloud provider (AWS, GCP, Azure, DigitalOcean, etc.).

**Basic Concept:**

You write `.tf` files describing what you want. Terraform figures out how to make it happen.

**Simple Example:**

```hcl
# main.tf

# Configure provider (DigitalOcean in this case)
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

# Variables
variable "do_token" {
  description = "DigitalOcean API token"
  type        = string
  sensitive   = true
}

# Create a VPS (droplet)
resource "digitalocean_droplet" "web" {
  name   = "web-server"
  region = "nyc3"
  size   = "s-1vcpu-1gb"
  image  = "ubuntu-22-04-x64"

  ssh_keys = [digitalocean_ssh_key.default.id]

  # Run commands after creation
  user_data = <<-EOF
              #!/bin/bash
              apt-get update
              apt-get install -y nginx
              systemctl enable nginx
              systemctl start nginx
              EOF
}

# SSH key
resource "digitalocean_ssh_key" "default" {
  name       = "terraform-key"
  public_key = file("~/.ssh/id_rsa.pub")
}

# Output the IP address
output "web_ip" {
  value = digitalocean_droplet.web.ipv4_address
}
```

**Terraform Workflow:**

```bash
# Initialize (download providers)
terraform init

# See what would be created (dry run)
terraform plan

# Create the infrastructure
terraform apply

# See outputs
terraform output

# Destroy everything
terraform destroy
```

**What Happens:**

```
terraform apply
    |
    v
1. Terraform reads .tf files
2. Compares to current state
3. Calculates what needs to change
4. Shows you the plan
5. You confirm
6. Terraform makes API calls to create/update resources
7. Saves state to terraform.tfstate
```

### State Files Explained

**terraform.tfstate**: This file tracks what infrastructure exists.

```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "resources": [
    {
      "type": "digitalocean_droplet",
      "name": "web",
      "instances": [
        {
          "attributes": {
            "id": "123456",
            "ipv4_address": "1.2.3.4",
            "name": "web-server"
          }
        }
      ]
    }
  ]
}
```

**Why State Matters:**

Without state, Terraform doesn't know what it previously created. It would try to create duplicates.

**State File Best Practices:**

1. **Never edit manually**
2. **Never commit to git** (contains sensitive data)
3. **Use remote state** (S3, Terraform Cloud) for team environments
4. **Lock state** (prevent concurrent modifications)

**Remote State Example:**

```hcl
terraform {
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"

    # Enable state locking
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

Now multiple team members can use Terraform without conflicts.

### When NOT to Use Terraform

**Don't use Terraform for:**

**1. Configuration Management**
```hcl
# Bad - using Terraform to manage app config
resource "local_file" "config" {
  filename = "/etc/myapp/config.json"
  content  = jsonencode({
    api_key = "secret"
  })
}

# Better: Use Ansible, Chef, Puppet for config
# Or just use systemd environment files
```

**2. Secrets Management**
```hcl
# Bad - storing secrets in Terraform
variable "db_password" {
  default = "secret123"  # This ends up in state file!
}

# Better: Use Vault, AWS Secrets Manager, etc.
```

**3. One-Off Tasks**
```bash
# Bad: Writing Terraform for a one-time server
# Just spin it up manually or with a script

# Terraform is for:
# - Reproducible infrastructure
# - Multiple environments (dev, staging, prod)
# - Resources you'll modify over time
```

**4. Application Deployment**
```hcl
# Bad - using Terraform to deploy app code
resource "null_resource" "deploy" {
  provisioner "remote-exec" {
    inline = ["git pull", "npm install"]
  }
}

# Better: Use CI/CD, Ansible, or deployment scripts
```

**Use Terraform for:**
- Creating VPS instances
- Setting up networks, firewalls
- Managing DNS records
- Cloud resources (databases, storage, etc.)
- Anything you want to version control and recreate

### Common Beginner Mistakes

**Mistake 1: Not using variables**
```hcl
# Bad - hardcoded
resource "digitalocean_droplet" "web" {
  size = "s-1vcpu-1gb"
}

# Good - parameterized
variable "droplet_size" {
  default = "s-1vcpu-1gb"
}

resource "digitalocean_droplet" "web" {
  size = var.droplet_size
}
```

**Mistake 2: Massive single file**
```
# Bad
main.tf  # 1000 lines

# Good
main.tf          # Main resource definitions
variables.tf     # Variable declarations
outputs.tf       # Outputs
providers.tf     # Provider configuration
```

**Mistake 3: Not using modules**
```hcl
# Instead of repeating:
resource "digitalocean_droplet" "web1" { ... }
resource "digitalocean_droplet" "web2" { ... }
resource "digitalocean_droplet" "web3" { ... }

# Create a module and use it 3 times
```

**Mistake 4: Committing sensitive state**
```bash
# .gitignore
terraform.tfstate
terraform.tfstate.backup
*.tfvars  # Variable files might contain secrets
.terraform/
```

### Hands-On Practice Tasks

**Task 1: Install Terraform**
```bash
# Ubuntu/Debian
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# Verify
terraform version
```

**Task 2: Create your first Terraform config**
```bash
mkdir terraform-test
cd terraform-test

# Create main.tf
cat > main.tf << 'EOF'
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "~> 2.0"
    }
  }
}

resource "local_file" "hello" {
  filename = "hello.txt"
  content  = "Hello from Terraform!"
}

output "file_path" {
  value = local_file.hello.filename
}
EOF

# Initialize
terraform init

# Plan (see what will be created)
terraform plan

# Apply (create the file)
terraform apply

# Check
cat hello.txt

# Destroy
terraform destroy
```

**Task 3: Use variables**
```bash
# variables.tf
cat > variables.tf << 'EOF'
variable "filename" {
  description = "Name of the file to create"
  type        = string
  default     = "output.txt"
}

variable "content" {
  description = "Content of the file"
  type        = string
}
EOF

# main.tf
cat > main.tf << 'EOF'
resource "local_file" "demo" {
  filename = var.filename
  content  = var.content
}
EOF

# Apply with variables
terraform apply -var="content=Hello World"

# Or use a variables file
cat > terraform.tfvars << 'EOF'
filename = "myfile.txt"
content  = "Content from tfvars"
EOF

terraform apply
```

### Mini Project: Provision a Complete VPS with Terraform

```hcl
# variables.tf
variable "do_token" {
  description = "DigitalOcean API token"
  type        = string
  sensitive   = true
}

variable "ssh_key_path" {
  description = "Path to SSH public key"
  type        = string
  default     = "~/.ssh/id_rsa.pub"
}

# main.tf
terraform {
  required_version = ">= 1.0"

  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

# SSH Key
resource "digitalocean_ssh_key" "terraform" {
  name       = "terraform-key"
  public_key = file(var.ssh_key_path)
}

# VPC Network
resource "digitalocean_vpc" "main" {
  name     = "production-vpc"
  region   = "nyc3"
  ip_range = "10.10.0.0/16"
}

# Web Server
resource "digitalocean_droplet" "web" {
  name   = "web-server"
  region = "nyc3"
  size   = "s-1vcpu-1gb"
  image  = "ubuntu-22-04-x64"

  ssh_keys = [digitalocean_ssh_key.terraform.id]
  vpc_uuid = digitalocean_vpc.main.id

  user_data = file("cloud-init.yml")

  tags = ["web", "production"]
}

# Database Server
resource "digitalocean_droplet" "db" {
  name   = "db-server"
  region = "nyc3"
  size   = "s-2vcpu-2gb"
  image  = "ubuntu-22-04-x64"

  ssh_keys = [digitalocean_ssh_key.terraform.id]
  vpc_uuid = digitalocean_vpc.main.id

  tags = ["database", "production"]
}

# Firewall
resource "digitalocean_firewall" "web" {
  name = "web-firewall"

  droplet_ids = [digitalocean_droplet.web.id]

  # Allow HTTP
  inbound_rule {
    protocol         = "tcp"
    port_range       = "80"
    source_addresses = ["0.0.0.0/0", "::/0"]
  }

  # Allow HTTPS
  inbound_rule {
    protocol         = "tcp"
    port_range       = "443"
    source_addresses = ["0.0.0.0/0", "::/0"]
  }

  # Allow SSH from specific IP
  inbound_rule {
    protocol         = "tcp"
    port_range       = "22"
    source_addresses = ["your-ip/32"]  # Replace with your IP
  }

  # Allow all outbound
  outbound_rule {
    protocol              = "tcp"
    port_range            = "all"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }

  outbound_rule {
    protocol              = "udp"
    port_range            = "all"
    destination_addresses = ["0.0.0.0/0", "::/0"]
  }
}

# Domain
resource "digitalocean_domain" "main" {
  name = "example.com"  # Replace with your domain
}

# DNS Records
resource "digitalocean_record" "www" {
  domain = digitalocean_domain.main.name
  type   = "A"
  name   = "www"
  value  = digitalocean_droplet.web.ipv4_address
  ttl    = 3600
}

resource "digitalocean_record" "root" {
  domain = digitalocean_domain.main.name
  type   = "A"
  name   = "@"
  value  = digitalocean_droplet.web.ipv4_address
  ttl    = 3600
}

# Outputs
output "web_ip" {
  value       = digitalocean_droplet.web.ipv4_address
  description = "Web server public IP"
}

output "db_ip" {
  value       = digitalocean_droplet.db.ipv4_address
  description = "Database server public IP"
}

output "db_private_ip" {
  value       = digitalocean_droplet.db.ipv4_address_private
  description = "Database server private IP"
}
```

**cloud-init.yml** (run on first boot):
```yaml
#cloud-config
package_update: true
package_upgrade: true

packages:
  - nginx
  - nodejs
  - npm
  - git

runcmd:
  - systemctl enable nginx
  - systemctl start nginx
  - echo "Server provisioned at $(date)" > /var/www/html/index.html
```

**Usage:**
```bash
# Set your API token
export TF_VAR_do_token="your-digitalocean-token"

# Initialize
terraform init

# Plan
terraform plan

# Apply
terraform apply

# Get outputs
terraform output web_ip

# SSH to server
ssh root@$(terraform output -raw web_ip)

# When done, destroy everything
terraform destroy
```

This creates:
- A VPC network
- Two VPS instances (web and database)
- Firewall rules
- DNS records
- All managed as code, version controlled, reproducible

---

## 12. Monitoring, Logging & Alerts

### Metrics vs Logs vs Traces

When production breaks at 3am, you need to know:
1. **What** broke (metrics)
2. **Why** it broke (logs)
3. **Where** in the code (traces)

**Metrics**: Numbers over time
```
cpu_usage: 75%
requests_per_second: 1250
response_time_p95: 250ms
error_rate: 0.5%
```

**Logs**: Events that happened
```
2026-02-07T10:30:45Z ERROR Database connection timeout
2026-02-07T10:30:46Z INFO Retrying connection
2026-02-07T10:30:47Z ERROR Failed after 3 retries
```

**Traces**: Request path through system
```
Request → API Gateway (5ms)
       → Auth Service (20ms)
       → Database Query (200ms)  ← slow!
       → Cache Update (10ms)
Total: 235ms
```

### Prometheus Mental Model

**Prometheus** collects metrics by scraping (pulling) them from your services.

**The Architecture:**

```
Your App                    Prometheus Server
    |                              |
    | Exposes metrics at           |
    | /metrics endpoint            |
    |                              |
    | <-------- scrapes ---------- |
    |                              |
    |                              v
    |                       Stores time-series data
                                   |
                                   v
                            Grafana queries and visualizes
```

**How It Works:**

1. Your app exposes metrics at `/metrics`
2. Prometheus scrapes this endpoint every 15s (configurable)
3. Stores metrics in time-series database
4. You query metrics using PromQL
5. Grafana visualizes the data

**Instrumenting Your App:**

```javascript
// Node.js with prom-client
const client = require('prom-client');
const express = require('express');

const app = express();

// Create a Registry
const register = new client.Registry();

// Add default metrics (CPU, memory, etc.)
client.collectDefaultMetrics({ register });

// Custom metrics
const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.5, 1, 2, 5]
});

const httpRequestTotal = new client.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code']
});

register.registerMetric(httpRequestDuration);
register.registerMetric(httpRequestTotal);

// Middleware to track requests
app.use((req, res, next) => {
  const start = Date.now();

  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;

    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);

    httpRequestTotal
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .inc();
  });

  next();
});

// Your routes
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

// Metrics endpoint for Prometheus
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000);
```

**Prometheus Configuration:**

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['localhost:3000']

  - job_name: 'node-exporter'
    static_configs:
      - targets: ['localhost:9100']
```

**Installing Node Exporter** (system metrics):

```bash
# Download and run node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
cd node_exporter-1.7.0.linux-amd64
./node_exporter

# Access metrics
curl localhost:9100/metrics

# You'll see hundreds of metrics:
# node_cpu_seconds_total
# node_memory_MemAvailable_bytes
# node_disk_io_time_seconds_total
# etc.
```

**PromQL Queries:**

```promql
# Current CPU usage
rate(node_cpu_seconds_total[5m])

# Memory usage percentage
100 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100)

# Request rate
rate(http_requests_total[5m])

# 95th percentile response time
histogram_quantile(0.95, http_request_duration_seconds_bucket)

# Error rate
rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m]) * 100
```

### Grafana Dashboards That Matter

**Don't create dashboard bloat.** Focus on what you actually need.

**The Four Golden Signals:**

1. **Latency**: How long requests take
2. **Traffic**: How many requests
3. **Errors**: How many requests fail
4. **Saturation**: How full your resources are

**A Useful Dashboard:**

```
Row 1: System Health
- CPU usage (%)
- Memory usage (%)
- Disk usage (%)
- Network I/O

Row 2: Application Performance
- Requests per second
- Response time (p50, p95, p99)
- Error rate (%)
- Active connections

Row 3: Business Metrics
- New users (per hour)
- Failed logins (per minute)
- API calls by endpoint

Row 4: Logs (recent errors from Loki)
```

**Setting Up Grafana:**

```bash
# Docker Compose setup
cat > docker-compose.yml << 'EOF'
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"

  grafana:
    image: grafana/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - "3001:3000"
    depends_on:
      - prometheus

  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
  grafana_data:
EOF

docker compose up -d

# Access Grafana at http://localhost:3001
# Login: admin / admin
# Add Prometheus data source: http://prometheus:9090
# Import dashboard: ID 1860 (Node Exporter Full)
```

### Alert Fatigue and Real Alerts

**Bad Alerts:**

```yaml
# Fires every time CPU hits 51%
- alert: HighCPU
  expr: node_cpu_usage > 50
  for: 0s
```

This creates noise. You'll ignore alerts.

**Good Alerts:**

```yaml
# Only fires if sustained high CPU for 5 minutes
- alert: HighCPU
  expr: node_cpu_usage > 80
  for: 5m
  annotations:
    summary: "High CPU usage on {{ $labels.instance }}"
    description: "CPU usage is {{ $value }}% for 5 minutes"
```

**Alert Rules in Prometheus:**

```yaml
# alerts.yml
groups:
  - name: critical
    rules:
      # Disk will fill in 4 hours
      - alert: DiskFilling
        expr: predict_linear(node_filesystem_avail_bytes[1h], 4*3600) < 0
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "Disk will fill in 4 hours"

      # Service is down
      - alert: ServiceDown
        expr: up{job="my-app"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.job }} is down"

      # High error rate
      - alert: HighErrorRate
        expr: rate(http_requests_total{status_code=~"5.."}[5m]) / rate(http_requests_total[5m]) > 0.05
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Error rate above 5% for 5 minutes"

      # Slow response time
      - alert: SlowResponses
        expr: histogram_quantile(0.95, http_request_duration_seconds_bucket) > 2
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "95th percentile response time above 2s"
```

**Alertmanager Configuration:**

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m

route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'team'

  routes:
    # Critical alerts go to PagerDuty
    - match:
        severity: critical
      receiver: 'pagerduty'

    # Warnings go to Slack
    - match:
        severity: warning
      receiver: 'slack'

receivers:
  - name: 'team'
    email_configs:
      - to: 'team@example.com'

  - name: 'slack'
    slack_configs:
      - api_url: 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
        channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}\n{{ end }}'

  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'your-pagerduty-key'
```

### What Actually Happens Under the Hood

**When an alert fires:**

```
1. Prometheus evaluates rules every 15s

2. If condition is true for the "for" duration:
   - Alert transitions to FIRING state

3. Prometheus sends alert to Alertmanager

4. Alertmanager:
   - Groups similar alerts
   - Deduplicates
   - Applies silences
   - Routes to receivers (Slack, PagerDuty, email)

5. Receiver gets notification

6. When condition resolves:
   - Alert transitions to RESOLVED
   - Alertmanager sends resolved notification
```

**Grafana rendering a graph:**

```
1. User opens dashboard

2. Grafana sends PromQL query to Prometheus:
   "rate(http_requests_total[5m])"

3. Prometheus:
   - Parses query
   - Fetches time-series data from storage
   - Calculates rate over 5-minute windows
   - Returns data points

4. Grafana receives data:
   [
     {timestamp: 1623456000, value: 120},
     {timestamp: 1623456015, value: 125},
     ...
   ]

5. Grafana renders line graph
```

### Common Beginner Mistakes

**Mistake 1: Too many dashboards**
```
Don't create 50 dashboards. Create 3 good ones:
1. System overview (CPU, memory, disk)
2. Application metrics (requests, errors, latency)
3. Business metrics (users, revenue, key actions)
```

**Mistake 2: Meaningless alerts**
```yaml
# Bad - fires constantly
- alert: AnyError
  expr: error_count > 0

# Good - alerts on rate
- alert: HighErrorRate
  expr: rate(errors[5m]) > 10
  for: 5m
```

**Mistake 3: Not setting "for" duration**
```yaml
# Bad - fires on transient spikes
- alert: HighCPU
  expr: cpu > 80

# Good - sustained issues only
- alert: HighCPU
  expr: cpu > 80
  for: 5m
```

**Mistake 4: No runbook links**
```yaml
# Bad
annotations:
  summary: "Database is slow"

# Good
annotations:
  summary: "Database is slow"
  description: "p95 query time is {{ $value }}s"
  runbook: "https://wiki.example.com/runbooks/slow-database"
```

**Mistake 5: Monitoring without context**
```
CPU is at 80%
Is that bad? Depends:
- During peak traffic? Normal.
- At 3am with no traffic? Problem.

Add context: Compare to expected baselines.
```

### Hands-On Practice Tasks

**Task 1: Set up Prometheus and Grafana**
```bash
# Use Docker Compose from earlier section
docker compose up -d

# Access Prometheus: http://localhost:9090
# Access Grafana: http://localhost:3001

# In Grafana:
# 1. Add Prometheus data source (http://prometheus:9090)
# 2. Import dashboard 1860 (Node Exporter Full)
# 3. See system metrics!
```

**Task 2: Instrument your app**
```bash
# Add metrics to Node.js app (code from earlier)
npm install prom-client

# Run app and check metrics
curl localhost:3000/metrics

# Add app to prometheus.yml:
scrape_configs:
  - job_name: 'my-app'
    static_configs:
      - targets: ['host.docker.internal:3000']

# Reload Prometheus
docker compose restart prometheus

# Query in Prometheus:
# http_requests_total
# rate(http_requests_total[5m])
```

**Task 3: Create an alert**
```yaml
# Create alerts.yml
cat > alerts.yml << 'EOF'
groups:
  - name: test
    rules:
      - alert: HighMemory
        expr: node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes < 0.1
        for: 1m
        annotations:
          summary: "Less than 10% memory available"
EOF

# Add to prometheus.yml:
rule_files:
  - '/etc/prometheus/alerts.yml'

# Mount in docker-compose.yml:
volumes:
  - ./alerts.yml:/etc/prometheus/alerts.yml

# Restart and check Alerts tab in Prometheus
```

### Mini Project: Complete Monitoring Stack

```yaml
# docker-compose-monitoring.yml
version: '3.8'

services:
  # Application
  app:
    build: ./app
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production

  # Prometheus
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'

  # Alertmanager
  alertmanager:
    image: prom/alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager:/etc/alertmanager
    command:
      - '--config.file=/etc/alertmanager/alertmanager.yml'

  # Grafana
  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    depends_on:
      - prometheus

  # Node Exporter (system metrics)
  node-exporter:
    image: prom/node-exporter
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    volumes:
      - /:/host:ro,rslave

  # Loki (logs)
  loki:
    image: grafana/loki
    ports:
      - "3100:3100"
    volumes:
      - loki_data:/loki

  # Promtail (log shipper)
  promtail:
    image: grafana/promtail
    volumes:
      - /var/log:/var/log:ro
      - ./promtail:/etc/promtail
    command:
      - '--config.file=/etc/promtail/promtail.yml'

volumes:
  prometheus_data:
  grafana_data:
  loki_data:
```

This gives you:
- Application metrics (from your app)
- System metrics (from node-exporter)
- Logs (Loki + Promtail)
- Visualizations (Grafana)
- Alerts (Alertmanager)

---

## 13. Security for Real Systems

### SSH Hardening

Your VPS is on the public internet. Bots are constantly trying to SSH in with default passwords.

**Default SSH is insecure:**
- Allows password authentication
- Allows root login
- Runs on port 22 (everyone knows this)
- No rate limiting

**Hardened SSH:**

```bash
# Edit SSH config
vim /etc/ssh/sshd_config

# Make these changes:

# Disable root login
PermitRootLogin no

# Disable password authentication (key-only)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Only allow specific user
AllowUsers yourusername

# Change port (security through obscurity, but helps)
Port 2222

# Disable empty passwords
PermitEmptyPasswords no

# Limit authentication attempts
MaxAuthTries 3

# Disconnect if no successful login
LoginGraceTime 30

# Restart SSH
systemctl restart sshd
```

**IMPORTANT:** Before disabling password auth, make sure you have SSH keys set up!

```bash
# On your local machine
ssh-keygen -t ed25519 -C "your_email@example.com"

# Copy public key to server
ssh-copy-id -p 2222 user@your-server.com

# Test connection (with key)
ssh -p 2222 user@your-server.com

# Only THEN disable password auth
```

**SSH Key Best Practices:**

```bash
# Generate strong key
ssh-keygen -t ed25519 -a 100 -f ~/.ssh/production

# Set passphrase (encrypts private key)
# If someone steals your laptop, they still can't use the key

# Restrict key permissions
chmod 600 ~/.ssh/production
chmod 644 ~/.ssh/production.pub

# Use SSH agent to avoid typing passphrase repeatedly
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/production
```

### Firewalls

**Defense in depth**: Even if a service has a vulnerability, the firewall blocks access.

**UFW (Uncomplicated Firewall) Setup:**

```bash
# Default policies
ufw default deny incoming
ufw default allow outgoing

# SSH (use your custom port if changed)
ufw allow 2222/tcp

# HTTP and HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# Allow from specific IP only (for admin panel)
ufw allow from 1.2.3.4 to any port 3001

# Enable
ufw enable

# Status
ufw status verbose
```

**Port Knocking** (advanced):

Hide SSH behind a sequence of port knocks.

```bash
# Install knockd
apt install knockd

# Configure
cat > /etc/knockd.conf << 'EOF'
[openSSH]
    sequence    = 7000,8000,9000
    seq_timeout = 5
    command     = /usr/sbin/ufw allow from %IP% to any port 2222
    tcpflags    = syn

[closeSSH]
    sequence    = 9000,8000,7000
    seq_timeout = 5
    command     = /usr/sbin/ufw delete allow from %IP% to any port 2222
    tcpflags    = syn
EOF

# Enable
systemctl enable knockd
systemctl start knockd

# To connect:
# 1. Knock ports: nc -z server.com 7000 8000 9000
# 2. SSH immediately: ssh -p 2222 user@server.com
# 3. Close: nc -z server.com 9000 8000 7000
```

### Fail2Ban

Automatically ban IPs that have too many failed login attempts.

```bash
# Install
apt install fail2ban

# Configure
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime  = 1h
findtime = 10m
maxretry = 5
destemail = you@example.com
sendername = Fail2Ban
action = %(action_mwl)s

[sshd]
enabled = true
port = 2222
logpath = /var/log/auth.log
maxretry = 3

[nginx-http-auth]
enabled = true
logpath = /var/log/nginx/error.log

[nginx-limit-req]
enabled = true
logpath = /var/log/nginx/error.log
EOF

# Start
systemctl enable fail2ban
systemctl start fail2ban

# Check status
fail2ban-client status

# See banned IPs
fail2ban-client status sshd

# Unban an IP
fail2ban-client set sshd unbanip 1.2.3.4
```

### Secrets Management

**Never store secrets in:**
- Git repositories
- Plain text files with wide permissions
- Environment variables visible to all users
- Application code

**Better Approaches:**

**1. Environment files with strict permissions:**
```bash
# Store secrets
cat > /etc/myapp/secrets.env << 'EOF'
DB_PASSWORD=actuallySecure123
API_KEY=sk_live_realkey
JWT_SECRET=randomlongstring
EOF

# Strict permissions (only root and app user can read)
chown root:myapp /etc/myapp/secrets.env
chmod 640 /etc/myapp/secrets.env

# Load in systemd
[Service]
EnvironmentFile=/etc/myapp/secrets.env
```

**2. HashiCorp Vault** (for multiple servers):
```bash
# App fetches secrets at startup
VAULT_TOKEN=s.abc123 vault kv get secret/myapp/database

# Vault features:
# - Secrets encrypted at rest
# - Access logged
# - Secrets can be rotated
# - Temporary credentials
```

**3. Cloud provider secret managers:**
- AWS Secrets Manager
- GCP Secret Manager
- Azure Key Vault

### Principle of Least Privilege

**Give minimum necessary permissions.**

**Bad:**
```bash
# Everything runs as root
sudo node app.js
```

**Good:**
```bash
# Dedicated user for each service
useradd --system --no-create-home --shell /bin/false webapp
useradd --system --no-create-home --shell /bin/false worker

# Each service runs as its own user
[Service]
User=webapp
ExecStart=/usr/bin/node /opt/webapp/app.js

[Service]
User=worker
ExecStart=/usr/bin/node /opt/worker/worker.js
```

**File Permissions:**

```bash
# App can read, not write
chown root:webapp /opt/webapp
chmod 750 /opt/webapp

# Config can read, not write
chown root:webapp /etc/webapp/config.json
chmod 640 /etc/webapp/config.json

# Logs can write
chown webapp:webapp /var/log/webapp
chmod 750 /var/log/webapp

# Secrets can only read
chown root:webapp /etc/webapp/secrets.env
chmod 640 /etc/webapp/secrets.env
```

**systemd Sandboxing:**

```ini
[Service]
User=webapp
Group=webapp

# Restrict filesystem
PrivateTmp=yes
ProtectSystem=strict
ProtectHome=yes
ReadWritePaths=/var/log/webapp

# Restrict network
RestrictAddressFamilies=AF_INET AF_INET6

# Restrict capabilities
NoNewPrivileges=yes
CapabilityBoundingSet=

# Restrict syscalls
SystemCallFilter=@system-service
SystemCallErrorNumber=EPERM
```

### What Actually Happens Under the Hood

**When an SSH brute force attack happens:**

```
1. Attacker tries to connect on port 22
   - Firewall: Blocked (you changed port to 2222)
   OR
   - Port 22 open: Connection allowed

2. Attacker tries passwords:
   user: root, password: admin
   user: root, password: 123456
   user: root, password: password

3. SSH logs failures to /var/log/auth.log

4. Fail2Ban monitors logs:
   - Sees 3 failed attempts in 10 minutes
   - Adds iptables rule to ban IP

5. Next attempt:
   - Firewall drops packets from that IP
   - Attacker sees connection timeout

6. After 1 hour, Fail2Ban unbans IP

7. If attacker tries again, cycle repeats
```

**When your app reads a secret:**

```
1. systemd starts service

2. Reads EnvironmentFile=/etc/myapp/secrets.env
   - Checks permissions (must be readable by service user)
   - Loads key=value pairs into environment

3. Starts process with environment variables set

4. App reads process.env.DB_PASSWORD

5. Secret is only in:
   - Process memory (not on disk)
   - Environment (not visible to other users)

6. When process ends, secret is gone from memory
```

### Common Beginner Mistakes

**Mistake 1: Committing secrets to git**
```bash
# Check before committing
git diff --cached

# If you accidentally committed a secret:
# 1. Change the secret immediately
# 2. Remove from git history (complex, use git-filter-branch or BFG)
# 3. Force push (breaks for collaborators)

# Better: Use .gitignore from the start
echo ".env" >> .gitignore
echo "secrets/" >> .gitignore
```

**Mistake 2: Running everything as root**
```bash
# Bad
sudo docker run myapp

# Good
# Create user, run as that user
# Docker: USER directive in Dockerfile
# Systemd: User= in service file
```

**Mistake 3: Exposing unnecessary services**
```bash
# Bad - database accessible from internet
# In docker-compose.yml:
postgres:
  ports:
    - "5432:5432"

# Good - only accessible within Docker network
postgres:
  # No ports section
```

**Mistake 4: Weak SSH keys**
```bash
# Bad
ssh-keygen -t rsa -b 1024

# Good
ssh-keygen -t ed25519 -a 100
```

**Mistake 5: Not updating packages**
```bash
# Bad: Never update
# Security vulnerabilities pile up

# Good: Regular updates
apt update && apt upgrade -y

# Set up automatic security updates
apt install unattended-upgrades
dpkg-reconfigure --priority=low unattended-upgrades
```

### Hands-On Practice Tasks

**Task 1: Harden SSH**
```bash
# Create non-root user
adduser deploy
usermod -aG sudo deploy

# Set up key-based auth
# (On local machine)
ssh-keygen -t ed25519
ssh-copy-id deploy@your-server.com

# Test key login
ssh deploy@your-server.com

# Edit SSH config (as root)
vim /etc/ssh/sshd_config
# Change: PermitRootLogin no
# Change: PasswordAuthentication no

# Restart SSH
systemctl restart sshd

# From new terminal, verify you can still login with key
ssh deploy@your-server.com

# Try to login with password (should fail)
```

**Task 2: Set up Fail2Ban**
```bash
apt install fail2ban

# Check it's watching SSH
fail2ban-client status

# Simulate attack (from another machine)
# Try to SSH with wrong password 5 times

# Check if banned
fail2ban-client status sshd

# Check logs
tail -f /var/log/fail2ban.log
```

**Task 3: Lock down a service**
```bash
# Create restricted user
useradd --system --shell /bin/false webapp

# Create app directory
mkdir /opt/webapp
chown root:webapp /opt/webapp
chmod 750 /opt/webapp

# Create secrets file
cat > /etc/webapp/secrets.env << 'EOF'
SECRET_KEY=test123
EOF

chown root:webapp /etc/webapp/secrets.env
chmod 640 /etc/webapp/secrets.env

# Try to read as different user (should fail)
sudo -u nobody cat /etc/webapp/secrets.env

# Create systemd service
cat > /etc/systemd/system/webapp.service << 'EOF'
[Service]
User=webapp
EnvironmentFile=/etc/webapp/secrets.env
ExecStart=/usr/bin/node /opt/webapp/app.js
ProtectSystem=strict
ProtectHome=yes
PrivateTmp=yes
EOF
```

### Mini Project: Secure Production Server

**Goal:** Harden a fresh VPS to production standards.

```bash
#!/bin/bash
# secure-server.sh

set -e

echo "=== Securing Server ==="

# Update system
apt update && apt upgrade -y

# Create deploy user
useradd -m -s /bin/bash -G sudo deploy
echo "deploy:$(openssl rand -base64 32)" | chpasswd

# Set up SSH keys (add your public key)
mkdir -p /home/deploy/.ssh
echo "YOUR_PUBLIC_KEY_HERE" > /home/deploy/.ssh/authorized_keys
chown -R deploy:deploy /home/deploy/.ssh
chmod 700 /home/deploy/.ssh
chmod 600 /home/deploy/.ssh/authorized_keys

# Harden SSH
cat >> /etc/ssh/sshd_config << 'EOF'

# Security hardening
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
PrintMotd no
AcceptEnv LANG LC_*
Subsystem sftp /usr/lib/openssh/sftp-server
MaxAuthTries 3
LoginGraceTime 20
AllowUsers deploy
EOF

systemctl restart sshd

# Set up firewall
ufw default deny incoming
ufw default allow outgoing
ufw allow 22/tcp  # SSH (change if you changed port)
ufw allow 80/tcp  # HTTP
ufw allow 443/tcp # HTTPS
ufw --force enable

# Install and configure Fail2Ban
apt install -y fail2ban
cat > /etc/fail2ban/jail.local << 'EOF'
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 3

[sshd]
enabled = true
EOF

systemctl enable fail2ban
systemctl start fail2ban

# Disable unnecessary services
systemctl disable bluetooth
systemctl stop bluetooth

# Set up automatic security updates
apt install -y unattended-upgrades
cat > /etc/apt/apt.conf.d/50unattended-upgrades << 'EOF'
Unattended-Upgrade::Allowed-Origins {
    "${distro_id}:${distro_codename}-security";
};
Unattended-Upgrade::Automatic-Reboot "false";
EOF

# Install security tools
apt install -y \
    ufw \
    fail2ban \
    aide \
    rkhunter \
    logwatch

# Set up log rotation
cat > /etc/logrotate.d/custom << 'EOF'
/var/log/custom/*.log {
    daily
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 root adm
}
EOF

# Create secure directory structure
mkdir -p /opt/apps/{webapp,api,worker}
mkdir -p /etc/apps/{webapp,api,worker}
mkdir -p /var/log/apps/{webapp,api,worker}

# Set ownership
chown root:root /opt/apps
chmod 755 /opt/apps

# Create app users
for app in webapp api worker; do
    useradd --system --no-create-home --shell /bin/false $app
    chown root:$app /opt/apps/$app
    chmod 750 /opt/apps/$app
    chown root:$app /etc/apps/$app
    chmod 750 /etc/apps/$app
    chown $app:$app /var/log/apps/$app
    chmod 750 /var/log/apps/$app
done

echo "=== Server Secured ==="
echo "Next steps:"
echo "1. Test SSH login with key as deploy user"
echo "2. Verify firewall rules: ufw status"
echo "3. Check fail2ban: fail2ban-client status"
echo "4. Set up monitoring"
```

Run this script on a fresh VPS to harden it.

---

## 14. From VPS to Cloud (Mental Transition)

### Mapping VPS Concepts to AWS/GCP

You've been working with a VPS. The cloud is the same concepts, but abstracted and managed.

**Mental Map:**

| VPS Concept | AWS | GCP | What Changed |
|-------------|-----|-----|--------------|
| VPS | EC2 Instance | Compute Engine | Same: A virtual server |
| Storage/Disk | EBS Volume | Persistent Disk | Same: Block storage |
| Files | S3 Bucket | Cloud Storage | New: Object storage, not filesystem |
| VPS IP | Elastic IP | Static IP | Same: Public IP address |
| Load Balancer | ALB/ELB | Cloud Load Balancing | New: Managed, highly available |
| DNS | Route53 | Cloud DNS | Same: Manage DNS records |
| Firewall (ufw) | Security Groups | Firewall Rules | Same concept, managed UI |
| Systemd service | ECS/Fargate | Cloud Run | New: Managed container orchestration |
| Docker Compose | ECS | GKE | New: Managed Kubernetes |
| PostgreSQL (self-hosted) | RDS | Cloud SQL | New: Managed database |
| Redis (self-hosted) | ElastiCache | Memorystore | New: Managed cache |
| Cron jobs | EventBridge | Cloud Scheduler | New: Managed scheduling |
| Monitoring (Prometheus) | CloudWatch | Cloud Monitoring | New: Managed monitoring |

### What Cloud Actually Abstracts

**VPS:**
```
You manage:
- OS updates
- Security patches
- Service restarts
- Backups
- Scaling (manually)
- Monitoring setup
- Log aggregation

You get:
- Full control
- Lower cost
- Simple mental model
```

**Managed Cloud Services:**
```
Cloud manages:
- OS updates (transparent)
- Security patches (automatic)
- Service restarts (automatic)
- Backups (automated, point-in-time)
- Scaling (automatic, based on load)
- Monitoring (built-in)
- Log aggregation (built-in)

You get:
- Less operational burden
- Higher availability (multi-AZ)
- Pay more
- Less control
- Vendor lock-in
```

**Example: Database**

**Self-Hosted on VPS:**
```bash
# Install PostgreSQL
apt install postgresql

# Configure replication manually
# Set up backups with cron + pg_dump
# Monitor with Prometheus
# Handle failover manually
# Scale by provisioning larger VPS

Total work: Days to set up, ongoing maintenance
Cost: $20/month (VPS)
```

**AWS RDS:**
```bash
# Click "Create Database" in console
# Select multi-AZ for high availability
# Automated backups enabled by default
# Monitoring built-in to CloudWatch
# Failover automatic
# Scale with a click or API call

Total work: 10 minutes to set up, minimal maintenance
Cost: $50-200/month depending on size
```

**Trade-off:**
- VPS: More work, cheaper, full control
- RDS: Less work, expensive, less control

### Common Cloud Misconceptions

**Misconception 1: "Cloud is always cheaper"**

False. For a single VPS workload, a VPS is usually cheaper.

```
VPS: $10/month gets you 2 CPU, 4GB RAM
AWS EC2 equivalent: $30-50/month
```

Cloud becomes cost-effective when you need:
- Auto-scaling (handle traffic spikes)
- High availability (multi-region)
- Managed services (databases, caches, queues)

**Misconception 2: "Cloud means no servers"**

"Serverless" is a marketing term. There are always servers, you just don't manage them.

```
AWS Lambda = Code runs on AWS-managed servers
"Serverless" just means you don't provision/manage the servers
```

**Misconception 3: "Cloud is more secure"**

Cloud is secure by default, but you can misconfigure it:

```bash
# Misconfigured S3 bucket = data leak
# Open security group = exposed database
# No MFA = compromised account

Cloud gives you tools, but you must use them correctly.
```

**Misconception 4: "Moving to cloud is easy"**

Migrating from VPS to cloud is a project, not a task:

```
1. Redesign architecture for cloud (can't just lift-and-shift everything)
2. Learn cloud-specific concepts (VPCs, IAM, security groups)
3. Rewrite deployment pipelines
4. Train team
5. Manage costs (easy to overspend)

Timeline: Weeks to months, not days
```

### When to Use VPS vs Cloud

**Use VPS when:**
- Starting out (learn fundamentals first)
- Single server is enough
- Budget is tight
- Simple application
- You want full control
- Predictable, low traffic

**Use Cloud when:**
- Need auto-scaling
- Need high availability (multi-region)
- Need managed services (databases, queues, caches)
- Large team (cloud abstractions reduce coordination)
- Unpredictable traffic
- Compliance requirements (certifications)

**Hybrid Approach:**
```
Common pattern:
- Application servers: Cloud (auto-scale)
- Database: Cloud managed service (RDS, Cloud SQL)
- Object storage: Cloud (S3, Cloud Storage)
- Background jobs: VPS (cheap, predictable)
- Dev/staging: VPS (cost savings)
```

### What Actually Happens Under the Hood

**VPS:**
```
Physical server
    └─ Hypervisor (KVM, Xen)
        └─ Your VPS
            └─ Your OS
                └─ Your apps
```

**Cloud (EC2/Compute Engine):**
```
Physical server
    └─ AWS/GCP Hypervisor
        └─ Your EC2 instance
            └─ Your OS
                └─ Your apps

Same architecture, but:
- AWS manages hypervisor
- AWS handles hardware failures
- AWS provides API to manage instances
- AWS monitors and replaces failed hardware
```

**Serverless (Lambda/Cloud Functions):**
```
Physical server
    └─ AWS manages
        └─ Container with runtime
            └─ Your function code

Differences:
- Container starts on demand
- Stops after execution
- You pay per invocation
- Cold starts (latency on first call)
```

### Hands-On: Map VPS to Cloud

**Your VPS Setup:**
```bash
# What you have:
- 1 VPS
- Nginx (reverse proxy)
- Node.js app
- PostgreSQL database
- Redis cache
- Let's Encrypt SSL
- Systemd services
```

**Equivalent AWS Architecture:**
```
- EC2 instance (VPS equivalent)
- Application Load Balancer (Nginx equivalent)
- Node.js app in Docker on ECS
- RDS PostgreSQL (managed database)
- ElastiCache Redis (managed cache)
- ACM certificate (SSL)
- Auto Scaling Group (replaces systemd restart logic)
```

**Equivalent GCP Architecture:**
```
- Compute Engine instance (VPS equivalent)
- Cloud Load Balancing
- Cloud Run (serverless containers)
- Cloud SQL PostgreSQL
- Memorystore Redis
- Google-managed SSL
- Cloud Run auto-scaling (built-in)
```

### Common Beginner Mistakes

**Mistake 1: Starting with cloud before learning VPS**
```
Without VPS experience:
- Don't understand what cloud abstracts
- Don't know what's happening under the hood
- Can't debug when abstractions leak
- Waste money on unnecessary managed services

Learn VPS first, then cloud makes sense.
```

**Mistake 2: Treating cloud like a VPS**
```bash
# Bad: SSH into EC2 instance and manually configure
ssh ec2-user@instance
sudo apt install stuff
# This defeats the purpose of cloud

# Good: Use infrastructure as code (Terraform, CloudFormation)
# Instances should be immutable and replaceable
```

**Mistake 3: Not using cloud-native features**
```
Using EC2 like a VPS:
- Manually backing up with cron
- Manually scaling
- Manually setting up monitoring

Using cloud properly:
- Automated backups (built-in)
- Auto Scaling Groups
- CloudWatch (built-in monitoring)
```

**Mistake 4: Cost ignorance**
```bash
# Easy to rack up huge bills
# - Left test instances running
# - Didn't configure auto-scaling properly
# - Used expensive services for dev/test

# Set up billing alerts!
# Use AWS Budgets, GCP Budget Alerts
```

---

## FINAL PROJECT: Complete End-to-End DevOps System

### The Challenge

Build and deploy a complete production-ready application with:
- Frontend (React)
- Backend API (Node.js)
- Database (PostgreSQL)
- Cache (Redis)
- Background worker
- Full CI/CD pipeline
- Monitoring & alerting
- Proper security
- Zero-downtime deployments

### Architecture

```
                                    Internet
                                        |
                                        v
                                  Cloudflare CDN
                                  (caching, DDoS)
                                        |
                                        v
                    +-------------------+-------------------+
                    |                                       |
                    v                                       v
              [HTTP/HTTPS]                           [WebSocket]
                    |                                       |
                    v                                       v
            +---------------+                       +--------------+
            |   Nginx       |                       |  Nginx       |
            | (Port 80/443) |                       | (Port 3001)  |
            +-------+-------+                       +------+-------+
                    |                                      |
          +---------+---------+                            |
          |                   |                            |
          v                   v                            v
    +----------+        +----------+              +----------------+
    |  Web     |        |  API     |              | WebSocket      |
    | (static) |        | (Node.js)|              | Server         |
    +----------+        +-----+----+              +----------------+
                              |                            |
                    +---------+----------+--------+--------+
                    |         |          |        |
                    v         v          v        v
            +----------+ +-------+  +--------+ +--------+
            |PostgreSQL| | Redis |  | Worker | | Queue  |
            |  (Primary)| |(Cache)|  |(Node.js)| |(Bull)|
            +----+-----+ +-------+  +--------+ +--------+
                 |
                 v
         +---------------+
         | PostgreSQL    |
         | (Replica)     |
         | (Read-only)   |
         +---------------+

         Monitoring Stack:
         +------------+     +----------+     +-----------+
         | Prometheus | <-- | Node     | --> | Grafana   |
         | (Metrics)  |     | Exporter |     | (Visualize)|
         +------------+     +----------+     +-----------+
                |
                v
         +-------------+
         | Alertmanager|
         | (Alerts)    |
         +-------------+
```

### Step-by-Step Implementation

**Step 1: Set Up Infrastructure**

```bash
# Create project structure
mkdir fullstack-devops
cd fullstack-devops

# Directory structure
mkdir -p {frontend,backend,worker,nginx,monitoring,terraform}

# Initialize git
git init
cat > .gitignore << 'EOF'
node_modules/
.env
*.log
terraform.tfstate*
.terraform/
dist/
build/
EOF
```

**Step 2: Backend API**

```javascript
// backend/src/app.js
const express = require('express');
const { Pool } = require('pg');
const Redis = require('ioredis');
const Bull = require('bull');
const prometheus = require('prom-client');

const app = express();
app.use(express.json());

// Prometheus metrics
const register = new prometheus.Registry();
prometheus.collectDefaultMetrics({ register });

const httpRequestDuration = new prometheus.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.1, 0.3, 0.5, 0.7, 1, 3, 5, 7, 10]
});
register.registerMetric(httpRequestDuration);

// Database connection pool
const pool = new Pool({
  host: process.env.DB_HOST || 'localhost',
  port: process.env.DB_PORT || 5432,
  database: process.env.DB_NAME || 'myapp',
  user: process.env.DB_USER || 'postgres',
  password: process.env.DB_PASSWORD,
  max: 20,
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Redis client
const redis = new Redis({
  host: process.env.REDIS_HOST || 'localhost',
  port: process.env.REDIS_PORT || 6379,
  retryStrategy: (times) => Math.min(times * 50, 2000),
});

// Job queue
const emailQueue = new Bull('email', {
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
  },
});

// Middleware: Request timing
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = (Date.now() - start) / 1000;
    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode)
      .observe(duration);
  });
  next();
});

// Health check endpoint
app.get('/health', async (req, res) => {
  const checks = {
    server: 'ok',
    database: 'checking',
    redis: 'checking',
  };

  try {
    await pool.query('SELECT 1');
    checks.database = 'ok';
  } catch (err) {
    checks.database = 'error';
  }

  try {
    await redis.ping();
    checks.redis = 'ok';
  } catch (err) {
    checks.redis = 'error';
  }

  const allOk = Object.values(checks).every(v => v === 'ok');
  res.status(allOk ? 200 : 503).json(checks);
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.end(await register.metrics());
});

// API endpoints
app.get('/api/users', async (req, res) => {
  try {
    // Try cache first
    const cached = await redis.get('users');
    if (cached) {
      return res.json({ source: 'cache', data: JSON.parse(cached) });
    }

    // Query database
    const result = await pool.query('SELECT id, name, email FROM users LIMIT 100');

    // Cache for 60 seconds
    await redis.setex('users', 60, JSON.stringify(result.rows));

    res.json({ source: 'database', data: result.rows });
  } catch (err) {
    console.error('Error fetching users:', err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.post('/api/users', async (req, res) => {
  try {
    const { name, email } = req.body;

    const result = await pool.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [name, email]
    );

    // Invalidate cache
    await redis.del('users');

    // Queue welcome email
    await emailQueue.add({
      to: email,
      subject: 'Welcome!',
      body: `Hello ${name}, welcome to our app!`,
    });

    res.status(201).json(result.rows[0]);
  } catch (err) {
    console.error('Error creating user:', err);
    res.status(500).json({ error: 'Internal server error' });
  }
});

// Graceful shutdown
const server = app.listen(process.env.PORT || 3000, () => {
  console.log(`Server running on port ${process.env.PORT || 3000}`);
});

process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully');

  server.close(async () => {
    await pool.end();
    await redis.quit();
    await emailQueue.close();
    console.log('Closed all connections');
    process.exit(0);
  });

  // Force close after 10 seconds
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 10000);
});
```

**Step 3: Background Worker**

```javascript
// worker/src/worker.js
const Bull = require('bull');
const nodemailer = require('nodemailer');

const emailQueue = new Bull('email', {
  redis: {
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
  },
});

// Email transporter
const transporter = nodemailer.createTransporter({
  host: process.env.SMTP_HOST,
  port: process.env.SMTP_PORT,
  auth: {
    user: process.env.SMTP_USER,
    pass: process.env.SMTP_PASS,
  },
});

// Process email jobs
emailQueue.process(async (job) => {
  console.log(`Processing email job ${job.id}`);

  const { to, subject, body } = job.data;

  try {
    await transporter.sendMail({
      from: process.env.EMAIL_FROM,
      to,
      subject,
      text: body,
    });

    console.log(`Email sent to ${to}`);
    return { success: true };
  } catch (err) {
    console.error(`Failed to send email to ${to}:`, err);
    throw err; // Retry
  }
});

emailQueue.on('completed', (job, result) => {
  console.log(`Job ${job.id} completed`);
});

emailQueue.on('failed', (job, err) => {
  console.error(`Job ${job.id} failed:`, err);
});

console.log('Worker started, waiting for jobs...');

// Graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down worker');
  await emailQueue.close();
  process.exit(0);
});
```

**Step 4: Docker Compose (Development)**

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: devpassword
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./backend/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5

  backend:
    build: ./backend
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=development
      - DB_HOST=postgres
      - DB_PASSWORD=devpassword
      - REDIS_HOST=redis
    volumes:
      - ./backend/src:/app/src
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy

  worker:
    build: ./worker
    environment:
      - NODE_ENV=development
      - REDIS_HOST=redis
      - SMTP_HOST=${SMTP_HOST}
      - SMTP_PORT=${SMTP_PORT}
      - SMTP_USER=${SMTP_USER}
      - SMTP_PASS=${SMTP_PASS}
    depends_on:
      - redis

  frontend:
    build: ./frontend
    ports:
      - "8080:80"
    depends_on:
      - backend

  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./monitoring/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus

  grafana:
    image: grafana/grafana
    ports:
      - "3001:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana_data:/var/lib/grafana

volumes:
  postgres_data:
  prometheus_data:
  grafana_data:
```

**Step 5: CI/CD Pipeline**

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json

      - name: Install dependencies
        run: |
          cd backend
          npm ci

      - name: Run linter
        run: |
          cd backend
          npm run lint

      - name: Run tests
        run: |
          cd backend
          npm test

      - name: Run security audit
        run: |
          cd backend
          npm audit --production --audit-level=high

  build:
    needs: test
    runs-on: ubuntu-latest
    strategy:
      matrix:
        service: [backend, worker, frontend]
    steps:
      - uses: actions/checkout@v3

      - name: Log in to Container Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v4
        with:
          context: ./${{ matrix.service }}
          push: true
          tags: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}:${{ github.sha }}
          cache-from: type=registry,ref=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}-${{ matrix.service }}:latest
          cache-to: type=inline

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://app.example.com
    steps:
      - uses: actions/checkout@v3

      - name: Deploy to production
        env:
          SSH_KEY: ${{ secrets.PRODUCTION_SSH_KEY }}
          HOST: ${{ secrets.PRODUCTION_HOST }}
        run: |
          mkdir -p ~/.ssh
          echo "$SSH_KEY" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa

          # Deploy script
          ssh -i ~/.ssh/id_rsa -o StrictHostKeyChecking=no deploy@$HOST << 'ENDSSH'
            set -e

            cd /opt/production

            # Pull latest images
            echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin

            export IMAGE_TAG=${{ github.sha }}

            # Update services with zero downtime
            docker compose -f docker-compose.prod.yml pull
            docker compose -f docker-compose.prod.yml up -d --no-deps --build

            # Health check
            sleep 10
            curl -f http://localhost:3000/health || exit 1

            # Cleanup old images
            docker image prune -af --filter "until=24h"

            echo "✓ Deployment successful"
ENDSSH

      - name: Notify deployment
        if: always()
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          webhook_url: ${{ secrets.SLACK_WEBHOOK }}
```

**Step 6: Production Deployment Script**

```bash
#!/bin/bash
# deploy.sh - Run on production server

set -e

echo "=== Deploying to Production ==="

# Configuration
REPO_URL="https://github.com/yourusername/fullstack-devops.git"
DEPLOY_DIR="/opt/production"
BACKUP_DIR="/opt/backups"
IMAGE_TAG=${IMAGE_TAG:-latest}

# Create backup
echo "Creating backup..."
BACKUP_FILE="$BACKUP_DIR/backup-$(date +%Y%m%d-%H%M%S).tar.gz"
tar -czf $BACKUP_FILE $DEPLOY_DIR 2>/dev/null || true

# Pull latest code (if not using images)
cd $DEPLOY_DIR
git pull origin main

# Pull latest Docker images
docker compose -f docker-compose.prod.yml pull

# Run database migrations
docker compose -f docker-compose.prod.yml run --rm backend npm run migrate

# Deploy with zero downtime
echo "Deploying services..."

# Start new backend containers
docker compose -f docker-compose.prod.yml up -d --no-deps --scale backend=2 backend

# Wait for health checks
sleep 10
for i in {1..30}; do
  if curl -sf http://localhost:3000/health > /dev/null; then
    echo "Backend healthy"
    break
  fi
  echo "Waiting for backend..."
  sleep 2
done

# Scale down old containers
docker compose -f docker-compose.prod.yml up -d --no-deps --scale backend=1 backend

# Deploy worker and frontend
docker compose -f docker-compose.prod.yml up -d worker frontend

# Cleanup
echo "Cleaning up..."
docker image prune -af
docker volume prune -f

# Verify deployment
echo "Verifying deployment..."
curl -f http://localhost:3000/health || {
  echo "❌ Health check failed, rolling back..."
  tar -xzf $BACKUP_FILE -C /
  docker compose -f docker-compose.prod.yml up -d
  exit 1
}

echo "✅ Deployment successful!"

# Send notification
curl -X POST $SLACK_WEBHOOK -H 'Content-Type: application/json' -d '{
  "text": "✅ Production deployment successful"
}' || true
```

### What Breaks First in Production (and Why)

**1. Database Connections Exhausted**
```
Symptom: "Error: Too many clients"
Why: Connection pool too small for traffic
Fix:
- Increase pool size in backend
- Use connection pooler (PgBouncer)
- Scale backend instances
```

**2. Memory Leak**
```
Symptom: Container OOMKilled, restart loop
Why: JavaScript closures holding references
Debug:
- docker stats (watch memory)
- node --inspect for heap snapshots
- Set memory limits in docker-compose
```

**3. Disk Full**
```
Symptom: ENOSPC errors, database can't write
Why: Logs or temp files filling disk
Fix:
- Log rotation (logrotate)
- Clean up old Docker images/volumes
- Monitor disk usage (node_exporter)
```

**4. SSL Certificate Expired**
```
Symptom: Browser shows SSL error
Why: Certbot renewal failed
Debug:
- Check /var/log/letsencrypt/
- Test renewal: certbot renew --dry-run
- Check Certbot systemd timer
```

**5. Database Replica Lag**
```
Symptom: Users see stale data
Why: Writes to primary, reads from lagging replica
Debug:
- Check replication lag in Postgres
- Use read-your-writes pattern
- Route critical reads to primary
```

**6. Cache Stampede**
```
Symptom: Sudden load spike, slow responses
Why: Popular cache key expired, all requests hit DB
Fix:
- Stagger cache expiration times
- Use probabilistic early expiration
- Implement cache warming
```

### Debugging Failures Calmly

**Step 1: Don't Panic**

Take a breath. Production issues are normal. You have backups and rollback plans.

**Step 2: Assess Impact**

```bash
# Is the service down completely?
curl https://yourapp.com/health

# What's the error rate?
# Check Grafana dashboard

# How many users affected?
# Check metrics
```

**Step 3: Check Recent Changes**

```bash
# What was deployed recently?
git log --oneline -5

# Check deployment logs
journalctl -u your-app -n 100

# Check CI/CD run history
```

**Step 4: Gather Information**

```bash
# Application logs
docker compose logs --tail=100 backend

# System logs
journalctl -xe

# Resource usage
docker stats
df -h  # Disk space
free -h  # Memory

# Database status
docker compose exec postgres psql -U postgres -c "SELECT count(*) FROM pg_stat_activity"

# Check for connection issues
netstat -an | grep 3000
```

**Step 5: Form Hypothesis**

Based on symptoms:
- Slow responses → Database overloaded? Memory leak?
- 500 errors → Application crash? Database connection issue?
- Timeout → Network issue? External service down?

**Step 6: Test Hypothesis**

```bash
# Is database slow?
docker compose exec postgres psql -U postgres -c "SELECT * FROM pg_stat_activity WHERE state = 'active'"

# Is memory exhausted?
docker stats | grep backend

# Is external service down?
curl -v https://external-api.com/health
```

**Step 7: Fix or Rollback**

```bash
# If fix is obvious and quick:
# - Restart service
# - Increase resources
# - Fix configuration

# If not obvious, rollback:
cd /opt/production
git checkout <previous-good-commit>
./deploy.sh

# Or use tagged release
git checkout tags/v1.2.3
./deploy.sh
```

**Step 8: Verify Fix**

```bash
# Check health
curl https://yourapp.com/health

# Check metrics (error rate should drop)
# Check logs (errors should stop)

# Monitor for 15 minutes to ensure stability
```

**Step 9: Post-Mortem**

After the fire is out, document:

```markdown
## Incident Post-Mortem: 2026-02-07 Database Connection Issue

### What Happened
At 14:30, error rate spiked to 25%. Users saw 500 errors.

### Root Cause
Database connection pool (max 20) exhausted due to slow queries.
Backend couldn't acquire connections, timed out.

### Detection
- Grafana alert fired at 14:31
- Slack notification received
- Engineer checked logs within 2 minutes

### Resolution
- Increased connection pool from 20 to 50 (14:35)
- Restarted backend services (14:37)
- Error rate returned to normal (14:40)

### Total Downtime
10 minutes

### Action Items
- [ ] Add alert for connection pool utilization
- [ ] Optimize slow queries identified in logs
- [ ] Document connection pool sizing
- [ ] Add query timeout to prevent connection hogging

### Timeline
14:30 - Issue started
14:31 - Alert fired
14:33 - Engineer investigated
14:35 - Fix deployed
14:40 - Confirmed resolved
```

---

## Conclusion

You've now covered the complete DevOps journey from first principles:

1. ✅ **DevOps Philosophy** - Ownership and responsibility
2. ✅ **Linux Fundamentals** - The foundation of everything
3. ✅ **Networking** - How requests reach your server
4. ✅ **Process Management** - Running applications reliably
5. ✅ **Reverse Proxies** - Routing traffic with Nginx
6. ✅ **HTTPS & Security** - Protecting user data
7. ✅ **Git & Deployment** - Version control and releases
8. ✅ **CI/CD** - Automating the boring stuff
9. ✅ **Containers** - Consistent environments
10. ✅ **Orchestration** - Managing multiple services
11. ✅ **Infrastructure as Code** - Reproducible systems
12. ✅ **Monitoring** - Knowing when things break
13. ✅ **Security** - Keeping bad actors out
14. ✅ **Cloud Transition** - Scaling beyond one server

**What You Should Do Next:**

1. **Build the final project** - Actually deploy it, break it, fix it
2. **Run a service for 90 days** - Experience operational burden
3. **Get paged at 3am** - Nothing teaches like production fires
4. **Read post-mortems** - Learn from others' mistakes
5. **Automate what annoys you** - If you do it twice, script it

**The Real Learning:**

DevOps isn't about tools (Kubernetes, Terraform, Docker). It's about:
- Understanding systems deeply
- Debugging with confidence
- Automating repetitively
- Monitoring proactively
- Responding to incidents calmly
- Learning from failures

You now have the fundamentals. Everything else is just elaboration on these concepts.

**Go break things. Then fix them. That's how you learn.**

---

*This handbook is a living document. As you gain experience, you'll return to these concepts with deeper understanding. Each production incident will teach you something these pages cannot. Good luck, and remember: every senior engineer was once confused by DNS propagation too.*

