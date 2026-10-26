# Campus Events RSVP: student starter

Finish a read feature and an RSVP write feature, then trace each request:

```text
Browser -> Next.js API route on Vercel -> Supabase database
```

Use fictional names and emails only.

## In-class activity

### 1. Create your repository and deploy

1. Create your own GitHub repository from this starter template.
2. Import your repository into Vercel with **Root Directory = `starter`**.
3. Deploy without database variables. Confirm Sample data mode, four `[Sample]` events, and a form that does not save yet.
4. Save your repository and deployed URL. Confirm you can edit/push using your existing AI coding tool and sign in to Supabase.

Follow the [deployment guide](docs/deployment-guide.md) for detailed setup. Complete this initial deployment **at the start of class**; local Node.js setup is optional.

### 2. Build the database and request flows

1. Design and create the tables in your own isolated Supabase teaching project.
2. Set `SUPABASE_URL` and `SUPABASE_SECRET_KEY` as server-side variables in Vercel, then redeploy. Test the provided `/api/events/1` route.
3. Implement GET `/api/events`, push, wait for Ready, and inspect the deployed JSON before connecting the frontend.
4. Connect the frontend read flow and deploy it.
5. Implement POST, connect the form, and remove starter notices in one grouped push; test success, invalid input, and duplicate RSVPs.

Connecting Vercel to Supabase supplies configuration only; you still complete the routes and frontend. Never prefix privileged credentials with `NEXT_PUBLIC_` or commit real environment files.

## Project files

| Path | Purpose |
|---|---|
| [starter/](starter/) | Next.js app with sample data, unfinished GET/POST routes, and marked frontend changes |
| [database/](database/) | Schema, seed data, and [Supabase setup](database/README.md) |
| [docs/student-guide.md](docs/student-guide.md) | Activity and submission instructions |
| [docs/deployment-guide.md](docs/deployment-guide.md) | Vercel setup and push checkpoints |
| [docs/ai-prompts.md](docs/ai-prompts.md) | Bounded prompts and recovery guidance |
| [docs/api-reference.md](docs/api-reference.md) | Endpoint contracts and deployed request tests |
| [docs/troubleshooting.md](docs/troubleshooting.md) | Deployment, environment, and API fixes |
| [requests/campus-events.http](requests/campus-events.http) | Example requests; set the base URL to your Vercel app |

Open `starter/` in your AI coding tool for the coding stages. The completed reference is held by your instructor; ask for a hint or stage-specific recovery file if needed.

## Optional local development

With Node.js 22 or newer:

```bash
cd starter
npm install
npm run dev
```

Open `http://localhost:3000`. For local database tests, copy `.env.example` to `.env.local` in `starter/`, fill in your teaching project's values, and restart. Vercel environment settings are separate.
