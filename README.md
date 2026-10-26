# YVONNIE2005 WTSUP — installable messaging PWA

An original branded Progressive Web App (PWA) built with Next.js App Router, TypeScript, Supabase Auth/PostgreSQL/Realtime, and browser WebRTC. It is not WhatsApp and does not connect to WhatsApp accounts.

## Implemented in this source package

- Responsive branded interface for desktop and Android-sized screens.
- PWA web manifest, home-screen icons, Apple web-app metadata, and a service worker for best-effort shell caching.
- Email/password signup and login via Supabase Auth, email confirmation callback, cookie-aware SSR clients, and middleware session refresh.
- Profile directory for signed-in users, private one-to-one conversations, and realtime messages.
- Group creation with member selection, group member list, admin-controlled adding/removing members, and leave-group action.
- Server-side WTSUP AI endpoint using an OpenRouter-compatible API; the secret stays on the server. An API key is required to make it answer.
- Voice/video call signaling and media using WebRTC over private Supabase Realtime channels. Browser media permission and HTTPS are required. STUN is configured by default; a TURN provider is strongly recommended for reliability across mobile networks and carrier NAT.
- PostgreSQL Row Level Security policies for private conversations, memberships, and messages; guarded RPCs for direct chat and group membership actions; private channel authorization for call signaling.

## Still required before this can be a live service

This source is **not yet deployed** or connected to your Supabase project. No credentials are embedded. I have not been able to run a verified production build in this environment, so run the checks below before public launch. Voice/video may fail on some networks without TURN credentials. AI is unavailable until you configure an OpenRouter API key and a model available to that key.

The current PWA is not an offline-first messenger: the service worker can cache some app resources, but sending/receiving messages, authentication, AI, and calls require internet access. It does not claim end-to-end encryption; messages are protected by TLS and database access controls, not client-side E2EE. The AI assistant is available as a private panel, and the group bot is command-triggered with `.ai question` or `@wtsup question`; it is not a passive monitor and does not read every group message.

## Requirements

- Node.js 20.9 or newer (Node.js 22 LTS recommended)
- npm
- A Supabase project
- An OpenRouter key if you want AI answers
- A TURN provider if you need reliable calls through restrictive mobile networks

## 1. Create and configure Supabase

1. Create a project at <https://database.new>.
2. Open **SQL Editor** in the Supabase dashboard.
3. Run `supabase/migrations/0001_messaging_foundation.sql` first.
4. Run `supabase/migrations/0002_groups_realtime_calls.sql` second.
5. Run `supabase/migrations/0003_group_ai_bot.sql` third.
6. In Project Settings → API (or Connect), copy the Project URL and **publishable** key. Do not use a secret/service-role key in the browser.
7. Under Authentication → URL Configuration, set the Site URL to your local URL for development (`http://localhost:3000`) and add `http://localhost:3000/auth/callback` to allowed redirect URLs. Add your real deployed URL and its `/auth/callback` URL after deployment.

The group bot responds when a member sends `.ai your question` or `@wtsup your question` in a group. The response is visible to the group. It requires both an OpenRouter key and a server-only Supabase service-role key so the server can create a bot-marked message; never expose that service-role key to the browser. The SQL grants access through Row Level Security. Do not disable RLS. Test that a user who is not in a conversation cannot read its messages. Before public production use, review Realtime authorization against the current Supabase project settings.

## 2. Configure environment variables

Copy `.env.example` to `.env.local`, then fill the values:

```env
NEXT_PUBLIC_SUPABASE_URL=https://YOUR_PROJECT_REF.supabase.co
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY=sb_publishable_your_actual_key
OPENROUTER_API_KEY=your_server_side_openrouter_key
# Needed only for group-bot replies; server-side secret, never NEXT_PUBLIC_.
SUPABASE_SERVICE_ROLE_KEY=your_supabase_service_role_secret
OPENROUTER_MODEL=meta-llama/llama-3.3-70b-instruct:free
# Optional TURN values; keep credential secret on the server if you redesign this config.
NEXT_PUBLIC_TURN_URLS=
NEXT_PUBLIC_TURN_USERNAME=
NEXT_PUBLIC_TURN_CREDENTIAL=
```

Only the first two values are needed for authentication and messaging. The OpenRouter key and Supabase service-role secret are required for the group bot (and the OpenRouter key for the private AI panel). Keep the service-role secret server-side and never commit it. The OpenRouter key is optional and must never use the `NEXT_PUBLIC_` prefix. The default model can change availability; select a model that is enabled for your OpenRouter account. If using TURN, the current example exposes the TURN username/credential to the browser because WebRTC clients need them. Prefer short-lived, time-limited TURN credentials from a provider before a public launch rather than permanent credentials.

## 3. Run locally

From this project folder:

```bash
npm install
npm run build
npm run dev
```

Open `http://localhost:3000`. For signup, use an email inbox you can access. To test messaging, create two user accounts on separate browsers/devices and start a chat by searching for the other account. For calls, test on a deployed HTTPS origin or localhost, grant microphone/camera permission, and have both users open the same direct conversation.

## 4. Deploy

A straightforward route is Vercel:

1. Create a private repository on GitHub and upload this source folder.
2. Import the repository at <https://vercel.com/new>.
3. Add `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY` as Vercel environment variables. Add `OPENROUTER_API_KEY` and `OPENROUTER_MODEL` only if AI is enabled. Add TURN configuration only if you have a TURN service.
4. Deploy and wait for the build to pass.
5. In Supabase Auth URL Configuration, add the deployed URL and `https://YOUR_DEPLOYED_DOMAIN/auth/callback`.
6. Test signup, login, private messaging with two users, group creation and membership controls, AI responses, and voice/video on different mobile networks.
7. On Android Chrome, open the deployed site, tap the browser menu, then **Install app** or **Add to Home screen** (the wording depends on the browser).

## Security and operational checklist

Before opening registration to the public: configure reliable email delivery/SMTP; add rate limiting and CAPTCHA/bot protection; define account deletion, privacy policy and terms; configure database backups and monitoring; audit RLS/RPC and Realtime policies; use temporary TURN credentials; add abuse reporting and moderation; review AI provider privacy/data retention; and conduct security testing. Do not claim end-to-end encryption without implementing and auditing client-side encryption.

## Folder map

```text
app/manifest.ts                         PWA manifest
app/api/ai/route.ts                     authenticated server-side AI proxy
app/auth/callback/route.ts              Supabase email confirmation callback
app/login/page.tsx                      sign-in / sign-up
app/page.tsx                            messaging home
components/Messenger.tsx                main messaging UI
components/GroupCreator.tsx             create group and choose members
components/GroupManager.tsx             admin member management / leave group
components/AIAssistant.tsx              AI assistant UI
components/CallPanel.tsx                WebRTC voice/video UI and signaling
components/PWARegister.tsx              service-worker registration
public/sw.js                            best-effort shell cache
supabase/migrations/0001_messaging_foundation.sql
supabase/migrations/0002_groups_realtime_calls.sql
supabase/migrations/0003_group_ai_bot.sql
```
