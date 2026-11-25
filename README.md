# Submission Watcher Agent

Monitors literary magazine submission windows and emails you the moment they open. Deploys as a Vercel cron job.

## What It Does

- **Watches submission pages** — Currently monitors One Story's Submittable page for magazine submission links (filtering out workshops, conferences, teen programs)
- **Sends email alerts** — One notification per submission window via Resend, so you never miss an opening
- **Tracks state** — Uses Upstash Redis to avoid duplicate notifications and expose status via API
- **Runs on schedule** — Vercel cron checks twice daily (9 AM and 5 PM UTC)

## Install

### As a Claude Code plugin

```bash
/plugin marketplace add highschoolsmokers/submission-watcher-agent
/plugin install submission-watcher-agent@submission-watcher-agent
```

The plugin includes an agent that helps you set up, configure, deploy, and add new magazine watchers.

### Manual deploy

1. Clone this repo
2. Copy `.env.example` to `.env` and fill in your keys
3. Deploy to Vercel: `vercel --prod`

## Environment Variables

| Variable | Description |
|----------|-------------|
| `UPSTASH_REDIS_REST_URL` | Upstash Redis REST endpoint |
| `UPSTASH_REDIS_REST_TOKEN` | Upstash Redis auth token |
| `RESEND_API_KEY` | Resend API key for sending email |
| `RESEND_FROM` | Sender address for notifications |
| `NOTIFY_EMAIL` | Your email address |
| `CRON_SECRET` | Secret for authenticating cron requests |

## Project Structure

```
api/
  cron.mjs                — Vercel cron handler, runs all watchers
  status.mjs              — GET /api/status — check last run + notification state
  _lib/redis.mjs          — Upstash Redis client
one-story-watcher/
  index.mjs               — Main run(): check → notify if open
  check-local.mjs         — Local test (no email, no Redis)
  lib/
    check-submissions.mjs — Fetches + parses Submittable page
    notify.mjs            — Sends notification via Resend
```

## Local Testing

```bash
node one-story-watcher/check-local.mjs
```

Checks the Submittable page and prints status without sending email or touching Redis.

## Adding a New Watcher

1. Create a directory (e.g., `example-magazine-watcher/`)
2. Implement `export async function run()` returning `{ agent, open, checkedAt }`
3. Register it in `api/cron.mjs`:
   ```js
   import { run as exampleMagazineWatcher } from "../example-magazine-watcher/index.mjs";
   const agents = [
     { name: "one-story-watcher", run: oneStoryWatcher },
     { name: "example-magazine-watcher", run: exampleMagazineWatcher },
   ];
   ```
4. Deploy

## API Endpoints

- **`GET /api/status`** — Returns last check result and notification state for all watchers
- **`POST /api/cron`** — Triggered by Vercel cron; runs all watchers (requires `CRON_SECRET` auth)

## License

MIT. See [LICENSE](LICENSE).
