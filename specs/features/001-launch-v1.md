<!-- product-strategist-Opus5.5-agent, 2026-10-27 -->

# 001: Launch v1, the smallest version that can take money

## Goal

Reach the first paying landlord as early as possible. Build only what a landlord and a renter need to list, find, contact and pay. Everything else waits until payments are live.

"Dev spec" below means the media and location developer spec that came with the design mocks.

## In scope

### 1. Landlord lists a home

- Landlord signs up and signs in by email one-time code (see "Sign-in" below).
- Landlord adds a phone or WhatsApp number to their profile. It's used only when they choose to reveal it to a renter, and it isn't verified in v1.
- Stepped editor as in the mocks: Photos, Location, Review, Publish.
- Publishing needs only three things (dev spec A3): the pin is confirmed, there are at least 3 photos, and a cover photo is set.
- Basic facts: price per month, home type, bedrooms, bathrooms, size, furnishing, available-from date and deposit.
- Owner intro video: optional, but nudged hard. The listing card shows "Video" or "No video".
- Owner intro video length: up to 3 minutes. The editor suggests 60 to 90 seconds. This replaces the 90-second limit in dev spec B7 and B9.
- Exact or approximate location (dev spec C2).
- Google Maps link: optional (see "Google Maps link on map cards" below).

#### Google Maps link on map cards

Our maps are drawn with MapLibre (spec 003). Clicking a home's map card, or its photo, opens that home in Google Maps. People already know Google Maps and can start navigating from there.

- **Landlord input (optional):** in the Location step, the landlord pastes a Google Maps share link or the embed code. We accept only Google Maps addresses (`google.com/maps`, `maps.google.com`, `maps.app.goo.gl`, `goo.gl/maps`). From embed code we keep only the link inside it. Anything else is rejected with "Paste a link from Google Maps".
- **Check it matches the pin:** the server opens the link, reads its location, and compares it with the confirmed pin. If they're more than 200 m apart, the landlord sees "This Google Maps link points somewhere else. Check it or remove it." This is a warning only. It never blocks publishing (dev spec A3).
- **No link pasted:** we build one automatically from the confirmed pin, using Google's public Maps link format (`https://www.google.com/maps/search/?api=1&query={lat},{lng}`). This needs no Google account or key, so every exact-location home gets the feature.
- **Approximate location (privacy):** no Google Maps link is shown publicly, because it would give away the exact spot. The card shows "Exact location shared after you enquire". The landlord can share the link in their reply.
- **Where a card click goes:**
  - **Map on a home's own page:** clicking the map card or its photo opens Google Maps.
  - **Search results map (many homes):** clicking the card or its photo opens our listing page. The card also has a small "Open in Google Maps" button with the map-pin icon (Heroicons), so navigation is one tap away. Renters see the photos, video and price before they leave the site.
  - The button has a 44 px tap area and the visible text "Open in Google Maps", never an icon alone.
- **The link:** opens in a new tab, and on phones it opens the Google Maps app. Marked `rel="nofollow noopener"`. It's an outside link, not a page we want Google to rank.
- We never embed Google's own map on our pages. That keeps pages fast and avoids Google's billing.
- Autosave. No Save button.
- "Preview as renter" before publishing.

### 2. Renter finds and contacts

- Home page, area search, results list and listing page, as in the mocks.
- The listing page shows the gallery, the owner intro, key facts, what's nearest, price and deposit.
- The Enquire button sends a message to the landlord. The landlord's phone number is never shown publicly. The landlord sees the lead and chooses to reveal it (as in the mocks).
- A safety line on every listing: "Never pay a deposit before viewing the unit."
- Saved homes and My enquiries, for signed-in renters.

### 3. Landlord pays

| Plan | Includes |
|---|---|
| Free | 1 live listing |
| Small (yearly) | Up to 3 live listings, each with a tenancy record |
| Large (yearly) | More than 3 live listings, each with a tenancy record |
| Tenancy record add-on (yearly, per home) | A tenancy record for the free plan's home |

- Plan names, prices and listing limits come from settings in the database, never from code. That way prices can change without a release.

- Payment method at launch: **Pay with QR (DuitNow QR) only**, confirmed by hand by the admin. FPX and card through a gateway come later (see Out of scope).
- **Pay with QR:**
  - The pay screen shows a DuitNow QR. The landlord scans it with any Malaysian banking or e-wallet app.
  - On a phone, the QR is shown with a "Save QR" button and the steps "open your bank app, scan from gallery", since people can't scan their own screen.
  - The QR image is a static merchant QR, uploaded by the admin and kept in settings (not in the repo).
  - A static QR doesn't tell us who paid or how much. So after paying, the landlord types the bank reference number and can attach a screenshot of the receipt. The plan shows "Payment being checked". The admin sees a "QR payments to confirm" list, checks the bank statement, and taps Confirm. The plan then activates and the receipt goes out.
  - Promise shown to the landlord: confirmed within 1 working day.
  - Trust line on the pay screen, above the QR: "Payments go to {legal entity}, the company behind Sewa Rumah by Puascari." The QR shows the legal entity's name in the payer's bank app, so the line stops that name from looking unfamiliar.
