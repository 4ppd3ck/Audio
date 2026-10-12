# Uploading Vedic Stotrams audio (Drive primary · GitHub secondary)

The folder `vedic/` next to the `mythology/` folder is the upload payload —
one archive per stotra per voice:

    vedic/
    ├── achyutashtakam-mohan.zip        (flat inside: manifest.json + mp3/cues)
    ├── achyutashtakam-shruti.zip
    ├── aditya-hridayam-mohan.zip
    └── … (66 zips, 33 stotras × 2 voices)

All current zips are under 10 MB, so none is split; if a future stotram's zip
ever exceeds 90 MB, `tools/build-audio-hosting.py` splits it into
`<name>.zip.partN` chunks automatically (index.json records `parts` + per-part
sha256s) and the app reassembles them at download time.

Upload that tree, then record the links in ONE file:

    <app>/data/audio/index.json

Three top-level fields (shared by every archive) — already filled in:

    "driveRootId": "1mQd98kvZ0rHXSKAJ_9ZzSVie8oqABIIn",
    "githubBase":  "https://raw.githubusercontent.com/4ppd3ck/Audio/main/vedic",
    "driveKey":    "<Drive API key — still to be filled in>"

The app's downloader joins `<base>/<stotra>-<voice>.zip(.partN)` itself,
fetches each file (Drive first, GitHub automatically as fallback), verifies it
against the sha256 recorded in index.json, then installs every file listed in
the zip's manifest.json. The server reads index.json per request, so updating
links needs **no rebuild and no restart**.

## Google Drive (primary)

1. Upload the whole `Audio` folder to Drive (drag it in — the app subfolders
   and zips come along). The pasted `driveRootId` may be the Audio root or
   any folder on the path down to the files — the downloader walks down
   until it finds the archive's files.
2. Share the folder as "Anyone with the link → Viewer" (restricted sharing
   returns 401 to the downloader).
3. One-time setup: console.cloud.google.com → create/select a project →
   enable "Google Drive API" → Credentials → API key restricted to the
   Drive API.
4. Paste the API key as `driveKey` into index.json.

## GitHub (secondary) — LIVE

    githubBase: https://raw.githubusercontent.com/4ppd3ck/Audio/main/vedic

The repo is public and already carries the whole tree, so downloads work
right now with no key; Drive becomes the faster primary once `driveKey` is
filled in.

Keep the same zip names when you regenerate later and bump the `version` —
`tools/build-audio-hosting.py` refreshes index.json (sizes, whole-zip hashes)
automatically and preserves your `driveRootId` / `githubBase` / `driveKey`.
