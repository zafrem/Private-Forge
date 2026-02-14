English Study: System Technical Requirements Specification

  1. System Overview

  An interactive English study platform for software engineers that generates audio from
  dialogue scripts via TTS and serves them through a web-based player with multi-mode
  learning capabilities (Listening, Reading, Writing, Speaking).

  ---
  2. System Architecture

  ┌──────────────────-────────────────────────────────────────┐
  │                    Content Pipeline                       │
  │                                                           │
  │  script/*.json ──► tts_generator.py ──► site/audio/*.mp3  │
  │                         │                                 │
  │                         ▼                                 │
  │               generate_playlist.py ──► site/playlist.js   │
  └───────────────────--──────────────────────────────────────┘
                            │
                      git push main
                            │
                            ▼
  ┌─────────────────────--────────────────────────────────────┐
  │              GitHub Actions (deploy.yml)                  │
  │  Checkout ──► Upload site/ ──► Deploy to GitHub Pages     │
  └───────────────────────--──────────────────────────────────┘
                            │
                            ▼
  ┌─────────────────────────--────────────────────────────────┐
  │                   Web Application (SPA)                   │
  │                                                           │
  │  index.html + app.js + style.css + playlist.js            │
  │  ┌───────-────┐  ┌──────────────┐  ┌──────────────────┐   │
  │  │  Sidebar   │  │  Player Bar  │  │  Content Area    │   │
  │  │  - Playlist│  │  - Audio     │  │  - Script View   │   │
  │  │  - Groups  │  │  - Controls  │  │  - Writing View  │   │
  │  │  - Nav     │  │  - MediaAPI  │  │  - Calendar View │   │
  │  └────────-───┘  └──────────────┘  └──────────────────┘   │
  │                                                           │
  │  Service Worker (sw.js) ──► Offline caching (PWA)         │
  │  IndexedDB ──► Study attendance persistence               │
  └─────────────────────────────────────────--────────────────┘

  ---
  3. Component Specifications

  3.1 Script Data Format (JSON Schema)

  ┌──────────────────────────────┬────────┬──────────┬─────────────────────────────────┐
  │            Field             │  Type  │ Required │           Description           │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ title                        │ string │ Yes      │ Display title for playlist      │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ description                  │ string │ Yes      │ Short description               │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ output_filename              │ string │ Yes      │ Target MP3 filename             │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ speakers                     │ object │ Yes      │ Speaker-to-voice mapping        │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ speakers[name].language_code │ string │ Yes      │ BCP-47 code (e.g. en-US, en-AU) │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ speakers[name].voice_name    │ string │ Yes      │ Google Neural2 voice ID         │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ speakers[name].gender        │ string │ Yes      │ MALE / FEMALE / NEUTRAL         │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ dialogue                     │ array  │ Yes      │ Ordered list of utterances      │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ dialogue[].speaker           │ string │ Yes      │ Must match a key in speakers    │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ dialogue[].text              │ string │ Yes      │ The spoken text                 │
  ├──────────────────────────────┼────────┼──────────┼─────────────────────────────────┤
  │ dialogue[].section           │ string │ No       │ Optional section divider label  │
  └──────────────────────────────┴────────┴──────────┴─────────────────────────────────┘

  3.2 Audio Generation Engine (tts_generator.py)

  - Runtime: Python 3.12+
  - Dependency: google-cloud-texttospeech >= 2.14.0
  - Auth: GOOGLE_APPLICATION_CREDENTIALS env var pointing to a GCP service account JSON key
  - Output: MP3 (concatenated per-utterance chunks)
  - Options: --speaking-rate (0.25–4.0), --output dir, --publish (auto-copy to site/audio/ +
   regenerate playlist), --list-voices
  - Publish flow: Copy MP3 → update site/audio/metadata.json → invoke generate_playlist.py

  3.3 Playlist Generator (generate_playlist.py)

  - Input: Scans site/audio/ for audio files (.mp3, .wav, .ogg, .m4a, .webm)
  - Metadata resolution priority: metadata.json > matching script/*.json > auto-generated
  from filename
  - Output: site/playlist.js — a JS file exporting const PLAYLIST = [...]
  - Playlist entry fields: title, description, file, category (optional), dialogue (embedded
   from script)

  3.4 Web Application (Frontend SPA)

  ┌──────────────────┬────────────────────────────────────────────────────────┐
  │      Aspect      │                         Detail                         │
  ├──────────────────┼────────────────────────────────────────────────────────┤
  │ Framework        │ Vanilla JS (no build tools, no dependencies)           │
  ├──────────────────┼────────────────────────────────────────────────────────┤
  │ Styling          │ Single CSS file with CSS custom properties for theming │
  ├──────────────────┼────────────────────────────────────────────────────────┤
  │ Data source      │ playlist.js loaded as a global PLAYLIST constant       │
  ├──────────────────┼────────────────────────────────────────────────────────┤
  │ State management │ Module-scoped variables in app.js                      │
  ├──────────────────┼────────────────────────────────────────────────────────┤
  │ Persistence      │ localStorage (theme), IndexedDB (study attendance)     │
  └──────────────────┴────────────────────────────────────────────────────────┘

  3.4.1 Learning Modes

  ┌──────────────────┬────────────┬────────────────────────────────────────────────────────┐
  │       Mode       │ Category   │                        Behavior                        │
  │                  │    Tab     │                                                        │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Listening        │ listening  │ Audio playback + chat-style transcript bubbles,        │
  │                  │            │ auto-advance to next track                             │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Reading          │ reading    │ Clean document-style script view (no audio player bar) │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Writing — Pen &  │ writing    │ Blurred text with click-to-mark-done tracking          │
  │ Paper            │            │                                                        │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Writing — Typing │ writing    │ Blurred reference text + textarea for transcription    │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Writing — Email  │ writing    │ Scenario prompt + email editor + toggleable model      │
  │                  │            │ answer                                                 │
  ├──────────────────┼────────────┼────────────────────────────────────────────────────────┤
  │ Speaking         │ speaking   │ Category tab present, content TBD                      │
  └──────────────────┴────────────┴────────────────────────────────────────────────────────┘

  3.4.2 UI Features

  - Dark/Light theme: CSS custom properties, toggled via button, persisted in localStorage
  - Playlist grouping: Hardcoded groups (Daily Stand-ups, Netflix Interview, Amazon
  Interview, Workplace Culture, Small talk, Documents) — matched by title keyword
  - Collapsible groups: CSS max-height animation
  - Mobile responsive: Sidebar as slide-out drawer (768px breakpoint), FAB toggle button
  - Repeat mode: audio.loop toggle
  - Media Session API: Lock-screen / notification controls (title, play/pause)
  - Study Calendar: Month-view grid, click dates to toggle "studied" status, stored in
  IndexedDB

  3.5 PWA Support

  Component: manifest.json
  Detail: standalone display, portrait, icons (192/512 + maskable)
  ────────────────────────────────────────
  Component: Service Worker
  Detail: Cache-first strategy, versioned cache (study-english-v5), old cache cleanup on
    activate
  ────────────────────────────────────────
  Component: Cached assets
  Detail: index.html, style.css, app.js, playlist.js, manifest.json, logo, favicon, icons
  ────────────────────────────────────────
  Component: Note
  Detail: Audio files are NOT pre-cached (network-only)

  3.6 CI/CD Pipeline

  - Trigger: Push to main or manual workflow_dispatch
  - Process: Checkout → Upload site/ directory → Deploy to GitHub Pages
  - No build step — static files served directly

  ---
  4. External Dependencies

  Dependency: Google Cloud TTS API (Neural2)
  Purpose: Audio synthesis
  Cost: ~$16/1M chars (free tier: 1M chars/month)
  ────────────────────────────────────────
  Dependency: GCP Service Account
  Purpose: API authentication
  Cost: Free
  ────────────────────────────────────────
  Dependency: GitHub Pages
  Purpose: Static hosting
  Cost: Free
  ────────────────────────────────────────
  Dependency: GitHub Actions
  Purpose: CI/CD deployment
  Cost: Free (public repos)

  ---
  5. Directory Structure

  project-root/
  ├── .env.example              # GCP credentials template
  ├── .github/workflows/
  │   └── deploy.yml            # GitHub Pages deployment
  ├── Makefile                   # refresh / clean shortcuts
  ├── requirements.txt           # Python deps (google-cloud-texttospeech)
  ├── script/                    # Source dialogue JSONs
  │   ├── template.json          # Schema reference
  │   └── *.json                 # Individual scripts
  ├── src/                       # Backend tooling
  │   ├── tts_generator.py       # Audio generation + publish
  │   ├── generate_playlist.py   # Playlist.js generator
  │   ├── create_silent_mp3.py   # Placeholder audio utility
  │   ├── generate_icons.py      # PWA icon generator
  │   └── cleanup_metadata.py    # Metadata maintenance
  ├── site/                      # Deployed web root
  │   ├── index.html             # SPA entry point
  │   ├── app.js                 # Application logic (~360 LOC)
  │   ├── style.css              # Styles (~915 LOC)
  │   ├── playlist.js            # Auto-generated data
  │   ├── sw.js                  # Service Worker
  │   ├── manifest.json          # PWA manifest
  │   ├── audio/                 # Generated MP3 files
  │   │   └── metadata.json      # Audio metadata overrides
  │   └── icons/                 # PWA icons
  └── images/                    # Static image assets

  ---
  6. Additional Information Needed to Build From Scratch

  To fully recreate this system, you would still need the following:

  Content & Design Assets

  1. Logo and favicon images (logo.jpg, favicon.jpg) — the brand assets
  2. PWA icons (192px, 512px, maskable variants) — or a source image + generate_icons.py
  3. Dialogue scripts — the actual JSON content files that drive the entire platform

  Configuration & Credentials

  4. Google Cloud project with Text-to-Speech API enabled
  5. GCP service account JSON key for TTS authentication
  6. GitHub repository configured with Pages (Source: GitHub Actions)

  Design Decisions Not Captured in Code

  7. Playlist grouping logic — currently hardcoded by keyword matching in app.js:231-242.
  What groups should exist and how should items be categorized?
  8. Speaker bubble alignment rules — which speakers render as "self" (right-aligned, accent
   color) vs "other" (left-aligned)? Currently hardcoded as CSS class lists in
  style.css:417-431
  9. Category assignment — how are scripts assigned to listening / reading / writing /
  speaking? Currently via optional category field in metadata.json, defaulting to listening
  10. Speaking mode — the tab exists but has no implementation yet. What is the intended
  feature?
  11. Voice selection strategy — which Google Neural2 voices to assign to which speaker
  archetypes (interviewer, colleague, etc.)

  Operational

  12. Cache versioning strategy — when to bump CACHE_NAME in sw.js
  13. Audio file size budget — no compression or duration limits are defined
