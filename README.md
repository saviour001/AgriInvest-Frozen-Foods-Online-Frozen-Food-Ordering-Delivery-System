# AgriInvest Frozen Foods — Online Frozen Food Ordering & Delivery System
### A Case Study of AgriInvest Ltd — v2 (Production-Architecture Edition)

## 1. Overview
A full-stack Flask web application for ordering and delivering frozen food
online. v2 is a ground-up rebuild of the original prototype into a properly
layered, secure, scalable codebase — structured the way a real production
Flask service is structured, not a single-file demo.

It satisfies the three stated project objectives, plus a security/scalability
pass on top:

1. **Web-based ordering platform** — storefront, search, category filtering,
   pagination, product detail pages, basket and checkout.
2. **Product catalog management module** — the shop owner (admin) account can
   log in, list new frozen food items with photos, and remove them; products
   are soft-deleted so historical orders never break.
3. **Secure online payment gateway** — Paystack inline checkout (card, bank
   transfer, USSD), with a documented path to server-side verification.

## 2. What changed in v2
- **Restructured into an application factory + Blueprints** (`auth`, `shop`,
  `admin`) instead of one flat `app.py` — the standard pattern for a Flask
  app meant to grow and be tested.
- **Real authentication & accounts for both sides**: customers register/log
  in for themselves; the shop owner has a separate `admin` role on the same
  `User` model, gated by an `admin_required` decorator (HTTP 403, not just a
  hidden link, if a customer tries to reach `/admin`).
- **Server-side form validation with Flask-WTF/WTForms** — every form
  (register, login, checkout, add-product) validates on the server, not just
  HTML5 `required`, with inline field errors.
- **CSRF protection** (Flask-WTF `CSRFProtect`) on every state-changing
  request, including the basket and the Paystack JS verification callback.
- **Rate limiting** (Flask-Limiter) on login, to blunt brute-force attempts.
- **Password policy**: minimum 8 characters, at least one letter and one
  number, hashed with Werkzeug's salted hashing — never stored in plain text.
- **Product photo uploads** — the shop owner can attach a real photo per
  product (validated extension + 4MB limit, stored under a generated UUID
  filename via `secure_filename`, never the user-supplied name directly).
- **Soft-delete for products** — "delete" deactivates a product instead of
  hard-removing it, so past orders that reference it stay intact; this is
  also why product deletion is a CSRF-protected POST, not a bare link.
- **Pagination** on the public catalog and the admin product/order tables.
- **Config via environment variables** (`Config` / `DevelopmentConfig` /
  `ProductionConfig` / `TestingConfig`), with a `ProductionConfig.validate()`
  guard that refuses to boot with the default `SECRET_KEY`.
- **Database migrations** wired up via Flask-Migrate/Alembic, so schema
  changes are tracked instead of hand-edited.
- **Security headers** (`X-Content-Type-Options`, `X-Frame-Options`,
  `Referrer-Policy`) and secure session-cookie flags (`HttpOnly`, `SameSite`,
  `Secure` in production).
- **Custom branded logo** — a hand-drawn SVG mark (hexagon = crate/packaging,
  snowflake spokes = frozen, orange droplet = fresh produce) replacing the
  placeholder emoji icon, used in the nav, footer and favicon.
- **Proper error pages** (404 / 403 / 429 / 500) instead of Flask's default
  debug traceback or blank pages.
- **Rotating file logging** in production mode.

## 2b. What changed in v3 (design & navigation pass)
- **Role-aware navigation**: guests see Home / Shop / How it works / Contact.
  Once signed in, customers see only Shop / My Orders / Contact — no Home or
  How-it-works clutter. Admins see Dashboard / Products / Orders only.
  Logout is now a clearly labelled button (icon + text), not an ambiguous icon.
- **Dedicated `/contact` page** — a clean, professional info layout (address,
  phone, email) separate from the homepage's footer-style contact block.
- **New colour palette**: deep teal + warm copper on white, replacing the old
  near-black navy. Every colour is a CSS variable in `style.css`, so the
  whole site re-themes from one place.
- **New typography**: Sora (headings) + Inter (body) + JetBrains Mono (data/
  prices) — a cleaner, more professional pairing than the original serif.
- **Real food photography — fully offline**: every category (and any product
  without an admin-uploaded photo) now shows a styled, locally-bundled food
  image generated at build time and stored in `app/static/img/`. No internet
  connection is required for images to load — the app works 100% offline.
  Admin-uploaded photos still take priority when present.
- **Emoji clean-up**: nearly all emoji glyphs across the site (nav, trust
  badges, empty states, error pages, admin sidebar) were replaced with a
  small hand-built set of line-style SVG icons (`partials/icons.html`) for a
  more professional look.

