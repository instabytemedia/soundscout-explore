# 06 - Entity CRUD

> **Agent:** Entities Agent 📦
> **Goal:** Create complete CRUD for all Entities

---

## Context

**Product:** SoundScout Explore
**Entities:** Event, Artist, Venue
**Entity Count:** 3

---

## Overview

Per Entity will be created:
- 1x TypeScript Types File
- 1x Zod Schema File
- 2x API Routes (List/Create + Detail)
- 1x SWR Hooks File
- 2x Pages (List + Detail)
- 2x Components (List + Detail)

**Total Files per Entity:** 9
**Total Files overall:** 27

| Entity | Table | API Endpoints | Pages |
|--------|-------|---------------|-------|
| Event | events | /api/events | /events, /events/[id] |
| Artist | artists | /api/artists | /artists, /artists/[id] |
| Venue | venues | /api/venues | /venues, /venues/[id] |

---


---

## Entity 1: Event

**Description:** Live music event
**Table Name:** `events`

**Fields:**
- `id`: uuid - Primary key
- `created_at`: datetime - Creation timestamp
- `updated_at`: datetime - Last update timestamp
- `user_id`: uuid - Owner user ID
- `name`: string - Event name
- `description`: text (optional) - Event description
- `location`: string - Event location
- `start_time`: datetime - Event start time
- `end_time`: datetime - Event end time

**Relationships:**
- many_to_many → Artist: An event can have multiple artists, and an artist can perform at multiple events
- one_to_one → Venue: An event is held at one venue

**Related Flows:**
- **Event Creation:** Create event → Add event details
- **Event Discovery:** Search events → View event details

**API Endpoints:**
- `POST /api/events`: Create a new live music event
- `GET /api/events`: Get a list of live music events
- `GET /api/events/{id}`: Get live music event details
- `PUT /api/events/{id}`: Update live music event details

### Instructions: Event

#### 1. TypeScript Types

Create **Types File** (`types/event.ts`):

**⚠️ CRITICAL: No Duplicate Fields!**
Each field must appear EXACTLY ONCE. Use consistent naming (snake_case for DB fields).

```typescript
// ✅ CORRECT: Each field once
export interface Event {
  id: string;
  user_id: string;
  created_at: string;
  updated_at: string;
  // Custom fields:
  name: string;
  description: string;
  location: string;
  start_time: string;
  end_time: string;
}

// ❌ WRONG: Duplicate fields!
export interface Event {
  id: string;        // 1x
  userId: string;    // mixed camelCase
  created_at: Date;  // wrong type (use string)
  user_id: string;   // DUPLICATE (snake_case)!
  id: string;        // DUPLICATE!
}
```

**EventListItem Interface:**
- Reduced version for List Views
- Only essential fields: id, created_at + first 3 custom fields

#### 2. Zod Validation Schema

Create **Schema File** (`lib/schemas/event.ts`):

**CreateEventSchema:**
- Zod Object with all custom fields
- Required vs Optional based on Field Definition
- Type Mapping:
  - string/text → z.string()
  - number/integer → z.number() / z.number().int()
  - boolean → z.boolean()
  - date/datetime → z.string() (ISO format)

**UpdateEventSchema:**
- Use Create Schema with .partial()
- All fields optional for updates

**Exports:**
- Schemas + Inferred Types

#### 3. API Routes - List & Create

Create **List/Create Route** (`app/api/events/route.ts`):

**GET /api/events (List):**
- Auth Check: User from Supabase
- Query Params: limit (default 20), cursor (pagination)
- Supabase Query:
  - Filter: user_id = current user
  - Order: created_at DESC
  - Limit: limit + 1 (for hasMore detection)
  - Cursor: created_at < cursor (if provided)
- Response: { data: items, meta: { next_cursor, has_more } }
- Error Handling: 401 (unauthorized), 500 (server error)

**POST /api/events (Create):**
- Auth Check: User from Supabase
- Body Validation: CreateEventSchema with safeParse
- Insert: Merge validated data + user_id
- Select: .select().single() for Created Object
- Response: { data } with 201 Status
- Error Handling: 400 (validation), 500 (server error)

#### 4. API Routes - Detail Operations

Create **Detail Route** (`app/api/events/[id]/route.ts`):

**GET /api/events/[id] (Read):**
- Params: id (await params promise)
- Auth Check: User from Supabase
- Query: .select().eq("id", id).eq("user_id", user.id).single()
- Response: { data }
- Error: 404 if not found

