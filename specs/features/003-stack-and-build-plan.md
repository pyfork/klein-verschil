<!-- lead-dev-Opus5.5-agent, 2026-10-27 -->

# 003: Tech stack and build plan

Core stack: **Go (chi), React with TypeScript, and Postgres.** Each choice and its reason is below.

## Stack

| Layer | Choice | Why |
|---|---|---|
| Server | Go, `chi` router | One static binary to deploy. `chi` stays close to the standard `net/http`, so there's little framework to learn or fight. |
| Public pages (listing, area, near, home, about) | Go `html/template`, server-rendered | Meets SEO spec 002 section 1 (all content in the first HTML response) without running a second Node server. Fast on phones. |
| Interactive bits on public pages (gallery lightbox, enquiry sheet, nearby map with MapLibre) | Small React and TypeScript "islands", mounted onto server-rendered HTML | The page works and reads fine before JavaScript loads. React adds only the interactive parts. |
| Signed-in app (owner editor, leads, tenancy record, payments, admin) | React and TypeScript single-page app, built with Vite, served by Go under `/owner`, `/account`, `/admin` and `/tenancy` | Rich UI that never needs to be indexed (SEO spec section 4 blocks these paths). |
| Database | Postgres 18, with PostGIS for location search | One database for data, job queue and sessions. PostGIS handles distances, areas and "nearest" searches. |
| Database access | `pgx` and `sqlc` (SQL written by hand, Go code generated) | Plain SQL, type-safe, no ORM magic. |
| Migrations | `goose`, SQL files | Simple and versioned. The app role isn't a superuser, so every migration ends with `GRANT … TO` that role. Extensions are created once by an admin. |
| Background jobs | `river` (a Postgres-backed job queue) | Used for media processing, emails, sitemap rebuilds and nearby calculations. No Redis needed. |
| Sessions | Server-side sessions in Postgres, `HttpOnly` + `Secure` + `SameSite=Lax` cookie | Easy to revoke, and no tokens kept in the browser. |
| Sign-in | Email 6-digit one-time code (spec 001) | Codes stored hashed, with rate limits in Postgres. |
| Platform text | Message catalogue files (`en` now, `ms` reserved), used by both Go templates and React | Spec 002 section 3: Malay later means adding text, not rewriting. |
| Files (photos, video, tenancy documents) | S3-compatible object storage. Public media goes behind a CDN. Tenancy files are in a private bucket and only served through expiring signed links. | Keeps big files off the server. Private files are never public (spec 001 section 4). |
| Image and video processing | `ffmpeg` and `vips` in a separate, resource-capped worker process | Resizing, WebP/AVIF, video for the web, poster frames, and EXIF stripping (dev spec B8). See "Sharing the server safely". |
| Front-end checks | TypeScript strict mode, ESLint, Vitest, and Playwright for end-to-end tests | |
| Go checks | `go vet`, `staticcheck`, `go test` | |
| Speed checks | Lighthouse CI budgets on listing, area and home pages | SEO spec section 9. |

## Sharing the server safely (resource limits and queue)

Production runs on a small server. Media processing must never starve the web process, or anything else on the machine, of CPU, memory or disk.

**Three separate processes, each with its own limits:**

| Process | What it does | Limits |
|---|---|---|
| `web` | Serves pages and the API | Memory capped at 400 MB, at most 1 CPU core, normal priority |
| `worker` (media) | Photo and video processing with `vips` and `ffmpeg` | Runs inside a capped group: at most **half of one CPU core**, lowest CPU and disk priority, slows down above 400 MB of memory, hard stop at 600 MB, never uses swap |
| `worker` (light) | Emails, sitemap rebuilds, nearby calculations | Memory capped at 200 MB, low priority |

The same Go binary runs as `web` or `worker`. The heavy work never runs inside `web`, so a big video can't slow page loads.

**Queue rules (everything else waits its turn):**
- Every upload becomes a job in the Postgres job queue (`river`). Nothing is processed during the upload request itself.
- **One photo job and one video job at a time**, site-wide. All other jobs wait in the queue, first in, first out.
- Photos have their own lane, so a long video never holds up the 3 photos a landlord needs to publish.
- **Back-off when the server is busy:** before starting a media job, the worker checks the server load. If load is high (1-minute load average above 1.5) or free disk is under 5 GB, the job goes back in the queue and retries 5 minutes later.
- Each tool runs single-threaded (`ffmpeg -threads 1`, `VIPS_CONCURRENCY=1`).
- Time limits: 2 minutes per photo, 45 minutes per video. A failed job retries 3 times with growing gaps, then shows "Processing failed, please re-upload" to the landlord (dev spec B8).
- Every job works in its own temporary folder, deleted when the job ends, pass or fail.
- Upload limits protect the queue: photos up to 25 MB, videos up to 3 minutes and 500 MB. Videos are converted to at most 1080p.
- Tenancy documents (agreements and move-in files) are stored as uploaded. They only get a small preview image, which goes through the photo lane.

