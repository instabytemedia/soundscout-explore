# 08 - Forms & Validation

> **Agent:** Forms Agent 📝
> **Goal:** Create forms with react-hook-form + Zod

---

## Context

**Product:** SoundScout Explore
**Entities with Forms:** Event, Artist, Venue

---

## Instructions

### 1. Check Dependencies

Ensure the following dependencies are installed:
- `react-hook-form` - Form State Management
- `@hookform/resolvers` - Zod Integration

If not present, install them.

**Important:**
- Zod should already be installed from Setup
- Schemas should already exist in lib/schemas/ (from Phase 06)


### 2. Event Form

Create **EventForm Component** (`components/event/EventForm.tsx`):

**Props Interface:**
- `event?`: Event Object (if provided = Edit Mode)
- `onSuccess?`: Callback after successful submit

**Functionality:**
- Client Component ("use client")
- react-hook-form with zodResolver
- Use CreateEventSchema for Validation
- Distinguish Create vs Edit Mode (based on event prop)
- useCreateEvent() Hook for Create
- useUpdateEvent(id) Hook for Edit
- Loading State during Submit
- Error Handling with Alert
- After Success: onSuccess Callback + Router Push to List + Router Refresh

**Form Fields:**
- `id` (text) - Required: Primary key
- `created_at` (text) - Required: Creation timestamp
- `updated_at` (text) - Required: Last update timestamp
- `user_id` (text) - Required: Owner user ID
- `name` (text) - Required: Event name
- `description` (text) - Optional: Event description
- `location` (text) - Required: Event location
- `start_time` (text) - Required: Event start time
- `end_time` (text) - Required: Event end time

**Layout:**
- Card Container
- CardHeader: Title ("New Event" or "Edit Event")
- CardContent: Form Fields (space-y-4)
- CardFooter: Actions (justify-end gap-2)
  - Cancel Button (variant="outline", onClick → router.back())
  - Submit Button (type="submit", disabled during Loading)

**Field Layout (per field):**
- Container with space-y-2
- Label with htmlFor
- Input with register()
- Error Message (if errors.fieldName)

**Important:**
- register() with valueAsNumber for Number fields
- defaultValues from event prop (for Edit Mode)
- Proper Error Display under each field

---

