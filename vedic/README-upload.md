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

Three top-level fields (shared by every archive):

    "driveRootId": "<id of the vedic folder on Drive>",
    "githubBase":  "https://raw.githubusercontent.com/<user>/<repo>/main/vedic",
    "driveKey":    "<Drive API key>"

The app's downloader joins `<base>/<stotra>-<voice>.zip` itself, fetches the
zip (Drive first, GitHub automatically as fallback), verifies it against the
whole-zip sha256 recorded in index.json, then installs every file listed in
the zip's manifest.json. The server reads index.json per request, so updating
links needs **no rebuild and no restart**.

## Google Drive (primary)

1. Upload the whole `vedic` folder to Drive (drag it in — the zips come
   along).
2. Share the `vedic` folder as "Anyone with the link → Viewer".
3. One-time setup (share links alone won't work for scripted downloads):
   console.cloud.google.com → create/select a project → enable "Google Drive
   API" → Credentials → API key restricted to the Drive API.
4. Paste into index.json:
   - the folder id from the share link (the long token after /folders/) as
     `driveRootId`
   - the API key as `driveKey`.

The app lists the folder via the Drive API at download time, so per-file ids
are never needed.

## GitHub (secondary)

    cd ~/Desktop/Audio/vedic
    git init && git add . && git commit -m "Vedic Stotrams audio"
    # push to a repo, e.g. github.com/<user>/vedic-audio (main branch)

Then `githubBase` is
`https://raw.githubusercontent.com/<user>/vedic-audio/main/vedic`.
(GitHub release assets are flat, so a repo tree is the right shape here.)

Keep the same zip names when you regenerate later and bump the `version` —
`tools/build-audio-hosting.py` refreshes index.json (sizes, whole-zip hashes)
automatically and preserves your `driveRootId` / `githubBase` / `driveKey`.
