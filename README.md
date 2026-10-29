# AssignFlow — Academic Assignment Management System

AssignFlow is a modern, responsive full-stack Assignment Management System built specifically for college and university students. It empowers students to organize course assignments, track deadlines with urgent countdowns, prioritize academic work, and monitor completion progress.

Built with **Next.js (App Router)**, **React & TypeScript**, **Tailwind CSS**, and **Supabase (PostgreSQL with Row Level Security & `@supabase/ssr`)**.

---

## 🌟 Key Features

- **Supabase Authentication**: Secure email & password signup, login, session persistence, and server-side cookie authentication via `@supabase/ssr`.
- **Database Row Level Security (RLS)**: Strictest data isolation ensuring each student can read, insert, update, and delete only their own assignments using `auth.uid() = user_id`.
- **Academic Dashboard**:
  - Live statistics calculated directly from PostgreSQL: Total, Pending, In Progress, Completed, Overdue.
  - Academic Progress Tracker with completion percentage and milestone guidance.
  - Deadline Radar highlighting urgent upcoming submissions.
- **Full Assignment CRUD**:
  - Create new assignments with course, due date/time, priority level (Low, Medium, High), status (Pending, In Progress, Completed), and detailed instructions.
  - Interactive assignment list with responsive cards, course badges, and relative deadline countdowns.
  - Assignment detail view with status updates and metadata timestamps.
  - Edit view with pre-filled inputs and validation.
  - Safe deletion with confirmation dialog modal.
- **Search, Filtering & Sorting**:
  - Real-time search across assignment titles and course subjects.
  - Filter by status, priority, or subject.
  - Sort by deadline (earliest/latest), creation timestamp, or title.
  - One-click clear filters option.
- **Responsive Modern UI**:
  - Desktop sidebar navigation with student avatar and quick links.
  - Mobile slide-over drawer and top navigation bar.
  - Accessible contrast, clear form validation, and feedback messages.

---

## 🛠️ Technology Stack

