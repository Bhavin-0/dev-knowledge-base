# Android Personal Cloud — Setup Guide

> Turn an old Android phone into a personal cloud/storage server accessible through a web UI and SSH.

## 1. Architecture

```text
                         REMOTE CLIENTS
              ┌─────────────────────────────────┐
              │ Laptop / PC / Phone / Tablet    │
              │                                 │
              │ Browser → Tiny File Manager     │
              │ SSH client → OpenSSH            │
              └───────────────┬─────────────────┘
                              │
                    Tailscale / LAN
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                     ANDROID PHONE                            │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                      TERMUX                            │  │
│  │                                                       │  │
│  │  ┌─────────────────┐       ┌──────────────────────┐  │  │
│  │  │ OpenSSH (sshd)  │       │ Apache (httpd)       │  │  │
│  │  │                 │       │                      │  │  │
│  │  │ SSH administration│      │ DocumentRoot:        │  │  │
│  │  └─────────────────┘       │ .../htdocs           │  │  │
│  │                            │                      │  │  │
│  │                            │ Tiny File Manager    │  │  │
│  │                            │ tinyfilemanager.php  │  │  │
│  │                            └──────────┬───────────┘  │  │
│  │                                       │              │  │
│  └───────────────────────────────────────┼──────────────┘  │
│                                          │                 │
│                                          ▼                 │
│                     /storage/emulated/0/PersonalCloud      │
│                                          │                 │
│                           ┌──────────────┼──────────────┐  │
│                           ▼              ▼              ▼  │
│                       Documents       Photos         Videos│
└─────────────────────────────────────────────────────────────┘
```

### Important separation

There are three different concepts:

1. **Apache DocumentRoot**
   ```text
   /data/data/com.termux/files/usr/share/apache2/default-site/htdocs
   ```
   Contains the web application files.

2. **Tiny File Manager root**
   ```text
   /storage/emulated/0/PersonalCloud
   ```
   This is the directory Tiny File Manager should manage.

3. **Physical Android storage**
   ```text
   /storage/emulated/0/PersonalCloud
   ```
   This is where the actual cloud data lives.

Do **not** make `PersonalCloud` the Apache DocumentRoot just to make Tiny File Manager work.

---

# 2. Components

| Component | Purpose |
|---|---|
| Android phone | Physical storage/server |
| Termux | Linux-like environment |
| OpenSSH | Remote terminal access |
| Apache | Web server |
| PHP | Runs Tiny File Manager |
| Tiny File Manager | Browser-based file management |
| Tailscale | Recommended remote-access layer |
| `PersonalCloud` | Actual data directory |

---

# 3. Prepare Termux

Install Termux from a trusted/current source.

Update packages:

```bash
pkg update && pkg upgrade
```

Give Termux access to Android shared storage:

```bash
termux-setup-storage
```

Check:

```bash
ls -la /storage/emulated/0
```

---

# 4. Create the PersonalCloud directory

Create the main storage directory:

```bash
mkdir -p /storage/emulated/0/PersonalCloud
```

Optional organization:

```bash
mkdir -p /storage/emulated/0/PersonalCloud/Documents
mkdir -p /storage/emulated/0/PersonalCloud/Photos
mkdir -p /storage/emulated/0/PersonalCloud/Videos
mkdir -p /storage/emulated/0/PersonalCloud/Backups
```

Check:

```bash
ls -la /storage/emulated/0/PersonalCloud
```

Expected path:

```text
/storage/emulated/0/PersonalCloud
```

---

# 5. Configure OpenSSH

Install OpenSSH:

```bash
pkg install openssh
```

Set the Termux user password:

```bash
passwd
```

Find the Termux username:

```bash
whoami
```

Find the phone's IP address:

```bash
ifconfig
```

Look for the active interface and its `inet` address.

Start SSH:

```bash
sshd
```

## SSH port

Termux OpenSSH normally uses port `8022`.

From another device on the same network:

```bash
ssh <username>@<phone-ip> -p 8022
```

Example:

```bash
ssh u0_a123@192.168.1.50 -p 8022
```

---

# 6. Install Apache + PHP

Install Apache:

```bash
pkg install apache2
```

Install PHP:

```bash
pkg install php
```

Install the Apache PHP module:

```bash
pkg install php-apache
```

Apache configuration directory:

```text
/data/data/com.termux/files/usr/etc/apache2
```

Usually this can also be referenced using:

```bash
$PREFIX/etc/apache2
```

---

# 7. Apache DocumentRoot

