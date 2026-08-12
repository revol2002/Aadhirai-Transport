# Aadhirai Transport — Full Website + Admin Panel + Backend

Proprietor: **Gunasekaran S** · Sathyamangalam, Tamil Nadu – 638401
Phone: 89030 96477 · Email: aadhiraitransport@gmail.com
All India permit fleet — any vehicle type arranged.

## What's in this folder

```
aadhirai/
├── index.html          → public website (open directly in a browser to preview)
├── admin.html           → admin panel (login: admin / admin123 demo)
├── logo.png              → your uploaded logo, used across the site
├── backend/              → Node.js + Express API
│   ├── server.js
│   ├── config/db.js               (MySQL connection)
│   ├── config/authMiddleware.js   (JWT check for admin routes)
│   └── routes/
│       ├── auth.js         → POST /api/auth/login
│       ├── tracking.js     → GET  /api/tracking/:lr   (public)
│       ├── enquiries.js    → POST /api/enquiries       (public contact form)
│       └── adminShipments.js → GET/POST/PATCH shipments, GET enquiries (admin only)
└── database/
    ├── schema.sql   → run this first to create tables
    └── seed.sql     → default admin login + one demo shipment
```

## What's new

1. **Backend connection** — both `index.html` and `admin.html` now try the real backend first (`API_BASE_URL` at the top of each `<script>` block, default `http://localhost:5000/api`) and automatically fall back to demo data if it's not reachable. So the site still previews standalone, but the moment your backend is running, it's live.
2. **LR auto-generation** — the admin "New Shipment" button no longer asks for an LR number. The backend generates one itself (`AT-2026-00001`, incrementing per year) via the `lr_sequence` table, so numbers never clash.
3. **Customer reviews** — a "Reviews" section on the public site shows approved reviews and lets visitors submit their own (held for approval). The admin panel has a Reviews table to Approve / Hide each one.
4. **WhatsApp click-to-chat** — the phone number in the nav bar and contact section, plus a floating button in the bottom-right corner, now open a WhatsApp chat to 89030 96477 with a pre-filled message, instead of just dialing.

## 1. Preview right now (no setup)

`index.html` and `admin.html` work standalone in any browser — open them directly.
They try to reach a backend at `http://localhost:5000/api` first; if none is running, they fall back to **demo data** automatically, so you can see the design and flows immediately either way. The tracking page has one working demo ID: `AT-2026-00842`. Admin demo login: `admin` / `admin123`.

## 2. Set up the real backend + database

**Requirements:** Node.js (v18+), MySQL (v8+)

```bash
# 1. Create the database
mysql -u root -p < database/schema.sql
mysql -u root -p < database/seed.sql

# 2. Configure the backend
cd backend
cp .env.example .env
# edit .env — set DB_PASSWORD and a random JWT_SECRET

# 3. Install and run
npm install
npm start
# API now running at http://localhost:5000
```

## 3. Connect the frontend to the real backend

This is already wired up — both pages call the real API first and only use demo data as a fallback. All you need to do:

1. Get the backend running (step 2 above).
2. In `index.html` and `admin.html`, find the line near the top of the `<script>` block:
   ```js
   const API_BASE_URL = 'http://localhost:5000/api';
   ```
   Update it to wherever your backend is actually running (e.g. your deployed backend domain).
3. That's it — tracking, the contact form, reviews, admin login, and the admin dashboard will all start talking to the real database automatically. If the backend is ever unreachable, the pages quietly fall back to demo data instead of breaking.

**Endpoints in use:**
| Purpose | Method & Path |
|---|---|
| Track a shipment | `GET /api/tracking/:lr` |
| Submit contact enquiry | `POST /api/enquiries` |
| List approved reviews | `GET /api/reviews` |
| Submit a review | `POST /api/reviews` |
| Admin login | `POST /api/auth/login` |
| Admin dashboard stats | `GET /api/admin/stats` |
| Admin shipment list | `GET /api/admin/shipments` |
| Create shipment (auto LR) | `POST /api/admin/shipments` `{ origin, destination, vehicle_type }` |
| Update shipment status | `PATCH /api/admin/shipments/:id` `{ status, event_text }` |
| Admin enquiry list | `GET /api/admin/enquiries` |
| Admin review list (incl. pending) | `GET /api/admin/reviews` |
| Approve / hide a review | `PATCH /api/admin/reviews/:id` `{ is_approved }` |

Note: admin routes require `Authorization: Bearer <token>` from the login response — already handled in `admin.html`'s `authHeaders()`.

## 4. Deploying for real

- **Frontend**: host `index.html`, `admin.html`, and `logo.png` on any static host (GitHub Pages, Netlify, Vercel, or your own hosting).
- **Backend**: deploy the `backend/` folder to a Node host (Railway, Render, a VPS, etc.) and point it at a managed MySQL instance (PlanetScale, RDS, or your host's MySQL).
- Update the `http://localhost:5000` URLs above to your real backend domain, and set `JWT_SECRET` to a strong random value in production.
- **Change the default admin password** (`admin123`) immediately after your first real login — you can add a "change password" endpoint or update the `admins` table directly.

## Design notes

Colors and type are pulled directly from your logo: steel blue (#1E4A70 / #3E7CA6) and gold (#F2A93D) on a near-black base, with the logo's flag/speed-stripe motif reused as a recurring section divider and hover accent. Headings use Oswald (a condensed, industrial face) and tracking numbers use JetBrains Mono for a manifest/logbook feel.
