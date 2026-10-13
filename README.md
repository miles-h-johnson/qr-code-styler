# QR Styler

A web app that turns a plain QR code into AI-generated artwork that still scans. Users upload a QR code, describe a style, and get back an image generated around the code; results are saved to a personal gallery. Usage is metered by subscription tier.

## Stack

- **App:** Next.js 15 (App Router), TypeScript, Tailwind CSS, shadcn/ui
- **Generation:** an image-conditioned model on [Replicate](https://replicate.com); the uploaded QR code is sent as the conditioning image with the user's prompt
- **Auth:** NextAuth with Google sign-in
- **Data:** PostgreSQL on Neon via Drizzle ORM (`migrations/`), generated images in Vercel Blob
- **Payments:** Stripe Checkout for Basic and Premium tiers; a webhook (`app/api/webhook/stripe`) handles checkout completion, subscription updates and cancellation, and paid or failed invoices

## How a generation works

1. `POST /api/predictions` checks the user's tier and remaining monthly quota (`src/utils/subscription.ts`), then starts a Replicate prediction with the QR image, prompt and the tier's maximum resolution.
2. The client polls `GET /api/predictions/[id]` until the prediction finishes.
3. The finished image is copied to Vercel Blob and recorded against the user, so it appears in their gallery (`/api/user/images`).

## Running locally

```bash
npm install
cp .env.example .env.local   # fill in the values
npm run migrate:dev
npm run dev
```

`.env.example` lists every variable the app reads. Stripe webhooks need a public URL in development; `stripe listen --forward-to localhost:3000/api/webhook/stripe` works.

## Status

Not currently deployed. Imported from a private repository without its earlier history.
