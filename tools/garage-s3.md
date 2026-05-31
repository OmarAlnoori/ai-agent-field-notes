# garage-s3 (self-hosted S3-compatible storage)

**Verdict:** ⚠️ Unverified — failed in our initial setup; may work with correct config

---

## What It Is

[Garage](https://garagehq.deuxfleurs.fr/) is an open-source, self-hosted,
S3-compatible object store. It is designed for multi-node, geographically
distributed deployments but can run on a single machine. It exposes the S3 API,
so any tool that speaks S3 (including n8n's S3 node) can use it.

The appeal for this use case: you control the server, you control the bucket
policy, and you can make objects publicly readable without relying on Google or
AWS.

---

## Why We Tried It

After Google Drive failed (see [`google-drive.md`](google-drive.md)), we wanted
a self-hosted alternative that would give us a truly public, direct image URL
under our own domain — no auth wall, full control over CORS and caching headers.

---

## What Happened in Our Setup

Integration with n8n's S3 node did not complete successfully. The exact failure
details are still being captured.

> **This section needs a real failure report.** If you attempted Garage and hit
> errors, please open a PR with:
> - The Garage version and deployment mode (single node / multi-node).
> - The n8n S3 node configuration (endpoint, region, path-style vs. virtual-hosted).
> - The exact error from n8n or from the Garage logs.
> - The bucket policy you applied (paste the JSON).

---

## Known Configuration Gotchas

These are common Garage + S3-client issues reported in the Garage community,
collected here as a starting checklist:

**Path-style vs. virtual-hosted addressing**

Most self-hosted S3 stores require path-style URLs
(`https://garage.example.com/bucket-name/key`) rather than virtual-hosted URLs
(`https://bucket-name.garage.example.com/key`). In n8n's S3 node, set
`Force path style: true`.

**Region string**

Garage accepts any region string but it must match exactly between the server
config and the client. A common safe value is `garage`. Mismatches produce
`SignatureDoesNotMatch` errors.

**Public bucket policy**

To make objects publicly readable without a pre-signed URL, apply this bucket
policy via the Garage admin API or `aws s3api put-bucket-policy`:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
    }
  ]
}
```

**CORS**

If n8n is making client-side requests (unlikely in node context, but worth
checking), add a CORS rule. For server-side n8n this usually doesn't matter.

---

## What a Working Setup Should Produce

A successful upload should return a URL in the form:

```
https://garage.yourserver.com/your-bucket/filename.jpg
```

That URL, when fetched with a plain `curl` (no auth headers), should return the
image binary with a `Content-Type: image/jpeg` header.

Test before wiring into n8n:

```bash
curl -I https://garage.yourserver.com/your-bucket/test.jpg
# Expected: HTTP/1.1 200 OK, Content-Type: image/jpeg
```

---

## Alternatives If You Can't Get Garage Working

| Option | Notes |
|---|---|
| [WordPress Media Library](wordpress-media.md) | Simplest if WordPress is already in the stack |
| AWS S3 + public bucket | Managed, reliable, costs money |
| Cloudflare R2 | S3-compatible, generous free tier, CDN included |
| Backblaze B2 | S3-compatible, cheap egress |

---

## See Also

- [Garage documentation](https://garagehq.deuxfleurs.fr/documentation/)
- [Case study 001](../case-studies/001-social-media-image-publishing.md)
- [Lesson: public image URLs for social](../lessons/public-image-urls-for-social.md)
