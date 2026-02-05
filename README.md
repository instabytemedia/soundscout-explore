# SoundScout Explore

> Uncover Hidden Music Gems

This variant is designed for adventurous music fans, offering a gamified experience that encourages users to explore new music and discover emerging artists. SoundScout Explore uses AI-generated playlists and AR-powered scavenger hunts to guide users through a curated selection of music, rewarding them with exclusive content and virtual badges. By leveraging gamification elements, this variant makes music discovery a fun and engaging experience.

## Features

- AI-generated playlists
- AR-powered scavenger hunts
- Gamified rewards and challenges

## Tech Stack

- **Framework:** Next.js 14 (App Router)
- **Database:** Supabase (PostgreSQL)
- **Auth:** Supabase Auth
- **Styling:** Tailwind CSS
- **Language:** TypeScript

## Getting Started

1. Clone this repository
2. Copy `.env.example` to `.env.local` and fill in your credentials
3. Run `npm install`
4. Run `npm run dev`

## Project Structure

```
├── app/                  # Next.js App Router pages
├── components/           # React components
├── lib/                  # Utilities and helpers
├── supabase/            # Database schema
└── INSTRUCTIONS.md      # Detailed build guide for AI assistants
```

## Database

This project uses 3 main entities:
- **Event**: Live music event
- **Artist**: Music artist
- **Venue**: Music venue

## Build Instructions

For detailed step-by-step build instructions, see [`INSTRUCTIONS.md`](./INSTRUCTIONS.md).

This file contains comprehensive guidance for building this project with AI coding assistants like Claude Code, Cursor, or Windsurf.

---

*Generated with [Claudery](https://claudery.io) - AI-powered blueprint generator*
