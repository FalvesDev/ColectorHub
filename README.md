<p align="center">
  <img src="docs/assets/logo-placeholder.png" alt="ColectorHub Logo" width="120" />
</p>

<h1 align="center">ColectorHub</h1>

<p align="center">
  <strong>Scan your shelf. Build your collection. Own your legacy.</strong>
</p>

<p align="center">
  <a href="#-about">About</a> &bull;
  <a href="#-features">Features</a> &bull;
  <a href="#-tech-stack">Tech Stack</a> &bull;
  <a href="#-architecture">Architecture</a> &bull;
  <a href="#-screenshots">Screenshots</a> &bull;
  <a href="#-getting-started">Getting Started</a> &bull;
  <a href="#-roadmap">Roadmap</a> &bull;
  <a href="#-license">License</a>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Flutter-02569B?style=for-the-badge&logo=flutter&logoColor=white" alt="Flutter" />
  <img src="https://img.shields.io/badge/Dart-0175C2?style=for-the-badge&logo=dart&logoColor=white" alt="Dart" />
  <img src="https://img.shields.io/badge/Supabase-3FCF8E?style=for-the-badge&logo=supabase&logoColor=white" alt="Supabase" />
  <img src="https://img.shields.io/badge/Riverpod-00B4D8?style=for-the-badge&logo=dart&logoColor=white" alt="Riverpod" />
  <img src="https://img.shields.io/badge/Google%20ML%20Kit-4285F4?style=for-the-badge&logo=google&logoColor=white" alt="ML Kit" />
  <img src="https://img.shields.io/badge/Gemini%20AI-8E75B2?style=for-the-badge&logo=google&logoColor=white" alt="Gemini" />
</p>

---

## About

**ColectorHub** is a mobile app for physical game collectors. Instead of manually typing each game title, you simply take a photo of your shelf — the app uses **OCR + AI** to automatically identify game titles from spines and covers, then registers them in your digital collection with rich metadata.

Built with a focus on the **PlayStation 1** library as the initial platform, ColectorHub combines on-device text recognition (Google ML Kit), intelligent fuzzy matching, and cloud-based AI fallback (Gemini Vision) to deliver a seamless cataloging experience.

### The Problem

Collectors with hundreds of physical games have no easy way to digitize their collection. Manually searching and adding games one by one is tedious and time-consuming.

### The Solution

Point your camera at a shelf of games. ColectorHub reads the spines, identifies the titles, pulls metadata (cover art, genre, year, developer, publisher) from gaming databases, and adds everything to your organized digital collection — in seconds.

---

## Features

### Core (v1.0)

| Feature | Description |
|---|---|
| **Smart Scan** | Take a photo of game spines/covers and let AI identify titles automatically |
| **OCR Pipeline** | On-device text recognition using Google ML Kit with image preprocessing |
| **Fuzzy Matching** | Levenshtein + Jaro-Winkler algorithms match OCR text against the PS1 catalog |
| **AI Fallback** | Gemini Vision API identifies games when OCR alone isn't enough |
| **Rich Metadata** | Cover art, genre, release year, developer, publisher — all pulled from IGDB |
| **Digital Collection** | Browse your collection in grid or list view, organized by platform |
| **Game Details** | View detailed info, track condition (mint/good/fair/poor), box & manual status |
| **Manual Search** | Search the full PS1 catalog and add games manually |
| **Wishlist** | Track games you want but don't own yet |
| **Auth** | Email/password and Google Sign-In via Supabase Auth |
| **User Profile** | Display name, avatar, bio, and collection stats |

### Scan Pipeline

```
Camera Capture → Image Preprocessing → OCR (ML Kit) → Text Filtering
    → Fuzzy Matching → AI Fallback (Gemini) → User Confirmation → Save
```

---

## Tech Stack

### Frontend
| Technology | Purpose |
|---|---|
| **Flutter** | Cross-platform mobile framework |
| **Dart** | Programming language |
| **Riverpod** | State management |
| **go_router** | Declarative routing |
| **Hive** | Local cache (offline catalog) |

### Backend
| Technology | Purpose |
|---|---|
| **Supabase** | Auth, PostgreSQL database, Storage, Edge Functions |
| **PostgreSQL** | Relational database with Row Level Security |
| **Supabase Edge Functions** | Serverless functions (Deno) for secure API token management |

### AI / ML
| Technology | Purpose |
|---|---|
| **Google ML Kit** | On-device OCR (Text Recognition v2) |
| **Gemini Vision API** | Cloud AI fallback for complex image recognition |
| **Levenshtein + Jaro-Winkler** | Fuzzy string matching algorithms |

### External APIs
| API | Purpose |
|---|---|
| **IGDB** (Internet Game Database) | Game metadata, covers, genres, ratings |
| **TheGamesDB** | Regional box art (NTSC-U, PAL, NTSC-J) |

---

## Architecture

ColectorHub follows a **Feature-First + Clean Architecture** pattern:

```
lib/
├── core/            # Constants, errors, network, utilities
├── features/
│   ├── auth/        # Authentication & user profile
│   ├── collection/  # User's game collection management
│   ├── scanner/     # Camera, OCR, AI identification
│   └── catalog/     # Game database browsing & search
└── shared/          # Reusable widgets & providers
```

Each feature is organized in three layers:

