#H3Proxy

Hybrid proxy browser for New Nintendo 3DS: server-side Firefox renders the modern web into a tile stream for a homebrew 3DS client.

H3Proxy is an independent homebrew project and is not affiliated with, endorsed by, sponsored by, or approved by Nintendo.

Nintendo 3DS and related names are trademarks of Nintendo. This project does not include Nintendo firmware, SDK files, copyrighted assets, game files, keys, or circumvention tools.

The H3Proxy source code is licensed under the Apache License 2.0. Third-party dependencies retain their own licenses.

#What H3Proxy Does

H3Proxy lets a New Nintendo 3DS act like a lightweight browser terminal.

The 3DS does not run a modern browser engine itself. Instead:

PC/server:
  runs Firefox through Playwright
  loads modern websites
  handles TLS, JavaScript, CSS, DOM, media, and page rendering
  captures the visible 400x240 viewport
  splits the viewport into small tiles
  sends changed tiles and media frames to the 3DS

New Nintendo 3DS:
  connects to the H3Proxy server over WebSocket
  sends input events
  receives tile/media packets
  decodes PNG/JPEG payloads with stb_image
  converts/draws the image data to the 3DS framebuffer
  reports hardware capability and runtime stats back to the server

In simple terms:

Firefox does the modern web.
The 3DS displays the result and sends controls back.
Current Status

This is an early prototype hardware build.

Working:

New Nintendo 3DS homebrew client
SD logging
SD config file
hardware calibration
server pairing screen
WebSocket connection
client_caps and client_stats
tile stream protocol
touch input
scroll input
keyboard/text input
browser back/forward/reload
experimental media/canvas mode
server dashboard
PC debug client

Known issues:

UI and rendering bugs remain
media mode is experimental
some browser automation errors can happen during navigation
LAN ws:// mode is the main tested path
remote encrypted mode is not finalized
not tested on old 3DS hardware
Hardware Target

Primary target:

New Nintendo 3DS LL / New Nintendo 3DS XL

Other hardware may work later, but this build is designed around New 3DS-class homebrew hardware.

Repository / Release Layout

The GitHub repo itself is mainly the project landing page.

Actual source snapshots and runnable builds are distributed through GitHub Releases.

A typical release contains:

h3proxy_3ds.3dsx
  Homebrew Launcher build for the 3DS

H3Proxy-<version>-source.zip
  Frozen source snapshot containing:
    client_3ds/
    h3proxy/
    tools/
Installing The 3DS Client

Download the .3dsx file from the latest GitHub Release.

Put it on the 3DS SD card like this:

SD:/
  3ds/
    h3proxy_3ds/
      h3proxy_3ds.3dsx

Launch it from the Homebrew Launcher.

The client writes logs here:

sdmc:/h3proxy.log

The client stores config here:

sdmc:/h3proxy.cfg
Server Requirements

The server side needs Python 3 and a Playwright Firefox install.

Python dependencies used by the server:

fastapi
uvicorn
playwright
pillow

Install dependencies:

pip install fastapi uvicorn playwright pillow
python -m playwright install firefox

A virtual environment is recommended.

Starting The Server

For LAN testing, the server should listen on all interfaces so the 3DS can reach it.

From the source snapshot root, use:

tools\start_lan_server.cmd

That script sets:

H3PROXY_HOST=0.0.0.0
H3PROXY_PORT=8766
H3PROXY_HEADLESS=true

Then starts:

python -m h3proxy

Manual start example:

set H3PROXY_HOST=0.0.0.0
set H3PROXY_PORT=8766
set H3PROXY_HEADLESS=true
python -m h3proxy

Open the dashboard on the server PC:

http://127.0.0.1:8766/

From another LAN device, use the server LAN IP:

http://SERVER-IP:8766/

Example:

http://192.168.1.116:8766/
3DS First Launch / Pairing

On launch, the 3DS client runs:

hardware calibration
server pairing
WebSocket connection

The pairing screen lets you choose the server IP and port directly on the 3DS.

Pairing controls:

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

Supported private ranges in the pairing screen:

10.0.0.0/8
172.16.0.0/12
192.168.0.0/16

Default config if no config exists:

server=http://192.168.1.100:8766
start_url=/start
quality=normal
token=

