# H3Proxy

Hybrid proxy browser for New Nintendo 3DS: server-side Firefox renders the modern web into a tile/media stream for a homebrew 3DS client.

H3Proxy is an independent homebrew project and is not affiliated with, endorsed by, sponsored by, or approved by Nintendo.

Nintendo 3DS and related names are trademarks of Nintendo. This project does not include Nintendo firmware, SDK files, copyrighted assets, game files, keys, ROMs, CIAs, or circumvention tools.

The H3Proxy source code is licensed under the Apache License 2.0. Third-party dependencies retain their own licenses.

## Important: The Source Zip Is Required

The `.3dsx` file is only the 3DS client.

You also need the source zip because it contains the H3Proxy server code and tools. The 3DS client does not browse the web by itself. It connects to the H3Proxy server, and the server does the actual modern browser work.

A normal release should include:

```text
h3proxy_3ds.3dsx
  The 3DS Homebrew Launcher client.

H3Proxy-<version>-source.zip
  The server code, tools, and frozen source snapshot.
```

If you only download the `.3dsx`, you do not have a working H3Proxy setup.

## What H3Proxy Does

H3Proxy turns a New Nintendo 3DS into a lightweight browser terminal.

The server runs Firefox through Playwright, loads modern websites, handles TLS, JavaScript, CSS, DOM, page rendering, and media/canvas capture, then sends a compressed tile/media stream to the 3DS.

The 3DS receives that stream, decodes the image data, displays it, and sends input events back to the server.

In simple terms:

```text
The server runs the modern web.
The 3DS displays it and controls it.
```

## Current Build

Current release line:

```text
2.12.6.18SEWCM#SecureCore
```

This is the current working SecureCore prototype.

The version includes `E` because the SecureCore transport/network path is experimental. Secure transport support exists in the code, but it has not been fully real-hardware-tested yet.

Meaning of the important status letters here:

```text
S = stable enough structurally
E = experimental component present
W = working
C = checkpoint
M = main/current line
```

## Current Status

Working / implemented:

```text
New Nintendo 3DS homebrew client
Firefox/Playwright server backend
server dashboard
LAN server mode
3DS-side WebSocket transport
SD config file
SD logging
hardware calibration
New 3DS high-clock/L2 speedup request
400x240 viewport streaming
100x60 tile updates
PNG/JPEG image decode
touch input
scroll input
text input
browser back/forward/reload controls
client capability reporting
runtime client stats
experimental media/canvas mode
tools for LAN and secure server launch
```

SecureCore experimental features:

```text
HTTPS server mode
WSS secure WebSocket mode
3DS-side secure transport path
access-token protected server mode
secure deployment support
```

Known limitations:

```text
SecureCore transport/network is experimental and not fully hardware-tested.
Media/canvas mode is experimental.
Some websites may behave badly or need tuning.
The server code is required; the .3dsx alone is not enough.
Old 3DS hardware is not the target for this build.
```

## Hardware Target

Primary target:

```text
New Nintendo 3DS LL / New Nintendo 3DS XL
```

Other hardware may work later, but this build is designed around New 3DS-class homebrew hardware.

## Release Layout

The GitHub repo itself is mostly the project landing page.

Actual source snapshots and runnable builds are distributed through GitHub Releases.

Download both:

```text
1. h3proxy_3ds.3dsx
2. H3Proxy-<version>-source.zip
```

The source zip should contain folders like:

```text
client_3ds/
h3proxy/
tools/
```

## Quick Start

### 1. Download Both Release Assets

Download:

```text
h3proxy_3ds.3dsx
H3Proxy-2.12.6.18SEWCM-SecureCore-source.zip
```

The `.3dsx` goes on the 3DS.

The source zip gets extracted on the PC/server.

### 2. Install The 3DS Client

Put the `.3dsx` on the 3DS SD card:

```text
SD:/
  3ds/
    h3proxy_3ds/
      h3proxy_3ds.3dsx
```

Then launch it from the Homebrew Launcher.

The client writes logs here:

```text
sdmc:/h3proxy.log
```

The client stores config here:

```text
sdmc:/h3proxy.cfg
```

### 3. Extract The Source Zip On The Server PC

Extract the source zip somewhere on the PC that will run the server.

