# File Structure Scaffold - SoundScout Explore

## Complete Project Structure

```
soundscout-explore/
├── app/                          # Next.js App Router
│   ├── layout.tsx                # Root layout
│   ├── page.tsx                  # Landing page
│   ├── globals.css               # Global styles
│   │
│   ├── (auth)/                   # Auth group (no layout)
│   │   ├── login/
│   │   │   └── page.tsx
│   │   ├── signup/
│   │   │   └── page.tsx
│   │   └── forgot-password/
│   │       └── page.tsx
│   │
│   ├── (app)/                    # Authenticated app group
│   │   ├── layout.tsx            # App layout with sidebar
│   │   ├── dashboard/
│   │   │   └── page.tsx
│   ├── event/
│   │   ├── page.tsx              # List view
│   │   ├── [id]/
│   │   │   └── page.tsx          # Detail view
│   │   └── new/
│   │       └── page.tsx          # Create form
│   ├── artist/
│   │   ├── page.tsx              # List view
│   │   ├── [id]/
│   │   │   └── page.tsx          # Detail view
│   │   └── new/
│   │       └── page.tsx          # Create form
│   ├── venue/
│   │   ├── page.tsx              # List view
│   │   ├── [id]/
│   │   │   └── page.tsx          # Detail view
│   │   └── new/
│   │       └── page.tsx          # Create form
│   │   └── settings/
│   │       └── page.tsx
│   │
│   └── api/                      # API Routes
│       ├── auth/
│       │   ├── signin/route.ts
│       │   ├── signup/route.ts
│       │   └── signout/route.ts
│   ├── events/
│   │   ├── route.ts              # GET list, POST create
│   │   └── [id]/
│   │       └── route.ts          # GET, PUT, DELETE
│   ├── artists/
│   │   ├── route.ts              # GET list, POST create
│   │   └── [id]/
│   │       └── route.ts          # GET, PUT, DELETE
│   ├── venues/
│   │   ├── route.ts              # GET list, POST create
│   │   └── [id]/
│   │       └── route.ts          # GET, PUT, DELETE
│       └── webhooks/
│           └── stripe/route.ts
│
├── components/                   # React Components
│   ├── ui/                       # shadcn/ui components
│   │   ├── button.tsx
│   │   ├── input.tsx
│   │   ├── card.tsx
│   │   ├── dialog.tsx
│   │   ├── dropdown-menu.tsx
│   │   └── ...
│   │
│   ├── layout/                   # Layout components
│   │   ├── header.tsx
│   │   ├── sidebar.tsx
│   │   ├── footer.tsx
│   │   └── nav.tsx
│   │
│   ├── forms/                    # Form components
│   │   ├── login-form.tsx
│   │   ├── signup-form.tsx
│   │   └── event-form.tsx
│   │   └── artist-form.tsx
│   │   └── venue-form.tsx
│   │
│   └── features/                 # Feature components
│       ├── event/
│       │   ├── event-card.tsx
│       │   ├── event-list.tsx
│       │   └── event-detail.tsx
│       ├── artist/
│       │   ├── artist-card.tsx
│       │   ├── artist-list.tsx
│       │   └── artist-detail.tsx
│       ├── venue/
│       │   ├── venue-card.tsx
│       │   ├── venue-list.tsx
│       │   └── venue-detail.tsx
│
├── lib/                          # Utilities & Services
│   ├── supabase/
│   │   ├── client.ts             # Browser client
│   │   ├── server.ts             # Server client
│   │   └── types.ts              # DB types
│   │
│   ├── utils/
│   │   ├── cn.ts                 # classnames helper
│   │   ├── format.ts             # Formatters
│   │   └── validation.ts         # Zod schemas
│   │
│   └── hooks/
│       ├── use-user.ts
│       ├── use-toast.ts
│       └── use-media-query.ts
│
├── types/                        # TypeScript types
│   ├── index.ts                  # Main types
│   └── api.ts                    # API types
│
├── styles/                       # Additional styles
│   └── components.css
│
├── public/                       # Static assets
│   ├── favicon.ico
│   ├── og-image.png
│   └── images/
│
├── supabase/                     # Supabase config
│   └── migrations/
│       └── 001_initial_schema.sql
│
├── .env.example                  # Env template
├── .env.local                    # Local env (gitignored)
├── .gitignore
├── next.config.mjs
├── package.json
├── postcss.config.mjs
├── tailwind.config.ts
├── tsconfig.json
└── README.md
```

---

## Key Files to Create First

### 1. Core Configuration
- [ ] `package.json` - Dependencies
- [ ] `next.config.mjs` - Next.js config
- [ ] `tailwind.config.ts` - Tailwind config
- [ ] `.env.example` - Environment template

### 2. Supabase Setup
- [ ] `lib/supabase/client.ts` - Browser client
- [ ] `lib/supabase/server.ts` - Server client
- [ ] `supabase/migrations/001_initial_schema.sql`

### 3. Layout & Auth
- [ ] `app/layout.tsx` - Root layout
- [ ] `app/(auth)/login/page.tsx` - Login
- [ ] `app/(app)/layout.tsx` - App layout

### 4. Core Features
- [ ] `app/(app)/event/page.tsx` - Event list
- [ ] `app/(app)/artist/page.tsx` - Artist list
- [ ] `app/(app)/venue/page.tsx` - Venue list

---

## Component Naming Conventions

| Type | Pattern | Example |
|------|---------|---------|
| Page | lowercase with dashes | `app/user-settings/page.tsx` |
| Component | PascalCase | `components/UserCard.tsx` |
| Hook | camelCase with `use` prefix | `hooks/useUser.ts` |
| Utility | camelCase | `lib/utils/formatDate.ts` |
| Type | PascalCase | `types/User.ts` |
| API Route | lowercase with dashes | `api/user-profile/route.ts` |