Leaving token blank is recommended. The 3DS will create a fresh session with the server automatically.

Manual 3DS Config

You can edit:

sdmc:/h3proxy.cfg

Example:

server=http://192.168.1.116:8766
start_url=/start
quality=normal
token=

start_url can be a full URL:

https://example.com
https://google.com
https://en.m.wikipedia.org/wiki/Main_Page

or a server-local path:

/start
/debug/animation
/debug/input
/debug/media
Runtime Controls

Main browser controls:

Circle Pad:
  scroll page

Touchscreen top/page area:
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
Bottom Screen Layout

The bottom screen is split into:

top 180 px:
  page/touch pad area

bottom 60 px:
  dock controls

The top page pad maps touch input into the 400x240 browser viewport.

The dock area provides quick browser controls.

Current dock behavior in this prototype:

top dock row:
  tap left/middle area to edit URL/start_url
  tap right side to Go/open current URL

lower dock area:
  tap left/middle area to open text keyboard and send text to the page
  tap right side to send Enter

Some UI text may mention DOM selection or quality cycling. Those controls are still prototype/incomplete in this build.

Debug Pages

The server includes local test pages:

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

/
  server dashboard

These are useful before testing heavy external sites.

Recommended first test:

/debug/animation

Then:

/debug/input

Then:

/start

Then try external pages.

How The Protocol Works

The 3DS creates or uses a server session.

Session creation:

POST /sessions

The server returns:

token
websocket_path
viewport
tile size
auto_update flag
protocol max FPS

The 3DS then connects:

ws://SERVER-IP:8766/ws/<token>

The 3DS sends:

{"type":"client_caps", "...":"..."}
{"type":"hello","viewport":{"width":400,"height":240},"quality":"normal"}
{"type":"open_url","url":"/start"}

The server sends text messages for:

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

The server sends binary messages for:

tile
media_frame

Binary packets use:

4 byte big-endian JSON header length
JSON header
binary payload

Tile packets contain small PNG/JPEG images for changed regions of the 400x240 viewport.

Default tile size:

100x60

So the visible top-screen viewport is normally:

4 columns x 4 rows = 16 tiles
Media Mode

H3Proxy can detect video/canvas-like regions and offer media mode.

Media mode sends a JPEG stream for the active media rectangle instead of treating everything as page tiles.

This is experimental.

Expected behavior:

server detects media/canvas region
server sends media_offer
3DS requests start_media
server sends media_frame binary packets
3DS decodes/draws media frames
3DS sends stats back
server may throttle based on decode/queue/dropped-frame pressure
Client Hardware Calibration

On boot, the 3DS client measures:

safe RAM budget
SD write speed
RGB565 tile/frame conversion timing
texture/cache estimates

The client reports this as client_caps.

Example capabilities:

{
  "type": "client_caps",
  "profile": "New 3DS LL / XL",
  "calibrated": true,
  "max_page_fps": 60,
  "max_media_fps": 60,
  "safe_ram_bytes": 76336332,
  "safe_texture_cache_tiles": 256
}

The server uses this information to adjust page/media FPS and runtime policy.

Server Environment Variables

Useful server variables:

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
Building The 3DS Client From Source

Expected toolchain:

devkitPro
devkitARM
libctru

Build helper:

tools\build_3ds.cmd

Or from inside client_3ds/:

make

The output .3dsx is the Homebrew Launcher build.

The .elf is a debug/build artifact and is not required for normal users.

#Third-Party Code

The 3DS client vendors:

client_3ds/third_party/stb_image.h

stb_image.h is from the stb single-file public-domain/MIT-style library collection. See the header itself for its license text.

Third-party components retain their own licenses.

#timeperiod
project first started 5/27/2026 @ 8:00am

#License

H3Proxy is licensed under the Apache License 2.0.

Original project by Phsphorus.

See:

LICENSE
NOTICE
Legal Notice

3H3Proxy is an independent homebrew project.

It is not affiliated with, endorsed by, sponsored by, or approved by Nintendo.

Nintendo 3DS and related names are trademarks of Nintendo.

This project does not include Nintendo firmware, SDK files, copyrighted assets, game files, keys, or circumvention tools.