Example:

```text
D:\H3Proxy\
```

The extracted folder should contain:

```text
h3proxy/
client_3ds/
tools/
```

### 4. Install Server Dependencies

The server needs Python 3 and Playwright Firefox.

Install the Python dependencies:

```cmd
pip install fastapi uvicorn playwright pillow
python -m playwright install firefox
```

If the release includes a `requirements.txt`, you can use:

```cmd
pip install -r requirements.txt
python -m playwright install firefox
```

### 5. Start LAN Server

From the extracted source folder:

```cmd
tools\start_lan_server.cmd
```

Or start manually:

```cmd
set H3PROXY_HOST=0.0.0.0
set H3PROXY_PORT=8766
set H3PROXY_HEADLESS=true
python -m h3proxy
```

Open the dashboard on the server PC:

```text
http://127.0.0.1:8766/
```

From another LAN device, use the server LAN IP:

```text
http://SERVER-LAN-IP:8766/
```

Example:

```text
http://192.168.1.116:8766/
```

### 6. Point The 3DS At The Server

On first launch, the 3DS client can create/edit:

```text
sdmc:/h3proxy.cfg
```

Example LAN config:

```text
server=http://192.168.1.116:8766
start_url=/start
quality=normal
token=
auth_token=
```

Leaving `token` blank is preferred for normal testing. The 3DS will ask the server to create a fresh session.

## SecureCore / Secure Server Mode

Secure mode is experimental.

For reliable testing, use LAN mode first.

Secure server mode requires a certificate, key, and access token.

Example:

```cmd
set H3PROXY_ACCESS_TOKEN=put-a-long-random-token-here
set H3PROXY_SSL_CERTFILE=C:\path\to\cert.pem
set H3PROXY_SSL_KEYFILE=C:\path\to\privkey.pem
set H3PROXY_PORT=8766
tools\start_secure_server.cmd
```

Secure dashboard:

```text
https://SERVER-HOST:8766/
```

Example secure 3DS config:

```text
server=https://SERVER-HOST:8766
start_url=https://google.com
quality=normal
token=
auth_token=put-a-long-random-token-here
```

Do not expose H3Proxy publicly without an access token.

Secure transport/network is marked experimental because this path is implemented but not fully tested on real hardware.

## 3DS Pairing / First Launch

On launch, the 3DS client runs:

```text
hardware calibration
server setup / pairing
network connection
WebSocket session
```

Pairing controls:

```text
D-Pad Up / Down:
  switch private IP range

D-Pad Left / Right:
  select editable IP/port field

A:
  increase selected field

B:
  decrease selected field

X:
  test and pair with selected server

START:
  exit pairing
```

Supported private ranges:

```text
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

Default config if no config exists:

```text
server=http://192.168.1.100:8766
start_url=/start
quality=normal
token=
auth_token=
```

## Runtime Controls

Main browser controls:

```text
Circle Pad:
  scroll page

Touchscreen page area:
  touch/click/drag inside the browser viewport

L:
  browser back

R:
  browser forward

ZL:
  reload

A:
  Enter

B:
  Escape / deselect

X:
  keep page view on top display

Y:
  mirror page into bottom touch area

SELECT:
  toggle debug/log overlay

Hold START for about 3 seconds:
  quit the client
```

## Bottom Screen Layout

The bottom screen is split into:

```text
top 180 px:
  page/touch pad area

bottom 60 px:
  dock controls
```

The top page pad maps touch input into the 400x240 browser viewport.

The dock area provides quick browser controls.

Prototype dock behavior:

```text
top dock row:
  tap left/middle area to edit URL/start_url
  tap right side to Go/open current URL

lower dock area:
  tap left/middle area to open text keyboard and send text to the page
  tap right side to send Enter
```

Some UI behavior is still prototype-level and may change.

## Debug Pages

The server includes local test pages:

```text
/
  server dashboard

/start
  simple H3Proxy start page

/debug/animation
  CSS motion test

/debug/input
  input field/button test

/debug/media
  media/canvas test

/debug/client
  PC browser debug client