- Receipts, terms and the privacy policy name the **legal entity** (from settings) as the seller. Everything else in the app says **Sewa Rumah by Puascari**.
- Receipt by email and in the app.
- Yearly plans don't renew automatically in v1. A reminder goes out 14 days before the end. This avoids surprise charges and refund disputes.
- Trying to publish a 2nd home on the free plan opens the upgrade screen. Nothing is lost while the landlord decides.
- The admin sees a count of paying landlords.

### 4. Tenancy record

- After "Mark as rented", the landlord sets up the tenancy: tenant name, start and end dates, rent, deposits and rent due day (as in the mocks).
- The landlord invites the tenant by WhatsApp or email link. The tenant opens the record by signing in with their own email one-time code. The invite link alone never opens the record.
- Both sides can upload and download:
  - the signed tenancy agreement (PDF or photos)
  - move-in photos and videos, time-stamped by the server on upload
- Once uploaded, files can't be edited. Either side can add more, but nobody can quietly replace a file. Each file shows who uploaded it and when.
- Files stay available for the whole tenancy plus 1 year, even if the plan lapses. The landlord just can't add new records.
- Files are private. They're never public, never in search engines, and downloads use expiring links.

### Sign-in (everyone: landlords, renters, tenants, admin)

- Email plus a **6-digit one-time code**. No passwords and no SMS.
- The code is valid for 10 minutes and works once. After 5 wrong tries, the person waits 15 minutes.
- Limit: 5 codes per email per hour, and a limit per device or network too, so nobody can flood an inbox.
- The email shows the code in large text and also has a one-tap sign-in link, valid for the same 10 minutes.
- The person stays signed in for 30 days on that device. The admin must sign in again every 12 hours.
- Codes are sent from a domain with SPF, DKIM and DMARC set up, so they don't land in spam.

### 5. SEO (must have)

Everything in `specs/features/002-seo.md`, sections 1 to 13 (13 is AEO and GEO). Summary:

- Public pages are server-rendered and URLs are clean. The site launches in English only. All platform text goes through a translation layer, so Malay can be added later.
- Sitemaps, `robots.txt` and canonicals are set up. Rented, deleted and draft pages are handled correctly.
- Structured data includes the video for owner intros.
- Area and "near" pages only appear once at least 5 homes qualify.
- Mobile speed targets are met.
- Private pages and approximate locations are never visible to search engines.
- Search Console is set up before launch.

### 6. Admin

- Listings directory, duplicate check and override (as in the mocks).
- Remove a listing and block a user.
- Paying-landlord count.
- "QR payments to confirm" list: landlord, plan, amount, reference, receipt screenshot, and Confirm or Reject (Reject asks for a reason, which is sent to the landlord).

## Out of scope for v1

- Boost (paid extra views for a listing)
- Rent reminders and rent tracking for the tenant
- Malay language (parked). The `/ms/` URL prefix is reserved for it.
- Chinese language
- Owner-added nearby places (dev spec C13)
- Guides and blog
- Automatic plan renewal
- Payment gateway (FPX and card with automatic confirmation). Add it once checking QR payments by hand gets busy. Stripe or Billplz, chosen then.

## Done means

- A landlord can sign up, list a home with video, and publish it in under 15 minutes on a phone.
- A renter can find that home on Google (once it's indexed), view it and send an enquiry.
- A landlord can pay for any plan or the add-on by DuitNow QR and enter the reference. Once the admin confirms the payment, the plan activates and the admin count goes up by 1.
- A landlord and tenant can both open the same tenancy record and download the agreement.
- Every item on the SEO launch checklist (`specs/features/002-seo.md` section 12) passes.
- Every screen works at 320 px wide, with no sideways scrolling (responsiveness table in the mocks).

## Decisions

| Decision | Technical reason |
|---|---|
| Owner intro video up to 3 minutes, 60 to 90 seconds suggested | Long enough to show a home. The suggestion keeps most videos short, so they're watched to the end and process quickly. |
| DuitNow QR only at launch, confirmed by the admin | No payment gateway to integrate before launch. Works with every Malaysian banking and e-wallet app. A gateway can be added later without changing plans. |
| Email one-time code for everyone, no SMS | One sign-in method to build and secure. No per-message cost and no SMS provider to integrate. |
| `property.puascari.com`, with `puascari.com/sewarumah` redirecting there (301) | One indexable host. The short link keeps any ranking it earns. |
| English first, Malay later under `/ms/` | One language to build and test first. A translation layer from day one keeps Malay cheap to add. |
