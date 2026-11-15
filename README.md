# Secure File Sharing System

A small web app where users sign up, upload files to a private store, and share them through short-lived signed links. Built with React and Supabase (Auth, Postgres, Storage).

## Features

- Email + password signup/login (minimum 6 characters)
- Protected dashboard with header (app name, user email, Logout)
- Upload via file picker or drag-and-drop, with client-side validation and a progress/loading state
- "My Files" table: name, size, type, upload date, Share and Delete
- Share modal: link expiry of 5 minutes, 1 hour or 24 hours, with Copy button and exact expiry time
- Delete with confirmation (removes storage object and database row)
- Loading, error and success states throughout

## Tech stack

- Vite + React 19 + TypeScript
- React Router
- Supabase (`@supabase/supabase-js`): Auth, Postgres, Storage
- Plain CSS, no UI libraries
- Deployed on Vercel

## Security design

- **Private bucket** – `secure-files` is not public; objects are only reachable with an authenticated session or a signed URL.
- **Row Level Security** – `public.files` has RLS enabled; users can only select, insert and delete rows where `owner_id = auth.uid()`. There is no update policy.
- **Folder-per-user storage policies** – objects live at `<user_id>/<timestamp>-<sanitized filename>`. Policies on `storage.objects` only allow insert/select/delete when the first folder equals `auth.uid()`.
- **Expiring signed URLs** – sharing uses `createSignedUrl()` with a 5 min / 1 h / 24 h lifetime, so links stop working automatically.
- **Client-side file validation** – only PDF, PNG, JPEG, TXT and ZIP up to 10 MB are accepted before upload. The bucket also enforces the same limits server-side (`file_size_limit`, `allowed_mime_types`), so client checks are a convenience, not the only defence.
- The anon key is safe to expose in the browser because RLS and storage policies enforce access. Never put the `service_role` key in this app.

## Folder structure

```
.
├── supabase/schema.sql        # bucket, table, RLS and storage policies
├── src/
│   ├── lib/
│   │   ├── supabase.ts        # Supabase client + env check
│   │   ├── auth.tsx           # AuthProvider / useAuth
│   │   └── files.ts           # validation, formatting helpers, types
│   ├── components/
│   │   ├── ProtectedRoute.tsx
│   │   ├── UploadArea.tsx
│   │   ├── ShareModal.tsx
│   │   └── ConfirmDialog.tsx
│   ├── pages/
│   │   ├── Login.tsx
│   │   └── Dashboard.tsx
│   ├── App.tsx                # routes
│   ├── main.tsx
│   └── index.css
├── vercel.json                # SPA rewrite for React Router
├── .env.example
└── package.json
```

## Setup

1. **Create a Supabase project** at [supabase.com](https://supabase.com).
2. **Run the schema**: open the SQL Editor, paste the contents of `supabase/schema.sql`, and run it.
3. **Auth settings** (optional): under Authentication → Providers → Email you can disable "Confirm email" for easier local testing.
4. **Configure env**: copy the project URL and anon key (Project Settings → API) into `.env.local`:
   ```
   VITE_SUPABASE_URL=https://your-project.supabase.co
   VITE_SUPABASE_ANON_KEY=your-anon-key
   ```
5. **Run locally**:
   ```bash
   npm install
   npm run dev
   ```
6. **Deploy to Vercel**: push the repo to GitHub, import it in Vercel, add the two `VITE_` environment variables, and deploy. `vercel.json` rewrites all routes to `/index.html`. Add your Vercel URL to Supabase → Authentication → URL Configuration (Site URL).

## Notes

- `supabase-js` does not expose upload progress events, so the UI shows a per-file progress bar based on files completed plus an "Uploading…" state.
