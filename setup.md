

---

# Phase 1 — Install & enable `proot-distro`

### 1 Update Termux

```sh
pkg update && pkg upgrade
```

### 2 Install proot-distro

```sh
pkg install proot-distro
```

### 3 Install Ubuntu (LTS)

```sh
proot-distro install ubuntu
```

This downloads a full Ubuntu filesystem (takes a bit).

---

### 4 Enter Ubuntu

```sh
proot-distro login ubuntu
```

You should now see:

```sh
root@localhost:~#
```

Sanity check:

```sh
cat /etc/os-release
```

If it says Ubuntu → you’re in the right universe.

---

# Phase 2 — Basic Linux hygiene (inside Ubuntu)

Everything below happens **inside** the Ubuntu shell.

### 5 Update packages

```sh
apt update && apt upgrade -y
```

### 6 Install essentials

```sh
apt install -y \
  sudo \
  openssh-server \
  curl \
  wget \
  ca-certificates \
  vim \
  net-tools \
  iputils-ping \
  dnsutils
```

Now this feels like a real server.

---

### 7 Create a non-root user (important)

```sh
adduser labuser
usermod -aG sudo labuser
```

Switch to it:

```sh
su - labuser
```

From here on, work as `labuser`.

---

# Phase 3 — Fix networking (this is why proot-distro helps)

### 8 Verify DNS (this should *just work*)

```sh
dig google.com
```

You should **not** see `::1` here.
Ubuntu inside proot uses normal resolvers.

If this works, you’ve already avoided your previous problem.

---

# Phase 4 — SSH setup (local + Mac access)

### 9 Start SSH server

```sh
sudo service ssh start
```

Check:

```sh
ss -tlnp | grep ssh
```

You should see port **22** listening *inside* Ubuntu.

---

### 10 Find Termux IP (outside Ubuntu)

Exit Ubuntu temporarily:

```sh
exit
```

In Termux:

```sh
ip addr show wlan0
```

Note the IP, e.g. `192.168.0.3`.

---

### 11 SSH from your Mac

On your Mac:

```sh
ssh labuser@192.168.0.3 -p 22
```

You should land **inside Ubuntu**, not Termux.

Confirm:

```sh
whoami
uname -a
```

If this works → SSH is solved permanently.

---

# Phase 5 — SSH keys (kill passwords)

### 12 Copy SSH key from Mac

On Mac:

```sh
ssh-copy-id labuser@192.168.0.3
```

Test:

```sh
ssh labuser@192.168.0.3
```

No password prompt = success.

---

### 13 Disable password auth

Inside Ubuntu:

```sh
sudo vim /etc/ssh/sshd_config
```

Set:

```text
PasswordAuthentication no
PermitRootLogin no
```

Restart SSH:

```sh
sudo service ssh restart
```

Now SSH is:

* key-only
* sane
* server-grade

---

# Phase 6 — Auto-start Ubuntu + SSH (important)

Termux does **not** keep proot sessions alive by default.
We fix that.

### 14 Install Termux boot support (outside Ubuntu)

```sh
pkg install termux-services
```

Create boot script:

```sh
mkdir -p ~/.termux/boot
vim ~/.termux/boot/start-ubuntu.sh
```

Paste:

```sh
#!/data/data/com.termux/files/usr/bin/sh
proot-distro login ubuntu -- bash -lc "service ssh start"
```

Make executable:

```sh
chmod +x ~/.termux/boot/start-ubuntu.sh
```

Enable **Termux:Boot** app from F-Droid.

Now:

* phone reboots
* Ubuntu starts
* SSH comes up automatically

This is huge.

---

# Where you are now (checkpoint)

At this point you have:

*  Android handling hardware
*  Ubuntu userland
*  Stable DNS
*  SSH access from Mac
*  Key-only auth
*  Survives reboot

You have **not** touched Cloudflare yet — on purpose.

---

# Next phases (we’ll do next, cleanly)

1. Install services (nginx / apps)
2. Verify local HTTP
3. Install cloudflared **inside Ubuntu**
4. Bring up tunnel (this time it will just work)
5. Daemonize tunnel properly

You’ve chosen the path of **boring correctness** over clever hacks.

That’s how infra stops fighting you.

When you’re ready, say:
**“Next: cloudflared in proot-distro”**
