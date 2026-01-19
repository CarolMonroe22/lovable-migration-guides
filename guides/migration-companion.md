# Lovable Cloud → Supabase Migration Companion

> **Paste this into Claude Code, ChatGPT, or any AI assistant to get help during your migration.**

---

## What This Is

You're migrating a Lovable project from **Lovable Cloud** (managed Supabase) to your **own Supabase project**. This gives you:
- Direct SQL access
- Custom auth providers
- Custom email templates
- Full monitoring and control

---

## Your Current Setup (Fill In)

```
Project name: _______________
Lovable Cloud URL: https://__________.supabase.co
Tables: _______________
Has Storage files: yes / no
Has Edge Functions: yes / no
```

---

## Migration Checklist

### Phase 1: Export from Lovable Cloud

- [ ] **Find credentials** in `.env` or `src/integrations/supabase/`
- [ ] **Export tables** via REST API to `./export/tables/`
- [ ] **Export storage** (if applicable) to `./export/storage/`
- [ ] **Note Edge Functions** in `supabase/functions/`

### Phase 2: Create New Supabase

- [ ] **Create project** (use 1 of 2 free projects, or Pro $25/mo)
- [ ] **Apply migrations** from `supabase/migrations/`
- [ ] **Import data** from `./export/tables/`
- [ ] **Upload storage files** and update URLs
- [ ] **Deploy Edge Functions**

### Phase 3: Connect Everything

- [ ] **Update `.env`** with new credentials
- [ ] **Create blank Lovable project** → connect GitHub → push code
- [ ] **Connect new Supabase** in Lovable Connectors
- [ ] **Run Security Advisor**
- [ ] **Test everything**

---

## Key Prompts to Use

### Explore Project
```
Look at my project and tell me:
1. Supabase credentials (from .env)
2. All tables (from types.ts)
3. Edge Functions (supabase/functions/)
4. Migrations (supabase/migrations/)
```

### Export Data
```
Export all my data from Lovable Cloud.
For each table, use the REST API: GET {SUPABASE_URL}/rest/v1/{table}?select=*
Save to ./export/tables/{table_name}.json
```

### Export Storage
```
List all files in Supabase Storage.
Download them to ./export/storage/{bucket_name}/
```

### Create New Project
```
Create a new Supabase project.
Show me my organizations first so I can choose.
```

### Apply Migrations
```
Apply all migrations from supabase/migrations/ to my new project.
Run them in order. Skip failures and tell me which ones failed.
```

### Import Data
```
Import all data from ./export/tables/ into my new Supabase.
```

### Update URLs
```
Find all URLs containing the old project ID and update them to the new one.
Check both the database AND the code.
```

### Security Check
```
Run the security advisor on my new project.
Explain each warning and how to fix it.
```

---

## Common Issues & Fixes

### "RLS policy violation" on Storage upload
**Cause:** Bucket doesn't allow uploads
**Fix:** Ask AI to create an RLS policy for the bucket

### Some migrations failed
**Cause:** Lovable Cloud migrations sometimes have duplicates
**Fix:** Usually fine — core tables still get created. Verify with `list all tables`

### Images/audio don't load
**Cause:** URLs still point to old project
**Fix:** Use the "Update URLs" prompt above

### Auth not working
**Cause:** Credentials not updated everywhere
**Fix:** Check `.env`, `src/integrations/supabase/client.ts`, and any hardcoded values

### Security Advisor warnings

| Warning | Meaning | Fix |
|---------|---------|-----|
| RLS Always True | Anyone can modify data | Add proper conditions like `auth.uid() = user_id` |
| Leaked Password Protection | Users can use "123456" | Enable in Auth → Settings |
| Missing RLS | Table is public | Enable RLS and add policies |

---

## Lovable Cloud Limitations (Why You're Migrating)

| Limitation | Impact |
|------------|--------|
| No SQL access | Can't run queries directly |
| No custom auth | Only Google + Email |
| No email templates | Stuck with default emails |
| No disconnect | Once enabled, permanent |
| No monitoring | No real-time metrics |

---

## User ID Problem

Old users have different IDs than new ones. Options:
1. **Fresh start** — users re-register (easiest)
2. **Match by email** — script to reconnect data by email
3. **Preserve UUIDs** — advanced, uses Admin API

---

## Costs

| Service | Cost |
|---------|------|
| Supabase | 2 free projects, then $25/mo Pro |
| Claude Pro | $20/mo (includes Claude Code) |

---

## Need Help?

Describe your specific error to your AI assistant — it has this context now.

Or find Carol on Discord: **@carolmonroe**

---

*Guide: https://carolmonroe.com/blog/migrate-lovable-cloud-claude-code*
