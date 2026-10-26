# YVONNIE2005 WTSUP — quick launch checklist

1. Create a Supabase project at https://database.new.
2. In Supabase SQL Editor, run migrations in this exact order: `0001_messaging_foundation.sql`, `0002_groups_realtime_calls.sql`, `0003_group_ai_bot.sql`.
3. Copy `.env.example` to `.env.local`; fill the Supabase URL and publishable key. For AI, add an OpenRouter API key and enabled model. For group bot replies, add the Supabase service-role secret as a server-only variable. Never prefix it with `NEXT_PUBLIC_`.
4. On a computer with Node.js 22 installed, run `npm install`, then `npm run build`. Fix any errors before publishing.
5. Push the folder to a private GitHub repository and import it at https://vercel.com/new. Add the same environment variables in Vercel settings and deploy.
6. Add the deployed domain and `/auth/callback` to Supabase Authentication → URL Configuration.
7. Test signup, direct messaging with two users, group creation/member management, `.ai your question` in a group, and calls over different networks.
8. On Android Chrome, open the HTTPS website, tap the three-dot menu, and choose **Install app** or **Add to Home screen**.

**Not done for you:** The app has not been connected to a live Supabase account or deployed because those require your accounts and secrets. A production build could not be verified in the current environment because npm dependency installation timed out. The PWA source is prepared, but it is not yet a published website. TURN credentials are recommended for reliable voice/video calling on mobile networks.
