# Smart Bookmark App

A real-time bookmark manager built with Next.js, Supabase, and Tailwind CSS.

## Live Demo
[Add your Vercel URL here after deployment]

## Features
- Google OAuth authentication
- Add bookmarks (URL + title)
- Private bookmarks per user (Row Level Security)
- Real-time sync across tabs
- Delete bookmarks
- Responsive design

## Tech Stack
- Next.js 15 (App Router)
- Supabase (Auth, Database, Realtime)
- Tailwind CSS
- TypeScript

## Local Setup

1. Clone the repository
```bash
git clone <your-repo-url>
cd smart-bookmarks
```

2. Install dependencies
```bash
npm install
```

3. Create `.env.local` file with your Supabase credentials
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
```

4. Run development server
```bash
npm run dev
```

5. Open [http://localhost:3000](http://localhost:3000)

## Supabase Setup

### 1. Create the bookmarks table
```sql
CREATE TABLE bookmarks (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  url TEXT NOT NULL,
  title TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);
```

### 2. Enable Row Level Security
```sql
ALTER TABLE bookmarks ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own bookmarks" ON bookmarks
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own bookmarks" ON bookmarks
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own bookmarks" ON bookmarks
  FOR DELETE USING (auth.uid() = user_id);
```

### 3. Enable Realtime
Go to Database > Replication and enable realtime for the `bookmarks` table.

### 4. Configure Google OAuth
1. Create OAuth credentials in Google Cloud Console
2. Add redirect URI: `https://<your-supabase-project>.supabase.co/auth/v1/callback`
3. Enable Google provider in Supabase Dashboard (Authentication > Providers)
4. Add your Google Client ID and Secret

## Deployment

### Deploy to Vercel
1. Push code to GitHub
2. Import project in Vercel
3. Add environment variables:
   - `NEXT_PUBLIC_SUPABASE_URL`
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY`
4. Deploy

### Update OAuth URLs
After deployment, update:
- Supabase: Site URL and Redirect URLs in Authentication > URL Configuration
- Google Console: Authorized redirect URIs (if needed)

## Problems Encountered & Solutions

### Problem 1: [Add your issues here]
**Solution:** [How you fixed it]

### Problem 2: [Add your issues here]
**Solution:** [How you fixed it]

## License
MIT
