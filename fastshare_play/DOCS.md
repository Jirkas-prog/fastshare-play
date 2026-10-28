# FastShare Play

A separate Home Assistant app for your local film collection. Three real frames
from each movie, a sharp navy-and-lime library, saved playback position and NOVA
resume compatibility. No API keys, metadata accounts or cloud lookup required.

The existing WorkFlow FastShare downloader stays unchanged. Point both apps at
the same media directory. Play can also index films obtained elsewhere.

## Install

1. Add `https://github.com/Jirkas-prog/fastshare-play` to Home Assistant's app
   store repositories, install **FastShare Play**, then start it.
2. Open its web UI through Home Assistant and select media folders in Settings,
   for example `/media/movies`. The app mounts `/media` and `/share` read-only.
3. To open the player directly on your network, enable **Playback without
   sign-in** and select a free port (default **8111**). Save; the app restarts
   automatically when the port mapping changes. The separate player has no
   settings or file management operations.

The optional hostname setting builds a convenient URL. Create your own DNS
record pointing that name to the Home Assistant host. It does not configure a
DNS server, router, external access or certificates.

English is the default; Czech is selectable in Settings.

## Library and playback

- Search titles and filenames; sort newest added, title, year, rating, last played
  or episode order. Filter movies, series, favorites, unwatched, year, folder,
  collection and genre. Genre, collection, cast and rating require local NFO
  data; they cannot be inferred reliably from video frames.
- Three previews are sampled at stable random positions, one in each third of
  the middle 60% of the film. They cycle every six seconds while visible and
  expand in the detail gallery. Reduced-motion preferences disable cycling.
- Previews and watch state live in private app storage, never alongside the
  source movie. One low-priority FFmpeg process runs at a time with one decoding
  thread, bounded timeouts and 640-pixel output. Existing previews are reused.
  First indexing of a large collection takes time; normal browsing uses cached
  JPEGs. Preview generation can be disabled without deleting saved previews.
- Resume, watched and favorites persist across restarts and browser devices.
  State is shared by the household, not split into user profiles. A newer
  playback session prevents an older tab from overwriting its progress.
- NOVA `.archos.resume.xml` and local NOVA/Kodi NFO progress are imported when
  newer than the saved browser state. This is **one-way import**; source sidecars
  and movie files are never written. SRT/VTT sidecar subtitles are supported.
- No plot descriptions or online movie lookup are used. Local NFO titles,
  genres, collections, cast, ratings and local posters are recognized.
- Direct video delivery supports HTTP Range/HEAD for seeking. The browser
  decodes the original file. There is **no live transcoding**: MP4/H.264/AAC is
  widely supported, but MKV/AVI, HEVC/DTS and embedded subtitles depend on the
  browser and device. Your existing Android/TV player remains useful for those.
- The app has no delete, move, rename, download-queue or email endpoints. Its
  anonymous player can browse/play and update only watch state and favorites.
  Administration is restricted to Home Assistant Ingress's actual peer.

## NOVA attribution and scope

Selected filename, NFO and resume parsing logic was ported from
[NOVA Video Player](https://github.com/nova-video-player/aos-AVP), with attribution
and modification notices under Apache 2.0. This is a browser adaptation, **not a
complete Android code port**. Android decoders, Trakt, torrents, audio passthrough
and Android TV UI are not included. Credits are in About and source notices.

See [LICENSE](LICENSE), [NOTICE](NOTICE) and [upstream details](docs/UPSTREAM.md).
FFmpeg is a separate Debian program with its own licenses; its copyright files
remain in the locally built container. This repository publishes source, not
prebuilt container images or FFmpeg binaries.

## Development and checks

Python 3.9+ (container: 3.12), Node for syntax/translation checks, FFmpeg/FFprobe
for real preview extraction. Install `fastshare_play/requirements.txt`, then:

```sh
cd fastshare_play
python -m unittest discover -s tests -v
node --check www/app.js
python -m player_app.app
```

Local development serves administration only to loopback on port 8099. Media
must be under the `PLAY_DATA` directory (default `.runtime`). In Home Assistant,
only configured `/media` and `/share` paths are accepted. Tests cover public
access restrictions, path confinement, range requests, sidecar parsing,
persistence, stale sessions, replacement files, preview caching and real FFmpeg
extraction when the executables are installed.

Limits: 5,000 videos, 30,000 directory entries and 16 nested levels per scan;
12 configured roots; sidecar XML 1 MB; subtitle files 2 MB. Missing mounts do
not wipe saved progress. Replacing a file resets its old watch position.
