UbuntuEduAI — README (How this single-file app is developed)

UbuntuEduAI is a single-page, single-file HTML app that mimics a modern “ChatGPT/Gemini-style” education assistant UI, with category quick-access, multi-provider model selection, optional voice loop (STT + TTS), accessibility toggles, and optional server streaming.

1) High-level structure (what you built)

This file is organized into four main layers:

UI + Styling (HTML/CSS)

Layout: Left sidebar (brand + quick access categories + conversation list) and Main chat (topbar + messages + composer).

Modals: Settings, Access, Library.

Live Panel (hidden by default): voice/dictation controls, transcript drawer, and live mode toggles.

Data definitions (JS constants)

CULTURE_ROWS, LEARNING_ROWS: Populate “Library” tables.

CATEGORIES: Defines the Quick access category menu + the required auto-response text + the “Generate” and “Explain” prompts.

State management (JS variables)

config: User settings + accessibility toggles + API mode.

conversations, conversationHistory, activeConversationId: Conversation list and messages.

Mode toggles: imageMode, videoMode, streamMode, voiceMode.

Voice state: recognition, isListening, pendingAutoSend, autoSendTimer.

Behavior/logic (JS functions)

Rendering: renderCategoryList(), renderChat(), addBubble(), renderConversations()

Prompting: buildSystemPrompt()

API calls: server (/api/chat, /api/chat/stream, /api/image, /api/video) and client providers (OpenAI/Anthropic/Google)

Voice: Web Speech API STT + SpeechSynthesis TTS

Persistence: localStorage for non-sensitive settings; sessionStorage for keys (optional)

2) UI sections and how each was developed
A) Left Sidebar

Purpose: navigation + “agent-like” entry points
Key parts:

Brand block + “New conversation”

Quick access categories (rendered from CATEGORIES)

Conversation list (each item switches active conversation)

Footer row (session info + export)

How it works:

renderCategoryList() loops through CATEGORIES and creates buttons.

startNewChat() creates a new conversation object, sets it active, clears modes/files, and shows the welcome screen.

B) Topbar (App controls)

Purpose: “enterprise assistant” feel + fast access to modes
Controls include:

Sidebar collapse / mobile drawer

Provider model dropdown (OpenAI/Anthropic/Google)

Image mode, Video mode, Live mode, Clear, Settings

How it works:

Mode toggles update state variables (e.g., streamMode) then call:

refreshComposerMode() to update placeholder and mode label

updateLivePanelBadges() to show/hide the Live panel and reflect status

C) Main Messages Area

Purpose: chat transcript + system panels
Key parts:

Welcome card (shows when no messages)

Live controls panel (shows only if voice/captions/live/video is on)

Transcript drawer (shows only if captions enabled)

Chat bubbles (addBubble())

How it works:

renderChat() rebuilds the messages UI from conversationHistory.

addBubble(kind, content, type) adds a message to UI and optionally stores it in history.

D) Composer (Message box)

Purpose: typed input + attachments + send behavior
Key parts:

Attach file button (adds file names into the outbound message)

“Use local context” button (demo only: stores names in config.localFiles)

Textarea + Send button

How it works:

Press Enter → sendMessage() unless Shift+Enter.

sendMessage() routes the request:

Video mode → handleVideoGeneration()

Image mode → handleImageGeneration()

Text → handleChat()

3) Quick access categories (the “agent” behavior rule)

Each category is defined in CATEGORIES as an object with:

key, label

response: the immediate welcome/breakdown message shown when clicked

generatePrompt: inserted into the composer when user clicks “Generate something for me”

explainPrompt: inserted into the composer when user clicks “Explain an item in more detail”

What happens when a category is clicked:

selectCategory(key) runs:

Adds the category’s welcome/breakdown message instantly

Adds three next-step buttons via buildCategoryActionsHTML()

This ensures the category menu behaves like a structured assistant, not a static list.

4) System prompt + response style (where “ChatGPT/Gemini-like” comes from)

All provider calls use a single system prompt built by:

buildSystemPrompt()

It merges:

Foundations (CRP, Funds of Knowledge, etc.)

Disability-aware support schema

Category behavior rule

“Final response requirement” structure

User context (country/region/curriculum/language)

Optional “Sign-friendly output” style rules

To make outputs more “narrative, summary-first, token-efficient” you adjust the OUTPUT STYLE and FINAL RESPONSE REQUIREMENT sections inside buildSystemPrompt() (this is the controlling layer for tone and verbosity).

5) Voice features (STT + TTS + pause/delay)
A) Dictation (STT)

Uses Web Speech API (SpeechRecognition / webkitSpeechRecognition)

Controlled by:

config.stt (Dictation enabled)

voiceMode (loop behavior: listen → auto-send after pause)

Auto-send pause behavior:

scheduleAutoSend() triggers an auto-send after a fixed delay (currently 1200ms).

Dictation “pause” is implemented via this timer (silence leads to auto-send).

B) Voice replies (TTS)

Uses speechSynthesis and SpeechSynthesisUtterance

