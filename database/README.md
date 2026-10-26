# Database setup and design

## 1. Use an isolated teaching project

Create a **new, separate Supabase project** just for this class. Do not use a project that holds real data or another prototype.

1. Go to https://supabase.com/dashboard and sign in.
2. Select **New project**. Name it something like `campus-events-teaching` and pick any region near you. Save the database password somewhere safe, although this exercise does not use it.
3. Wait until the project finishes provisioning, which takes about 1–2 minutes.

Why a separate project? Our setup script **drops and recreates tables**, and the seed script **deletes all rows**. Students will also be sharing screenshots of their requests. An isolated project means none of that can damage real work.

Free-plan projects are **paused after about a week without activity**. If yours is paused, open it in the dashboard and choose **Restore** before class.

## 2. Create the tables

1. In your project, open **SQL Editor** → **New query**.
2. Paste the whole of [schema.sql](schema.sql) and select **Run**. You should see "Success. No rows returned".
3. Open a new query, paste [seed.sql](seed.sql), and select **Run**. You should see four events with dates in the coming weeks.
4. Open **Table Editor** and look at `events` and `rsvps`. `rsvps` starts empty.

If the dashboard warns that the script contains destructive operations (the `drop table` and `truncate`), confirm. That warning is expected in a teaching project.

**Refreshing dates later:** the seed dates are 7, 14, 21, and 28 days from the day you run it. If you reuse the project months later and the events have passed, just run `seed.sql` again. (That also clears test RSVPs.)

## 3. Connect the app (server-side only)

1. In Supabase, open **Connect** or **Project Settings → Data API** and copy your teaching project's **Project URL**.
2. Under **Project Settings → API Keys**, copy a **Secret key** (`sb_secret_...`). For older projects use the legacy **`service_role`** key.
3. In your Vercel project → **Settings → Environment Variables**, add:

   ```text
   SUPABASE_URL=https://abcdefghijkl.supabase.co
   SUPABASE_SECRET_KEY=sb_secret_...
   ```

   You can connect the Supabase integration instead, but verify it supplies these exact names. This app does not read a connection string or browser publishable key. Apply the variables to **Production**, plus **Preview** if you test branch deployments.
4. Save, **redeploy**, and wait for Ready. Open `<your-vercel-url>/api/events/1` to check the provided route returns an event. Older deployments keep their previous environment settings.

Connecting Vercel to Supabase supplies configuration only. You still need the tables above, the unfinished GET and POST handlers, and frontend `fetch()` calls. The path is **Browser → Next.js API route on Vercel → Supabase database**. See [the deployment guide](../docs/deployment-guide.md).

Never put privileged credentials in `NEXT_PUBLIC_` variables, browser code, GitHub, AI prompts, or screenshots. If a key leaks, revoke/rotate it in Supabase, update Vercel, and redeploy. Do **not** use the publishable/anon key: this schema blocks it deliberately.

**Optional local tests:** copy `.env.example` to `.env.local` inside `starter/`, fill in the same two variables, and restart `npm run dev`. The file is git-ignored; Vercel configuration is separate.


## 4. The design

```
events                               rsvps
──────────────────────────           ──────────────────────────────
id          integer  PK  ◄─────┐     id          integer  PK
title       text                └──── event_id    integer  FK → events.id
description text                      name        text
location    text                      email       text
starts_at   timestamptz                created_at  timestamptz
created_at  timestamptz                UNIQUE (event_id, email)
```

One event has many RSVPs, and each RSVP belongs to exactly one event.

### Primary keys (`id`)

Each row gets a number that the database generates (`generated always as identity`). Titles and names can repeat or change, but the `id` never does, so other tables and URLs (like `/api/events/2`) can point at a row reliably.

Note that IDs can have gaps. A failed insert, such as a duplicate RSVP, still uses up a number. That's normal.

### Foreign key (`rsvps.event_id → events.id`)

