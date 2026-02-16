# Smart Bookmark App

A real-time bookmark manager built with Next.js 15, Supabase, and Tailwind CSS. Features instant synchronization across multiple tabs/devices and secure user authentication.

## 🚀 Live Demo
**[https://smart-bookmarks-4l7w.vercel.app](https://smart-bookmarks-4l7w.vercel.app)**

## ✨ Features
- **Google OAuth Authentication** - Secure, passwordless login
- **Real-time Sync** - Changes appear instantly across all open tabs
- **Private Bookmarks** - Row Level Security ensures users only see their own data
- **CRUD Operations** - Add, view, and delete bookmarks seamlessly
- **Responsive Design** - Works beautifully on desktop and mobile
- **Type-safe** - Built with TypeScript for better developer experience

## 🛠️ Tech Stack
- **Next.js 15** (App Router, Server Components)
- **Supabase** (PostgreSQL, Auth, Realtime subscriptions)
- **Tailwind CSS** (Utility-first styling)
- **TypeScript** (Type safety)

## 📋 Problems Encountered & Solutions

### Problem 1: Real-time Updates Not Showing Immediately After Insert
**Issue:** When adding a bookmark, it would only appear after a page refresh or when another tab triggered a real-time event.

**Root Cause:** The `INSERT` operation wasn't immediately updating the local state. The real-time subscription would catch changes from other sources, but not the current user's own inserts.

**Solution:** Modified the `addBookmark` function to:
1. Use `.select().single()` to return the inserted row
2. Immediately update local state with the new bookmark
3. Keep real-time subscription for cross-tab/cross-device sync

```typescript
const { data } = await supabase
  .from('bookmarks')
  .insert({ url, title, user_id: user.id })
  .select()
  .single()

if (data) {
  setBookmarks(prev => [data, ...prev])
}
```

### Problem 2: TypeScript Compilation Errors on Vercel Deployment
**Issue:** Build failed with "Parameter 'cookiesToSet' implicitly has an 'any' type" in both `middleware.ts` and `utils/supabase/server.ts`.

**Root Cause:** Vercel's strict TypeScript checks caught implicit `any` types that weren't flagged in local development.

**Solution:** Added explicit type annotations:
```typescript
setAll(cookiesToSet: any) {
  cookiesToSet.forEach(({ name, value, options }: any) => {
    // ... cookie operations
  })
}
```

### Problem 3: OAuth Redirect to Localhost in Production
**Issue:** After Google sign-in on the deployed app, users were redirected to `http://localhost:3000/?code=...` instead of the production URL.

**Root Cause:** Supabase's Site URL and Redirect URLs were not configured for the production domain.

**Solution:** Updated Supabase Authentication settings:
- Site URL: `https://smart-bookmarks-4l7w.vercel.app`
- Redirect URLs: `https://smart-bookmarks-4l7w.vercel.app/**`

This ensures OAuth callbacks route to the correct domain in production.

### Problem 4: React2Shell Security Vulnerability (CVE-2025-66478)
**Issue:** Vercel deployment blocked due to vulnerable Next.js version (15.1.3).

**Root Cause:** Next.js versions 15.0.0 through 16.0.6 contained a critical security vulnerability in React Server Components.

**Solution:** 
1. Updated `package.json` to Next.js 15.1.7 (patched version)
2. Ran `npm audit fix --force` to resolve dependency conflicts
3. Redeployed to Vercel

## 🏗️ Architecture Decisions

### Why Supabase Real-time?
Chose Supabase's real-time subscriptions over polling because:
- Lower latency (WebSocket-based)
- Reduced server load
- Built-in filtering by user_id
- Automatic reconnection handling

### Why Row Level Security?
Implemented RLS policies instead of API-level authorization because:
- Security enforced at database level (defense in depth)
- Prevents accidental data leaks
- Works automatically with Supabase client
- No additional middleware needed

### Why Server/Client Component Split?
Used Next.js App Router patterns:
- Server Components for auth checks (middleware)
- Client Components for interactive UI (dashboard)
- Optimizes bundle size and improves performance

## 📦 Local Setup

```bash
# Clone repository
git clone https://github.com/om21kk/smart-bookmarks.git
cd smart-bookmarks

# Install dependencies
npm install

# Create .env.local
echo "NEXT_PUBLIC_SUPABASE_URL=your_url" > .env.local
echo "NEXT_PUBLIC_SUPABASE_ANON_KEY=your_key" >> .env.local

# Run development server
npm run dev
```

## 🗄️ Database Schema

```sql
CREATE TABLE bookmarks (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE NOT NULL,
  url TEXT NOT NULL,
  title TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Row Level Security Policies
ALTER TABLE bookmarks ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view own bookmarks" ON bookmarks
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own bookmarks" ON bookmarks
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own bookmarks" ON bookmarks
  FOR DELETE USING (auth.uid() = user_id);
```

## 🚢 Deployment

Deployed on Vercel with automatic CI/CD:
1. Push to `main` branch triggers build
2. Environment variables injected from Vercel dashboard
3. Next.js optimizes and deploys to edge network

## 🔐 Security Features
- OAuth 2.0 authentication (no password storage)
- Row Level Security (database-level authorization)
- HTTPS-only in production
- Environment variables for sensitive keys
- CSRF protection via Supabase session cookies

## 📈 Future Improvements
- [ ] Add bookmark folders/categories
- [ ] Implement search and filtering
- [ ] Add bookmark tags
- [ ] Export bookmarks (JSON/CSV)
- [ ] Browser extension for quick saves
- [ ] Bookmark preview thumbnails

## 📝 License
MIT

---

**Built with ❤️ by Om Kale**