Create **New Event Page** (`app/(app)/events/new/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Render EventForm Component

**Layout:**
- Container with max-w-2xl + py-8
- Centered Form Layout



### 3. Artist Form

Create **ArtistForm Component** (`components/artist/ArtistForm.tsx`):

**Props Interface:**
- `artist?`: Artist Object (if provided = Edit Mode)
- `onSuccess?`: Callback after successful submit

**Functionality:**
- Client Component ("use client")
- react-hook-form with zodResolver
- Use CreateArtistSchema for Validation
- Distinguish Create vs Edit Mode (based on artist prop)
- useCreateArtist() Hook for Create
- useUpdateArtist(id) Hook for Edit
- Loading State during Submit
- Error Handling with Alert
- After Success: onSuccess Callback + Router Push to List + Router Refresh

**Form Fields:**
- `id` (text) - Required: Primary key
- `created_at` (text) - Required: Creation timestamp
- `updated_at` (text) - Required: Last update timestamp
- `user_id` (text) - Required: Owner user ID
- `name` (text) - Required: Artist name
- `genre` (text) - Optional: Artist genre

**Layout:**
- Card Container
- CardHeader: Title ("New Artist" or "Edit Artist")
- CardContent: Form Fields (space-y-4)
- CardFooter: Actions (justify-end gap-2)
  - Cancel Button (variant="outline", onClick → router.back())
  - Submit Button (type="submit", disabled during Loading)

**Field Layout (per field):**
- Container with space-y-2
- Label with htmlFor
- Input with register()
- Error Message (if errors.fieldName)

**Important:**
- register() with valueAsNumber for Number fields
- defaultValues from artist prop (for Edit Mode)
- Proper Error Display under each field

---

Create **New Artist Page** (`app/(app)/artists/new/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Render ArtistForm Component

**Layout:**
- Container with max-w-2xl + py-8
- Centered Form Layout



### 4. Venue Form

Create **VenueForm Component** (`components/venue/VenueForm.tsx`):

**Props Interface:**
- `venue?`: Venue Object (if provided = Edit Mode)
- `onSuccess?`: Callback after successful submit

**Functionality:**
- Client Component ("use client")
- react-hook-form with zodResolver
- Use CreateVenueSchema for Validation
- Distinguish Create vs Edit Mode (based on venue prop)
- useCreateVenue() Hook for Create
- useUpdateVenue(id) Hook for Edit
- Loading State during Submit
- Error Handling with Alert
- After Success: onSuccess Callback + Router Push to List + Router Refresh

**Form Fields:**
- `id` (text) - Required: Primary key
- `created_at` (text) - Required: Creation timestamp
- `updated_at` (text) - Required: Last update timestamp
- `user_id` (text) - Required: Owner user ID
- `name` (text) - Required: Venue name
- `location` (text) - Required: Venue location

**Layout:**
- Card Container
- CardHeader: Title ("New Venue" or "Edit Venue")
- CardContent: Form Fields (space-y-4)
- CardFooter: Actions (justify-end gap-2)
  - Cancel Button (variant="outline", onClick → router.back())
  - Submit Button (type="submit", disabled during Loading)

**Field Layout (per field):**
- Container with space-y-2
- Label with htmlFor
- Input with register()
- Error Message (if errors.fieldName)

**Important:**
- register() with valueAsNumber for Number fields
- defaultValues from venue prop (for Edit Mode)
- Proper Error Display under each field

---

Create **New Venue Page** (`app/(app)/venues/new/page.tsx`):

**Functionality:**
- Server Component for Auth Check
- Redirect to /login if not authenticated
- Render VenueForm Component

**Layout:**
- Container with max-w-2xl + py-8
- Centered Form Layout


---

## Validation Checklist

Verify for EACH Entity:
- [ ] Form Component created
- [ ] New Page created with Auth Check
- [ ] react-hook-form integrated
- [ ] Zod Validation works
- [ ] Create Mode works
- [ ] Edit Mode works (when event is passed)
- [ ] Loading State during Submit
- [ ] Error Messages are displayed
- [ ] Redirect after Success

**Test per Entity:**
1. **Create Test:**
   - Go to /[entity]s/new
   - Leave field empty → Validation Error displayed
   - Fill all fields → Submit
   - Redirected to list
   - New entry appears in list

2. **Validation Test:**
   - Required fields empty → Error
   - Number field with text → Error (if applicable)
   - Correct input → No Error

3. **Edit Test (if Detail Page exists):**
   - Open Entity Detail
   - Click Edit
   - Fields are pre-filled
   - Change value → Submit
   - Changes saved

---

## Troubleshooting

**Form submitted but nothing happens:**
- handleSubmit correctly passed to form onSubmit?
- async/await in onSubmit Handler?
- Errors being logged?

**Validation not working:**
- zodResolver correctly imported?
- Schema correctly passed to zodResolver?
- Schema exported from lib/schemas?

**Register not working:**
- Field Name exactly like in Schema?
- Input has {...register("fieldName")} spread?
- For Number: valueAsNumber Option?

**defaultValues being ignored:**
- useForm with defaultValues option?
- event prop being passed?
- Object spread correct?

**No redirect after Submit:**
- router.push() called?
- router.refresh() for SSR Update?
- No Error in try/catch?

**TypeScript Errors:**
- CreateEventSchema imported?
- Type Inference from Zod: type CreateEvent = z.infer<typeof CreateEventSchema>
- useForm Generic: useForm<CreateEvent>()

---

**Continue to:** `09-landing.md`