**PATCH /api/events/[id] (Update):**
- Params: id (await params promise)
- Auth Check + Body Validation
- Update: .update().eq("id").eq("user_id").select().single()
- Response: { data }
- Error: 400 (validation), 404 (not found)

**DELETE /api/events/[id] (Delete):**
- Params: id (await params promise)
- Auth Check
- Delete: .delete().eq("id").eq("user_id")
- Response: 204 No Content
- Error: 404 (not found)

**Important for all Routes:**
- User ID Check prevents access to other users' data
- NextRequest/NextResponse for Types
- await params as Promise (Next.js 15)

#### 5. SWR Data Fetching Hooks

Create **Hooks File** (`hooks/useEvents.ts`):

**useEvents() Hook:**
- SWR for List Fetching
- URL: /api/events
- Returns: { events, isLoading, error, mutate }

**useEvent(id) Hook:**
- SWR for Single Item Fetching
- URL: /api/events/{id}
- Conditional: only fetch if id is provided
- Returns: { event, isLoading, error, mutate }

**useCreateEvent() Hook:**
- POST Request Function
- Mutate List after Success
- Error Handling with throw

**useUpdateEvent(id) Hook:**
- PATCH Request Function
- Mutate Item + List after Success
- Error Handling with throw

**useDeleteEvent() Hook:**
- DELETE Request Function
- Mutate List after Success
- Error Handling with throw

**Important:**
- All Hooks are Client Components ("use client")
- Fetcher Function for SWR
- Proper Error Handling

#### 6. List Page

