# Arcana API (Cloudflare Workers + D1)

## Required setup before production
1. Create a Cloudflare D1 database named `arcana-db`; copy its database ID into `wrangler.toml`.
2. Apply schema: `npx wrangler d1 execute arcana-db --remote --file=./migrations/0001_init.sql`.
3. Configure the exact site origin in `ALLOWED_ORIGIN` and your admin email in `ADMIN_EMAIL`.
4. Add the Gemini key as a Worker secret: `npx wrangler secret put GEMINI_API_KEY`.
5. Deploy the Worker and attach the `/api/*` route to the same site domain (or configure a same-origin reverse proxy). Keep the API same-origin so the HttpOnly cookie works safely.
6. After registering your admin account, promote it once with a controlled D1 command:
   `UPDATE users SET role='admin' WHERE email='YOUR-ADMIN-EMAIL';`
7. Remove placeholders before production. Never put API keys in frontend code.

## API
- `GET /api/health`
- `POST /api/auth/register`, `POST /api/auth/login`, `POST /api/auth/logout`
- `GET /api/me`
- `GET/POST /api/readings`, `DELETE /api/readings/:id`
- `GET/POST /api/favorites`, `DELETE /api/favorites/:card_id`
- `POST /api/interpret` (Gemini secret required; authenticated and rate-limited)
- `GET /api/admin/stats` (admin only)

Sessions use random opaque tokens stored only as SHA-256 hashes in D1 and HttpOnly/Secure/SameSite cookies. Passwords use PBKDF2-SHA-256. Configure HTTPS and a strict origin. Add Turnstile, email verification, password reset and a scheduled cleanup job before a public high-traffic launch.

The API is not live until you create D1, set secrets, configure the route, and deploy. No credentials are committed here.