```

Recommended test order:

```text
/debug/animation
/debug/input
/start
then external websites
```

## How The Protocol Works

The 3DS creates or uses a server session.

Session creation:

```text
POST /sessions
```

The server returns:

```text
token
websocket_path
viewport
tile size
auto_update flag
protocol max FPS
```

The 3DS connects to:

```text
ws://SERVER-IP:8766/ws/<token>
```

or, in SecureCore mode:

```text
wss://SERVER-HOST:8766/ws/<token>
```

The 3DS sends messages like:

```json
{"type":"client_caps"}
{"type":"hello","viewport":{"width":400,"height":240},"quality":"normal"}
{"type":"open_url","url":"/start"}
{"type":"client_stats"}
{"type":"touch_down","x":100,"y":80}
{"type":"touch_up","x":100,"y":80}
{"type":"text","text":"example"}
```

The server sends text messages for:

```text
hello
status
error
page_meta
dom_map
frame_done
media_offer
media_start
media_stop
client_policy
server_policy
```

The server sends binary messages for:

```text
tile
media_frame
```

Binary packets use:

```text
4 byte big-endian JSON header length
JSON header
binary payload
```

Default viewport:

```text
400x240
```

Default tile size:

```text
100x60
```

That makes the normal top-screen viewport:

```text
4 columns x 4 rows = 16 tiles
```

## Media Mode

H3Proxy can detect video/canvas-like regions and offer media mode.

Media mode sends a JPEG stream for the active media rectangle instead of treating everything as normal page tiles.

This is experimental.

Expected flow:

```text
server detects media/canvas region
server sends media_offer
3DS requests start_media
server sends media_frame binary packets
3DS decodes/draws media frames
3DS sends stats back
server may throttle based on decode/queue/dropped-frame pressure
```

## Client Hardware Calibration

On boot, the 3DS client measures:

```text
safe RAM budget
SD write speed
RGB565 tile/frame conversion timing
texture/cache estimates
```

The client reports this as `client_caps`.

Example:

```json
{
  "type": "client_caps",
  "profile": "New 3DS LL / XL",
  "calibrated": true,
  "max_page_fps": 60,
  "max_media_fps": 60,
  "safe_ram_bytes": 76336332,
  "safe_texture_cache_tiles": 256
}
```

The server uses this information to adjust page/media FPS and runtime policy.

## Server Environment Variables

Useful server variables:

```text
H3PROXY_HOST
  default: 127.0.0.1
  LAN testing: 0.0.0.0

H3PROXY_PORT
  default: 8765
  LAN test scripts use: 8766

H3PROXY_HEADLESS
  default: true

H3PROXY_VIEWPORT_WIDTH
  default: 400

H3PROXY_VIEWPORT_HEIGHT
  default: 240

H3PROXY_TILE_WIDTH
  default: 100

H3PROXY_TILE_HEIGHT
  default: 60

H3PROXY_JPEG_QUALITY
  default: 64

H3PROXY_PROTOCOL_MAX_FPS
  default: 60

H3PROXY_ACCESS_TOKEN
  required for secure/public-style mode

H3PROXY_SSL_CERTFILE
  certificate path for HTTPS/WSS server mode

H3PROXY_SSL_KEYFILE
  private key path for HTTPS/WSS server mode
```

## Building The 3DS Client From Source

Expected toolchain:

```text
devkitPro
devkitARM
libctru
```

Build helper:

```cmd
tools\build_3ds.cmd
```

Or from inside `client_3ds/`:

```cmd
make
```

The `.3dsx` is the Homebrew Launcher build.

## Third-Party Code

The 3DS client vendors:

```text
client_3ds/third_party/stb_image.h
```

`stb_image.h` is from the stb single-file public-domain/MIT-style library collection. See the header itself for its license text.

Third-party components retain their own licenses.

## Project Timeline

Project first started:

```text
May 27, 2026 around 8:00 AM
```

## License

H3Proxy is licensed under the Apache License 2.0.

Original project by Phsphorus.

See:

```text
LICENSE
NOTICE
```

## Legal Notice

H3Proxy is an independent homebrew project.

It is not affiliated with, endorsed by, sponsored by, or approved by Nintendo.

Nintendo 3DS and related names are trademarks of Nintendo.

This project does not include Nintendo firmware, SDK files, copyrighted assets, game files, keys, ROMs, CIAs, or circumvention tools.
