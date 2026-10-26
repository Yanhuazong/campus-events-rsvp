# Student guide: tracing a full-stack feature

In this session you will finish two features in a small campus events app and **trace each request** from the browser, through our API, to the database, and back.

By the end you should be able to explain, for one request, **which code runs in the browser, which runs on the server, and what the database does**.

> 🔒 Use **fictional names and emails only**, like "Alex Example" and `alex@example.com`. Your screenshots will be shared.

---

## Map of the files you'll touch

```
starter/
├── components/EventsApp.js       BROWSER: fetch() calls (STUDENT CHANGE 1, 2, 3)
├── app/api/events/route.js       SERVER: GET /api/events       (TODOs, returns 501)
├── app/api/events/[id]/route.js  SERVER: GET /api/events/[id]  (finished example)
├── app/api/rsvps/route.js        SERVER: POST /api/rsvps       (TODOs, returns 501)
├── lib/supabase-server.js        SERVER ONLY: database connection
└── .env.example                  variable names; set real values in Vercel
```

Rule of thumb: **`components/` runs in the browser. `app/api/` and `lib/` run on the server.** Only the server talks to Supabase.

---

## During class

Complete setup and the feature work in class. **Browser → Next.js API route on Vercel → Supabase database** remains our architecture. Connecting Vercel to Supabase supplies configuration; you still complete the routes and frontend.

