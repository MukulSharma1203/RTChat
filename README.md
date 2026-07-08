# RTChat

Lightweight real-time chat application with a Node/Express + MongoDB backend and a React + Vite frontend.

## Features

- Real-time messaging via Socket.io
- User sync via Clerk (webhooks keep user profiles in MongoDB)
- File uploads (images & videos) stored through ImageKit
- Simple REST API for messages and user lists
- Small seeded dataset for local development

## Tech stack

- Backend: Node.js, Express, MongoDB (Mongoose), Socket.io
- Auth: Clerk (webhooks + Clerk middleware)
- Media: ImageKit
- Frontend: React + Vite, Zustand, Clerk React, Tailwind (styles)

## Repository layout

- `backend/` — Express server, API routes, webhooks, and database models
- `frontend/` — React app built with Vite

## Prerequisites

- Node.js (18+ recommended)
- npm or yarn
- A MongoDB connection URI
- Clerk account (for auth) with keys and webhook signing secret
- ImageKit account (optional, for media uploads)

## Environment variables (backend)

Create a `.env` file in `backend/` with the following values (placeholders shown):

- `PORT` — port for the backend (e.g. `3000`)
- `NODE_ENV` — `development` or `production`
- `MONGO_URI` — your MongoDB connection string
- `CLERK_PUBLISHABLE_KEY` — Clerk publishable key
- `CLERK_SECRET_KEY` — Clerk secret key
- `CLERK_WEBHOOK_SIGNING_SECRET` — Clerk webhook signing secret (required for webhook verification)
- `IMAGEKIT_PRIVATE_KEY` — ImageKit private key (optional; required to enable media uploads)
- `FRONTEND_URL` — URL of the frontend used for CORS (e.g. `http://localhost:5173`)

Do NOT commit real secrets to source control. Keep keys in an environment-specific secret store.

## Local development

1. Start the backend

```bash
cd backend
npm install
cp .env.example .env   # create and edit .env with real values
npm run dev
```

The backend exposes API routes under `/api` and serves a webhook at `/api/webhooks/clerk`.

2. Start the frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend uses Clerk for authentication and expects the backend API to be available at `http://localhost:3000/api` when in development.

## API reference (important endpoints)

- Health check: `GET /health`
- Clerk webhook (raw body required): `POST /api/webhooks/clerk`
  - This endpoint verifies incoming Clerk webhook signatures using `CLERK_WEBHOOK_SIGNING_SECRET` and upserts/deletes user profiles in MongoDB.
- Auth check: `GET /api/auth/check` (protected; Clerk middleware + `protectRoute`)

Messages (protected routes; all require a logged-in user):

- `GET /api/messages/users` — list other users for the sidebar
- `GET /api/messages/conversations` — list conversations ordered by most recent
- `GET /api/messages/:id` — fetch messages between current user and `:id`
- `POST /api/messages/send/:id` — send message to user `:id` (multipart `media` field allowed)
  - Media uploads are validated (images/videos only, max 25 MB). If `IMAGEKIT_PRIVATE_KEY` is set, files are uploaded to ImageKit and the message stores the returned URL.

Authentication and route protection are implemented with Clerk and the `protectRoute` middleware which looks up the user in the `users` collection.

## Websockets

- The backend runs Socket.io on the same server and exposes real-time events.
- The frontend connects using `socket.io-client` and sends `userId` as a query parameter when connecting.
- Events:
  - `getOnlineUsers` — server broadcasts currently online user IDs
  - `newMessage` — server emits this to a receiver's socket when a new message is created

## Seeding the database

To insert seed users locally:

```bash
cd backend
npm run db:seed
```

This will upsert a set of sample users (useful for local testing without Clerk).

## Production build & deploy notes

Typical flow to deploy a single server-hosted app:

1. Build the frontend:

```bash
cd frontend
npm run build
```

2. Copy the built `dist`/public files into the backend `public/` directory (the server serves `public/` in production).

3. Configure environment variables on the server (including Clerk keys and webhook signing secret) and start the backend:

```bash
cd backend
npm install --production
NODE_ENV=production npm start
```

Notes:
- The clerk webhook endpoint must be reachable from Clerk (use a public URL or a tunnel during development). The webhook expects the raw request body to verify the signature.
- Ensure `FRONTEND_URL` matches the deployed frontend origin for correct CORS behavior.

## Development tips

- Use the seed script to create dummy users locally if you don't want to configure Clerk for development.
- If you run into media upload issues, verify `IMAGEKIT_PRIVATE_KEY` is set and accessible to the backend process.
- Check the server logs for webhook verification failures — they usually indicate a missing or incorrect `CLERK_WEBHOOK_SIGNING_SECRET`.

## Contributing

Open an issue or submit a pull request. Keep changes focused and include tests where appropriate.

## License

This repository does not include a license file. Add one if you intend to make this project public.
# RTChat
