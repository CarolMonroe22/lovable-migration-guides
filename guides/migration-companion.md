# Lovable Cloud → Supabase Migration Companion

> **Paste this into Claude Code, ChatGPT, or any AI assistant to get guided help during your migration.**

---

## Understanding Your Situation

### What is Lovable Cloud?

When you click "Connect Supabase" in Lovable and don't select your own project, Lovable creates a **managed Supabase instance** for you. This is called **Lovable Cloud**.

It's convenient — you get a database without any setup. But it comes with limitations:

- **No SQL access** — You can't run queries directly. Need to delete a user? You have to ask Lovable support and wait 3-5 days.
- **No custom auth** — Only Google and Email login. No GitHub, Apple, or custom providers.
- **No email templates** — All emails come from `no-reply@auth.lovable.cloud`. You can't customize them.
- **No disconnect** — Once enabled, you cannot disconnect Lovable Cloud. It's permanent.
- **No monitoring** — No real-time metrics, no query analysis, no performance insights.
- **No mobile deep links** — Only `https://` URLs work. Custom schemes like `com.myapp://` are rejected.

### What You're Doing

You're migrating from Lovable Cloud to **your own Supabase project**. After migration, you'll have:

- Full SQL access (run any query, anytime)
- Any auth provider (Google, Apple, GitHub, SAML, etc.)
- Custom email templates (your brand, your domain)
- Real monitoring and analytics
- Complete control over your data

### What You'll Need

- **GitHub account** (free) — Your code needs to live somewhere
- **Supabase account** (free) — Where your new database will be
- **Claude Pro** ($20/month) — For Claude Code with Supabase MCP integration
- **Your Lovable project** — Already connected to GitHub

### Time Required

About 30-45 minutes, depending on how much data you have.

---

## Before You Start: Fill In Your Details

```
Project name: _______________
Lovable Cloud URL: https://__________.supabase.co
Project ID: __________ (the part before .supabase.co)

Tables I have:
- _______________
- _______________
- _______________

Storage buckets: _______________
Edge Functions: _______________
```

This helps your AI assistant understand your specific project.

---

## Phase 1: Get Your Code and Data Out

### Step 1.1: Transfer Repository Ownership

Your code currently lives in a GitHub repo controlled by Lovable. You need to transfer it to YOUR account.

**Why?** So you can make changes freely without Lovable overwriting them.

**How:**
1. Open your project at lovable.dev
2. Click the GitHub button in the toolbar
3. Go to Settings
4. Find "Transfer ownership" and click it
5. Select your personal GitHub account
6. Confirm the transfer

The repo now belongs to you.

### Step 1.2: Clone and Explore

Open your terminal and navigate to where you want the project:

```bash
cd ~/projects
claude
```

Tell Claude Code:
```
Clone my GitHub repository [YOUR-REPO-URL] to this folder.
Then explore what I have:
1. Find Supabase credentials in .env or src/integrations/supabase/
2. List all tables from src/integrations/supabase/types.ts
3. List Edge Functions in supabase/functions/
4. List migrations in supabase/migrations/

Give me a summary of everything.
```

**Save this summary** — you'll need it to verify everything migrated correctly.

### Step 1.3: Export Your Data

This is the critical step. You're extracting all your data from Lovable Cloud.

Tell Claude:
```
Export all my data from Lovable Cloud.

For each table:
1. Use the REST API: GET {SUPABASE_URL}/rest/v1/{table}?select=*
2. Save to ./export/tables/{table_name}.json

Create the folders first, then export each table. Show me progress.
```

**What's happening:** Claude is using the Supabase REST API with your existing credentials to download all your data as JSON files.

### Step 1.4: Export Storage Files (If Applicable)

If your project has file uploads (images, audio, documents), you need to export those too.

Ask Claude:
```
Does this project use Supabase Storage?
Check for buckets and list all files.
If there are files, download them to ./export/storage/{bucket_name}/
```

**Important:** Storage files have URLs that point to your old project. We'll need to update these later.

---

## Phase 2: Create Your New Supabase

### Step 2.1: Create the Project

