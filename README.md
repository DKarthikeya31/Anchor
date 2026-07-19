# Anchor

**An on-device life-orchestration agent for iOS — powered by Apple Foundation Models, App Intents, and a custom LoRA adapter.**

---

## 1. Problem

Modern productivity apps are still built around **manual navigation**: you open Calendar, then open Reminders, then open Health, and manually stitch your week together yourself. Even with "smart" suggestions, the user still does all the coordination work across apps.

At the same time, Apple's 2026 platform introduced three capabilities that, combined, remove that burden — but almost nobody has actually wired them together end-to-end:

- **Foundation Models** — a language model that runs fully on-device
- **App Intents** — lets an app trigger actions across other apps, not just within itself
- **LoRA adapters** — lets you specialize the base model for a specific domain without retraining it

**The gap:** No existing app lets a user say one loose sentence ("I've got a rough week, sort me out") and have it actually reorganize their calendar, reminders, and health goals — privately, on-device, in one step.

---

## 2. Idea / Solution

**Anchor** takes a single natural-language request per day and turns it into real, executed actions across Calendar, Reminders, and Health — without the user touching any of those apps directly.

**Example:**
> User says: *"I've got deadlines Thursday and keep skipping workouts — sort me out."*
> Anchor: moves a workout to Wednesday evening, blocks focus time before Thursday's meetings, and updates reminders — automatically.

Everything happens **on-device**. No request, plan, or personal data ever leaves the phone.

---

## 3. Why This, Why Now

| Trend | What it enables |
|---|---|
| Apple Foundation Models (iOS 26) | On-device reasoning without cloud calls or API cost |
| App Intents maturity | Apps can now execute multi-step actions across other apps, not just expose shortcuts |
| LoRA adapter support | You can specialize the base model for a narrow domain (scheduling/habit language) while keeping Apple's privacy guarantees |

This is the first year all three are usable *together* — which is exactly why almost no one has built this yet.

---

## 4. Core Workflow

```
 User Input (natural language)
        │
        ▼
┌───────────────────────────┐
│  Foundation Models         │   ← reasons about intent,
│  (on-device reasoning)     │     current calendar load,
│                            │     habits, and tradeoffs
└───────────────────────────┘
        │
        ▼
┌───────────────────────────┐
│  LoRA Adapter              │   ← specializes model on
│  (domain fine-tune)        │     scheduling/habit vocabulary
│                            │     ("rough week", "crunch")
└───────────────────────────┘
        │
        ▼
┌───────────────────────────┐
│  Structured Plan (JSON)    │   ← e.g. move event, add
│                            │     reminder, adjust workout
└───────────────────────────┘
        │
        ▼
┌───────────────────────────┐
│  App Intents Executor      │   ← chains actions across
│                            │     Calendar, Reminders, Health
└───────────────────────────┘
        │
        ▼
   Week Reorganized ✅
```

**Step-by-step:**

1. **Input** — user speaks or types one loose request.
2. **Context gathering** — app reads existing Calendar events, open Reminders, and recent Health/Fitness trends (all local, read-only at this stage).
3. **Reasoning** — Foundation Models generates a plan: what should move, what should be added, what should be flagged.
4. **Domain specialization** — the LoRA adapter ensures the model correctly interprets scheduling/habit-specific phrasing rather than treating it as generic text.
5. **Structured output** — the model returns a plan in a fixed JSON schema (not free text) so it can be executed reliably.
6. **Execution** — App Intents chain runs the plan across the relevant apps.
7. **Confirmation** — user sees a short summary of what changed and can undo any single action.

---

## 5. Tech Stack

| Layer | Technology |
|---|---|
| Language | Swift, SwiftUI |
| On-device reasoning | Apple Foundation Models framework |
| Domain specialization | LoRA adapter (trained via Apple's on-device training tools) |
| Cross-app execution | App Intents framework |
| Data sources | EventKit (Calendar), Reminders framework, HealthKit |
| Structured output | JSON schema-constrained generation from Foundation Models |
| Persistence | SwiftData (local only) |
| Hardware requirement | A17 Pro or newer (iPhone), M1 or newer (iPad/Mac) |
| Privacy | 100% on-device — no network calls, no external servers |

---

## 6. What You Need Before Starting

- Xcode 26 (or current version supporting Foundation Models + App Intents)
- A physical device with A17 Pro or newer (Foundation Models won't run in Simulator for on-device inference testing)
- Sample/synthetic calendar, reminders, and health data for early testing (don't use real sensitive data during dev)
- A small labeled dataset of scheduling/habit phrases for LoRA training (can be self-generated — 100-300 examples is a reasonable starting point)

---

## 7. Suggested Build Order

1. **Reasoning first** — get Foundation Models generating a sensible plan from a prompt, no LoRA yet, no execution yet. Just verify the reasoning quality.
2. **Single-app execution** — wire up App Intents for Calendar only. Confirm the plan → action pipeline works end-to-end for one domain.
3. **Expand domains** — add Reminders, then Health.
4. **Train the LoRA adapter** — once you know exactly which phrases/edge cases the base model struggles with, fine-tune to fix those specifically.
5. **Polish the confirmation/undo UX** — this matters a lot for trust, since the app is taking real actions on the user's behalf.

---

## 8. Success Criteria (What "Done" Looks Like)

- User can say one sentence and see a real, correct multi-app change happen.
- No data leaves the device at any point (verifiable via network monitoring during demo).
- User can undo any individual action from the plan, not just the whole batch.
- The LoRA adapter measurably improves interpretation of domain-specific phrasing vs. the base model alone (have a small before/after test set to show this).

---

## 9. Demo Angle

A strong 3-5 minute demo:
1. Show a messy week (calendar conflicts, skipped workout reminders).
2. Say one sentence to Anchor.
3. Show the before/after — calendar and reminders updated live.
4. Show a network monitor confirming zero outbound requests.
5. Briefly show the LoRA adapter's improvement over the base model on a tricky phrase.

This is the kind of demo that works equally well as a personal portfolio piece or an internal show-and-tell.
