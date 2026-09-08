# Phase-wise Planning — AI Cold-Calling Voice Agent

Companion document to `project-details.md`. Each phase lists the goal, tasks, tech touched, and the exit criteria that mark it done — don't move to the next phase until the exit criteria are actually met.

---

## Phase 0 — Setup
**Goal**: All accounts and the local dev environment are ready.

- [ ] Create a Twilio trial account; verify your own number plus 1–2 test numbers
- [ ] Get a Groq API key (covers both STT and LLM)
- [ ] Get a Gemini API key (fallback LLM)
- [ ] Get an ElevenLabs free-tier API key
- [ ] Set up a Neon or Supabase Postgres instance
- [ ] Set up an Upstash Redis instance
- [ ] Install Pipecat + Pipecat Flows locally; set up the Python project and virtual environment
- [ ] Install ngrok and verify the tunnel works against a placeholder FastAPI server

**Exit criteria**: A test Twilio call reaches your local ngrok endpoint and you can see the webhook payload logged.

---

## Phase 1 — Pipeline skeleton
**Goal**: A working, even if scripted, voice loop — call connects, the agent speaks a fixed line, listens, transcribes, and responds.

- [ ] Wire Twilio Media Streams to Pipecat through the FastAPI webhook
- [ ] Configure the Pipecat pipeline: STT (Groq Whisper) → passthrough → TTS (ElevenLabs)
- [ ] Place a real test call to your verified number and confirm round-trip audio works
- [ ] Measure and log round-trip latency per turn

**Tech touched**: Pipecat, Twilio, Groq STT, ElevenLabs TTS, FastAPI, ngrok

**Exit criteria**: You can have a live voice exchange with the agent over a real phone call, even if the content is scripted.

---

## Phase 2 — State machine (Pipecat Flows)
**Goal**: Replace the fixed script with the real conversation flow.

- [ ] Define every state from the state diagram (Greeting through Wrapup)
- [ ] Define the tool schemas for every transition-triggering function
- [ ] Implement each state's prompt and its allowed tool calls in Pipecat Flows
- [ ] Manually test every branch, including each objection path

**Exit criteria**: You can walk the agent through every branch of the state diagram on a live call and it transitions correctly each time.

---

## Phase 3 — Business research tool
**Goal**: The agent opens with lead-specific context instead of a generic script.

- [ ] Load the Google Maps lead list into the `leads` table
- [ ] Build a lightweight scraper to pull 1–2 lines of context from each business's website or listing, cached in `research_notes`
- [ ] Inject `research_notes` into the agent's context before each call

**Exit criteria**: A test call opens with a specific, correct reference to the business — not a generic line.

---

## Phase 4 — Retry & fallback logic
**Goal**: The agent survives real-world messiness instead of breaking on the first edge case.

- [ ] Enable Twilio AMD; route voicemail detections to `log_voicemail` + hangup
- [ ] Add an ASR-confidence check with a "could you repeat that" fallback
- [ ] Add tool-call schema validation with a bounded retry, then a scripted fallback line
- [ ] Add a secondary-LLM-provider fallback on API timeout or error
- [ ] Add a silence-timeout handler

**Exit criteria**: Deliberately trigger each failure mode (mumble, go silent, hang up mid-sentence) and confirm the agent degrades gracefully instead of crashing or looping.

---

## Phase 5 — Data layer & cost tracking
**Goal**: Every call is fully logged and costed.

- [ ] Create the `calls`, `transcript_turns`, and `cost_logs` tables per the ER diagram
- [ ] Log every turn's transcript and any tool calls
- [ ] Log STT seconds, LLM tokens, and TTS characters per call; compute the dollar cost
- [ ] Build a simple query or script for a weekly cost/outcome summary

**Exit criteria**: After 10 test calls, you can query total cost, cost per call, and the outcome breakdown directly from Postgres.

---

## Phase 6 — Eval harness
**Goal**: Automated, repeatable scoring of agent reliability. This is the centerpiece of the portfolio story.

- [ ] Write 30–50 synthetic personas covering each objection type plus a few edge cases (rude, abrupt hangup)
- [ ] Build the text-only simulated-caller loop — a second LLM plays the persona against your Flows logic directly, no audio involved
- [ ] Implement scoring: task completion, correct-tool-sequence rate, hallucination rate, turns-to-resolution
- [ ] Run a baseline eval and record the score
- [ ] Make one deliberate prompt change, re-run the eval, and confirm the score moves — this proves the harness actually detects regressions

**Exit criteria**: A baseline eval report you can put directly in your README, plus at least one documented before/after showing the harness catching a real regression or improvement.

---

## Phase 7 — Persistent deployment & production number
**Goal**: Move off local + ngrok and go live against the real lead list at small scale.

- [ ] Deploy the FastAPI + Pipecat service to Koyeb or Cloud Run
- [ ] Set up GitHub Actions for CI (tests + lint) and CD (deploy on merge)
- [ ] Complete TRAI DLT registration and buy the official Indian telephony number
- [ ] Cross-check the lead list against the National DNC registry
- [ ] Run against 10–20 real leads, watching the cost dashboard and call logs closely

**Exit criteria**: A small batch of real calls completed end-to-end with acceptable cost per call and no unhandled crashes.

---

## Phase 8 — Stretch goals (optional, ongoing)

- [ ] Arize Phoenix tracing dashboard for per-call debugging
- [ ] Expand the eval persona set based on real call transcripts that went wrong
- [ ] Add a genuine human-handoff/escalation path (real transfer, not just a log entry)
- [ ] Strengthen Hindi-English code-switching handling if it isn't already solid
