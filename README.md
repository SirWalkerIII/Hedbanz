# Hedbanz 2.0 — GitHub Pages + Supabase

A browser-hosted party game with private invite-code rooms, anonymous player sessions, shared custom cards, host-controlled rounds, and server-side card assignments.

## Files

- `index.html` — the website. Host this on GitHub Pages.
- `supabase.sql` — database schema and secure RPC functions. Run once in Supabase.
- `README.md` — these setup instructions.

## 1. Create the free Supabase project

1. Open https://supabase.com/ and create a free account.
2. Create a new project. Save the database password somewhere safe.
3. Wait for the project to finish provisioning.
4. Open **SQL Editor** in the Supabase dashboard.
5. Open `supabase.sql`, copy the entire file, paste it into a new SQL query, and click **Run**.
6. Go to **Authentication → Providers** (sometimes shown under Sign In / Providers).
7. Enable **Anonymous Sign-Ins**. The game uses anonymous authentication so each browser has its own player identity; players do not need to register.
8. Open **Project Settings → API** (or **Data API / API Keys**, depending on dashboard layout).
9. Copy the **Project URL** and the **anon / publishable key**. Never use or publish the `service_role` or secret key.

## 2. Publish the website with GitHub Pages

1. Create a GitHub repository, for example `hedbanz-game`.
2. Upload `index.html` into the repository root.
3. Open **Settings → Pages**.
4. Under **Build and deployment**, choose **Deploy from a branch**.
5. Choose branch `main` and folder `/ (root)`, then save.
6. Wait for GitHub Pages to publish. Your site URL will look like `https://YOUR-USERNAME.github.io/hedbanz-game/`.

## 3. Connect the site to Supabase

1. Open your published GitHub Pages URL.
2. Paste your Supabase **Project URL** and **anon / publishable key** into the connection screen.
3. Click **Connect**.
4. Create a room and invite friends.

Connection settings are saved in that browser's local storage. Each friend connects their own browser to the same Supabase project once. The public anon key is designed to be present in a client app; database access is restricted to the provided RPC functions. Do not expose the service-role key.

## 4. Playing

1. One person creates a room with their display name.
2. Share the room code or **Copy invite link** with friends.
3. Friends enter their own names and the room code.
4. Everyone can add custom words and optional direct image URLs to the shared room deck.
5. The host starts the game when at least two players have joined and there are enough cards.
6. Each player sees their own card hidden, while the other players' cards are visible.
7. When a player guesses their card, they press **Reveal my card** and confirm.
8. Only the host can advance to the next round.

Cards are randomly assigned on the server for each round. No player receives a duplicate card within a round. Cards may be reused in later rounds.

## Private-room model and limitations

- Rooms are not listed or searchable. The random invite code is the room's access credential; share it only with people you want to invite.
- Players use anonymous Supabase Auth sessions stored in their browser. Clearing browser storage or switching browsers creates a new identity. Rejoining with the same identity/name in a lobby is supported.
- The browser receives other players' card data, but the current player's card is omitted from the room-state response. The current player's word/image is returned only by the reveal RPC.
- The game refreshes room state about every 2.2 seconds. This is polling, not a WebSocket push, but it works without extra Realtime publication configuration.
- The app uses the Supabase JS v2 CDN, so the browser needs internet access.
- Direct image URLs must allow browser hotlinking and should point to an image file or an image-serving endpoint. Some sites block embedding; if an image fails, the card still displays its word.
- This is a small casual-game setup, not a hardened identity platform. Anyone with the invite code can join while the room is in its lobby. Do not use the game to store sensitive information.
- Free-tier quotas and inactivity policies can change. Check the current Supabase pricing and project status in its dashboard.

## Troubleshooting

**"Anonymous sign-ins are disabled"** — enable Anonymous Sign-Ins in Supabase Authentication providers.

**"Could not find function..."** — rerun the complete `supabase.sql` in the SQL Editor and check for SQL errors.

**"Room not found"** — confirm the invite code exactly. Room codes are case-insensitive.

**Friends cannot join after the game starts** — this build locks the lobby when the host starts. Create a new room for a new group.

**Images do not show** — use a direct HTTPS image URL from a host that permits embedding. A normal webpage URL is not necessarily an image URL.

**You see "at least one card per player"** — everyone can add cards in the lobby; add enough unique words for the number of players.

## Security note

The SQL uses `SECURITY DEFINER` functions with a fixed search path and revokes direct table access from `anon` and `authenticated`. The website must use the Supabase anon/publishable key only. Never put a database password, service-role key, or secret key in `index.html`.