```
feature/
├── data/            # Repository implementations, data sources (Supabase, APIs, Hive)
├── domain/          # Models, repository interfaces, use cases
└── presentation/    # Screens, controllers (Riverpod), widgets
```

### System Architecture

```
┌─────────────────────────────────────────────────────┐
│                  FLUTTER APP                         │
│                                                     │
│  Auth  │  Collection  │  Scanner  │  Catalog        │
│                                                     │
│              Riverpod (State Management)             │
└──────────────────────┬──────────────────────────────┘
                       │
         ┌─────────────┼──────────────────┐
         │             │                  │
         ▼             ▼                  ▼
  ┌────────────┐ ┌───────────┐ ┌──────────────────┐
  │  Supabase  │ │ ML Kit    │ │   IGDB API       │
  │  Auth      │ │ (on-device│ │   (Game metadata) │
  │  Database  │ │  OCR)     │ └──────────────────┘
  │  Storage   │ └───────────┘          │
  │  Edge Func │       │               ▼
  └────────────┘       ▼       ┌──────────────────┐
                │ Gemini Vision│ │  TheGamesDB API  │
                │ (AI fallback)│ │  (Box art)       │
                └──────────────┘ └──────────────────┘
```

---

## Screenshots

> Coming soon — the app is currently in active development.

<!--
<p align="center">
  <img src="docs/screenshots/home.png" width="200" />
  <img src="docs/screenshots/scan.png" width="200" />
  <img src="docs/screenshots/results.png" width="200" />
  <img src="docs/screenshots/detail.png" width="200" />
</p>
-->

---

## Getting Started

### Prerequisites

- [Flutter SDK](https://docs.flutter.dev/get-started/install) (>= 3.22)
- [Dart](https://dart.dev/get-dart) (>= 3.4)
- A [Supabase](https://supabase.com) project
- [Twitch Developer](https://dev.twitch.tv/console) credentials (for IGDB API)
- Android Studio / Xcode (for emulators) or a physical device

### Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/FalvesDev/ColectorHub.git
   cd ColectorHub
   ```

2. **Install dependencies**
   ```bash
   flutter pub get
   ```

3. **Set up environment variables**

   Create a `.env` file in the project root:
   ```env
   SUPABASE_URL=your_supabase_url
   SUPABASE_ANON_KEY=your_supabase_anon_key
   IGDB_CLIENT_ID=your_twitch_client_id
   GEMINI_API_KEY=your_gemini_api_key
   ```

4. **Run the app**
   ```bash
   flutter run
   ```

### Supabase Setup

The complete database schema (tables, RLS policies, triggers) is available in [`PLANEJAMENTO.md`](PLANEJAMENTO.md#7-banco-de-dados).

---

## Database Schema

| Table | Description |
|---|---|
| `profiles` | User profile data (name, avatar, bio, total games) |
| `games_cache` | Global game metadata cache from IGDB |
| `collection` | User's game collection with condition tracking |
| `wishlist` | Games the user wants to acquire |

All tables are protected with **Row Level Security (RLS)** — each user can only access their own data.

---

## Roadmap

- [x] Project planning & documentation
- [ ] **v1.0** — PS1 collection with smart scan
  - [ ] Authentication (email + Google)
  - [ ] Game catalog & manual search
  - [ ] Collection management (CRUD)
  - [ ] OCR scan pipeline
  - [ ] AI fallback (Gemini Vision)
  - [ ] Wishlist
- [ ] **v1.1** — PS2 support
- [ ] **v1.2** — PS3 & PSP support
- [ ] **v2.0** — Nintendo consoles (SNES, N64, GBA)
- [ ] **v2.1** — Market value estimates
- [ ] **v2.2** — Public collection profiles
- [ ] **v3.0** — Social features (follow collectors, feed)
- [ ] **v4.0** — Barcode / QR code scanning

---

## Project Structure

```
ColectorHub/
├── lib/
│   ├── main.dart
│   ├── app.dart
│   ├── core/
│   │   ├── constants/       # Colors, strings, API config
│   │   ├── errors/          # Failure & exception classes
│   │   ├── network/         # HTTP client, interceptors
│   │   └── utils/           # Fuzzy matcher, image processing, text filter
│   ├── features/
│   │   ├── auth/            # Login, register, profile setup
│   │   ├── collection/      # User's game collection
│   │   ├── scanner/         # Camera, OCR, AI scan
│   │   └── catalog/         # Game database & search
│   └── shared/
│       ├── widgets/         # Reusable UI components
│       └── providers/       # Global Riverpod providers
├── supabase/
│   └── functions/           # Edge Functions (token management)
├── docs/
│   ├── assets/              # Logo, banners
│   └── screenshots/         # App screenshots
├── PLANEJAMENTO.md          # Full project planning document (PT-BR)
├── MONETIZACAO.md           # Monetization & scaling strategy (PT-BR)
└── README.md                # You are here
```

---

## Security

- **Row Level Security (RLS)** on all database tables
- **IGDB token** managed server-side via Supabase Edge Functions
- **Private storage bucket** for user photos (signed URLs)
- **No secrets in code** — all sensitive data via environment variables
- **Supabase `service_role` key** never exposed to the client

---

## License

This is a proprietary project. All rights reserved. See [LICENSE](LICENSE) for details.

---

<p align="center">
  Made with dedication by <a href="https://github.com/FalvesDev">FalvesDev</a>
</p>