- **Framework**: [Next.js](https://nextjs.org/) (v15 App Router)
- **Language**: [TypeScript](https://www.typescriptlang.org/) (Strict type checking)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/)
- **Icons**: [Lucide React](https://lucide.dev/)
- **Database**: [Supabase PostgreSQL](https://supabase.com/)
- **Authentication**: Supabase Auth with `@supabase/ssr`
- **Deployment**: Vercel ready

---

## 📋 Environment Variables

AssignFlow uses the exact environment variable names required:

```env
NEXT_PUBLIC_SUPABASE_URL=your_supabase_project_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_publishable_or_anon_key
```

### Setup Instructions

1. Copy `.env.example` to `.env.local`:
   ```bash
   cp .env.example .env.local
   ```
2. Retrieve your Supabase credentials:
   - Log into the [Supabase Dashboard](https://supabase.com/dashboard).
   - Select your project.
   - Navigate to **Project Settings** > **API**.
   - Copy **Project URL** into `NEXT_PUBLIC_SUPABASE_URL`.
   - Copy the **anon / public** key into `NEXT_PUBLIC_SUPABASE_ANON_KEY`.

> **Security Note**: Never commit `.env.local` or expose service role keys in client code. `.gitignore` is already pre-configured to exclude all `.env*.local` files.

---

## 🗄️ Database Setup & SQL Migration

Run the SQL migration script located at `supabase/migrations/20261029_create_assignments_table.sql` in your Supabase SQL Editor:

```sql
-- 1. Create assignments table
CREATE TABLE IF NOT EXISTS public.assignments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    title TEXT NOT NULL CHECK (char_length(trim(title)) > 0),
    description TEXT,
    subject TEXT NOT NULL CHECK (char_length(trim(subject)) > 0),
    due_date TIMESTAMPTZ NOT NULL,
    priority TEXT NOT NULL CHECK (priority IN ('Low', 'Medium', 'High')) DEFAULT 'Medium',
    status TEXT NOT NULL CHECK (status IN ('Pending', 'In Progress', 'Completed')) DEFAULT 'Pending',
    created_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now()),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT timezone('utc'::text, now())
);

-- 2. Performance Indexes
CREATE INDEX IF NOT EXISTS idx_assignments_user_id ON public.assignments(user_id);
CREATE INDEX IF NOT EXISTS idx_assignments_user_status ON public.assignments(user_id, status);
CREATE INDEX IF NOT EXISTS idx_assignments_user_due_date ON public.assignments(user_id, due_date);
CREATE INDEX IF NOT EXISTS idx_assignments_user_priority ON public.assignments(user_id, priority);
CREATE INDEX IF NOT EXISTS idx_assignments_user_subject ON public.assignments(user_id, subject);

-- 3. Automatic updated_at trigger
CREATE OR REPLACE FUNCTION public.handle_updated_at()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = timezone('utc'::text, now());
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

DROP TRIGGER IF EXISTS trigger_assignments_updated_at ON public.assignments;
CREATE TRIGGER trigger_assignments_updated_at
    BEFORE UPDATE ON public.assignments
    FOR EACH ROW
    EXECUTE FUNCTION public.handle_updated_at();

-- 4. Enable Row Level Security (RLS)
ALTER TABLE public.assignments ENABLE ROW LEVEL SECURITY;

-- 5. Strict Multi-Tenant RLS Policies
CREATE POLICY "Users can read own assignments"
    ON public.assignments FOR SELECT TO authenticated
    USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own assignments"
    ON public.assignments FOR INSERT TO authenticated
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own assignments"
    ON public.assignments FOR UPDATE TO authenticated
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can delete own assignments"
    ON public.assignments FOR DELETE TO authenticated
    USING (auth.uid() = user_id);
```

---

## 🚀 Local Development

1. Install dependencies:
   ```bash
   npm install
   ```

2. Start the development server:
   ```bash
   npm run dev
   ```
   Open [http://localhost:3000](http://localhost:3000) in your browser.

3. Run production build check:
   ```bash
   npm run build
   ```

---

## 🌐 Supabase Auth URL Configuration

In your Supabase project settings:
1. Go to **Authentication** > **URL Configuration**.
2. Set **Site URL** to:
   - For local development: `http://localhost:3000`
   - For production: `https://your-assignflow-domain.vercel.app`
3. Under **Redirect URLs**, add:
   - `http://localhost:3000/auth/callback`
   - `https://your-assignflow-domain.vercel.app/auth/callback`

---

## 🚢 Deploying to Vercel

1. Push your code to a GitHub repository:
   ```bash
   git init
   git add .
   git commit -m "Initial commit of AssignFlow"
   git remote add origin https://github.com/your-username/assignflow.git
   git push -u origin main
   ```
2. Go to [Vercel](https://vercel.com) and click **Add New Project**.
3. Import your GitHub repository.
4. Under **Environment Variables**, add:
   - `NEXT_PUBLIC_SUPABASE_URL` = your Supabase project URL
   - `NEXT_PUBLIC_SUPABASE_ANON_KEY` = your Supabase anon/publishable key
5. Click **Deploy**. Vercel will build and deploy the Next.js App Router application automatically.

---

## ✅ Production Verification Checklist

- [x] Next.js App Router static and dynamic route compilation (`npm run build` succeeds).
- [x] TypeScript strict type checking passes with 0 errors.
- [x] Row Level Security (RLS) enabled with `auth.uid() = user_id` on all CRUD operations.
- [x] Middleware route protection for `/dashboard`, `/assignments`, and `/settings`.
- [x] Auth redirects handle authenticated and unauthenticated sessions securely.
- [x] Form validation on client and server constraints in database schema.
- [x] Responsive layout tested for mobile and desktop screens.
- [x] Secrets excluded from git tracking (`.env*.local` in `.gitignore`).