Use your existing AI coding tool for changes, inspect the diff, then push to GitHub at the checkpoints below. Deployment delay is the main tradeoff: predict responses or trace the code while builds run. Check the latest commit is Ready and test the current production URL, not an older deployment URL. See the [checkpoint table](deployment-guide.md#push-and-verify-at-clear-checkpoints).

### 0–15 min: Create your repository and deploy (checkpoint 0)

Start setup together in class. Local Node.js installation and local secrets are optional.

1. Create your own GitHub repository from the shared starter and push the project files. Keep the full folder structure.
2. Import your repository into Vercel, choose Next.js, and set **Root Directory = `starter`**. Deploy without database variables.
3. Wait for Ready, then open your deployed URL. Confirm the yellow **Sample data mode** banner, four `[Sample]` events, and an RSVP form that says it does not save yet.
4. Save your repository link and Vercel production URL. Confirm your existing AI coding tool can edit the repository and push changes to the branch Vercel deploys.
5. Confirm you can sign in to Supabase and create your own isolated teaching project. Do not use real data.

Follow [deployment-guide.md](deployment-guide.md) for detailed instructions. If setup fails, ask your instructor or pair with a classmate; a hosted demonstration is also available.

### 15–20 min: Watch and predict

The instructor submits an RSVP in the finished app and shows the Network request and the new database row.

✏️ For each item, write **B** (browser), **S** (Next.js server), or **D** (database):

| | B / S / D |
|---|---|
| The form fields and the "Send RSVP" button | |
| `fetch("/api/rsvps", { method: "POST", ... })` | |
| Checking that the email looks valid (the trustworthy check) | |
| The secret Supabase key | |
| Refusing a second RSVP from the same email | |
| Showing "RSVP saved. See you there!" | |

### 20–27 min: Design the data

The app must let people browse events, view one, and RSVP with a name and email, with no duplicate RSVPs.

✏️ Before you look at `database/schema.sql`, sketch two tables. What columns does each need? How does an RSVP know which event it's for? What should stop duplicate RSVPs?

Then compare your sketch with the shared schema in [database/README.md](../database/README.md#4-the-design).

### 27–40 min: Set up the database and Vercel connection (checkpoint 1)

1. In your teaching project, open **SQL Editor**. Run all of `database/schema.sql`, then all of `database/seed.sql`.
2. Open **Table Editor** and look at both tables. Find:
   - a **primary key**
   - the **foreign key** (`rsvps.event_id`)
   - the **unique constraint** on `(event_id, email)` (it's near the bottom of `schema.sql`)
3. In Vercel → your project → **Settings → Environment Variables**, connect your teaching Supabase project or manually add **`SUPABASE_URL`** and **`SUPABASE_SECRET_KEY`**. Verify these exact names even if using the integration. Use the secret or legacy `service_role` key, never a privileged `NEXT_PUBLIC_` variable. See [database setup](../database/README.md#3-connect-the-app-server-side-only).
4. Apply the variables to **Production** (and Preview if testing branch deployments), save, and **redeploy**. Wait for Ready; refreshing an older deployment is not enough.
5. Open `<your-vercel-url>/api/events/1`. It should return `200` with `{"data": {...}}`. This provided route verifies the connection. The unfinished GET list and POST routes still return `501`.


🔒 Never paste your secret key into chat or show it in a screenshot.

### 40–60 min: The read flow (GET)

1. **Predict.** Open `app/api/events/route.js` and read the TODOs. Answer the five "pause and ask" questions below.
2. **Generate.** Use the **Stage 2a** prompt from [ai-prompts.md](ai-prompts.md).
3. **Inspect.** Compare the AI's code with `app/api/events/[id]/route.js`. Does it use `getSupabase()`? Does it hide database errors?
4. **Deploy and test (checkpoint 2).** Commit and push Stage 2a to GitHub. Wait for Vercel to show Ready for that commit, then open `<your-vercel-url>/api/events`. Inspect the JSON: you should get 200 with four events. Do this before connecting the frontend.
5. **Connect the frontend (checkpoint 3).** Use the **Stage 2b** prompt (STUDENT CHANGE 1 in `components/EventsApp.js`). Inspect, commit, and push; wait for the new deployment to be Ready before reloading your deployed app.
6. **Inspect in DevTools.** Reload the page, press F12, open **Network**, and filter **Fetch/XHR**. Click the `events` request. Look at the **Headers** tab (Request URL, Request Method, Status Code) and the **Response** tab. 📸 *Take screenshot 1.*
7. **Trace the finished endpoint.** Read `app/api/events/[id]/route.js`. Why does `/api/events/abc` give 400 but `/api/events/9999` give 404?

⭐ *Extension:* use the **Extension** prompt to load details with `GET /api/events/[id]`.

### 60–78 min: The write flow (POST)

1. **Predict.** Read the TODOs in `app/api/rsvps/route.js` and answer the five questions.
2. **Generate.** Use the **Stage 3a** prompt.
3. **Inspect.** Find each server-side check: JSON, eventId, name, email, length, format, event exists, duplicate.
4. **Connect the form.** Use the **Stage 3b** prompt (STUDENT CHANGE 2), then **3c** to remove the sample-mode notices.
5. **Deploy the POST flow (checkpoint 4).** Group Stages 3a, 3b, and 3c into one commit/push. Wait for Ready for that commit. **Submit fictional data** in your deployed form. In DevTools, click the `rsvps` request:
   - **Payload** tab: the JSON you sent
   - **Headers** tab: method `POST`, status `201`
   - **Response** tab: `{ "data": { "id": ..., "eventId": ... }, "message": ... }`
6. In Supabase **Table Editor → rsvps**, find your new row. 📸 *Take screenshot 2 (DevTools only; don't show any keys).*

### 78–85 min: Test success and failure

Use `requests/campus-events.http` or the browser console snippet in [api-reference.md](api-reference.md#3-browser-devtools-console-works-everywhere). Fill in the table:

| Test | What you sent | Expected | Actual status | Error code (if any) |
|---|---|---|---|---|
| Valid RSVP | event 2, new fictional email | 201 | | |
| Missing email (sent directly to the API) | no `email` field | 400 | | |
| Duplicate RSVP | same event + same email again | 409 | | |
| Unknown event | `eventId: 9999` | 404 | | |

Think about this: the form already has `required` and `type="email"`. Why did the server still need to check? (Hint: how did you just send a request without an email?)

### 85–90 min: Explain and transfer

1. Explain one complete request journey to a partner: **frontend → API route → database → response → screen**. Name the file at each step.
2. Look at your own prototype. Where does it do each job: UI, input validation, database access, and keeping keys secret?

   Your prototype might call Supabase directly from components, or use Server Actions, so its files may look different. The **responsibilities** are the same even when the file structure isn't.

---

## Pause and ask (before running any generated code)

- **Which endpoint** will receive this?
- **What method** does it use?
- **What data** will cross the network?
- **What code checks** the input?
- **What happens if the database operation fails?**

---

## What to submit

Keep it short. Use fictional data in everything. Include your GitHub repository link and working Vercel URL with the five items below.

1. **Screenshot 1:** one GET request in DevTools showing method, URL, status, and response.
2. **Screenshot 2:** one POST request and its result (status and response; the Supabase row is optional). No keys visible.
3. **Test-results table:** the table from 78–85 min, with at least one success and one failure.
4. **Trace (4–6 sentences):** follow one request from frontend → API route → database → response. Name the files.
5. **Transfer (1 sentence):** where are these same responsibilities in your own prototype?

### Template

```markdown
## Screenshots
1. GET: (image)
2. POST: (image)

## Test results
| Test | Expected | Actual | Code |
|---|---|---|---|
| Valid RSVP | 201 | | |
| Missing email | 400 | | |

## Trace
When I ..., the browser code in components/EventsApp.js sends ...

## My prototype
In my prototype, ...
```

Stuck? See [troubleshooting.md](troubleshooting.md), or use the recovery path at the bottom of [ai-prompts.md](ai-prompts.md#recovery-path-when-ai-output-differs-or-you-get-stuck).