Create **List Page** (`app/(app)/events/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Client Component for List Rendering

**Layout:**
- Container with Padding
- Header: Title + "New" Button (right)
- List Component Integration

#### 7. List Component

Create **List Component** (`components/event/EventList.tsx`):

**States:**
- Loading: Skeleton Grid (6 items)
- Error: Error Message Card
- Empty: EmptyState Component with "Create New" Action
- Success: Grid with Items

**Grid Layout:**
- Responsive: 1 col (mobile), 2 col (tablet), 3 col (desktop)
- Gap: gap-4

**Item Cards:**
- Link to Detail Page
- Hover Effect: border-primary
- Content: Title + Created Date (adapt to Entity fields)

**Important:**
- "use client" Directive
- useEvents() Hook for Data
- Proper Loading/Error/Empty States

#### 8. Detail Page

Create **Detail Page** (`app/(app)/events/[id]/page.tsx`):

**Functionality:**
- Server Component
- Auth Check
- Direct Supabase Fetch for SSR
- notFound() if Item doesn't exist
- Pass Data to Detail Component

**Security:**
- user_id Check in Query

#### 9. Detail Component

Create **Detail Component** (`components/event/EventDetail.tsx`):

**Props:**
- `event`: Event Object

**Layout:**
- Back Button (top left)
- Delete Button (top right)
- Card with Details:
  - ID
  - All Custom Fields (from Entity Definition)
  - Created/Updated Timestamps

**Delete Functionality:**
- Confirm Dialog
- useDeleteEvent() Hook
- Router Push to List after Success
- Loading State during Delete

**Important:**
- "use client" Directive
- Router for Navigation
- Confirmation before Delete



---

## Entity 2: Artist

**Description:** Music artist
**Table Name:** `artists`

**Fields:**
- `id`: uuid - Primary key
- `created_at`: datetime - Creation timestamp
- `updated_at`: datetime - Last update timestamp
- `user_id`: uuid - Owner user ID
- `name`: string - Artist name
- `genre`: string (optional) - Artist genre

**Relationships:**
- many_to_many → Event: An artist can perform at multiple events, and an event can have multiple artists





### Instructions: Artist

#### 1. TypeScript Types

Create **Types File** (`types/artist.ts`):

**⚠️ CRITICAL: No Duplicate Fields!**
Each field must appear EXACTLY ONCE. Use consistent naming (snake_case for DB fields).

```typescript
// ✅ CORRECT: Each field once
export interface Artist {
  id: string;
  user_id: string;
  created_at: string;
  updated_at: string;
  // Custom fields:
  name: string;
  genre: string;
}

// ❌ WRONG: Duplicate fields!
export interface Artist {
  id: string;        // 1x
  userId: string;    // mixed camelCase
  created_at: Date;  // wrong type (use string)
  user_id: string;   // DUPLICATE (snake_case)!
  id: string;        // DUPLICATE!
}
```

**ArtistListItem Interface:**
- Reduced version for List Views
- Only essential fields: id, created_at + first 3 custom fields

#### 2. Zod Validation Schema

Create **Schema File** (`lib/schemas/artist.ts`):

**CreateArtistSchema:**
- Zod Object with all custom fields
- Required vs Optional based on Field Definition
- Type Mapping:
  - string/text → z.string()
  - number/integer → z.number() / z.number().int()
  - boolean → z.boolean()
  - date/datetime → z.string() (ISO format)

**UpdateArtistSchema:**
- Use Create Schema with .partial()
- All fields optional for updates

**Exports:**
- Schemas + Inferred Types

#### 3. API Routes - List & Create

Create **List/Create Route** (`app/api/artists/route.ts`):

**GET /api/artists (List):**
- Auth Check: User from Supabase
- Query Params: limit (default 20), cursor (pagination)
- Supabase Query:
  - Filter: user_id = current user
  - Order: created_at DESC
  - Limit: limit + 1 (for hasMore detection)
  - Cursor: created_at < cursor (if provided)
- Response: { data: items, meta: { next_cursor, has_more } }
- Error Handling: 401 (unauthorized), 500 (server error)

**POST /api/artists (Create):**
- Auth Check: User from Supabase
- Body Validation: CreateArtistSchema with safeParse
- Insert: Merge validated data + user_id
- Select: .select().single() for Created Object
- Response: { data } with 201 Status
- Error Handling: 400 (validation), 500 (server error)

#### 4. API Routes - Detail Operations

Create **Detail Route** (`app/api/artists/[id]/route.ts`):

**GET /api/artists/[id] (Read):**
- Params: id (await params promise)
- Auth Check: User from Supabase
- Query: .select().eq("id", id).eq("user_id", user.id).single()
- Response: { data }
- Error: 404 if not found

**PATCH /api/artists/[id] (Update):**
- Params: id (await params promise)
- Auth Check + Body Validation
- Update: .update().eq("id").eq("user_id").select().single()
- Response: { data }
- Error: 400 (validation), 404 (not found)

**DELETE /api/artists/[id] (Delete):**
- Params: id (await params promise)
- Auth Check
- Delete: .delete().eq("id").eq("user_id")
- Response: 204 No Content
- Error: 404 (not found)

**Important for all Routes:**
- User ID Check prevents access to other users' data
- NextRequest/NextResponse for Types
- await params as Promise (Next.js 15)

#### 5. SWR Data Fetching Hooks

Create **Hooks File** (`hooks/useArtists.ts`):

**useArtists() Hook:**
- SWR for List Fetching
- URL: /api/artists
- Returns: { artists, isLoading, error, mutate }

**useArtist(id) Hook:**
- SWR for Single Item Fetching
- URL: /api/artists/{id}
- Conditional: only fetch if id is provided
- Returns: { artist, isLoading, error, mutate }

**useCreateArtist() Hook:**
- POST Request Function
- Mutate List after Success
- Error Handling with throw

**useUpdateArtist(id) Hook:**
- PATCH Request Function
- Mutate Item + List after Success
- Error Handling with throw

**useDeleteArtist() Hook:**
- DELETE Request Function
- Mutate List after Success
- Error Handling with throw

**Important:**
- All Hooks are Client Components ("use client")
- Fetcher Function for SWR
- Proper Error Handling

#### 6. List Page

Create **List Page** (`app/(app)/artists/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Client Component for List Rendering

**Layout:**
- Container with Padding
- Header: Title + "New" Button (right)
- List Component Integration

#### 7. List Component

Create **List Component** (`components/artist/ArtistList.tsx`):

**States:**
- Loading: Skeleton Grid (6 items)
- Error: Error Message Card
- Empty: EmptyState Component with "Create New" Action
- Success: Grid with Items

**Grid Layout:**
- Responsive: 1 col (mobile), 2 col (tablet), 3 col (desktop)
- Gap: gap-4

**Item Cards:**
- Link to Detail Page
- Hover Effect: border-primary
- Content: Title + Created Date (adapt to Entity fields)

**Important:**
- "use client" Directive
- useArtists() Hook for Data
- Proper Loading/Error/Empty States

#### 8. Detail Page

Create **Detail Page** (`app/(app)/artists/[id]/page.tsx`):

**Functionality:**
- Server Component
- Auth Check
- Direct Supabase Fetch for SSR
- notFound() if Item doesn't exist
- Pass Data to Detail Component

**Security:**
- user_id Check in Query

#### 9. Detail Component

Create **Detail Component** (`components/artist/ArtistDetail.tsx`):

**Props:**
- `artist`: Artist Object

**Layout:**
- Back Button (top left)
- Delete Button (top right)
- Card with Details:
  - ID
  - All Custom Fields (from Entity Definition)
  - Created/Updated Timestamps

**Delete Functionality:**
- Confirm Dialog
- useDeleteArtist() Hook
- Router Push to List after Success
- Loading State during Delete

**Important:**
- "use client" Directive
- Router for Navigation
- Confirmation before Delete



---

## Entity 3: Venue

**Description:** Music venue
**Table Name:** `venues`

**Fields:**
- `id`: uuid - Primary key
- `created_at`: datetime - Creation timestamp
- `updated_at`: datetime - Last update timestamp
- `user_id`: uuid - Owner user ID
- `name`: string - Venue name
- `location`: string - Venue location

**Relationships:**
- one_to_one → Event: A venue can host one event at a time





### Instructions: Venue

#### 1. TypeScript Types

Create **Types File** (`types/venue.ts`):

**⚠️ CRITICAL: No Duplicate Fields!**
Each field must appear EXACTLY ONCE. Use consistent naming (snake_case for DB fields).

```typescript
// ✅ CORRECT: Each field once
export interface Venue {
  id: string;
  user_id: string;
  created_at: string;
  updated_at: string;
  // Custom fields:
  name: string;
  location: string;
}

// ❌ WRONG: Duplicate fields!
export interface Venue {
  id: string;        // 1x
  userId: string;    // mixed camelCase
  created_at: Date;  // wrong type (use string)
  user_id: string;   // DUPLICATE (snake_case)!
  id: string;        // DUPLICATE!
}
```

**VenueListItem Interface:**
- Reduced version for List Views
- Only essential fields: id, created_at + first 3 custom fields

#### 2. Zod Validation Schema

Create **Schema File** (`lib/schemas/venue.ts`):

**CreateVenueSchema:**
- Zod Object with all custom fields
- Required vs Optional based on Field Definition
- Type Mapping:
  - string/text → z.string()
  - number/integer → z.number() / z.number().int()
  - boolean → z.boolean()
  - date/datetime → z.string() (ISO format)

**UpdateVenueSchema:**
- Use Create Schema with .partial()
- All fields optional for updates

**Exports:**
- Schemas + Inferred Types

#### 3. API Routes - List & Create

Create **List/Create Route** (`app/api/venues/route.ts`):

**GET /api/venues (List):**
- Auth Check: User from Supabase
- Query Params: limit (default 20), cursor (pagination)
- Supabase Query:
  - Filter: user_id = current user
  - Order: created_at DESC
  - Limit: limit + 1 (for hasMore detection)
  - Cursor: created_at < cursor (if provided)
- Response: { data: items, meta: { next_cursor, has_more } }
- Error Handling: 401 (unauthorized), 500 (server error)

**POST /api/venues (Create):**
- Auth Check: User from Supabase
- Body Validation: CreateVenueSchema with safeParse
- Insert: Merge validated data + user_id
- Select: .select().single() for Created Object
- Response: { data } with 201 Status
- Error Handling: 400 (validation), 500 (server error)

#### 4. API Routes - Detail Operations

Create **Detail Route** (`app/api/venues/[id]/route.ts`):

**GET /api/venues/[id] (Read):**
- Params: id (await params promise)
- Auth Check: User from Supabase
- Query: .select().eq("id", id).eq("user_id", user.id).single()
- Response: { data }
- Error: 404 if not found

**PATCH /api/venues/[id] (Update):**
- Params: id (await params promise)
- Auth Check + Body Validation
- Update: .update().eq("id").eq("user_id").select().single()
- Response: { data }
- Error: 400 (validation), 404 (not found)

**DELETE /api/venues/[id] (Delete):**
- Params: id (await params promise)
- Auth Check
- Delete: .delete().eq("id").eq("user_id")
- Response: 204 No Content
- Error: 404 (not found)

**Important for all Routes:**
- User ID Check prevents access to other users' data
- NextRequest/NextResponse for Types
- await params as Promise (Next.js 15)

#### 5. SWR Data Fetching Hooks

Create **Hooks File** (`hooks/useVenues.ts`):

**useVenues() Hook:**
- SWR for List Fetching
- URL: /api/venues
- Returns: { venues, isLoading, error, mutate }

**useVenue(id) Hook:**
- SWR for Single Item Fetching
- URL: /api/venues/{id}
- Conditional: only fetch if id is provided
- Returns: { venue, isLoading, error, mutate }

**useCreateVenue() Hook:**
- POST Request Function
- Mutate List after Success
- Error Handling with throw

**useUpdateVenue(id) Hook:**
- PATCH Request Function
- Mutate Item + List after Success
- Error Handling with throw

**useDeleteVenue() Hook:**
- DELETE Request Function
- Mutate List after Success
- Error Handling with throw

**Important:**
- All Hooks are Client Components ("use client")
- Fetcher Function for SWR
- Proper Error Handling

#### 6. List Page

Create **List Page** (`app/(app)/venues/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Client Component for List Rendering

**Layout:**
- Container with Padding
- Header: Title + "New" Button (right)
- List Component Integration

#### 7. List Component

Create **List Component** (`components/venue/VenueList.tsx`):

**States:**
- Loading: Skeleton Grid (6 items)
- Error: Error Message Card
- Empty: EmptyState Component with "Create New" Action
- Success: Grid with Items

**Grid Layout:**
- Responsive: 1 col (mobile), 2 col (tablet), 3 col (desktop)
- Gap: gap-4

**Item Cards:**
- Link to Detail Page
- Hover Effect: border-primary
- Content: Title + Created Date (adapt to Entity fields)

**Important:**
- "use client" Directive
- useVenues() Hook for Data
- Proper Loading/Error/Empty States

#### 8. Detail Page

Create **Detail Page** (`app/(app)/venues/[id]/page.tsx`):

**Functionality:**
- Server Component
- Auth Check
- Direct Supabase Fetch for SSR
- notFound() if Item doesn't exist
- Pass Data to Detail Component

**Security:**
- user_id Check in Query

#### 9. Detail Component

Create **Detail Component** (`components/venue/VenueDetail.tsx`):

**Props:**
- `venue`: Venue Object

**Layout:**
- Back Button (top left)
- Delete Button (top right)
- Card with Details:
  - ID
  - All Custom Fields (from Entity Definition)
  - Created/Updated Timestamps

**Delete Functionality:**
- Confirm Dialog
- useDeleteVenue() Hook
- Router Push to List after Success
- Loading State during Delete

**Important:**
- "use client" Directive
- Router for Navigation
- Confirmation before Delete


---

## Validation Checklist

Verify for EACH Entity:
- [ ] Types File created with correct Interfaces
- [ ] Zod Schema File with Create + Update Schemas
- [ ] API List/Create Route works
- [ ] API Detail Route works (GET, PATCH, DELETE)
- [ ] SWR Hooks File with all 5 Hooks
- [ ] List Page with Auth Check
- [ ] List Component with Loading/Error/Empty States
- [ ] Detail Page with Auth Check
- [ ] Detail Component with Delete Functionality

**Test per Entity:**
1. **Create:** Go to /events/new → Create Item → Redirect to List
2. **List:** List shows new Item
3. **Read:** Click on Item → Detail Page loads
4. **Update:** (built in Phase 08-forms)
5. **Delete:** Delete Button → Confirmation → Back to List

**API Tests:**
- POST /api/[entity]s → 201 with Created Item
- GET /api/[entity]s → 200 with Array
- GET /api/[entity]s/[id] → 200 with Object
- PATCH /api/[entity]s/[id] → 200 with Updated Item
- DELETE /api/[entity]s/[id] → 204

---

## Troubleshooting

**API 401 Errors:**
- Check Auth Check in all Routes
- Supabase Client correctly created?
- User is correctly retrieved?

**RLS Policy Errors:**
- Check if user_id is in Query
- RLS Policies active in Supabase?

**Pagination not working:**
- Cursor Logic: created_at < cursor
- hasMore Detection: length > limit

**SWR not updating:**
- Call mutate() after Mutations
- Fetcher Function correct?

**TypeScript Errors:**
- Types from database.types.ts generated?
- Zod Schemas export inferred Types

---

**Continue to:** `07-dashboard.md`
