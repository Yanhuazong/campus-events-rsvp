# Deploy the shared starter to Vercel

Start the activity by completing the initial deployment **in class**. You can use your existing AI coding tool to edit the repository and push changes to GitHub; a local Node.js installation and local database configuration are optional.

The request path stays:

```text
Browser → Next.js API route on Vercel → Supabase database
```

Connecting Vercel to Supabase supplies database configuration. It does **not** create the exercise tables, finish the GET/POST handlers, or connect the frontend. Those are the in-class activities.

## Step 1: create your repository and deploy in class

1. Create your own GitHub repository from the instructor's shared starter (use **Use this template** if offered, or copy the shared project into your own repository). Keep the student repository structure: `starter/`, `database/`, `docs/`, and `requests/`. Commit and push the project files, including the lockfiles; exclude `node_modules`, `.next`, and real `.env` files.
2. Sign in to [Vercel](https://vercel.com), add a new project, and import **your** GitHub repository. Grant Vercel access to that repository if prompted.
3. Select **Next.js** and set **Root Directory** to `starter`. This is the folder containing the student app's `package.json`; the repository root is not the student app. Keep the standard install/build settings (`npm install` / `npm run build`) and use a Node.js version supported by Vercel that meets `>=22` in `package.json`.
4. Deploy **without database variables**. Wait until the deployment is Ready, then open its URL. Confirm the yellow **Sample data mode** banner, four `[Sample]` events, and the form notice that it does not save yet. This is the expected starting state.
5. Save your GitHub repository link and Vercel production URL. Confirm your AI coding tool can edit this repository and push to its production branch (usually `main`; check Vercel's project settings).
6. Confirm you can sign in to Supabase. You will design and create tables in your **own isolated teaching project** during class. Do not use a project containing real data.

## Step 2: connect your teaching database

After designing the two tables and running `database/schema.sql` and `database/seed.sql`, configure the deployed server:

1. In Vercel, open your project → **Settings → Environment Variables**. You can connect the Supabase integration to your teaching project to supply configuration, or add the values manually from the Supabase dashboard.
2. Verify these **exact names**, which `lib/supabase-server.js` reads:

   | Variable | Value |
   |---|---|
   | `SUPABASE_URL` | Your teaching project's URL (`https://<project-ref>.supabase.co`) |
   | `SUPABASE_SECRET_KEY` | That project's secret key (`sb_secret_...`), or legacy `service_role` key |

   If the integration supplies different names, add these names explicitly. A database connection string or publishable/anon key does not replace the secret key this exercise needs. Do not add privileged credentials with a `NEXT_PUBLIC_` prefix.
3. Apply the variables to **Production**. Also configure **Preview** if you will test branch deployments, using the intended teaching project. Local Development variables alone do not configure deployed servers.
4. Save, then **redeploy** from Vercel's Deployments view. Environment changes apply to new deployments; refreshing an older deployment does not load them.
5. Wait for Ready and open `<your-vercel-url>/api/events/1`. The provided endpoint should return `200` with `{ "data": { ... } }`. This verifies the deployed server can reach your database. The event list and RSVP routes still return `501` until you implement them, and the frontend still shows sample data.

Keep credentials in Vercel's server-side environment settings. Do not put them in source files, GitHub, AI prompts, or screenshots. See [database setup](../database/README.md#3-connect-the-app-server-side-only) for the Supabase values.

## Push and verify at clear checkpoints

Use your existing AI coding tool to make and inspect the bounded changes in [ai-prompts.md](ai-prompts.md), then commit and push to the GitHub branch Vercel deploys. Each push triggers a deployment. Wait for **Ready** and check that the deployment shows your latest commit before testing. Use the current production URL consistently; a previous deployment's unique URL continues to show its older code/configuration.

| Checkpoint | Group these changes | Verify on Vercel |
|---|---|---|
| 0: classroom setup | Unmodified starter | Sample data mode loads |
| 1: database connection | Tables/seed in Supabase; environment variables in Vercel; redeploy | `/api/events/1` returns one event |
| 2: GET endpoint | Stage 2a only | Open `/api/events` directly; inspect `200` and JSON **before connecting the frontend** |
| 3: frontend read | Stage 2b | Reload the app; DevTools shows `GET /api/events`; events lose `[Sample]` |
| 4: POST flow | Stages 3a, 3b, and 3c together | Form returns `201`, row appears in Supabase, duplicate returns `409`; test invalid input directly too |

Deployment delay is the tradeoff. Inspect code, predict responses, or sketch the request journey while a deployment builds. Group related edits within each checkpoint rather than pushing every small edit. The optional details extension can wait until the core GET and POST flows work.

For POST testing, open DevTools on **your deployed app** and use the relative `fetch()` examples in [api-reference.md](api-reference.md). Set `@baseUrl` in `requests/campus-events.http` to your deployment URL if using a REST client.

## Optional local development

If your coding tool uses a local checkout, you may run `cd starter`, `npm install`, and `npm run dev` with Node.js 22+. Open `http://localhost:3000`. For local database tests only, copy `.env.example` to `.env.local`, fill in the two server variables, and restart the dev server. This file is ignored by Git and is **not** how Vercel receives configuration. Local setup is optional throughout this activity.

## Official references

- [Import and deploy a Git repository](https://vercel.com/docs/git)
- [Vercel environment variables](https://vercel.com/docs/environment-variables)
- [Supabase Vercel integration](https://supabase.com/docs/guides/integrations/vercel-marketplace)
