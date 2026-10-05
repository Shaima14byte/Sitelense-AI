# SiteLens AI — Site Analysis Tool

A professional architectural site analysis platform. Enter a project location and building program and receive a full environmental analysis: solar path, shadow study, solar heat absorption, 3D wind flow, climate data, SWOT analysis, bioclimatic strategies, and design suggestions by discipline — all driven by your actual building massing.

---

## Tech Stack

- **Framework:** Next.js 16 (App Router, TypeScript)
- **Database:** PostgreSQL via Prisma ORM
- **Auth:** NextAuth.js v5 (email + password credentials)
- **AI / LLM:** Abacus.AI API (SWOT + design suggestions via streaming)
- **Climate data:** Open-Meteo (free, no key needed)
- **Solar data:** NASA POWER (free, no key needed)
- **Maps:** Leaflet + OpenStreetMap (free, no key needed)
- **Styling:** Tailwind CSS + shadcn/ui
- **PDF export:** jsPDF (client-side, no server needed)

---

## Repository Structure

```
├── app/
│   ├── _components/          # Home wizard & dashboard shell
│   ├── analyze/              # 11-tab analysis dashboard
│   ├── api/                  # All backend API routes
│   │   ├── climate/          # Monthly climate data (Open-Meteo)
│   │   ├── solar/            # Solar radiation (NASA POWER)
│   │   ├── context/          # Site context (OpenStreetMap Overpass)
│   │   ├── elevation/        # Terrain elevation (Open-Meteo)
│   │   ├── geocode/          # Address → coordinates
│   │   ├── swot/             # AI SWOT + design suggestions (streaming)
│   │   ├── projects/         # CRUD for saved projects
│   │   ├── signup/           # User registration
│   │   └── auth/             # NextAuth handlers
│   ├── login/ signup/ projects/ new/
│   ├── globals.css
│   └── layout.tsx
├── components/
│   ├── diagrams/             # All 3D/2D study canvases
│   │   ├── massing-study.tsx # Shadow / Solar & Heat / Wind (mass-driven)
│   │   ├── massing-model.tsx # Interactive 3D block model
│   │   ├── sun-diagram.tsx   # Sun path diagram
│   │   ├── wind-diagram.tsx  # Wind rose / diagram
│   │   └── ...
│   ├── map/                  # Leaflet map + massing overlay
│   └── ui/                   # shadcn/ui component library
├── lib/
│   ├── solar-geometry.ts     # Pure solar math (altitude, azimuth, insolation)
│   ├── types.ts              # Shared TypeScript types + BuildingMass
│   ├── db.ts                 # Prisma client singleton
│   └── utils.ts
├── prisma/
│   └── schema.prisma         # Database schema (User, Project, Session…)
├── .env.example              # Required environment variables (copy → .env)
├── next.config.js
├── tailwind.config.ts
└── package.json
```

---

## Environment Variables

Copy `.env.example` to `.env` and fill in each value:

```bash
cp .env.example .env
```

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | ✅ | PostgreSQL connection string |
| `NEXTAUTH_SECRET` | ✅ | Random secret for session signing (min 32 chars) |
| `AUTH_SECRET` | ✅ | Same value as `NEXTAUTH_SECRET` (NextAuth v5 alias) |
| `NEXTAUTH_URL` | ✅ | Full public URL of your deployment (e.g. `https://yourdomain.com`) |
| `ABACUSAI_API_KEY` | ✅ | Abacus.AI API key for SWOT/design AI analysis |

> **Climate, solar, elevation, map, and geocoding data** all use free public APIs with no API key required.

### How to get each value

