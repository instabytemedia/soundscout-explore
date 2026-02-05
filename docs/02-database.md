# 02 - Database Schema

> **Agent:** Database Agent 🗄️
> **Goal:** Set up Supabase database with all tables and policies

---

## ⚠️ CRITICAL: Avoid Duplicate Fields

**NEVER define these fields in your custom schema - they are automatically added:**
- `id` - UUID Primary Key (auto-generated)
- `user_id` - UUID Foreign Key to auth.users (auto-generated)
- `created_at` - TIMESTAMPTZ (auto-generated)
- `updated_at` - TIMESTAMPTZ (auto-generated)

**Only define entity-SPECIFIC fields in your tables.**

Example of WRONG vs RIGHT:
```sql
-- WRONG: Duplicate fields!
CREATE TABLE items (
  id UUID PRIMARY KEY,       -- Already auto-added!
  user_id UUID,              -- Already auto-added!
  created_at TIMESTAMPTZ,    -- Already auto-added!
  name TEXT,
  id UUID,                   -- DUPLICATE! Will cause error!
  created_at TIMESTAMPTZ     -- DUPLICATE! Will cause error!
);

-- RIGHT: Only custom fields
CREATE TABLE items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  -- Only custom fields below:
  name TEXT NOT NULL,
  description TEXT,
  status TEXT NOT NULL DEFAULT 'active'
);
```

---

## Context

**Product:** SoundScout Explore
**Entities:** Event, Artist, Venue

### Data Model

**Event**: Live music event
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `name`: TEXT (required, indexed) - Event name
- `description`: TEXT - Event description
- `location`: TEXT (required, indexed) - Event location
- `start_time`: TIMESTAMPTZ (required, indexed, default: now) - Event start time
- `end_time`: TIMESTAMPTZ (required, indexed, default: now) - Event end time
Relationships:
  - many_to_many → Artist: An event can have multiple artists, and an artist can perform at multiple events
  - one_to_one → Venue: An event is held at one venue

**Artist**: Music artist
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `name`: TEXT (required, indexed) - Artist name
- `genre`: TEXT - Artist genre
Relationships:
  - many_to_many → Event: An artist can perform at multiple events, and an event can have multiple artists

**Venue**: Music venue
- `id`: UUID (required) - Primary key
- `created_at`: TIMESTAMPTZ (required) - Creation timestamp
- `updated_at`: TIMESTAMPTZ (required) - Last update timestamp
- `user_id`: UUID (required) - Owner user ID
- `name`: TEXT (required, indexed) - Venue name
- `location`: TEXT (required, indexed) - Venue location
Relationships:
  - one_to_one → Event: A venue can host one event at a time

---

## Instructions

### 1. Profiles Table

Create a **profiles** table that syncs with auth.users:

**Requirements:**
- Primary Key: `id` (UUID) - Reference to auth.users(id)
- Fields: username (unique), display_name, avatar_url, bio
- Timestamps: created_at, updated_at
- RLS: Everyone can view profiles, only owner can update
- Auto-Trigger: Automatically create profile on user signup

**Important:**
- ON DELETE CASCADE for foreign key to auth.users
- Trigger Function for auto-create on signup
- Public read access, owner-only write access

### 2. Entity Tables

Create a table for each entity:

**1. Event Table (`events`):**
Purpose: Live music event

```sql
CREATE TABLE events (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  name TEXT NOT NULL DEFAULT '' -- Event name,
  description TEXT DEFAULT '' -- Event description,
  location TEXT NOT NULL DEFAULT '' -- Event location,
  start_time TIMESTAMPTZ NOT NULL DEFAULT 'now' -- Event start time,
  end_time TIMESTAMPTZ NOT NULL DEFAULT 'now' -- Event end time

);

-- Indexes for performance
CREATE INDEX events_user_id_idx ON events(user_id);
CREATE INDEX events_created_at_idx ON events(created_at DESC);
CREATE INDEX events_name_idx ON events(name);
CREATE INDEX events_location_idx ON events(location);
CREATE INDEX events_start_time_idx ON events(start_time);
CREATE INDEX events_end_time_idx ON events(end_time);


-- RLS Policies (Granular)
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "events_select_own"
  ON events FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "events_insert_own"
  ON events FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "events_update_own"
  ON events FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "events_delete_own"
  ON events FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- many_to_many to Artist (foreign key: artist_id)
- one_to_one to Venue (foreign key: venue_id)

**2. Artist Table (`artists`):**
Purpose: Music artist

```sql
CREATE TABLE artists (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  name TEXT NOT NULL DEFAULT '' -- Artist name,
  genre TEXT DEFAULT '' -- Artist genre

);

