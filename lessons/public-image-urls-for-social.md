# Lesson: Social Platforms Require Truly Public, Direct Image URLs

**Applies to:** Any workflow that publishes images to Instagram, Facebook,
LinkedIn, Twitter/X, or any other social platform via API.

---

## The Principle

Social platforms fetch image URLs using a plain, unauthenticated HTTP GET from
their own servers. The URL must return the image binary directly. If the URL:

- redirects to a login page,
- requires cookies or auth headers,
- serves an HTML preview page instead of the raw file, or
- generates a transient pre-signed URL that expires before the platform fetches it,

...the image will fail to load — often silently.

---

## Why It Bites People

Most cloud file storage marketed as "public" is **browser-public**, not
**HTTP-public**. The distinction:

| | Browser public | HTTP public |
|---|---|---|
| Delivery mechanism | Redirect + cookie handshake | Direct binary response |
| What you need | A browser session | Nothing (plain GET) |
| Examples | Google Drive "Anyone with link", Dropbox shared link | S3 public bucket, WordPress media, your own web server |

AI agent tools tend to recommend the first option because it's the most familiar.
The failure is silent — the API accepts the request, the error only surfaces
when the platform tries to fetch the image later.

---

## How to Check Before Wiring It Up

Run this from a machine that is **not** your browser (a server, or `curl` on your
local machine):

```bash
curl -I "https://your-image-url.com/path/to/image.jpg"
```

A working URL returns:

```
HTTP/2 200
content-type: image/jpeg
content-length: 204800
```

A broken URL returns HTML (`content-type: text/html`) or a redirect to a login
page. If `curl` can't fetch it cleanly, a social platform's server can't either.

---

## Reliable Approaches (ranked by simplicity)

1. **WordPress Media Library** — if WordPress is already in the stack, upload
   there and use `source_url` from the response. Zero extra infrastructure.
   See [`tools/wordpress-media.md`](../tools/wordpress-media.md).

2. **S3-compatible bucket with public-read policy** — AWS S3, Cloudflare R2,
   Backblaze B2, or self-hosted Garage. Set a bucket policy that allows
   `s3:GetObject` for `Principal: "*"`. Verify with `curl -I` before using.
   See [`tools/garage-s3.md`](../tools/garage-s3.md).

3. **Your own web server** — serve the file from a path under your domain.
   As long as the path doesn't require auth, this is always reliable.

---

## Approaches That Fail

| Tool | Why it fails |
|---|---|
| Google Drive shared link | Auth redirect for headless clients |
| Dropbox shared link | Same issue — serves a preview page |
| OneDrive shared link | Same issue |
| Pre-signed S3 URLs with short TTLs | URL may expire before platform fetches |
| Localhost or private IP URLs | Not reachable from external servers |

---

## Related

- [Case study 001 — Auto-Publishing AI Images to Social Media](../case-studies/001-social-media-image-publishing.md)
- [Tool: Google Drive](../tools/google-drive.md)
- [Tool: WordPress Media Library](../tools/wordpress-media.md)
- [Tool: garage-s3](../tools/garage-s3.md)
