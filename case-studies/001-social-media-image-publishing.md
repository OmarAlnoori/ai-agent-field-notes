# 001 — Auto-Publishing AI Images to WordPress and Social Media

> **Status:** Complete  
> **Stack:** n8n · WordPress · OpenAI image generation · Instagram Graph API  
> **Context:** Dot.ID marketing automation project

---

## The Goal

Fully automated pipeline: an AI model generates a featured image for a blog post,
that image is attached to the WordPress post as its featured image, and the same
image is published to Instagram (and other social platforms) — all without any
manual upload step.

The hard constraint: social platforms fetch images server-side at publish time.
The image URL must be directly accessible by an external server, with no login
wall, no redirect dance, no auth token.

---

## What We Tried

### Attempt 1 — Google Drive

**Why we tried it:** Google Drive is the obvious first choice for storing files
in a workflow. It has an n8n node, you can set files to "Anyone with the link",
and the shared link looks like a public URL.

**How we set it up:**

1. n8n Google Drive node: upload the generated image binary.
2. Set sharing to "Anyone with the link → Viewer".
3. Grabbed the share URL in the format:
   ```
   https://drive.google.com/file/d/<FILE_ID>/view?usp=sharing
   ```
4. Passed that URL to the Instagram Graph API `media` endpoint as `image_url`.

**What happened:** Instagram's API accepted the request (HTTP 200) but the media
object never became publishable. The subsequent `publish` call returned an error
indicating the image could not be fetched. The root cause: Instagram's servers
hit the Drive URL and are served a Google login/consent page instead of the image
binary. Drive's "public" share is public to browsers with cookie-based auth flows,
not to headless HTTP clients making a plain GET request.

The same issue applies to any social platform that fetches images server-side
(Facebook, Twitter/X card validator, LinkedIn).

**Verdict:** ❌ Failed

---

### Attempt 2 — garage-s3 (self-hosted S3-compatible storage)

**Why we tried it:** A self-hosted S3-compatible store (Garage) was suggested as
a way to get a permanent, truly public direct URL under our own domain — no auth
wall, no CDN quirks, full control over CORS and bucket policies.

**How we set it up:**

- Garage instance running on our server.
- Bucket configured with public read policy.
- n8n S3 node pointed at the Garage endpoint.

**What happened:** Integration did not complete successfully in our setup. The
exact errors and configuration details are still being documented.

> TODO: paste the error messages and config that caused the failure so others can
> learn from or fix them. If you got Garage working, please open a PR with the
> working config.

**Verdict:** ❌ Failed (in our setup — may work with correct config; see
[`tools/garage-s3.md`](../tools/garage-s3.md))

---

### Attempt 3 — WordPress Media Library

**Why we tried it:** WordPress already serves every uploaded image as a direct,
publicly accessible URL. The site is the thing we're publishing to anyway, so
using it as the image host costs nothing extra and requires no new infrastructure.

**How we set it up:**

1. n8n HTTP Request node: `POST /wp-json/wp/v2/media` with the image binary and
   `Authorization: Basic <base64(user:app-password)>` header.
2. Parse `source_url` from the JSON response — this is the permanent direct URL
   (e.g. `https://yoursite.com/wp-content/uploads/2026/05/image.jpg`).
3. Use that `source_url` in two parallel branches:
   - Set as `featured_media` ID on the WordPress post (`POST /wp-json/wp/v2/posts`).
   - Pass as `image_url` to the Instagram Graph API `media` endpoint.

```json
// WordPress media upload response (relevant fields)
{
  "id": 1234,
  "source_url": "https://yoursite.com/wp-content/uploads/2026/05/ai-image.jpg",
  "media_type": "image",
  "mime_type": "image/jpeg"
}
```

**What happened:** Both the WordPress post and Instagram accepted the image
without issues. Instagram fetched the image successfully on the first attempt.
No auth, no redirect, no timeout.

**Verdict:** ✅ Worked

---

## What We Shipped

```
[n8n trigger]
    → [AI image generation node]
    → [WordPress /wp/v2/media upload]
    → source_url
         ├── [WordPress /wp/v2/posts — set featured_media]
         └── [Instagram Graph API — create media container → publish]
```

The WordPress media upload is the single source of truth for the image URL.
Everything downstream reads `source_url` from the upload response.

---

## Key Takeaway

Social platforms fetch image URLs with plain HTTP GET requests from their own
servers — no cookies, no auth headers. Any storage that requires a login flow
(Google Drive, Dropbox) silently breaks this. The most reliable image host is
whichever server you already control and serve publicly, which for most content
workflows is WordPress itself.

Full principle: [`lessons/public-image-urls-for-social.md`](../lessons/public-image-urls-for-social.md)

---

## Related

- [`tools/google-drive.md`](../tools/google-drive.md)
- [`tools/garage-s3.md`](../tools/garage-s3.md)
- [`tools/wordpress-media.md`](../tools/wordpress-media.md)
- [`lessons/public-image-urls-for-social.md`](../lessons/public-image-urls-for-social.md)
