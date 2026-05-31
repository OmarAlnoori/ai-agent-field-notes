# WordPress Media Library

**Verdict:** ✅ Worked — reliable public image host for social media automation

---

## What It Is

WordPress exposes a REST API endpoint (`/wp-json/wp/v2/media`) that accepts
binary file uploads and stores them in the site's media library. Every uploaded
file gets a permanent, direct URL under the site's own domain — no auth required
to fetch it.

---

## Why It Works for Social Media

Social platforms (Instagram, Facebook, LinkedIn, Twitter/X) fetch image URLs
with plain server-side HTTP GET requests. The URL must return the image binary
directly, with no redirect to a login page and no auth headers required.

WordPress media URLs satisfy all of those conditions:

- Served directly from the web server (Apache/Nginx), not behind an app auth layer.
- `Content-Type` is set correctly (`image/jpeg`, `image/png`, etc.).
- No cookies or tokens needed to fetch.
- URL is permanent — it doesn't expire like a pre-signed S3 URL.

---

## Setup

### Prerequisites

- WordPress site with the REST API enabled (enabled by default since WP 4.7).
- A WordPress user with the `upload_files` capability (Author role or higher).
- An [Application Password](https://make.wordpress.org/core/2020/11/05/application-passwords-integration-guide/)
  for that user (WP 5.6+). Do not use the account password directly.

### Uploading via n8n

Use an **HTTP Request** node with these settings:

| Field | Value |
|---|---|
| Method | `POST` |
| URL | `https://yoursite.com/wp-json/wp/v2/media` |
| Authentication | Generic Credential Type → HTTP Basic Auth |
| Username | WordPress username |
| Password | Application Password (e.g. `xxxx xxxx xxxx xxxx xxxx xxxx`) |
| Body Content Type | Binary |
| Input Binary Field | `data` (or whichever field holds your image) |
| Headers | `Content-Disposition: attachment; filename="image.jpg"` |

### Uploading via curl (for testing)

```bash
curl -X POST \
  -H "Authorization: Basic $(echo -n 'username:app-password' | base64)" \
  -H "Content-Disposition: attachment; filename=test.jpg" \
  -H "Content-Type: image/jpeg" \
  --data-binary @/path/to/test.jpg \
  https://yoursite.com/wp-json/wp/v2/media
```

### Response

```json
{
  "id": 1234,
  "source_url": "https://yoursite.com/wp-content/uploads/2026/05/image.jpg",
  "media_type": "image",
  "mime_type": "image/jpeg",
  "media_details": {
    "width": 1024,
    "height": 1024
  }
}
```

The `source_url` is the permanent public URL. The `id` is used to set the
featured image on a post.

---

## Using the URL Downstream

### Set as featured image on a WordPress post

```json
// POST /wp-json/wp/v2/posts/<post-id>
{ "featured_media": 1234 }
```

Or include `"featured_media": {{ $json.id }}` when creating the post.

### Pass to Instagram Graph API

```
POST https://graph.facebook.com/v18.0/<IG_USER_ID>/media
  ?image_url=https://yoursite.com/wp-content/uploads/2026/05/image.jpg
  &caption=Your caption here
  &access_token=<TOKEN>
```

---

## Gotchas

**Application Passwords vs. account password**

The WP REST API rejects plain account passwords over HTTP Basic Auth when
Application Passwords are in use. Generate one under Users → Profile → Application
Passwords. The format with spaces is correct — don't remove them.

**File size limits**

WordPress inherits PHP's `upload_max_filesize` and `post_max_size` limits
(often 2MB or 8MB by default). AI-generated images at high resolution can exceed
this. Raise the limits in `php.ini` or `.htaccess`, or resize before uploading.

**Filename in Content-Disposition**

Without this header, WordPress saves the file with a generic name. Include it
for clean, searchable media library entries.

**HTTPS**

The site should be served over HTTPS. Some social platforms reject plain HTTP
image URLs.

---

## Alternatives

| Situation | Consider instead |
|---|---|
| No WordPress in the stack | S3-compatible store with public-read bucket policy |
| Need a CDN in front | Cloudflare R2 or AWS CloudFront + S3 |
| Self-hosted, no WordPress | [`garage-s3.md`](garage-s3.md) |

---

## See Also

- [Case study 001](../case-studies/001-social-media-image-publishing.md)
- [Lesson: public image URLs for social](../lessons/public-image-urls-for-social.md)
- [WordPress Application Passwords guide](https://make.wordpress.org/core/2020/11/05/application-passwords-integration-guide/)
- [WP REST API media reference](https://developer.wordpress.org/rest-api/reference/media/)