**DATABASE_URL** — PostgreSQL connection string in the form:
```
postgresql://USER:PASSWORD@HOST:5432/DATABASE?sslmode=require
```
Free options: [Supabase](https://supabase.com), [Neon](https://neon.tech), [Railway](https://railway.app)

**NEXTAUTH_SECRET / AUTH_SECRET** — Generate with:
```bash
openssl rand -base64 32
```
Use the same value for both variables.

**NEXTAUTH_URL** — Set to your deployment URL, e.g.:
- Local: `http://localhost:3000`
- Vercel: `https://your-project.vercel.app`
- Custom domain: `https://yourdomain.com`

**ABACUSAI_API_KEY** — Sign in at [abacus.ai](https://abacus.ai), go to **API Keys** in account settings, and create a key.

---

## Local Development

### Prerequisites
- Node.js 18+
- Yarn (`npm install -g yarn`)
- A running PostgreSQL database

### Setup

```bash
# 1. Clone your GitHub repository
git clone https://github.com/YOUR_USERNAME/sitelens-ai.git
cd sitelens-ai

# 2. Install dependencies
yarn install

# 3. Configure environment
cp .env.example .env
# Edit .env and fill in all values

# 4. Push database schema
npx prisma db push

# 5. Start development server
yarn dev
```

Open [http://localhost:3000](http://localhost:3000) — you should see the login page.
Register a new account at `/signup` to get started.

---

## Deployment Options

### Option A — Vercel (Recommended for Next.js)

Vercel is purpose-built for Next.js: zero-config deployments, auto-scaling, edge network, and free for personal projects.

1. Push this repo to GitHub
2. Go to [vercel.com](https://vercel.com) → **New Project** → Import your repo
3. In the **Environment Variables** section, add all five variables from the table above
4. Click **Deploy**

Vercel will auto-deploy on every `git push` to `main`.

**Database:** Use [Neon](https://neon.tech) (free serverless PostgreSQL) and paste the connection string as `DATABASE_URL`.

After first deploy, run the database migration once:
```bash
npx prisma db push
```

---

### Option B — Railway (Full-stack with built-in PostgreSQL)

Railway provides a managed PostgreSQL database alongside the app in one project.

1. Go to [railway.app](https://railway.app) → **New Project**
2. Add a **PostgreSQL** service (Railway provisions it automatically)
3. Add a **GitHub Repo** service pointing to your repo
4. In the GitHub service settings, add environment variables:
   - Copy `DATABASE_URL` from the Railway PostgreSQL service panel
   - Add `NEXTAUTH_SECRET`, `AUTH_SECRET`, `NEXTAUTH_URL`, `ABACUSAI_API_KEY`
5. In **Settings → Deploy**, set the start command: `yarn start`
6. In **Settings → Build**, set the build command: `yarn build`

Railway auto-deploys on push and provides a `*.up.railway.app` URL.

---

### Option C — Hugging Face Spaces (Docker)

HF Spaces supports Next.js via a Docker container. Note: the filesystem is ephemeral — your database **must** be external (Supabase, Neon, Railway PostgreSQL).

1. Create a new Space at [huggingface.co/spaces](https://huggingface.co/spaces)
   - SDK: **Docker**
   - Visibility: Public or Private

2. Create a `Dockerfile` in the root of your repo:

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package.json yarn.lock .yarnrc.yml ./
RUN yarn install --frozen-lockfile
COPY . .
RUN yarn build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production
COPY --from=builder /app/.next ./.next
COPY --from=builder /app/public ./public
COPY --from=builder /app/package.json ./package.json
COPY --from=builder /app/node_modules ./node_modules
EXPOSE 7860
ENV PORT=7860
CMD ["yarn", "start", "-p", "7860"]
```

3. In your Space settings → **Repository secrets**, add all five environment variables.

4. Push the repo (including the Dockerfile) to your HF Space:
```bash
git remote add hf https://huggingface.co/spaces/YOUR_HF_USERNAME/sitelens-ai
git push hf main
```

5. After the build, run the database migration from your local machine:
```bash
DATABASE_URL="your_external_db_url" npx prisma db push
```

> **Important:** HF Spaces free tier sleeps after inactivity (cold start ~30s). For always-on hosting, upgrade to a paid Space or use Vercel/Railway.

---

## Database Migrations

After any schema change to `prisma/schema.prisma`, run:

```bash
# Development — push schema directly
npx prisma db push

# Production — generate and apply a migration
npx prisma migrate deploy
```

---

## Connecting Additional APIs and Storage

The app is designed to be extended. All external data calls go through API routes in `app/api/`.

### Add cloud storage (e.g. for user-uploaded files)

Install the AWS SDK or Supabase Storage client and add the credentials to `.env`:
```
S3_BUCKET=your-bucket
S3_REGION=us-east-1
S3_ACCESS_KEY=...
S3_SECRET_KEY=...
```
Then create an API route in `app/api/upload/route.ts`.

### Swap the AI provider

The SWOT and design-suggestions AI calls live in `app/api/swot/route.ts`. It currently uses the Abacus.AI API. To swap to OpenAI, Anthropic, or any other provider, replace the fetch call in that file and update the `ABACUSAI_API_KEY` variable name accordingly.

### Add Google Maps

Replace `components/map/leaflet-map.tsx` with a Google Maps React wrapper and add:
```
NEXT_PUBLIC_GOOGLE_MAPS_API_KEY=...
```

---

## Key Design Decisions

- **Physics frame vs render frame:** Solar, shadow, and wind calculations use absolute geographic coordinates (east = +x, north = +y). The 3D view rotates for aesthetics only — the physics is always geographically correct.
- **Massing as the source of truth:** All environmental studies (shadow, solar heat absorption, wind flow) read from the `BuildingMass[]` array defined in the project wizard — not generic floor/area parameters.
- **No demo data:** All climate, solar, and context data is fetched live from public APIs at analysis time.
- **Client-side PDF:** The report is generated entirely in the browser via jsPDF — no server processing or storage needed.

---

## License

MIT — free to use, modify, and deploy.

---

*Built with [Next.js](https://nextjs.org), [Prisma](https://prisma.io), [Tailwind CSS](https://tailwindcss.com), [shadcn/ui](https://ui.shadcn.com), [Open-Meteo](https://open-meteo.com), [NASA POWER](https://power.larc.nasa.gov), and [Abacus.AI](https://abacus.ai).*
