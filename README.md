# Zero Device — POS System

Single-file **Point of Sale (POS)** web app for a technology retail store (laptops, accessories, etc.).  
All data is stored in the browser (`localStorage`). Warranty reports can optionally sync to **Firebase Cloud Firestore** for online QR verification.

**File:** `zero-device-pos.html`  
**Version:** 1.2.0

---

## Features

| Area | What it does |
|------|----------------|
| **Dashboard** | Sales overview, charts, quick stats |
| **Billing** | Create invoices, print, payment status |
| **Inventory** | Products, stock, categories, buy/sell price |
| **Customers** | Customer records (name, NIC, mobile, email) |
| **Sales History** | Past bills and transactions |
| **Warranty** | Per-bill warranty codes + QR (online lookup via Firebase) |
| **Settings** | Business info, billing, warranty months, users, Firebase, backup |
| **Users** | Admin / staff accounts with login |
| **Backup** | Export / import JSON backups |
| **Reset** | Clear all local data + Firestore warranties (empty default) |

---

## How to open

1. Download / extract the project folder.
2. Open `zero-device-pos.html` in a modern browser (Chrome, Edge, Firefox).
3. For Firebase warranty QR to work reliably, serve the file over **http/https** (not `file://`):
   - Example: `npx serve .` or any local static server  
   - Or host on Netlify / GitHub Pages / your own domain.

---

## Default login

| Field | Value |
|-------|--------|
| **Username** | `admin` |
| **Password** | `admin123` |

Change the password after first login (Settings → User Management), or create additional staff accounts.

---

## First run vs empty state

- **First time** you open the app (no saved data yet): sample products and a few sample customers are loaded automatically so you can try the system.
- After **Reset All Local Data**, the app stays **empty** (no products, no customers, no bills, no warranties). Sample data is **not** re-seeded on reload.

---

## Firebase (online warranty)

Warranty reports can be stored in **Cloud Firestore** so a customer can open a QR link and view the warranty without logging into the POS.

### Setup steps

1. Go to [Firebase Console](https://console.firebase.google.com/) → create / select a project.
2. **Build → Firestore Database → Create database** (start in **test mode** if testing).
3. Project settings → **Your apps** → Web app → copy the config object (`apiKey`, `projectId`, etc.).
4. In the POS: **Settings → Firebase** → paste the config → **Connect**.
5. Set Firestore rules (recommended for this demo app):

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /warranties/{code} {
      allow read, write: if true;
    }
    match /_meta/{doc} {
      allow read, write: if true;
    }
  }
}
```

> **Note:** These rules are open (anyone with the document ID can read/write). Only warranty data is sent online — not products, prices, or login credentials. For production, tighten rules (e.g. authenticated writes only).

6. Publish the rules. Then create bills as usual; warranties sync when Firebase is connected.

### Public warranty page

Invoice QR links open a read-only warranty view using the hash:

`#warranty/ZD-WB-xxxxx-XXXXX`

Optional query param carries a compact Firebase project/key reference so the page works even without the POS session.

---

## Data & Backup

**Settings → Data & Backup**

| Action | Description |
|--------|-------------|
| **Export Backup** | Download a JSON file of products, customers, bills, warranties, settings |
| **Import Backup** | Restore from a previously exported JSON |
| **Reset All Local Data** | Danger zone — see below |

### Reset All Local Data

Deletes:

- All products & stock  
- All customers  
- All bills / sales history  
- All warranties **locally** and in **Firestore** (if Firebase is connected)

Keeps:

- Business settings  
- Firebase config / connection  
- Admin user(s)

After reset the app is **empty** (no sample data). Two confirmation dialogs are required.

If Firestore delete fails, an alert shows the error (often security rules or network). Local data is still cleared.

---

## Storage details

| Storage | Content |
|---------|---------|
| **localStorage** (`zero_device_pos_data`) | Products, customers, bills, warranties, users, settings, ID counters |
| **sessionStorage** (`zd_session`) | Current logged-in user (cleared on logout) |
| **Firestore** `warranties/{code}` | Online warranty documents for QR lookup |
| **Firestore** `_meta/connection-test` | Optional connection test document |

Clearing browser site data removes all local POS records. Always keep **Export Backup** files.

---

## Tech stack

- Single HTML file (no build step)
- [Tailwind CSS](https://tailwindcss.com/) (CDN)
- [Chart.js](https://www.chartjs.org/) (dashboard charts)
- [QRCode.js](https://github.com/davidshimjs/qrcodejs) (invoice / warranty QR)
- Font Awesome icons
- Firebase **REST API only** (no Firebase JS SDK) for Firestore

---

## Tips

- Export a backup before major changes or before reset.
- Low-stock threshold and currency are under **Settings → Billing**.
- Warranty duration (months) is under **Settings → Warranty**.
- Staff users cannot delete the protected `admin` account.
- Print invoices via the browser print dialog (layout is optimized for print).

---

## License / usage

For internal store use. Replace business name, address, phone, and description under **Settings**.  
No warranty is provided for this software; use at your own risk. Secure Firebase rules before production use.