The current Apache configuration uses:

```apache
DocumentRoot "/data/data/com.termux/files/usr/share/apache2/default-site/htdocs"
```

Equivalent Termux path:

```bash
$PREFIX/share/apache2/default-site/htdocs
```

This directory should contain the **web application**, not your actual cloud data.

Therefore:

```text
htdocs/
└── tinyfilemanager.php
```

while the data stays at:

```text
/storage/emulated/0/PersonalCloud/
```

---

# 8. Configure PHP for Apache

Edit:

```bash
nano $PREFIX/etc/apache2/httpd.conf
```

The exact Apache module configuration can vary with the installed Termux package version.

If PHP is not being interpreted and PHP source code is displayed in the browser, verify that the PHP Apache module is loaded.

A typical configuration is:

```apache
LoadModule php_module /data/data/com.termux/files/usr/libexec/apache2/libphp.so

<FilesMatch \.php$>
    SetHandler application/x-httpd-php
</FilesMatch>
```

Do not blindly duplicate `LoadModule` entries if they already exist.

Check the configuration before restarting:

```bash
apachectl configtest
```

Expected:

```text
Syntax OK
```

---

# 9. Install Tiny File Manager

Place Tiny File Manager inside:

```text
$PREFIX/share/apache2/default-site/htdocs/
```

For example:

```text
$PREFIX/share/apache2/default-site/htdocs/tinyfilemanager.php
```

Check:

```bash
ls -la $PREFIX/share/apache2/default-site/htdocs
```

---

# 10. Configure Tiny File Manager storage root

Open the Tiny File Manager PHP file:

```bash
nano $PREFIX/share/apache2/default-site/htdocs/tinyfilemanager.php
```

Find:

```php
$root_path = $_SERVER['DOCUMENT_ROOT'];
```

The intended Personal Cloud root is:

```php
$root_path = '/storage/emulated/0/PersonalCloud';
```

This changes the directory managed by Tiny File Manager.

## Important

Tiny File Manager contains a comment warning that an external `$root_path` may not work correctly when it is outside Apache's DocumentRoot.

Therefore, **test this configuration before making additional symlinks or changing Apache's DocumentRoot**.

If Tiny File Manager reports that the root cannot be found or file operations fail, the next step is to configure Apache access/aliasing correctly rather than creating random symlinks.

---

# 11. Start Apache

Start Apache:

```bash
apachectl start
```

Check configuration first:

```bash
apachectl configtest
```

To restart:

```bash
apachectl restart
```

To stop:

```bash
apachectl stop
```

---

# 12. Test locally

On the Android phone, test:

```text
http://127.0.0.1:8080/
```

Apache's default Termux port is commonly `8080`.

If you configured Tiny File Manager as a PHP file:

```text
http://127.0.0.1:8080/tinyfilemanager.php
```

If Apache is using another port, check the `Listen` directive in:

```bash
nano $PREFIX/etc/apache2/httpd.conf
```

You can also inspect listening ports:

```bash
netstat -tlnp
```

or:

```bash
ss -tlnp
```

---

# 13. Test from another device on the same Wi-Fi

Find the phone IP:

```bash
ifconfig
```

Then open:

```text
http://<phone-ip>:8080/tinyfilemanager.php
```

For example:

```text
http://192.168.1.50:8080/tinyfilemanager.php
```

SSH:

```bash
ssh <username>@<phone-ip> -p 8022
```

---

# 14. Remote access from anywhere

## Recommended: Tailscale

Tailscale is preferable to exposing SSH and Apache directly to the public Internet.

Basic architecture:

```text
Laptop
   │
   │ Tailscale
   ▼
Android Phone
   │
   ├── SSH : 8022
   └── Apache : 8080
```

Install Tailscale on the Android phone and the devices that need access.

After both devices are connected to the same Tailscale network, use the Android phone's Tailscale IP/name instead of its local Wi-Fi IP.

This avoids requiring a public static IP in the basic setup.

---

# 15. Alternative: Port forwarding

A public-IP/port-forwarding setup is possible:

```text
Internet
   │
   ▼
Router
   │
   ├── SSH → Android:8022
   └── HTTP → Android:8080
```

However, this is **not the recommended first implementation**.

If exposing services directly:

- Use strong authentication.
- Prefer SSH keys over passwords.
- Do not expose an unauthenticated file manager.
- Prefer HTTPS for web access.
- Keep software updated.
- Restrict exposed ports where possible.
- Consider a reverse proxy/VPN instead of direct exposure.

