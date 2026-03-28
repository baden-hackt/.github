# KI trifft Lagerhaltung — Hackathon Pitch

## TL;DR

A camera watches a shelf. AprilTags identify products. AI estimates how full each slot is. When stock is low, the system auto-reorders via email with a CSV — no human needed.

---

## How It Works

1. **Camera** captures the shelf at regular intervals
2. **OpenCV** detects changes between frames — only processes when something actually moved
3. **AprilTags** on each bin/slot map to a product number — rearrange the shelf, doesn't matter, the tag moves with the product
4. **OpenCV** crops the region around each detected tag
5. **Vision AI** (Claude API) receives the cropped image and estimates the fill level (0–100%)
6. **Logic layer** checks: is fill below threshold? Is there already a pending order?
7. If reorder needed → generate **CSV**, send **email** to supplier
8. **Dashboard** shows everything live — shelf status, fill levels, order history

---

## Where the AI Actually Is

**AI = fill-level estimation.** That's it. No bullshit.

Why this needs AI and not just pixel counting:

- Products have irregular shapes — bags, bottles, loose parts
- Items get shoved in sideways, stacked weirdly, partially occluded
- Lighting changes, shadows from shelf above
- A box might be present but empty
- Hand-tuned OpenCV thresholds break constantly in real conditions

A vision model just *looks* at the cropped slot and says "that's about 30% full." That's genuinely hard with classical CV and genuinely easy for a vision LLM.

**What is NOT AI:**

- Camera capture → OpenCV
- Product identification → AprilTags
- Change detection → OpenCV
- Order dedup → simple state logic
- Reorder trigger → threshold check

Every layer does exactly what it's best at. No AI for the sake of AI.

---

## Architecture

```
Camera
  │
  ▼
OpenCV (frame capture + change detection)
  │
  ▼
AprilTag detection → maps tag ID → product/supplier
  │
  ▼
Crop slot region around each tag
  │
  ▼
Vision AI (Claude API) → "how full is this slot? 0-100%"
  │
  ▼
Logic layer:
  ├── fill < threshold?
  ├── already ordered? (dedup check)
  └── → generate CSV, send email to supplier
  │
  ▼
Dashboard (live shelf status, fill levels, order log)
```

**One-liner:** Deterministic vision for sensing, AprilTags for identification, AI for the hard part — understanding what "low" looks like in messy reality.

---

## What Will Make Us Lose

- Demo breaks (lighting, tag detection, camera issues)
- No visible AI contribution — looks like "just OpenCV"
- No business logic (no dedup, no order tracking)
- Overcomplicated UI, underbuilt core
- AI shoehorned where it doesn't belong

## What Makes Us Win

- **Live camera demo** — remove items from shelf, watch fill level drop, order triggers automatically
- **AprilTags** — rearrange products mid-demo, system adapts instantly
- **Honest AI use** — judges see exactly where and why AI is used
- **Change detection** — system is efficient, doesn't spam the API every frame
- **Open orders + dedup** — actual business logic, not just a tech demo
- **Dashboard** with real-time shelf visualization (green/yellow/red per slot)
- **SAP-ready story** — CSV output maps to SAP Business One purchase order import
- **Cost story** — "CHF 50 in hardware, ~CHF 0.02 per API check"

---

## Team Split

Cleanly splits into three lanes:

**Said — Camera + Vision Pipeline**
OpenCV frame capture, change detection, AprilTag detection, cropping slot regions, calling the Claude Vision API, parsing the fill-level response. This is the core engine.

**Dijar — Backend Logic + Reorder**
Product master data config, order state tracking (SQLite), dedup logic, threshold checks, SAP-ready CSV generation, email sending. This is the business brain.

**Timo — Dashboard + Demo Setup**
Web frontend showing live shelf status (green/yellow/red per slot), order log, fill-level history. Also handles the physical demo setup — shelf, printed AprilTags, camera positioning.

Said outputs fill levels. Dijar consumes them and triggers orders. Timo visualizes everything. Clean interfaces, minimal dependencies between the three.

---

## Positioning

We are **not** building "an AI that sees products."

We are building **a reliable inventory system that uses AI where it actually matters: understanding messy real-world shelf conditions that deterministic CV can't handle.**

AprilTags for identification. OpenCV for efficiency. AI for perception. Clean separation. No magic, no hype.

---

## SAP Integration

No live SAP connection needed. The system is **SAP-ready out of the box**.

The CSV we generate for reorders uses SAP Business One's purchase order import column format — same field names, same structure. This means the output can be dropped straight into SAP B1 via the Data Import Framework with zero custom development.

This is not extra work — it's just naming our CSV columns correctly instead of making up our own. Person 2 handles this as part of normal CSV generation.

**In the pitch:** "Our output is SAP B1-compatible today. Plug in a real SAP RFC/API endpoint and this goes from prototype to production."

Data Unit are SAP people. They'll appreciate that we thought about the integration path without wasting hackathon time fighting with SAP.

---

## Tech Stack

| Layer | Tool |
|---|---|
| Camera | Webcam / Raspberry Pi cam |
| Frame capture + change detection | Python + OpenCV |
| Product identification | AprilTags (mapped to product master data) |
| Fill-level estimation | Claude Vision API |
| State / order tracking | SQLite |
| Reorder | Python → CSV generation → email (smtplib) |
| Dashboard | Web frontend (React or plain HTML) |
| Product master data | JSON/CSV config file |

