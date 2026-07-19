<div align="center">

# ⚓ Anchor

### On-device assistant that reconstructs context from your own notes, mail, and calendar — so you stop losing track of decisions you already made.

![Platform](https://img.shields.io/badge/platform-iOS%2026-black?style=for-the-badge&logo=apple&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-6.0-F05138?style=for-the-badge&logo=swift&logoColor=white)
![On--Device](https://img.shields.io/badge/AI-100%25%20On--Device-4CAF50?style=for-the-badge)
![Privacy](https://img.shields.io/badge/network%20calls-zero-2196F3?style=for-the-badge)
![License](https://img.shields.io/badge/status-concept-9C27B0?style=for-the-badge)

`ios` · `on-device-ai` · `foundation-models` · `app-intents` · `lora` · `privacy`

</div>

---

## The problem

You decide something in a meeting. Three weeks later someone asks "wait, what did we land on for this?" and you're scrolling through Notes, then Mail, then a Slack export, trying to reconstruct a conversation that happened across four different places.

Search doesn't fix this. Search finds files. It doesn't tell you *why* something was decided or which of five conflicting notes is the current one.

Nobody's really solved this on iOS because doing it properly means running real reasoning over personal data, and most people (rightly) don't want that reasoning happening on someone else's server.

## What Anchor does

You ask a plain question:

> "What did we land on for the Q3 timeline?"

Anchor looks across your Notes, Mail, and Calendar, figures out which pieces are actually relevant, and gives you an answer with a pointer back to where it came from — the note, the email thread, the meeting.

It doesn't summarize everything you own. It only surfaces what's relevant to the question, and it tells you where it got it from so you can go check.

Nothing leaves the device. Ever.

## Why now

Three things landed on iOS this year that make this actually buildable, where before it wasn't:

- **Foundation Models** gives you a real reasoning model running locally, not just autocomplete-grade text prediction.
- **LoRA adapter support** means you can teach that model your own shorthand — project names, team acronyms, recurring people — without retraining anything or losing the privacy guarantees of the base model.
- **App Intents** has matured enough that Anchor can answer from Spotlight or Siri directly, not just from inside its own app.

Before this year you'd have needed a cloud model to do the reasoning step well, which kills the whole point for something touching this much personal context.

## Architecture

The pipeline is simpler than it sounds — four stages, no networking anywhere in the loop.

**1. Local index**
A background process reads Notes, Mail, and Calendar (read-only) and builds a lightweight local index — not a copy of the content, just enough structure (timestamps, participants, subjects) to know what to pull later.

**2. Query understanding**
When you ask something, Foundation Models parses the question and figures out what it's actually asking for — a decision, a date, a person, a document.

**3. Retrieval + reasoning**
The relevant slice of the index gets pulled, and the model reasons across it to construct an answer, not just return the top match. This is the part a LoRA adapter helps with, since it's trained on your specific vocabulary — team names, project codenames, whatever shorthand your notes actually use.

**4. Answer + citation**
The response comes back with a short answer and a link to the source (the specific note, email, or event), so you can verify it yourself rather than just trusting it blind.

```
Notes / Mail / Calendar
        │
        ▼
   Local Index (on-device, read-only)
        │
        ▼
   Foundation Models — query understanding
        │
        ▼
   LoRA adapter — domain vocabulary
        │
        ▼
   Retrieval + reasoning over indexed content
        │
        ▼
   Answer + source citation
```

## Tech stack

| Layer | Tool |
|---|---|
| App | Swift, SwiftUI |
| Reasoning | Apple Foundation Models |
| Domain fine-tune | LoRA adapter, trained on your own note/email shorthand |
| System surfacing | App Intents (Spotlight, Siri) |
| Data access | EventKit, Mail (via approved entitlements), Notes |
| Local index | SwiftData |
| Hardware | A17 Pro+ / M1+ |
| Network calls | none |

## Before you start building

- Xcode with Foundation Models + App Intents support
- A real device — inference testing needs actual Neural Engine hardware, not the simulator
- A small set of your own real (or realistic synthetic) notes/emails to test retrieval quality against — this matters more than people expect, since a toy dataset won't surface the vocabulary problems the LoRA adapter is meant to fix
- 100-300 example phrases from your own notes for the LoRA training set, focused on the shorthand and acronyms you actually use

## Build order

Get the reasoning working before anything else. Ask a few real questions against a real (small) dataset with the base model, no adapter, no fancy retrieval — just see how far plain Foundation Models gets you. You'll be surprised how often it's "closer than you'd think but wrong on your specific vocabulary," which tells you exactly what the LoRA adapter needs to fix.

After that: build the local index, wire up retrieval, then layer the adapter in once you have real failure cases to train against, not before. Training an adapter before you've seen it fail is guessing.

App Intents and Spotlight surfacing come last — they're the polish layer, not the hard part.

## What "done" looks like

- You ask a real question about something you actually forgot, and get a correct answer with a source you can check.
- The adapter visibly helps on your own vocabulary — have 5-10 questions the base model gets wrong and the adapter fixes, and keep that list around as a regression test.
- Nothing shows up on a network monitor while any of this runs.

## Demo

Don't demo it with toy data. Pull up a real (or realistic) question you'd actually forget the answer to, ask Anchor, and show it pointing back to the exact note or email. Then show the same question against the base model without the adapter, so people see what the fine-tune actually bought you. That contrast is the whole pitch.
