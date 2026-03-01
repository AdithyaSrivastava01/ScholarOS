# ScholarOS — AI Tutor

An AI-powered tutor that teaches through Socratic questioning and interactive visualizations. Sage converses with students in real time using browser-native speech, renders math on Desmos/GeoGebra, runs science simulations in sandboxed HTML, and generates 3Blue1Brown-style animations via Manim.

---

## Architecture

```
Student speaks  ──>  Web Speech Recognition (browser-native)
                           |
                     useTutorBrain hook
                           |
                     POST /api/tutor/respond  (SSE stream)
                           |
                     Vercel AI SDK (streamText + tools)
                           |
            ┌──────────────┼──────────────┐
            v              v              v
       Azure OpenAI     Tool calls     Manim server
       GPT-4o           (canvas,       (video gen)
       response         sandbox,
       (speech text)    video,
            |           progress)
            v                |
    Web Speech Synthesis     v
    (browser TTS)       Frontend renders:
            |           - Desmos 2D/3D
            v           - GeoGebra
       Sage speaks      - HTML sandbox (iframe)
       to student       - Manim video player
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Framework | Next.js 16.1, React 19, TypeScript |
| Styling | Tailwind CSS v4 + shadcn/ui |
| State | Zustand 5 |
| Database | Drizzle ORM + PostgreSQL (Supabase) |
| AI | Vercel AI SDK (`ai@6`) + Azure OpenAI |
| TTS | Web Speech Synthesis (browser-native) |
| ASR | Web Speech Recognition (browser-native) |
| Math | Desmos API + GeoGebra |
| Animations | Manim (Python server) |
| Package Manager | pnpm |

### AI Models

| Model | Provider | ID |
|-------|----------|----|
| GPT-4o | Azure OpenAI | `azure-gpt-4o` |
| GPT-4o Mini | Azure OpenAI | `azure-gpt-4o-mini` |

---

## Getting Started

### Prerequisites

- Node.js 20+ (install via [nvm](https://github.com/nvm-sh/nvm): `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash && source ~/.bashrc && nvm install --lts`)
- pnpm 10+ (`npm install -g pnpm@10.29.3`)
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (for Manim animations)
- Chrome or Edge browser (Speech Recognition requires it)

### Step 1 — Environment Variables

Create `.env.local` in the project root:

```env
# Azure OpenAI (required)
AZURE_OPENAI_API_KEY=your-azure-openai-key
AZURE_OPENAI_RESOURCE_NAME=your-resource-name

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=sb_publishable_...
DATABASE_URL=postgresql://postgres.your-project:password@aws-X-region.pooler.supabase.com:6543/postgres

# Manim server
NEXT_PUBLIC_MANIM_URL=http://localhost:5000

# App URL
NEXT_PUBLIC_URL=http://localhost:3000
```

Create `manim-server/.env`:

```env
AZURE_OPENAI_API_KEY=your-azure-openai-key
AZURE_OPENAI_RESOURCE_NAME=your-resource-name
AZURE_OPENAI_MODEL=gpt-4o-mini
FLASK_HOST=0.0.0.0
FLASK_PORT=5000
BASE_URL=http://localhost:5000
```

> **Azure OpenAI setup:** Create an Azure OpenAI resource in the [Azure Portal](https://portal.azure.com). Deploy `gpt-4o` and `gpt-4o-mini` models. Get your API key and resource name from the resource's Keys and Endpoint page.

> **Supabase setup:** Create a free project at [supabase.com](https://supabase.com). Get your keys from Project Settings → Data API. For `DATABASE_URL`, use the **Transaction pooler** URI from Project Settings → Database.

### Step 2 — Install & Run

```bash
pnpm install
pnpm db:migrate       # creates 6 tables in Supabase
pnpm dev              # starts Next.js on http://localhost:3000
```

### Step 3 — Manim Server (for math animations)

In a second terminal:

```bash
cd manim-server
docker build -t sage-manim .
docker run -d --name sage-manim -p 5000:5000 --env-file .env sage-manim
```

### Step 4 — Seed Demo Data (optional)

```bash
npx tsx src/db/seed.ts
```

Creates two demo students:

| Name | Grade | PIN |
|------|-------|-----|
| Leo Wallace | 11th | `1234` |
| Maya Wallace | 4th | `5678` |

Open [http://localhost:3000](http://localhost:3000) in Chrome or Edge.

---

## Project Structure

```
src/
  app/
    page.tsx                     # Landing page
    login/page.tsx               # PIN-based student login
    student/session/page.tsx     # Main tutoring session UI
    parent/                      # Parent dashboard
    api/
      tutor/respond/route.ts     # Core SSE stream endpoint
      tutor/plan/route.ts        # AI learning plan generation
      session/route.ts           # Session CRUD
      session/summary/route.ts   # Post-session AI summary

  hooks/
    useSession.ts                # Session lifecycle orchestrator
    useAvatar.ts                 # Web Speech Synthesis TTS
    useTutorBrain.ts             # SSE stream consumer + tool execution
    useCanvas.ts                 # Multi-tool canvas manager
    useUserCamera.ts             # Native getUserMedia

  lib/
    ai/client.ts                 # Azure OpenAI client (GPT-4o, GPT-4o Mini)
    ai/prompts.ts                # Socratic teaching system prompts
    deepgram/client.ts           # Web Speech Recognition ASR
    canvas/                      # Desmos 2D/3D + GeoGebra wrappers
    manim/client.ts              # Manim video generation client
    sandbox/template.ts          # HTML sandbox wrapper
    camera/                      # Frame capture

  stores/sessionStore.ts         # Zustand session state
  types/session.ts               # Core type definitions
  db/                            # Drizzle schema + migrations
```

---

## Scripts

```bash
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm typecheck    # tsc --noEmit
pnpm lint         # ESLint
pnpm db:migrate   # Run migrations
pnpm db:studio    # Drizzle Studio
```
