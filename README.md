# CentricLabs

Personal ecommerce store for **Usman Abeeb** (Lagos, Nigeria).

Software development, IT consulting, web and mobile applications — plus a
shop for computer hardware and software, mobile phones, spares, accessories,
gadgets and general goods.

## Structure

```
centriclabs/
├── frontend/   Next.js 16 (App Router) + TypeScript + Tailwind CSS v4 — port 3000
├── backend/    Express + TypeScript + Mongoose (MongoDB, optional) — port 4000
└── README.md   this file
```

The two apps are **separate**: the frontend talks to the backend over HTTP
(`NEXT_PUBLIC_API_URL`, default `http://localhost:4000`). The frontend also
keeps an embedded sample catalogue (`frontend/src/lib/fallback.ts`) so the
shop still renders if the backend is not running.

## Design

Warm paper background `#FAF6EF`, ink black `#161513`, vivid orange `#FF5A1F`
accent and amber `#FFB627` secondary — a distinct retail look, deliberately
different from the portfolio's deep teal. The palette is defined once as
Tailwind v4 theme tokens in `frontend/src/app/globals.css`
(`--color-paper`, `--color-ink`, `--color-flame`, `--color-amber`, …) and
every component uses those token classes.

## Run it

### 1. Backend (port 4000)

```bash
cd backend
npm install
npm run dev        # tsx watch — http://localhost:4000
```

Check it: `curl http://localhost:4000/health` → `{ "ok": true, ... }`

### 2. Frontend (port 3000)

```bash
cd frontend
npm install
npm run dev        # http://localhost:3000
```

Open http://localhost:3000 — Home, Shop, product pages, Services (quote
form), Cart and Checkout.

Production builds:

```bash
cd backend  && npm run build && npm start
cd frontend && npm run build && npm start
```

## Database — MongoDB (optional)

The backend uses Mongoose (`backend/src/models.ts`: `Product`, `Service`,
`Order`, `Quote`).

- **Without MongoDB** it works anyway: if `MONGODB_URI` is not set, or
  MongoDB cannot be reached, it logs a warning and serves the in-memory
  sample data; orders/quotes are kept in memory until restart.
- **With MongoDB:** set the variable (see `backend/.env.example` —
  `MONGODB_URI=mongodb://localhost:27017/centriclabs`), start MongoDB, then:
  ```bash
  cd backend
  npm run seed   # inserts the 16 sample products + 4 services, only if empty
  ```
  Products/services are then read from MongoDB and orders/quotes are saved
  there. `/health` reports `"database": "mongodb"` or `"in-memory"`.

No real credentials are stored anywhere in this repo. Never commit a real
`.env` file.

## Swapping the sample catalogue

**One place: `backend/src/data.ts`** — the `products` array (16 sample
products in NGN across Phones, Laptops & Computers, Accessories, Spares,
Gadgets, Software, General Goods) and the `services` array (4 services).

1. Replace the products with real ones, keeping the same fields
   (`id`, `name`, `category`, `price`, `description`, `stock`, `image`;
   `oldPrice` and `badge` optional). Prices are whole Naira.
2. New `category` values automatically become new shop filters.
3. Restart the backend — no frontend change needed.
4. If you use MongoDB, re-seed (delete the products collection first, or
   edit the products in MongoDB directly).
5. Also update the frontend fallback copy in
   `frontend/src/lib/fallback.ts` so the offline fallback matches.

## Admin dashboard

- **Login:** http://localhost:3000/admin/login — demo password `admin123`
  (local development only — change with the `ADMIN_PASSWORD` environment
  variable, see `backend/.env.example`). The token is stored in
  localStorage and sent as a Bearer token on admin API calls.
- **Routes:** `/admin` (overview: products, orders, revenue, pending
  quotes, low stock), `/admin/products` (add / edit / delete, quick
  price & stock editing), `/admin/orders` (view details, update status:
  pending → confirmed → shipped → delivered / cancelled),
  `/admin/users` (customers derived from orders & quotes — no separate
  registration yet), `/admin/quotes` (mark as contacted / done).
- **Backend:** `POST /api/admin/login`, `GET /api/admin/stats`,
  `GET /api/admin/customers`, `GET /api/admin/quotes`,
  `POST/PUT/DELETE /api/products`, `PUT /api/orders/:id`,
  `PUT /api/quotes/:id`. Write/admin endpoints require the Bearer token;
  public shop endpoints stay open. Admin CRUD works against MongoDB when
  connected and against the in-memory sample data when it is not.

## Payments (placeholder)

Checkout currently records **Pay on delivery** or **bank transfer**. The
"Paystack (online)" option is shown as *coming soon* and is blocked at
checkout. Connect Paystack (or Flutterwave) in the backend and the checkout
page when ready — **no keys are stored in this repo**.

## API (backend)

| Method | Path | Description |
| --- | --- | --- |
| GET | `/health` | Status + database mode + product count |
| GET | `/api/products` | All products (`?category=` / `?search=`) |
| GET | `/api/products/:id` | One product |
| GET | `/api/categories` | Category names |
| GET | `/api/services` | The 4 services |
| POST | `/api/orders` | Create an order → returns order `id` |
| GET | `/api/orders` | Orders so far |
| POST | `/api/quotes` | Request a service quote → confirmation `id` |
| GET | `/api/quotes` | Quote requests so far |

See `backend/README.md` for request/response examples.