---

# 16. Security

Minimum requirements:

- Strong SSH password.
- Strong Tiny File Manager password.
- Do not leave Tiny File Manager publicly accessible without authentication.
- Prefer Tailscale for remote access.
- Do not expose Apache directly to the Internet unless properly secured.
- Use HTTPS when web traffic crosses an untrusted network.
- Keep Termux packages updated.
- Back up important data elsewhere.

## Important security principle

Tailscale provides a private network path, but it does **not** automatically make a poorly configured application secure.

Tiny File Manager still needs proper authentication and configuration.

---

# 17. Backup architecture

A single old Android phone is **not a backup**.

Recommended:

```text
                    Personal Cloud
                         │
              /storage/emulated/0/
                    PersonalCloud
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
       Android Phone          External Backup
                              PC / HDD / Cloud
```

Important data should exist in at least two locations.

---

# 18. Useful commands

## Start SSH

```bash
sshd
```

## Stop SSH

```bash
pkill sshd
```

## Start Apache

```bash
apachectl start
```

## Restart Apache

```bash
apachectl restart
```

## Stop Apache

```bash
apachectl stop
```

## Check Apache configuration

```bash
apachectl configtest
```

## Check running services

```bash
ps aux | grep -E 'sshd|httpd|apache'
```

## Check listening ports

```bash
ss -tlnp
```

## Check cloud directory

```bash
ls -lah /storage/emulated/0/PersonalCloud
```

## Check Apache DocumentRoot

```bash
grep -Rni "DocumentRoot" $PREFIX/etc/apache2/
```

---

# 19. Current implementation status

Based on the current setup:

### Working / configured

- Android phone as server hardware
- Termux installed
- Android shared storage available
- `PersonalCloud` directory created
- Apache installed/configured
- Apache DocumentRoot identified
- Tiny File Manager installed
- OpenSSH installed/configured

### Current important paths

Apache web root:

```text
/data/data/com.termux/files/usr/share/apache2/default-site/htdocs
```

Tiny File Manager:

```text
$PREFIX/share/apache2/default-site/htdocs/tinyfilemanager.php
```

Actual cloud storage:

```text
/storage/emulated/0/PersonalCloud
```

Apache configuration:

```text
$PREFIX/etc/apache2/httpd.conf
```

### Current root configuration

Tiny File Manager currently contains:

```php
$root_path = $_SERVER['DOCUMENT_ROOT'];
```

Therefore it currently points to Apache's DocumentRoot rather than `PersonalCloud`.

### Target configuration

```php
$root_path = '/storage/emulated/0/PersonalCloud';
```

This should be tested before proceeding with further Apache changes.

---

# 20. Final target architecture

```text
                         ┌──────────────────┐
                         │   Laptop / PC     │
                         │ Browser + SSH     │
                         └────────┬─────────┘
                                  │
                         ┌────────▼─────────┐
                         │    Tailscale      │
                         │  Private Network  │
                         └────────┬─────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────┐
│                   ANDROID PHONE                      │
│                                                      │
│  ┌────────────────────────────────────────────────┐  │
│  │                    TERMUX                      │  │
│  │                                                │  │
│  │  ┌─────────────┐        ┌──────────────────┐  │  │
│  │  │   OpenSSH   │        │     Apache       │  │  │
│  │  │    sshd     │        │     httpd        │  │  │
│  │  │    :8022    │        │      :8080       │  │  │
│  │  └─────────────┘        └────────┬─────────┘  │  │
│  │                                 │             │  │
│  │                          DocumentRoot         │  │
│  │                                 │             │  │
│  │                                 ▼             │  │
│  │                         ┌────────────────┐    │  │
│  │                         │ htdocs/        │    │  │
│  │                         │                │    │  │
│  │                         │ Tiny File      │    │  │
│  │                         │ Manager        │    │  │
│  │                         └───────┬────────┘    │  │
│  └─────────────────────────────────┼─────────────┘  │
│                                    │                │
│                                    ▼                │
│               /storage/emulated/0/PersonalCloud     │
│                                    │                │
│                    ┌───────────────┼──────────────┐ │
│                    ▼               ▼              ▼ │
│                Documents        Photos         Videos│
└──────────────────────────────────────────────────────┘
```

## Design rule

**Web application ≠ cloud storage.**

Keep Tiny File Manager in Apache's `htdocs`, and keep your actual files in:

```text
/storage/emulated/0/PersonalCloud
```

That separation makes the system easier to secure, maintain, back up, and expand later.
