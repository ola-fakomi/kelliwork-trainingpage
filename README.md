# KelliWorks Training Site

A cloud-synced training hub for Patricia Montan's onboarding at KelliWorks Accounting Firm.

## Quick Start

### 1. Configure Supabase

1. Go to [Supabase Dashboard](https://supabase.com/dashboard)
2. Create a new project (or use existing)
3. **Project Settings → API**
   - Copy the `Project URL` and `anon public` key
4. Paste values into `config/supabase-config.js`

### 2. Create the Progress Table

In Supabase SQL Editor, run:

```sql
CREATE TABLE progress (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id TEXT NOT NULL,
  data JSONB NOT NULL DEFAULT '{}',
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE progress ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can read own progress" ON progress
  FOR SELECT USING (true);

CREATE POLICY "Users can upsert own progress" ON progress
  FOR INSERT WITH CHECK (true);

CREATE POLICY "Users can update own progress" ON progress
  FOR UPDATE USING (true);
```

### 3. Deploy

**Option A: Static Hosting (Netlify)** (Recommended)
```bash
# 1. Drag src/index.html to netlify.com/drop
# 2. Or use Netlify CLI:
npm i -g netlify-cli
netlify deploy --prod
```

**Option B: Vercel**
```bash
npm i -g vercel
vercel --prod
```

**Option C: GitHub Pages**
```bash
# Push to GitHub repo
# Settings > Pages > Source: main branch
```

**Option D: Kelliworks.com cPanel**
```bash
# Upload src/index.html to public_html/
# via FileZilla or cPanel File Manager
```

## Project Structure

```
training-site/
├── src/
│   └── index.html          # Main training site (Supabase integrated)
├── config/
│   └── supabase-config.js  # Supabase settings (edit this file)
└── README.md              # You are here
```

## How Progress Sync Works

1. User completes checkboxes/takes notes
2. Data saves to localStorage (works offline)
3. Data syncs to Supabase database
4. Any device/browser can access the same progress

## Managing Multiple Trainees

To add another trainee, create a new `USER_ID` in `supabase-config.js`:

```javascript
const USER_ID = 'new-trainee-name';
```

Each trainee's progress is stored separately by `user_id` in the progress table.

## Troubleshooting

**Supabase not syncing?**
- Check browser console for errors
- Verify SUPABASE_URL and SUPABASE_ANON_KEY are correct
- Ensure the progress table exists in your Supabase project

**Progress lost?**
- Use the "Download Progress" button as backup
- localStorage persists even if Supabase fails