Tell Claude:
```
Create a new Supabase project for this migration.
Show me my organizations first so I can choose.
```

Claude will:
1. List your Supabase organizations
2. Ask you to choose one
3. Ask for project name and region
4. Show you the cost (usually free if you have slots available)
5. Create the project
6. Give you the new credentials

**About costs:**
- Supabase gives you **2 free projects**
- If you need more, Pro plan is $25/month
- Use a free slot if available!

**Save your new credentials** — you'll need the URL and anon key.

### Step 2.2: Apply Migrations

Your project has SQL migrations that define your database structure (tables, functions, policies).

Tell Claude:
```
Apply all migrations from supabase/migrations/ to my new project.
Run them in order by timestamp.
Skip any that fail and tell me which ones.
```

**Expect some failures.** Lovable Cloud migrations sometimes have:
- Duplicate statements
- References to tables that don't exist yet
- Partial migrations from interrupted syncs

This is normal. The important tables will be created. You can verify with:
```
List all tables in my new Supabase project and compare with what I had.
```

### Step 2.3: Import Your Data

Now put your data into the new database:

```
Import all data from ./export/tables/ into my new Supabase project.
Go table by table and show progress.
```

**Note on foreign keys:** If import fails because of foreign key constraints, ask Claude to import tables in the right order (parent tables first, then child tables).

### Step 2.4: Upload Storage Files

If you exported storage files:

```
Create the same storage buckets in my new project.
Upload all files from ./export/storage/
Make sure buckets have appropriate RLS policies.
```

**Common issue:** You might get RLS errors when uploading. If so:
```
Create an RLS policy that allows uploads to the {bucket_name} bucket for authenticated users.
```

### Step 2.5: Update Storage URLs

Your data might contain URLs pointing to the old project. Fix them:

```
Find all references to the OLD storage URL in my exported data and database.
The old project ID is: [OLD_PROJECT_ID]
The new project ID is: [NEW_PROJECT_ID]
Update all URLs in the database.
```

### Step 2.6: Deploy Edge Functions (If Applicable)

If you have Edge Functions:

```
Deploy all Edge Functions from supabase/functions/ to my new project.
```

**Important:** Edge Functions that use secrets (API keys, etc.) need those secrets configured manually:
1. Go to Supabase Dashboard
2. Edge Functions → Your function → Secrets
3. Add the required environment variables

---

## Phase 3: Connect Everything

### Step 3.1: Update Credentials in Code

Tell Claude:
```
Update this project to use my new Supabase.
Change .env from the old URL/key to the new ones.
Search for any hardcoded references to the old project too.
```

### Step 3.2: Connect to a Fresh Lovable Project

**Important:** Lovable does NOT import existing GitHub repos directly. The workflow is:

1. Create blank project in Lovable
2. Connect GitHub (this creates a NEW repo)
3. Push your migrated code to that new repo

**Steps:**

1. Go to lovable.dev and create a new project
2. When asked what to build, select "Other" and type something like "Import existing project"
3. Click the + button → Connectors → GitHub → Connect
4. This creates a new repo in your GitHub

Now tell Claude:
```
Clone the new Lovable repo [NEW-REPO-URL] to /tmp/lovable-import
Copy all my migrated code there (everything except .git and node_modules)
Commit and push to GitHub
```

5. Wait for Lovable to sync (1-2 minutes)
6. In Lovable, go to Connectors → Supabase
7. Select your NEW Supabase project

### Step 3.3: Run Security Advisor

After migration, always check for security issues:

```
Run the security advisor on my new Supabase project.
Show me all warnings and explain what each one means.
```

**Common warnings and fixes:**

**"RLS Policy Always True"**
- Meaning: A policy allows anyone to INSERT/UPDATE/DELETE without restrictions
- Fix: Add proper conditions like `auth.uid() = user_id`

**"Leaked Password Protection Disabled"**
- Meaning: Users can use compromised passwords like "123456"
- Fix: Enable in Dashboard → Auth → Settings → Security

**"Missing RLS on table"**
- Meaning: The table is publicly accessible to anyone
- Fix: Enable RLS and add appropriate policies

