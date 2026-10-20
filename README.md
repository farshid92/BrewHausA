# BrewHaus

A small specialty-coffee shop: React frontend, Express API, Postgres database, and email/password login.

**Live site:** https://brew-haus-a.vercel.app/

## Stack

| Part      | Tech                                           | Hosted on |
| --------- | ---------------------------------------------- | --------- |
| Frontend  | React 19, Vite, Tailwind CSS 4, React Router   | Vercel    |
| API       | Node.js, Express 5                             | Railway   |
| Database  | PostgreSQL via Drizzle ORM                     | Neon      |
| Auth      | better-auth (email + password, cookie sessions) | Railway   |

## Project structure

```
client/                 React app
  src/components/       Header, Hero, Shop, ProductCard, ProductDetail, Login, ...
  src/lib/              auth-client.js (better-auth), format.js (price formatting)
  vite.config.js        dev proxy: /api -> http://localhost:3000
  vercel.json           production rewrites: /api -> Railway, everything else -> index.html
server/                 Express API
  src/index.js          app setup, routes, 404 + error handlers
  src/auth.js           better-auth config
  src/routes/           /api/products
  src/db/               schema, connection, seed script and product data
  drizzle.config.js
  railway.json
```

## Running locally

Requires Node 20+ and a Postgres database (a free [Neon](https://neon.tech) project works).

```bash
# 1. Backend
cd server
npm install
cp .env.example .env      # then fill in DATABASE_URL and BETTER_AUTH_SECRET
npm run db:push           # create tables
npm run db:seed           # insert products
npm run dev               # http://localhost:3000

# 2. Frontend (second terminal)
cd client
npm install
npm run dev               # http://localhost:5173
```

Generate a value for `BETTER_AUTH_SECRET` with:

```bash
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

## Environment variables (`server/.env`)

| Variable             | Local value               | Production (Railway)          |
| -------------------- | ------------------------- | ----------------------------- |
| `DATABASE_URL`       | Neon pooled connection string | same                      |
| `BETTER_AUTH_SECRET` | long random string        | same                          |
| `BETTER_AUTH_URL`    | `http://localhost:3000`   | the Vercel URL                |
| `CLIENT_URL`         | `http://localhost:5173`   | the Vercel URL                |
| `PORT`               | `3000`                    | `3000`                        |

`.env` is git-ignored; only `.env.example` is committed. The frontend needs no environment variables, since it always calls relative `/api/...` paths.

## API

| Method | Path                    | Description                              |
| ------ | ----------------------- | ---------------------------------------- |
| GET    | `/api/health`           | health check                             |
| GET    | `/api/products`         | all products                             |
| GET    | `/api/products/:slug`   | one product, with its `images` gallery   |
| *      | `/api/auth/*`           | better-auth (sign up, sign in, session, sign out) |

Prices are stored as integer cents (`priceCents: 1850` = $18.50).

## Database scripts (run in `server/`)

- `npm run db:push`: apply `src/db/schema.js` to the database
- `npm run db:seed`: insert products from `src/db/products.data.js` (skips slugs that already exist)
- `npm run db:studio`: browse tables in the browser

To add a product, add it to `src/db/products.data.js` and run `npm run db:seed`.

## Deployment

- **Railway** builds `server/` and runs `npm start`. Pushes to `main` redeploy automatically.
- **Vercel** builds `client/`. `vercel.json` forwards `/api/*` to the Railway domain and serves `index.html` for all other paths so client-side routes survive a refresh.