An RSVP is meaningless without knowing which event it's for, so every RSVP stores the event's `id`. The foreign key makes the database **refuse** an RSVP whose `event_id` doesn't exist (Postgres error `23503`). `on delete cascade` means that deleting an event deletes its RSVPs too, so none are left orphaned.

Our API already checks that the event exists and returns a friendly 404. The foreign key is the backstop. It holds even if someone writes buggy code, or the event is deleted between the check and the insert.

### Unique constraint (`unique (event_id, email)`)

The same email may RSVP to **many events**, but only **once per event**. The database rejects a second row with the same pair (Postgres error `23505`), and the API translates that into `409 Conflict`.

This only works if `Alex@Example.com ` and `alex@example.com` are stored the same way, which is why the API trims and lowercases emails before saving. A `check` constraint in the database also rejects any email that isn't already lowercase and trimmed.

Why not just check for duplicates in JavaScript? Two requests can arrive at almost the same moment, both check "no RSVP yet", and both insert. A database constraint can't be raced like that.

### Check constraints

Length limits on text columns (for example, `name` is 1–100 characters) and the email-shape check repeat the API's rules as a backstop.

## 5. Permissions and Row Level Security

**Access approach:** the app reaches the database **only from the Next.js server**, using the secret key. Supabase maps that key to the `service_role` database role. Browsers have no key at all.

What `schema.sql` does:

| Statement | Effect |
|---|---|
| `enable row level security` on both tables, with **no policies** | The `anon` and `authenticated` roles (used by the publishable/anon key and by logged-in Supabase users) see **zero rows** and can insert nothing through Supabase's Data API. |
| `revoke all ... from anon, authenticated, service_role` | Removes the broad default privileges Supabase gives new tables. Public roles are now blocked twice: by missing privileges **and** by RLS. |
| `grant select on events to service_role` | The two GET endpoints can read events. |
| `grant select, insert on rsvps to service_role` | `POST /api/rsvps` can insert, and read back the new row's `id`. It cannot update or delete RSVPs. |

`service_role` **bypasses RLS**, but it still needs these table privileges. Granting only what our three endpoints use is the principle of **least privilege**. If code had a bug that tried to delete RSVPs, the database would refuse.

If a later class adds features, such as cancelling an RSVP, add the matching grant then (for example, `grant delete on rsvps to service_role;`).

## Who enforces what

| Protection | Next.js backend (`app/api/...`) | Database (`schema.sql`) |
|---|---|---|
| Body must be valid JSON | ✅ 400 `INVALID_JSON` | n/a |
| `eventId` is a positive integer; required fields present | ✅ 400 `VALIDATION_ERROR` | ✅ `not null`, integer column type |
| Trim name/email; lowercase email | ✅ does the normalizing | ✅ `check` rejects un-normalized emails |
| Length limits and email shape | ✅ friendly 400 messages | ✅ `check` constraints as backstop |
| Event must exist | ✅ looks it up and returns 404 | ✅ **foreign key** (23503) |
| One RSVP per email per event | Translates the error into 409 | ✅ **unique constraint** (23505): the real guarantee |
| Which fields leave the server (no names/emails) | ✅ only selects/returns public columns | n/a |
| Generic 500s; no SQL or keys in responses | ✅ `serverErrorResponse()` | n/a |
| Keep the secret key secret | ✅ `server-only`, no `NEXT_PUBLIC_`, Vercel server variables; local `.env.local` git-ignored | n/a |
| Block direct public access to tables | n/a | ✅ RLS with no policies + revoked privileges |
| Limit what the server's own role can do | n/a | ✅ grants: select events; select + insert RSVPs |

In this project, the backend is where **rules and friendly errors** live, and the database is where **guarantees** live. Some things only the backend can do, such as choosing which fields to return or writing good error messages. Some things only the database can promise, such as no duplicates even under simultaneous requests. Neither layer trusts the browser.

Not included in this exercise: user accounts, rate limiting, CAPTCHA, or email verification. Any visitor can RSVP with any fictional email. Raise these as discussion points about what a production system would add.
