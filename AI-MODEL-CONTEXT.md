# AI Model Context — Hemanth Portfolio

> This file provides a complete architectural overview for AI models to understand, modify, or extend this codebase efficiently.

---

## Project Overview

A dark-themed portfolio website for **Hemanth Kumar Chittiprolu**, an AI/ML Engineer. Features GSAP scroll animations, a LiveKit-powered AI avatar chat, and is optimized for mobile-first deployment on Vercel.

**Live URL**: Deploy to Vercel  
**Tech Stack**: Next.js 16 + TypeScript + Tailwind CSS 4 + GSAP 3 + LiveKit  
**Theme**: Dark (#0a0f1a base, emerald-400/teal accents)

---

## File Architecture

```
src/
├── app/
│   ├── layout.tsx          # Root layout — Inter font, JetBrains Mono, dark class
│   ├── page.tsx            # Homepage — orchestrates loading screen + all sections
│   ├── globals.css         # CSS variables (dark theme), scrollbar, selection styles
│   └── api/
│       └── session/
│           └── route.ts    # LiveKit room CRUD (POST create, DELETE teardown)
├── components/
│   ├── portfolio/
│   │   ├── loading-screen.tsx   # Full-screen loader with GSAP progress + cycling text
│   │   ├── navbar.tsx           # Sticky glass nav with mobile drawer
│   │   ├── hero.tsx             # Hero with GSAP split-line reveal + stats
│   │   ├── about.tsx            # Bio + Experience + Education + Achievements
│   │   ├── skills.tsx           # Tabbed skill categories with GSAP stagger tags
│   │   ├── projects.tsx         # 6 project cards with scroll-triggered stagger
│   │   ├── ai-chat.tsx          # LiveKit avatar (LEFT) + Chat panel (RIGHT)
│   │   ├── contact.tsx          # Contact form + social links
│   │   └── footer.tsx           # Minimal footer
│   └── ui/                      # shadcn/ui components (unchanged)
├── lib/
│   ├── data.ts             # ALL portfolio content — edit here to update text
│   ├── gsap-utils.ts       # GSAP hooks, presets (fadeInUp, staggerReveal, splitTextReveal)
│   ├── livekit.ts          # LiveKit server config, token generation
│   └── utils.ts            # Tailwind merge utility (cn)
└── hooks/
    ├── use-mobile.ts       # Mobile detection hook
    └── use-toast.ts        # Toast notification hook
```

---

## Key Design Decisions

### 1. Data Separation (`src/lib/data.ts`)
All portfolio content (name, bio, skills, projects, achievements) lives in a single data file. To update any text, edit ONLY this file. Components import from here.

### 2. GSAP Animation Strategy
- **Page load**: `LoadingScreen` plays a 2.2s progress animation, then fades out
- **Hero**: Split-line name reveal, staggered stats (fires after loader completes at ~2.5s delay)
- **Sections**: Each uses `ScrollTrigger` with `once: true` — animates on first viewport entry
- **Skill tabs**: Tags re-animate with `gsap.fromTo()` on category change
- **Cleanup**: All GSAP contexts are reverted on component unmount

### 3. AI Avatar Chat Layout
```
┌──────────────────────────────────────────┐
│ Avatar (LEFT)  │  Chat Panel (RIGHT)     │
│ - Video feed   │  - Messages with bot/   │
│ - Bot fallback │    user avatars         │
│ - Status light │  - Typing indicator     │
│ - Session ctrl │  - Input + send button  │
└──────────────────────────────────────────┘
```
On mobile: stacks vertically (avatar on top, chat below).

### 4. LiveKit Session Lifecycle
- Sessions only start on user click (no auto-connect)
- Room created via `POST /api/session` → returns token + wsUrl
- Auto-disconnect: 3min idle, 45s tab hidden, 20s warning countdown
- Room deleted via `DELETE /api/session?roomName=xxx`
- Requires env vars: `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, `LIVEKIT_URL`

### 5. Mobile-First Responsive
- Breakpoints: `sm:640`, `md:768`, `lg:1024`, `xl:1280`
- Touch targets: minimum 44px
- Chat panel: `min-h-[400px]` on mobile
- Skills: compact "all skills" view shown on mobile below tabs
- Navbar: hamburger drawer on mobile

---

## Environment Variables

```env
# LiveKit (required for AI Chat feature)
LIVEKIT_API_KEY=your_key
LIVEKIT_API_SECRET=your_secret
LIVEKIT_URL=wss://your-domain.livekit.cloud

# Python backend (separate deployment)
DEEPGRAM_API_KEY=your_key
OPENAI_API_KEY=your_key
OPENROUTER_API_KEY=your_key
ELEVEN_API_KEY=your_key
ELEVEN_VOICE_ID=your_voice_id
SIMLI_API_KEY=your_key
SIMLI_FACE_ID=your_face_id
```

---

## Common Modification Tasks

### Update portfolio content
Edit `src/lib/data.ts` — all text is there.

### Change the color theme
Edit `src/app/globals.css` — modify the `:root` and `.dark` CSS variable blocks. The accent color is emerald-based (oklch hue ~165).

### Add a new section
1. Add data to `src/lib/data.ts`
2. Create component in `src/components/portfolio/new-section.tsx`
3. Import and render in `src/app/page.tsx`
4. Add nav link in `data.ts` → `NAV_LINKS`

### Change GSAP animations
- Section scroll animations: each component self-registers `ScrollTrigger` in its `useEffect`
- Global presets: edit `src/lib/gsap-utils.ts`
- Loading animation: edit `src/components/portfolio/loading-screen.tsx`

### Fix AI Chat issues
- Connection errors: check LiveKit env vars are set
- Chat not sending: ensure `status === "live"` before calling `room.localParticipant.sendText()`
- Video not appearing: check that `AVATAR_IDENTITY = "avatar_worker"` matches the Python backend

---

## Running the Python Backend (REQUIRED for AI Chat)

The AI Chat feature requires a Python backend running alongside the Next.js frontend. This backend handles:
- **Deepgram** — Speech-to-Text (transcription of user voice)
- **OpenRouter/Deepseek** — LLM (generates AI responses)
- **ElevenLabs** — Text-to-Speech (speaks AI responses)
- **Simli** — Avatar video generation (creates talking head video)
- **LiveKit Agent** — Orchestrates the above into a real-time voice pipeline

### Step 1: Install Python Dependencies
```bash
cd Portfolio_hemanth
pip install -r requirements.txt
```

### Step 2: Set Up .env File
Create `Portfolio_hemanth/.env` with all the API keys:
```env
LIVEKIT_API_KEY=API2GeKUiuxBr8U
LIVEKIT_API_SECRET=NLro6aOf3odokLfprB6qge4rqAeCYpm2FvPj7eyoUwOC
LIVEKIT_URL=wss://portfolio-ejfgo35q.livekit.cloud
OPENROUTER_API_KEY=sk-or-v1-...
DEEPGRAM_API_KEY=ca95841d...
ELEVEN_API_KEY=sk_cc110d...
ELEVEN_VOICE_ID=TX3LPaxmHKxFdv7VOQHJ
SIMLI_API_KEY=dqdr3s5m6sswytzv6pu24
SIMLI_FACE_ID=dd10cb5a-d31d-4f12-b69f-6db3383c006e
```

### Step 3: Start the Dispatcher (Terminal 1)
```bash
cd Portfolio_hemanth
python dispatcher.py --port 8089
```
This starts a FastAPI server on port 8089 that launches avatar worker processes.

### Step 4: Start the Agent Worker (Terminal 2)
```bash
cd Portfolio_hemanth
python agent_worker.py dev --avatar-url http://localhost:8089/launch
```
This registers with LiveKit Cloud and waits for rooms to be created.

### Architecture Flow
```
Browser → Next.js API (/api/session) → Creates LiveKit Room + Token
Browser → Connects to LiveKit Room with token
LiveKit Cloud → Notifies Agent Worker (registered as ROOM worker type)
Agent Worker → Creates avatar token → POSTs to Dispatcher /launch
Dispatcher → Spawns simli_avatar_runner.py subprocess
Simli Runner → Connects to room → Streams avatar video
Agent Worker → Uses Deepgram STT → Deepseek LLM → ElevenLabs TTS
TTS Audio → Sent to Simli Runner → Lip-synced video published back
Browser → Receives avatar video + audio tracks → Displays in UI
```

## Vercel Deployment

1. Push to GitHub
2. Connect repo to Vercel
3. Set environment variables in Vercel dashboard (LIVEKIT_API_KEY, LIVEKIT_API_SECRET, LIVEKIT_URL)
4. The Python backend (agent_worker.py, dispatcher.py) must run on a separate server with the same LiveKit credentials
5. Build command: `next build` (via Vercel auto-detection)
6. Output: `standalone` mode configured in `next.config.ts`

---

## Dependencies (Key)

| Package | Version | Purpose |
|---------|---------|---------|
| next | ^16.1 | App Router, SSR, API routes |
| react | ^19 | UI |
| gsap | ^3.15 | Scroll animations, entrance effects |
| livekit-client | ^2.19 | Browser WebRTC for avatar video/audio |
| livekit-server-sdk | ^2.15 | Server-side room/token management |
| tailwindcss | ^4 | Utility-first CSS |
| lucide-react | ^0.525 | Icons |
| framer-motion | ^12 | Available but GSAP preferred for this project |

---

## Performance Notes

- GSAP is tree-shaken: only `gsap/core` and `gsap/ScrollTrigger` are bundled
- All sections are client components (`"use client"`) for GSAP access
- Images use `next/image` where applicable
- Fonts loaded via `next/font` for zero FOIT
- Loading screen ensures perceived performance is good even on slow connections
