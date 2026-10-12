# Uploading audio (Drive primary · GitHub secondary)

The folder `mythology/` next to this file is the upload payload — one archive
per kanda per voice, arranged per epic (zips over 90 MB are split into
`.partN` chunks — see the file-size note below):

    mythology/
    └── ramayana/
        ├── bala-kanda-mohan.zip.part1  (flat inside: manifest.json + mp3/cues)
        ├── bala-kanda-mohan.zip.part2
        ├── uttara-kanda-mohan.zip      (small zips stay whole)
        └── … (30 files, 7 kandas × 2 voices)

When another epic (e.g. Mahabharata) is added later it simply becomes a
sibling folder — `mythology/mahabharata/…` — no code changes.

Upload that tree, then record the links in ONE file:

    <app>/data/ramayana/audio/index.json

Three top-level fields (shared by every archive) — already filled in:

    "driveRootId": "1mQd98kvZ0rHXSKAJ_9ZzSVie8oqABIIn",
    "githubBase":  "https://raw.githubusercontent.com/4ppd3ck/Audio/main/mythology",
    "driveKey":    "<Drive API key — still to be filled in>"

The app's downloader joins `<base>/<epic>/<kanda>-<voice>.zip(.partN)` itself,
fetches each file (Drive first, GitHub automatically as fallback), verifies it
against the sha256 recorded in index.json, reassembles split archives, then
installs every file listed in the zip's manifest.json. The server reads
index.json per request, so updating links needs **no rebuild and no restart**.

## Google Drive (primary)

1. Upload the whole `Audio` folder to Drive (drag it in — the app subfolders
   and zips come along). The pasted `driveRootId` may be the Audio root or
   any folder on the path down to the files — the downloader walks down
   until it finds the archive's files.
2. Share the folder as "Anyone with the link → Viewer" (restricted sharing
   returns 401 to the downloader).
3. One-time setup (share links alone won't work for scripted downloads):
   console.cloud.google.com → create/select a project → enable "Google Drive
   API" → Credentials → API key restricted to the Drive API.
4. Paste the API key as `driveKey` into index.json.

The app lists the folder via the Drive API at download time, so per-file ids
are never needed.

## GitHub (secondary) — LIVE

    githubBase: https://raw.githubusercontent.com/4ppd3ck/Audio/main/mythology

The repo is public and already carries the whole tree (split parts included),
so downloads work right now with no key; Drive becomes the faster primary
once `driveKey` is filled in.

**File-size limit:** GitHub rejects any file over 100 MB. `tools/build-audio-hosting.py`
therefore splits every zip over 90 MB into balanced `<name>.zip.partN` chunks
(index.json's archives entry records `parts` + per-part sha256s), so every
file in this tree is committable — the app's downloader fetches the parts in
whatever number the entry declares, reassembles and verifies the whole zip.
Split and whole archives mix freely; nothing is hardcoded. (Alternatively,
Git LFS also works but its free 1 GB tier is under this payload's size.)

Keep the same zip names when you regenerate later and bump the `version` —
`tools/build-audio-hosting.py` refreshes index.json (sizes, whole-zip hashes)
automatically and preserves your `driveRootId` / `githubBase` / `driveKey`.