### Step 3.4: Test Everything

```bash
npm install
npm run dev
```

Test these things:
- [ ] Data displays correctly
- [ ] Sign up works
- [ ] Log in works
- [ ] File uploads work (if applicable)
- [ ] Audio/video plays (if applicable)
- [ ] All features work as expected

---

## Troubleshooting

### Storage Upload Fails with RLS Error

**Error:** `new row violates row-level security policy`

**Solution:**
```
Create an RLS policy that allows uploads to the {bucket_name} bucket for authenticated users.
```

### Some Migrations Failed

**Problem:** Migrations reference tables/functions that don't exist

**Solution:** This is usually fine. Verify your core tables exist:
```
List all tables in my new Supabase project and compare with what I had before.
```

If something's missing, you can manually create it or re-run just that migration.

### Images/Audio Don't Load

**Problem:** URLs still point to old project

**Solution:**
```
Find all URLs containing [OLD_PROJECT_ID] and update them to [NEW_PROJECT_ID].
Check both the database AND the code files.
```

### Auth Not Working

**Problem:** Can't log in or sign up

**Solution:** Check these places for old credentials:
- `.env` file
- `src/integrations/supabase/client.ts`
- Any hardcoded URLs in your code

```
Search the entire codebase for the old project ID and show me all occurrences.
```

### Lovable Shows Blank Page After Push

**Problem:** After pushing code, Lovable shows "Welcome to Your Blank App"

**Solution:**
1. Wait a minute — sync might still be in progress
2. Check that GitHub repo has your code
3. If it persists, create a fresh Lovable project and try again

---

## The User ID Problem

This is an important consideration: **old users have different IDs than new ones**.

When you migrate, existing users in your old database have UUIDs that were generated by Lovable Cloud's auth system. If those users sign up in your new project, they get NEW UUIDs.

**Options:**

1. **Fresh start** (Easiest)
   - Users re-register in the new system
   - Old data stays orphaned or is deleted
   - Best for apps still in development

2. **Match by email** (Medium)
   - Write a script that connects old data to new users by email
   - Users sign up again, script reconnects their data
   - Ask Claude: "Write a script to reconnect user data by matching emails"

3. **Preserve UUIDs** (Advanced)
   - Use Supabase Admin API to create users with specific UUIDs
   - Requires service role key and careful scripting
   - Only if you absolutely need to preserve exact user IDs

---

## Costs Summary

| Service | Cost | Notes |
|---------|------|-------|
| Supabase | Free (2 projects) or $25/mo Pro | Use free slots first |
| Claude Pro | $20/mo | Includes Claude Code |
| Lovable | Your existing plan | No additional cost |

---

## What You Gain After Migration

| Before (Lovable Cloud) | After (Your Supabase) |
|------------------------|----------------------|
| No SQL access | Full SQL access anytime |
| Only Google + Email auth | Any auth provider |
| Default email templates | Custom templates, your domain |
| No monitoring | Full analytics and insights |
| Can't disconnect | Full control |
| Limited storage options | Full storage control |

---

## Need Help?

1. **Describe your specific error** to your AI assistant — it has this full context now
2. **Include error messages** — copy the exact text
3. **Share what you already tried** — helps avoid repeating steps

Or find Carol on Discord: **@carolmonroe**

---

## Quick Reference: All Prompts

```
# Explore project
Look at my project and tell me: credentials, tables, Edge Functions, migrations.

# Export data
Export all data from Lovable Cloud to ./export/tables/

# Export storage
Download all storage files to ./export/storage/

# Create new project
Create a new Supabase project. Show my organizations first.

# Apply migrations
Apply all migrations from supabase/migrations/. Skip failures.

# Import data
Import all data from ./export/tables/

# Update URLs
Find and update all URLs with old project ID to new project ID.

# Security check
Run security advisor and explain all warnings.

# Push to Lovable
Clone [NEW-REPO], copy my code, push to GitHub.
```

---

*Full guide with screenshots: https://carolmonroe.com/blog/migrate-lovable-cloud-claude-code*

*GitHub: https://github.com/CarolMonroe22/lovable-migration-guides*