## 3. Tech Stack
- **Backend:** Python 3, Flask (application factory + Blueprints),
  Flask-SQLAlchemy, Flask-Login, Flask-WTF, Flask-Migrate, Flask-Limiter
- **Database:** SQLite by default (swap `DATABASE_URL` for Postgres/MySQL in
  production — the ORM layer doesn't change)
- **Frontend:** Server-rendered Jinja2 templates, hand-written CSS design
  system, vanilla JavaScript
- **Payments:** Paystack Inline JS + server-side order/payment state machine

## 4. Project Structure
```
agriinvest/
├── run.py                  # dev entry point — creates tables, seeds, runs
├── wsgi.py                 # production entry point (gunicorn wsgi:application)
├── requirements.txt
├── .env.example             # copy to .env and fill in real secrets
├── app/
│   ├── __init__.py          # application factory, error handlers, logging
│   ├── config.py            # Dev/Prod/Testing config classes
│   ├── extensions.py        # db, login_manager, csrf, limiter, migrate
│   ├── models.py            # User, Category, Product, Order, OrderItem
│   ├── forms.py             # WTForms: Register, Login, Checkout, Product
│   ├── utils.py             # cart helpers, secure image upload, order codes
│   ├── seed.py               # starter categories/products/demo accounts
│   ├── auth/                # /register /login /logout
│   ├── shop/                 # storefront, basket, checkout, payment
│   ├── admin/                 # shop-owner dashboard, catalog, orders
│   ├── static/{css,js,uploads}/
│   └── templates/            # all Jinja2 templates, including errors/
└── migrations/                # Flask-Migrate/Alembic migration history
```

## 5. Running the Project
```bash
cd agriinvest
python -m venv .venv && source .venv/bin/activate   # optional but recommended
pip install -r requirements.txt
cp .env.example .env        # edit SECRET_KEY / Paystack keys as needed
python run.py
```
Open **http://127.0.0.1:5000**. The database is created and seeded
automatically on first run (5 categories, 20 products, 1 shop-owner/admin
account, 1 demo customer account).

To manage schema changes going forward:
```bash
flask db init        # once, to create the migrations/ folder
flask db migrate -m "describe the change"
flask db upgrade
```

## 6. Demo Accounts
| Role               | Email                  | Password     |
|--------------------|-------------------------|--------------|
| Customer            | customer@example.com    | Customer123  |
| Shop owner (admin)  | admin@agriinvest.ng     | Admin12345   |

## 7. Payment Gateway Setup
1. Create a free account at https://paystack.com and copy your **public key**.
2. Put it in `.env` as `PAYSTACK_PUBLIC_KEY` (and `PAYSTACK_SECRET_KEY` for
   server-side verification).
3. **Before going live:** the `/payment/verify/<order_id>` route currently
   trusts the Paystack JS callback reference for demo purposes. For
   production, call Paystack's `GET /transaction/verify/{reference}`
   server-side with your secret key and confirm `status == "success"` and
   the returned amount matches `order.total` before marking the order paid.
   This is flagged with a `NOTE` comment directly in `app/shop/routes.py`.

No card details ever touch the AgriInvest server — Paystack's hosted iframe
handles all sensitive payment data.

## 8. Demonstrating to Your Supervisor
1. **Homepage** — the cold-chain themed hero, custom logo, category grid,
   featured products.
2. **Register a new customer account** — show server-side validation reject
   a weak password, then succeed with a strong one.
3. **Shop → filter/search → product detail → add to basket.**
4. **Checkout → Paystack test payment** (test card `4084 0840 8408 4081`,
   any future expiry, CVV `408`, PIN `0000`, OTP `123456`).
5. **My Orders** — order tracker.
6. **Shop-owner login** — dashboard (revenue, low stock), list a new product
   with a photo, delete a product, update an order's delivery status.
7. **Try visiting `/admin` while logged in as the customer** — show the 403
   page, to demonstrate the access control isn't just a hidden link.

## 9. Notes for the Project Report
- **ERD candidates (Chapter 4):** `User` (role: customer/admin), `Category`,
  `Product`, `Order`, `OrderItem`.
- **Architecture:** application factory + Blueprints (`auth`, `shop`,
  `admin`) — separates concerns the way a real Flask service would scale to
  a team, and makes the app independently testable per module.
- **Security measures implemented:** password hashing, CSRF protection,
  rate-limited login, server-side validation, secure file upload handling,
  role-based access control, secure session cookies, security response
  headers — useful talking points for a "Security Considerations" section.
- **Scalability measures implemented:** pagination, indexed/foreign-keyed
  schema, swappable database backend via `DATABASE_URL`, stateless
  session-based cart (works behind a load balancer once the secret key and
  session store are shared), migration tooling for schema evolution.
