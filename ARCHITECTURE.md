# get_still audio: architecture

Last updated 2026-10-11 (CT).

## 1. Domain map
- **getstill.space** is the main site, hosted on Vercel (A @ 76.76.21.21, CNAME www to cname.vercel-dns.com). Its DNS is on **Namecheap**, along with the Google Workspace mail for rob@getstill.space (MX smtp.google.com, SPF, DKIM `google._domainkey`, DMARC). **Never switch its nameservers.**
  - The legacy `audio.getstill.space` is a CNAME to Cloudflare Pages `getstill-audio.pages.dev`, built from this repo's `cdn/`. Keep it as a fallback until the app uses audio.getstill.site.
- **getstill.site** is on Cloudflare's Free plan (account Rob@fsyncos.com), with nameservers `malcolm.ns.cloudflare.com` and `veronica.ns.cloudflare.com`. It is used only for R2 audio hosting.

## 2. R2
- Bucket: `getstill-audio`. Custom domain: `audio.getstill.site` (Active). The `r2.dev` URL is disabled.
- CORS:
  `[{"AllowedOrigins":["*"],"AllowedMethods":["GET","HEAD"],"AllowedHeaders":["*"],"ExposeHeaders":["Content-Length","Content-Range","Accept-Ranges","ETag"],"MaxAgeSeconds":86400}]`
- Cache rule on the getstill.site zone: hostname equals `audio.getstill.site`, eligible for cache, edge TTL 1 month.
- An R2 billing/usage alert is set in Cloudflare Notifications.
- Not yet verified: the cache rule and the alert were being set up when this was written, so check them in the dashboard.

## 3. Uploads
- The GitHub Actions workflow `upload-r2` (`.github/workflows/upload-r2.yml`) syncs `r2/` to the bucket and checks range requests on `audio.getstill.site`. Its results go to `r2-report.txt`.
- It uses the repo secrets `R2_ACCOUNT_ID`, `R2_ACCESS_KEY_ID` and `R2_SECRET_ACCESS_KEY`. The token is Object Read & Write, scoped to this bucket only. **Never paste keys into any chat, file or log.**
- Files are the original 192k MP3s, including Kevin MacLeod's originals. Don't re-encode anything, and don't use AAC, m4a, FLAC or WAV. Content-Type is `audio/mpeg`.
- The first successful run was 2026-10-11 at 1:43 AM CT. Every track returned 206 for a range request.

## 4. Why R2 instead of Pages
- Pages has a 25 MiB per-file cap, which forced about 115 kbps for 30-minute tracks and cost audio quality.
- Seeking didn't work in the player on Pages.
- R2 has no egress fees. Its free tier covers 10 GB of storage, 1M Class A and 10M Class B operations a month; anything past that is billed, not throttled.

## 5. Verification
With `U=https://audio.getstill.site/temples/<file>.mp3`:
- `curl -sI $U` should return 200 with `audio/mpeg` and `accept-ranges: bytes`.
- `curl -s -o /dev/null -D - -H 'Range: bytes=1000000-1000099' $U` should return 206 with the right `content-range`.
- `curl -s -o /dev/null -w '%{http_code}' -H 'Range: bytes=999999999999-' $U` should return 416.
- Repeating the same request should show `cf-cache-status: HIT`. If it shows `DYNAMIC`, the cache rule is missing.
- Finally, drag the scrubber in Safari and Chrome.

## 6. Never
- Switch getstill.space's nameservers or edit its mail records (MX, SPF, DKIM, DMARC).
- Delete the `getstill-audio` bucket or its custom domain without asking Roberto.
- Enable `r2.dev`, or put R2 keys anywhere other than GitHub secrets.
