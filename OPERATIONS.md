# Lean Hippo — Booking system: how it works & how to never miss a booking

## Plain-English preference
Explain things to the owner simply (like to a beginner). Avoid jargon; use analogies.

## How a booking flows
1. Customer fills the booking form on leanhippo.io.
2. The form POSTs the details (JSON) to the n8n webhook: https://n8n.leanhippo.io/webhook/booking
3. n8n sends: (a) a branded confirmation to the customer, (b) a booking alert to contact@leanhippo.io.
4. n8n replies {"success":true} -> the customer sees the "Request Received" screen.

## Key facts
- The customer ONLY sees "Request Received" if n8n actually accepted and processed the booking.
  If n8n is down, the form shows an error telling them to email contact@leanhippo.io (it does NOT
  falsely say "confirmed").
- The n8n workflow ("Lean Hippo — Booking emails") must be **Published** (newer n8n's word for "active").
- Email send settings (Hostinger): host smtp.hostinger.com, port 465 (SSL on) OR 587 (SSL off / STARTTLS),
  user contact@leanhippo.io, password = the MAILBOX password (set in Hostinger hPanel -> Emails).
- Test the engine directly with the curl test; {"success":true} = healthy, 404 = not published.

## Deploy the website (build lives in this repo: leanhippo-deploy/out)
Pull the latest build onto the server:
  ssh root@213.210.21.81 'set -e; T=$(mktemp -d); curl -fsSL https://codeload.github.com/nuhayrhafiz1-star/leanhippo-site/tar.gz/refs/heads/main -o "$T/s.tgz"; tar -xzf "$T/s.tgz" -C "$T"; S=$(echo "$T"/*/leanhippo-deploy/out); test -f "$S/index.html"; cp -a "$S/." /var/www/leanhippo/; rm -rf "$T"; echo DEPLOYED'

## Never-miss-a-booking safety nets (fireproofing)
1. Google Sheet logbook: n8n appends every booking to a spreadsheet (source of truth; survives email loss/spam).
2. Phone alert (Telegram/WhatsApp): second channel, independent of email.
3. n8n Error Workflow: alerts the owner if processing fails.
4. Website fallback: if n8n is unreachable, the form auto-opens a pre-filled email so nothing is lost.
5. UptimeRobot (free): warns the owner if n8n goes down.

## Spreadsheet logbook
- Google Apps Script web app (owner: nuhayrhafiz1@gmail.com Google Drive).
- URL baked into the site as NEXT_PUBLIC_BOOKING_LOG_URL.
- Sheet "Lean Hippo Bookings" gets a row per booking, independent of n8n.
