# YT-simple

A minimal, single-file YouTube search and player. Everything lives in one HTML file with no build step and no dependencies to install. Network requests go through [libcurl.js](https://github.com/ading2210/libcurl.js) (libcurl compiled to WebAssembly) and a [Wisp](https://github.com/MercuryWorkshop/wisp-protocol) proxy server, so the page can talk to YouTube directly from the browser.

## Features

- Search YouTube, or paste a video link / 11-character video ID to play it straight away
- Quality selector: 360p, 480p, 720p (default) and 1080p
- Adaptive streaming: separate H.264 video and AAC audio are downloaded in chunks and played through the browser's MediaSource API
- Pre-buffers about 10 seconds of both tracks before playback starts, and waits for a small cushion to rebuild after a stall
- Falls back to a single-file 360p stream when higher quality is not available
- Tries several YouTube client identities in order and moves on if one is refused
- Timeouts and automatic retries on stalled downloads; a fresh proxy connection is made for every request
- Plain, lightweight UI that works on mobile browsers

## Quick start

1. Download `YT-simple.html`.
2. Open it in a modern browser (Chrome, Edge, Firefox or Safari).
3. Make sure the **Proxy (Wisp server)** box at the top holds a working Wisp server address (see below).
4. Type a search or paste a YouTube link, choose a quality, and press **Go**.

Opening the file from disk works. Hosting it over `http(s)` is only needed for the optional "Try embedded player" fallback.

## Proxy (Wisp server)

The page does not connect to YouTube on its own. It sends traffic through a Wisp server over a WebSocket, so you need one that is running and reachable.

- Enter its address in the proxy box as `wss://host/path/` (an `https://` address is converted automatically, and a trailing slash is added if missing).
- The address pre-filled in the file is only an example default. It may be offline or blocked at any time, so use your own.
- You can run your own with a Wisp server implementation such as [`wisp-server-node`](https://github.com/MercuryWorkshop/wisp-server-node) or [`wisp-server-python`](https://github.com/MercuryWorkshop/wisp-server-python).
- Where the server runs matters. YouTube is much more likely to challenge traffic coming from shared cloud or data-centre IP addresses than from a home or residential connection.

## How it works

1. **Transport:** libcurl.js loads as WebAssembly and sends requests through the Wisp server.
2. **Search:** the page calls YouTube's internal search API with a visitor ID and browser user agent. If that returns nothing, it falls back to reading the HTML results page.
3. **Streams:** for a video it first loads a visitor ID from the watch page, then asks several YouTube clients for stream URLs at once, in this order: `VISIONOS`, `ANDROID_VR` (two versions), `IOS`, `ANDROID`.
4. **Playback:**
   - If a client offers separate H.264 video and AAC audio up to the chosen quality, both are fetched in 4 MB range requests and played through MediaSource.
   - If a client is refused (for example HTTP 403) before playback starts, the next one is tried.
   - If nothing higher than 360p works, the single-file stream is downloaded in full and played from a blob URL, with a download link.

## Limitations

- It relies on YouTube's unofficial internal API. YouTube changes this regularly, so client versions and behaviour may need updating over time.
- Only H.264 (`avc1`) video and AAC (`mp4a`) audio are used, and the browser must support MediaSource.
- No sign-in, comments, playlists, Shorts feed, subtitles or live streams.
- Quality depends heavily on proxy speed. If playback is choppy, try a lower quality setting.
- YouTube may refuse requests from the proxy's IP with "Sign in to confirm you're not a bot". This cannot be fixed in the page; use a different Wisp server.

## Troubleshooting

| Message | Likely cause and fix |
| --- | --- |
| `libcurl failed to load` / `Still loading libcurl...` | Reload the page. Check that the browser supports WebAssembly. |
| `could not connect` / `code 7` | The proxy address is wrong or the Wisp server is down. |
| `timed out` | The proxy or YouTube did not respond. Try again or change the proxy. |
| `Sign in to confirm you're not a bot` | YouTube is flagging the proxy's IP. Use a different Wisp server. |
| `audio Download failed (HTTP 403)` | That client's stream links are refused. The page will try the next client. |
| `Could not read the search results (...)` | The error lists what the search API and results page returned. Pasting a video link still works. |
| Stays at 360p | The `Higher quality failed:` line under the video explains why each client was skipped. |

After updating the file, reload the page (or open the newest downloaded copy) so you are not running a cached version.

## Project layout

```
YT-simple.html   # the whole app: libcurl.js (embedded), UI and player logic
README.md
```

## Credits and licence

- [libcurl.js](https://github.com/ading2210/libcurl.js) by ading2210, embedded in the HTML file and licensed under the GNU LGPL v3.
- [Wisp protocol](https://github.com/MercuryWorkshop/wisp-protocol) and server implementations by Mercury Workshop.
- YouTube client details follow publicly documented behaviour in projects such as [yt-dlp](https://github.com/yt-dlp/yt-dlp).

Add a licence for the rest of the code here (for example MIT) before publishing.

## Disclaimer

This project is not affiliated with or endorsed by YouTube or Google. Use it in line with YouTube's Terms of Service and the laws that apply to you.
