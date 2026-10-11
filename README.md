# miniOS

A tiny Linux system you control entirely over SSH.

You connect, and instead of a bare shell you land on a small desktop: icons, animated windows and a slow red background with touches of pink and purple. Behind it there is a real SSH server, persistent storage, and a simple rule: every app runs in its own Docker container, created for you the first time you launch it.

I built it because I wanted a home server I could hand to someone who has never opened a terminal.

> **Status:** early. The build script hasn't been tested on every machine yet. If something breaks, please open an issue and paste the error.

## What you get

- **Real SSH** (Dropbear). Password and public-key login, host keys that survive reboots, and an optional `admin` account with a normal shell.
- **A smooth terminal interface** written in C with no dependencies. Truecolor, icons, window animations, and only the changed cells are sent over the wire, so it stays light on slow links.
- **One container per app.** Pick an app, press Enter, and miniOS pulls the image, creates the container, gives it a persistent volume and restarts it on boot.
- **Persistent storage.** Files, app data, SSH keys and settings live on an ext4 disk and survive restarts. The OS itself runs from RAM.
- **Built-in tools:** file manager, notes editor, system monitor (with per-container CPU and memory), network and SSH info, settings.
- **Nightly backups** with rotation, plus a health check you can run from the UI or over SSH.
- **A bootable ISO** (BIOS and UEFI) that can set up storage on a blank disk.
- **One script builds everything:** `miniOS.sh`.

## Included apps

nginx, a web file manager, Redis, Uptime Kuma, Adminer, MariaDB, PostgreSQL, Vaultwarden, Gitea, Node-RED, Grafana, Syncthing, Jellyfin, VS Code in the browser, Mosquitto, Memcached, a network test service, and a throwaway Alpine sandbox.

To add your own, press `a` in the Applications window and type an image and a port (for example `nginx:alpine 8080`), or drop a `.app` file into `/data/apps.d/`.

## Quick start

You need a Debian or Ubuntu machine. On Windows, use WSL.

```bash
git clone https://github.com/YOUR_USERNAME/miniOS.git
cd miniOS
chmod +x miniOS.sh
./miniOS.sh deps      # install build dependencies (once)
./miniOS.sh doctor    # check that your machine is ready
./miniOS.sh build     # compile everything; 20-40 minutes, about 12 GB
./miniOS.sh run       # start it in QEMU
```

Then, from another terminal:

```bash
ssh -p 2222 simple@localhost
```

The default password is `mini`. Change it right away in Settings. The interface will remind you until you do.

### Script commands

| Command | What it does |
|---|---|
| `deps` | Install build dependencies |
| `doctor` | Check your machine |
| `build` | Build the kernel, the system, the data disk and the ISO |
| `run`, `start`, `stop` | Run in the foreground, in the background, or stop |
| `ssh` | Connect to the local instance |
| `iso` | Rebuild `miniOS.iso` |
| `rebuild-ui` | Recompile only the interface |
| `status`, `clean` | Show build state, remove build files |

Environment variables: `MINI_PASSWORD`, `MINI_DISK_SIZE` (default 8G), `MINI_MEM` (default 1536 MB), `MINI_CPUS` (default 2), `MINI_RESET_DISK=1`.

## Using the ISO

The ISO boots with two entries:

1. **Try it:** touches no disk. Storage is temporary and Docker is unavailable.
2. **Install:** formats the first completely blank disk it finds (never one that contains data) and puts the storage and Docker on it.

To try it in QEMU:

```bash
qemu-img create -f raw disk.img 8G
qemu-system-x86_64 -m 1536 -smp 2 -cdrom miniOS.iso -boot d \
  -drive file=disk.img,format=raw,if=virtio \
  -nic user,model=virtio-net-pci,hostfwd=tcp::2222-:22
```

Pick the install entry on the first boot, then connect with `ssh -p 2222 simple@localhost`. On later boots the "try it" entry is enough: the disk is found automatically by its label.

Note that this is not a full disk install. The ISO stays your boot medium (a CD, a USB stick or a virtual disk); the disk only receives the storage and Docker, not a bootloader.

## Using it

| Key | Action |
|---|---|
| Arrows, Tab | Move around |
| Enter | Open, or the main action |
| 1-8 | Jump straight to a window |
| `/` | Search for a window or an app |
| `?` | Help |
| Esc | Close, or leave |

You can also run a few commands without opening the interface:

```bash
ssh simple@ip help
ssh simple@ip apps
ssh simple@ip start nginx
ssh simple@ip put notes/a.txt < a.txt
ssh simple@ip get notes/a.txt > a.txt
ssh simple@ip diag
```

That is the whole list. There is no shell behind it, so nothing else can be run through the `simple` account.

## How it is laid out

```
miniOS.sh                    build script, with all sources embedded
  miniui.c                   the interface
  rootfs/                    system scripts (init, appctl, sshcmd, ...)
  rootfs/etc/apps.d/         the app catalog, one file per app
```

## Limitations and security

- The `simple` account is root inside the OS, but it is locked into the menu. Don't expose the SSH port to the internet unless you have changed the passwords and switched to keys.
- Containers use the host network, so two apps can't listen on the same port.
- Docker needs at least 1.5 GB of RAM. Under WSL, QEMU has no hardware acceleration, so startup is slower.
- You need a terminal font with emoji support. Windows Terminal works well.

## License

miniOS is source-available, not open source. You can use, study, modify and share it for free, but only the author may sell it or distribute it for money. See [LICENSE](LICENSE) for the exact terms.

Third-party components keep their own licenses: Linux, BusyBox and e2fsprogs (GPLv2), Dropbear (MIT), Docker (Apache 2.0).