Controlled by config.tts

C) Deactivate/pause dictation

Already supported conceptually by:

toggleSTT() (turn dictation on/off)

toggleVoiceMode() (turn voice loop on/off)

stopListening() (stops recognition and cancels pending auto-send)

If you want an explicit “Pause dictation” button separate from OFF, implement a pausedListening flag that:

stops recognition,

preserves voiceMode as “On”,

and resumes with a “Resume” button without flipping all voice settings.

6) Server vs Client API mode (how requests are routed)
Server mode (recommended for production)

Frontend calls your backend:

GET /api/health (configuration check)

POST /api/chat (non-stream)

POST /api/chat/stream (stream mode, SSE-like data: lines)

POST /api/image (optional)

POST /api/video (optional; required for video mode)

Client mode (demo-only)

Frontend calls provider APIs directly using user-pasted keys:

OpenAI: v1/chat/completions, v1/images/generations

Anthropic: v1/messages

Google: v1beta/models/gemini-pro:generateContent

Keys can be kept session-only via sessionStorage (never localStorage).

7) Accessibility & transcripts

Accessibility toggles are in Settings and stored in config:

Sign-friendly output

Transcripts panel

Large text

Reduce motion

Theme selection (dark/light/gray/high-contrast)

Transcripts:

appendCaptions(text) writes time-stamped lines into captionsBox

Only active when config.showCaptions is enabled

8) Export + persistence

Conversations are stored in-memory for the session UI.

Configuration is stored in localStorage (non-sensitive only).

Keys can optionally be stored in sessionStorage for the tab session.

exportChat() downloads a .txt transcript including user context fields.

9) Where to edit when you want improvements

Category menu text / prompts: CATEGORIES

Narrative + summary-first style: buildSystemPrompt() → “OUTPUT STYLE” and “FINAL RESPONSE REQUIREMENT”

Token/length control: provider request settings (max_tokens, temperature) and the system prompt constraints

Dictation pause duration: scheduleAutoSend() delay (currently 1200)

Add “Pause dictation” button: add a paused state + a UI toggle in the Live panel

If you want, I can rewrite the system prompt block so outputs become:

summary-first (2–4 lines),

then a short narrative plan,

then compact bullets only when necessary,

and consistently optimized for streaming (clean sentence boundaries and fewer nested lists).

You said:
Coud you distsine the server and backend and front 
ChatGPT said:

Yes — here is the clean distinction between frontend, backend/server, and how they interact in UbuntuEduAI.

1) Frontend (Client)

This is the single index.html file running in the user’s browser.

What it does

Renders the UI: sidebar, chat, category menu, settings, modals, composer.

Captures user actions: typing, clicking categories, toggling voice/image/video modes.

Builds messages + the system prompt.

Displays model responses (including streaming text if enabled).

Runs optional browser-only features:

Speech-to-text (Web Speech API)

Text-to-speech (SpeechSynthesis)

Local UI accessibility toggles

Local storage of non-sensitive preferences

What it should NOT do (in production)

It should not store or use permanent API keys.

It should not directly call OpenAI/Google/Anthropic with secret keys if you want strong security.

2) Backend (Application Server)

This is your Node/Express (or Python/FastAPI) app that receives requests from the frontend and then talks to model providers.

What it does

Exposes your app endpoints:

POST /api/chat (normal chat)

POST /api/chat/stream (streaming chat)

POST /api/image (image generation)

POST /api/video (video generation)

GET /api/health (status/config check)

Holds secrets (API keys) safely in environment variables.

Enforces policies and guardrails:

rate limits, quotas, input size checks

content filtering/validation

logging (careful with PII)

Optional: stores data in a database (users, conversations, analytics).

Optional: caching, RAG retrieval, file handling, long documents, embeddings.

Why it matters

Security: keys stay on the server, not leaked to the browser.

Control: you can swap providers without changing frontend logic much.

Reliability: streaming, retries, fallbacks, instrumentation.

3) “Server” vs “Backend” (are they different?)

In most web apps they overlap, but a precise distinction is:

Server = the running service reachable over the network (the host + ports + deployed runtime).

Backend = the application logic behind the API (routes/controllers/services).

So: your backend runs on your server.
Sometimes “server” also includes infrastructure pieces (reverse proxy, load balancer, CDN), not just code.

4) How the flow works (production setup)

Step-by-step

User types message in frontend.

Frontend calls your backend: POST /api/chat (or /api/chat/stream).

Backend attaches:

system prompt

provider selection

keys (from environment)

safety rules

Backend calls the provider (OpenAI/Google/Anthropic, etc.).

Backend returns the response to frontend (streaming or one-shot).

Frontend renders bubbles + stores local conversation view.

5) Two operating modes (important)
A) Server Mode (recommended)

Frontend → your backend → Provider
✅ secure, scalable, production-ready.

B) Client Mode (demo-only)

Frontend → Provider directly
✅ easy for prototypes
⚠️ risky: keys are exposed in the browser and can be stolen.
