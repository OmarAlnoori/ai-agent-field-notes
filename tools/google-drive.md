# Google Drive

**Verdict:** ❌ Failed as a social-media image host

---

## What It Is

Google Drive is Google's cloud file storage. Files can be shared via link with
varying permission levels. It has a native n8n node and is a common first choice
for storing binary assets in workflows.

---

## Why People Try It

- Already in use for most teams; zero new infrastructure.
- n8n has a first-party Google Drive node.
- The "Anyone with the link" share setting sounds like "public URL".
- The share link visually resembles a direct file URL.

---

## The Problem

Google Drive's "public" sharing is **browser-public, not HTTP-public**.

When a headless client (Instagram's ingest server, Facebook's crawler, Twitter's
card validator) makes a GET request to a Drive share URL, it receives a Google
sign-in or consent page — not the file binary. The response is HTTP 200 with
HTML, which the client either rejects or silently drops.

This fails even when:
- The file is set to "Anyone with the link → Viewer".
- You use the `/uc?export=download` URL variant.
- You use the direct `lh3.googleusercontent.com` CDN URL (requires prior
  browser-side auth to generate and is not stable).

The underlying cause is that Google requires some form of session context (cookie,
OAuth token) to serve the file, which headless HTTP clients don't provide.

**Error pattern from Instagram Graph API:**

```
{
  "error": {
    "code": 36007,
    "message": "The image url you specified is not accessible...",
    "type": "OAuthException"
  }
}
```

Other social platforms fail silently — the media object is created but never
becomes publishable, or the post goes out with a missing image.

---

## Where It Does Work

- Storing files for download by humans clicking links.
- Passing files between n8n nodes using the binary data object (not a URL).
- Archiving generated assets for your own access — not for external consumption.

---

## Alternatives

| Use case | Better option |
|---|---|
| Public image URL for social | [`wordpress-media.md`](wordpress-media.md) |
| Self-hosted public storage | [`garage-s3.md`](garage-s3.md) |
| Truly public CDN | Any S3-compatible bucket with public-read policy |

---

## See Also

- [Case study 001](../case-studies/001-social-media-image-publishing.md)
- [Lesson: public image URLs for social](../lessons/public-image-urls-for-social.md)