-- Indexes for performance
CREATE INDEX artists_user_id_idx ON artists(user_id);
CREATE INDEX artists_created_at_idx ON artists(created_at DESC);
CREATE INDEX artists_name_idx ON artists(name);


-- RLS Policies (Granular)
ALTER TABLE artists ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "artists_select_own"
  ON artists FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "artists_insert_own"
  ON artists FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "artists_update_own"
  ON artists FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "artists_delete_own"
  ON artists FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- many_to_many to Event (foreign key: event_id)

**3. Venue Table (`venues`):**
Purpose: Music venue

```sql
CREATE TABLE venues (
  -- Standard fields
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),

  -- Entity-specific fields
  name TEXT NOT NULL DEFAULT '' -- Venue name,
  location TEXT NOT NULL DEFAULT '' -- Venue location

);

-- Indexes for performance
CREATE INDEX venues_user_id_idx ON venues(user_id);
CREATE INDEX venues_created_at_idx ON venues(created_at DESC);
CREATE INDEX venues_name_idx ON venues(name);
CREATE INDEX venues_location_idx ON venues(location);


-- RLS Policies (Granular)
ALTER TABLE venues ENABLE ROW LEVEL SECURITY;

-- SELECT: Users can only view their own records
CREATE POLICY "venues_select_own"
  ON venues FOR SELECT
  USING (auth.uid() = user_id);

-- INSERT: Users can only create records for themselves
CREATE POLICY "venues_insert_own"
  ON venues FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- UPDATE: Users can only update their own records
CREATE POLICY "venues_update_own"
  ON venues FOR UPDATE
  USING (auth.uid() = user_id)
  WITH CHECK (auth.uid() = user_id);

-- DELETE: Users can only delete their own records
CREATE POLICY "venues_delete_own"
  ON venues FOR DELETE
  USING (auth.uid() = user_id);
```

Relationships:
- one_to_one to Event (foreign key: event_id)


### 3. Updated-At Automation

Create a **Trigger Function** that automatically sets `updated_at`:
- Function Name: `update_updated_at()`
- Trigger: BEFORE UPDATE on all tables
- Logic: SET NEW.updated_at = now()

Apply the trigger to all tables (profiles + entities).

### 4. Storage Setup

Create a **Storage Bucket** for file uploads:
- Name: "uploads"
- Visibility: Private
- Folder Structure: `{user_id}/{filename}`

Storage Policies:
- Users can only upload to their own folder
- Users can only read/delete their own files
- No public access

Implement folder-based security with storage.foldername().

### 5. Type Safety

Generate TypeScript types from the schema:
- Use Supabase CLI: `supabase gen types typescript`
- Export to: `lib/database.types.ts`
- Use the types for type-safe queries

---

## Validation Checklist

Verify that:
- [ ] profiles table exists with RLS
- [ ] Auto-create Profile Trigger works
- [ ] Event table exists with correct fields
- [ ] Artist table exists with correct fields
- [ ] Venue table exists with correct fields
- [ ] All tables have RLS enabled
- [ ] Foreign Keys correctly set
- [ ] Indexes created for performance
- [ ] updated_at Trigger on all tables
- [ ] Storage Bucket configured
- [ ] TypeScript Types generated

**Test:**
- Signup new user → Profile automatically created
- Create Entity record → Only visible to owner
- Update record → updated_at automatically set
- Upload file → Only possible in own folder

---

## Troubleshooting

**RLS Issues:**
- Check if policies correctly use auth.uid()
- Test with SQL Editor: `SELECT auth.uid()`
- Policies must have USING and WITH CHECK

**Foreign Key Errors:**
- Check CASCADE on user_id
- Entities with relationships: Correct order when creating

**Storage Issues:**
- Folder structure must follow {user_id}/ pattern
- Policies use storage.foldername() for user isolation

---

**Continue to:** `03-auth.md`
