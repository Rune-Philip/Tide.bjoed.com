# Svinø pictures — backend contract

The album reads `photo-config.json` from the same folder as `index.html` at page load.
No config file present → pictures are stored in the visitor's own browser (IndexedDB)
and are not shared. Drop the file in and the album goes shared; no rebuild needed.

## Option A — your own host (kind: "rest")

`photo-config.json`:

    {
      "kind": "rest",
      "list":   "https://api.example.com/svino/photos",
      "upload": "https://api.example.com/svino/photos",
      "delete": "https://api.example.com/svino/photos"
    }

Three endpoints, CORS open to https://tide.bjoed.com:

**GET list** → JSON array, newest first:

    [ { "id": "abc123",
        "url": "https://cdn.example.com/svino/abc123.jpg",
        "uploader": "Mormor",
        "created_at": "2026-08-13T18:24:00Z" } ]

**POST upload** → `multipart/form-data` with two fields:
`file` (image/jpeg, already resized to 1600px and EXIF-stripped by the browser) and
`uploader` (string). Any 2xx counts as success.

**DELETE delete/{id}** → sent with header `x-admin-key: <key>`. The key is typed once
into the browser and kept in localStorage. Reject anything without the right key —
this is the only thing standing between the album and the public internet.

## Option B — Supabase (kind omitted)

    {
      "url": "https://xxxx.supabase.co",
      "key": "<anon key>",
      "bucket": "photos",
      "table": "photos"
    }

Bucket public for reads, insert allowed for anon; table `photos` with columns
`id uuid default gen_random_uuid()`, `path text`, `uploader text`,
`created_at timestamptz default now()`. Deletes are sent with the admin key as the
bearer token, so give delete rights to a service role, not to anon.

## Admin controls

Add `#admin` to the URL — https://tide.bjoed.com/#admin — and each picture gets a ×.
The first delete asks for the admin key and remembers it on that device.
