# GoPro Webcam for Linux, with a GUI

Use a GoPro HERO8–13 as a webcam on Linux, without a capture card. This fork adds a small desktop app on top of [jschmid1/gopro_as_webcam_on_linux](https://github.com/jschmid1/gopro_as_webcam_on_linux). The original command-line script is unchanged and still works on its own.

GoPro offers no official webcam support on Linux. The camera exposes a USB network interface and streams MPEG-TS over UDP. The `gopro` script finds that interface, switches the camera to webcam mode, and pipes the stream into a `v4l2loopback` device that Zoom, Chrome, OBS and others see as a normal camera.

## Quick start

```sh
git clone https://github.com/clyqramilian/gopro_as_webcam_on_linux
cd gopro_as_webcam_on_linux
sudo ./install.sh
gopro-gui
```

1. Set the camera's USB mode to **GoPro Connect** (Preferences → Connections → USB Connection), plug it in and switch it on until the charger icon shows.
2. Open **GoPro Webcam** from your app launcher, or run `gopro-gui`.
3. Pick a resolution and field of view, then press **Start webcam**. Enter your password when asked.
4. Select **GoPro** as the camera in your video app. By default it is `/dev/video42`.

## The GUI

`gopro-gui` is a GTK4/libadwaita app written in Python.

| Setting | What it does |
| --- | --- |
| Resolution | 1080p, 720p or 480p |
| Field of view | linear, wide, narrow or superview |
| Video device | creates `/dev/video<N>` (default 42) |
| UDP port | port the camera streams to (default 8554) |
| Camera IP, interface, interface pattern | override auto-detection if it picks the wrong network interface |
| Crop | `w:h:x:y` crop applied by ffmpeg |

The app runs as your normal user and calls `gopro webcam -a -n ...` through `pkexec`, so only the camera start needs root. A small root helper watches the connection to the GUI. **Stop webcam**, closing the window, or a GUI crash all shut down the script and ffmpeg, so nothing keeps `/dev/video42` busy.

The log pane shows the script's output, which is where to look first if something fails.

## Requirements

| Needed for | Packages |
| --- | --- |
| Webcam | `ffmpeg`, `curl`, `v4l2loopback` |
| GUI | `python-gobject`, `gtk4`, `libadwaita`, `polkit` |
| Preview (CLI only) | `vlc` |

Debian/Ubuntu: `sudo apt install ffmpeg curl v4l2loopback-dkms python3-gi gir1.2-gtk-4.0 gir1.2-adw-1 policykit-1`

Arch: `sudo pacman -S ffmpeg curl v4l2loopback-dkms python-gobject gtk4 libadwaita polkit`

The camera's network interface must be allowed through your firewall on the stream port, for example with `firewalld`:

```sh
sudo firewall-cmd --add-port 8554/udp
sudo firewall-cmd --add-port 8554/udp --permanent
```

## Command line

The GUI is optional. The script can be used directly:

```sh
sudo gopro webcam -a -n          # auto-detect, start ffmpeg, no prompts
sudo gopro webcam -p enx -a -n   # match the interface by pattern
sudo gopro webcam -i 172.27.187.52 -a -n
sudo gopro --help
```

| Option | Meaning |
| --- | --- |
| `-n`, `--non-interactive` | don't wait for input (for services and scripts) |
| `-a`, `--auto-start` | start ffmpeg and expose the camera as a video device |
| `-r`, `--resolution` | `1080`, `720` or `480` |
| `-f`, `--fov` | `wide`, `narrow`, `superview` or `linear` |
| `-c`, `--crop` | ffmpeg crop `w:h:x:y`, e.g. `1280:720:320:180` |
| `-d`, `--device` | full interface name. It changes on every reconnect, so don't use it in automation |
| `-p`, `--device-pattern` | interface name pattern such as `enx` |
| `-i`, `--ip` | camera IPv4 address |
| `--video-number` | creates `/dev/video<N>` (default 42) |
| `-P`, `--PORT` | UDP port (default 8554) |
| `-v`, `--preview`, `-u`, `--user` | preview in VLC, run as the given user because VLC refuses root |
| `-V`, `--verbose` | echo every command |

The camera offers a single format once in webcam mode: YU12 (planar YUV 4:2:0), 1920x1080 at 30 fps.

## Start on boot or plug-in

Copy `gopro_webcam.service` to `/etc/systemd/system/` and run `sudo systemctl enable --now gopro_webcam.service`. Follow the logs with `sudo journalctl -u gopro_webcam -f`.

To start and stop automatically when the camera is plugged in or removed, also copy `60-gopro.rules` to `/lib/udev/rules.d/`. The rule matches HERO9 by default. Copy and edit the line for other models. The first start after plugging in often fails because the network isn't up yet, and the retry about 15 seconds later succeeds.

The camera must be in standby (charger icon) when the computer boots.

## Troubleshooting

**No network interface found.** Check that the USB mode is GoPro Connect and not MTP. If the option is missing, update the camera firmware: <https://gopro.com/en/us/update>. Otherwise pass the interface in the GUI's Advanced section, or with `-p` or `-d`.

**`v4l2loopback` can't be loaded or unloaded.** Another program is using the device. Close apps that have the webcam open and try again.

**The camera starts but the video app shows nothing.** Check the firewall rule for the UDP port above.

## Credits and license

The `gopro` script is by [Joshua Schmid](https://github.com/jschmid1) and contributors, from the upstream project. Thanks to [KonradIT](https://github.com/KonradIT) for documenting the unofficial GoPro HTTP API.

Licensed under Apache 2.0, see [LICENSE](LICENSE).