**What the landlord sees:** "Processing" on each item while it waits. The listing can still publish once 3 photos are ready. The intro video is optional and shows up when it's done.

**Measured under the cap (2026-10-27):** a 30-second 1080p encode took about 1 minute and used about 210 MB of memory. Estimate: a 3-minute phone video takes 6 to 15 minutes, depending on its format. That's fine for a queued, optional video.

## Build plan (chunks, in order)

Each chunk ends deployed to the server and checked there.

1. **Skeleton.** Go and chi server, health check, Postgres connection, migrations, deploy to the server, staging set to `noindex` behind a password, `robots.txt` and an empty sitemap.
2. **Sign-in.** Email one-time code, sessions, and roles (landlord, renter, tenant, admin).
3. **Listing editor, part 1.** Create a listing, basic facts, address and pin confirmation, autosave.
4. **Media.** Photo upload, the processing worker, sections, cover photo, publish gate, owner intro video with captions.
5. **Public listing page.** Server-rendered page, quick facts box, gallery island, structured data, share previews, rented and deleted states.
6. **Find and contact.** Home page, search and results, area pages, enquiries, lead reveal, saved homes.
7. **Payments.** Plans, the free-plan limit, DuitNow QR pay screen, admin confirm list, receipts, and the paying-landlord counter.
8. **Tenancy record.** Mark as rented, set up the tenancy, invite the tenant, uploads that can't be changed, expiring download links.
9. **Nearby.** Places data, walking times, "What's nearest" rows, "near" pages, data answers. Data sources are set out under "Decisions" below.
10. **Admin.** Directory, duplicate check and merge, remove a listing or user.
11. **Launch pass.** SEO checklist (spec 002 section 12), speed budgets, about page, Search Console and Bing, and a logged-out privacy crawl.

**Payments go live at the end of chunk 7.** Chunks 8 to 11 ship after that, so revenue starts as early as possible. Chunk 9 is the slowest, and it isn't needed to take money.

## Decisions

| Decision | Technical reason |
|---|---|
| Public pages rendered by Go templates, with React islands | All content is in the first HTML response for search engines (spec 002 section 1), with no Node server to run beside Go. |
| Separate `web`, media `worker` and light `worker` processes from one binary | A long video encode can't slow page loads. Each process gets its own resource limits. |
| Media worker in a capped resource group, with one photo job and one video job at a time | Predictable load on a small server. Everything else waits in the queue instead of competing for CPU. |
| S3-compatible object storage (Singapore region). Public media behind a CDN. Tenancy documents in a separate private bucket with no CDN. | Photos load fast near Malaysian users. Private documents exist in one place only, never in CDN caches, and are only reachable through expiring signed links. A wrong setting on public media can't expose them. |
| Amazon SES for email, from a domain with DKIM, SPF and DMARC | Low cost per email. Authenticated domains keep sign-in codes out of spam. |
| PostGIS for location search | Real distance, area and nearest-neighbour queries with spatial indexes, instead of hand-rolled maths. |
| MapLibre GL JS for maps, loaded only when the map is scrolled into view or tapped | Open source, vector maps in our own style, and no per-view fee. Lazy loading keeps it off the critical path (spec 002 section 9). |
| OpenStreetMap vector tiles for Malaysia, served from object storage behind the CDN | No map-tile bill. In chunk 9, confirm the CDN supports the range requests the tile file needs. |
| OpenStreetMap places stored in PostGIS. Transit from data.gov.my GTFS. | The licence allows storing and indexing places, which the "near" pages need (spec 002). Google Places doesn't allow that. |
| A hosted routing service for walking and driving times | Self-hosting routing for Malaysia needs more memory than a small server has. |
| "Open in Google Maps" links rather than an embedded Google map | People get familiar navigation without Google's map on our pages, so pages stay fast and there's no Maps API bill (spec 001). |

Map attribution "© OpenStreetMap contributors" is always visible on the map.
