<!-- product-strategist-Opus5.5-agent, 2026-10-27 -->

# 001: Launch v1, the smallest version that can take money

## Goal

Get to the day landlords can pay, because that's when the 60-day test starts. To pass, 30 landlords pay within 60 days.

Build only what a landlord and a renter need for that. Everything else waits.

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
- Autosave. No Save button.
- "Preview as renter" before publishing.

### 2. Renter finds and contacts

- Home page, area search, results list and listing page, as in the mocks.
- The listing page shows the gallery, the owner intro, key facts, what's nearest, price and deposit.
- The Enquire button sends a message to the landlord. The landlord's phone number is never shown publicly. The landlord sees the lead and chooses to reveal it (as in the mocks).
- A safety line on every listing: "Never pay a deposit before viewing the unit."
- Saved homes and My enquiries, for signed-in renters.

### 3. Landlord pays

| Plan | Price | Includes |
|---|---|---|
| Free | RM0 | 1 live listing |
| Up to 3 | RM299 a year | 3 live listings, each with a tenancy record |
| More than 3 | RM599 a year | More than 3 live listings, each with a tenancy record |
| Tenancy record | RM99 a year per home | For the free plan's home |

- Payment method at launch: **Pay with QR (DuitNow QR) only**, confirmed by hand by the admin. FPX and card through a gateway come later (see Out of scope).
- **Pay with QR:**
  - The pay screen shows a DuitNow QR. The landlord scans it with any Malaysian banking or e-wallet app.
  - On a phone, the QR is shown with a "Save QR" button and the steps "open your bank app, scan from gallery", since people can't scan their own screen.
  - Source image (private, not in the repo): `docs/spike puascari property/uploads/duitnow-qr.jpg`. It's a static merchant QR for "KIP ADVISORS".
  - A static QR doesn't tell us who paid or how much. So after paying, the landlord types the bank reference number and can attach a screenshot of the receipt. The plan shows "Payment being checked". The admin sees a "QR payments to confirm" list, checks the bank statement, and taps Confirm. The plan then activates and the receipt goes out.
  - Promise shown to the landlord: confirmed within 1 working day.
  - Trust line on the pay screen, above the QR: "Payments go to Kip Advisors, the company behind Sewa Rumah by Puascari." Kip Advisors is the business entity, Sewa Rumah is the service, and Puascari is the brand.
- Receipts, terms and the privacy policy name **Kip Advisors** as the seller. Everything else in the app says **Sewa Rumah by Puascari**.
- Receipt by email and in the app.
- Yearly plans don't renew automatically in v1. A reminder goes out 14 days before the end. This avoids surprise charges and refund disputes.
- Trying to publish a 2nd home on the free plan opens the upgrade screen. Nothing is lost while the landlord decides.
- The admin sees a count of paying landlords against the target of 30, and the days left.

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
- Paying-landlord count and the 60-day countdown.
- "QR payments to confirm" list: landlord, plan, amount, reference, receipt screenshot, and Confirm or Reject (Reject asks for a reason, which is sent to the landlord).

## Out of scope for v1

- Boost (RM100)
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
- A landlord can pay RM299, RM599 or RM99 by DuitNow QR and enter the reference. Once the admin confirms the payment, the plan activates and the admin count goes up by 1.
- A landlord and tenant can both open the same tenancy record and download the agreement.
- Every item on the SEO launch checklist (`specs/features/002-seo.md` section 12) passes.
- Every screen works at 320 px wide, with no sideways scrolling (responsiveness table in the mocks).

## Open questions (one at a time, in this order)

1. ~~Owner intro video length~~ **Decided 2026-10-27:** up to 3 minutes, with 60 to 90 seconds suggested.
2. ~~Payment provider~~ **Decided 2026-10-27:** DuitNow QR only at launch, confirmed by hand. A gateway comes later.
3. ~~Tenant sign-in~~ **Decided 2026-10-27:** email one-time code for everyone. No SMS (it's complex to integrate and costs money per message).
4. ~~Website address~~ **Decided 2026-10-27:** `property.puascari.com`, with `puascari.com/sewarumah` sending people there through a permanent (301) redirect.
5. ~~First language~~ **Decided 2026-10-27:** English first. The Malay build is parked.
