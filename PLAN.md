# Tokenomics Board — Home / Rhythm / Priority Plan

Date: 2026-09-11. Status: draft for review. Board: `tokenomics/index.html` (18 lanes, verified).

## 1. Home
- Option A: `agentworth.dev/tokenomics` — public, SEO/GEO/AEO surface, cron-updated. Best for distribution + WorldTrainer bids.
- Option B: `spacepilot.dev/...` — private/fleet monitor via pets schedule. Best for personal alpha.
- Decision: ship A first (same dataset powers B later). One source file, two skins.

## 2. Rhythm
- Weekly cron (Mon 06:00 UTC): spot $/hr re-pull (Vast, Lambda, Together, CoreWeave), $5B+ deal watch, energization-date check.
- Event-driven extra: any $5B+ lease, backstop change, or grid-slip triggers same-day patch.
- Monthly: deep re-pull of one bottleneck layer (rotate power → memory → substrates → silicon).

## 3. Priority lens (balanced)
- Equity first in charts (who bills), product second (what to build). Every chart ends with both takeaways.
- Personal uses served: (a) equity investing, (b) product/market ideas, (c) existing products (chief-of-staff, WorldTrainer pool signals).

## 4. SEO / GEO / AEO surface
- Static HTML, semantic headings, dataset tables as real `<table>`, JSON-LD dataset block (add before publish).
- Clean URL: `/tokenomics`. Weekly `data-as-of` stamp + changelog. This is what ChatGPT/free monitors and pets schedules poll.

## 5. Cron spec (draft)
- Input: lane queries (spot, deals, grid). Output: `data/tokenomics.json` + board rebuild + git commit.
- Gate: only verified/reported tiers charted; single-source flagged, never plotted as fact.
- Alerts: spot floor <$0.25, GB200 listed, on-demand cut >20%, date slip >90d → notify.

## 6. Open questions
- Host: confirm A, B, or both?
- Cadence: weekly cron ok, or monthly deep only?
- Bid link: wire weekly pool-fill (0-100B) into WorldTrainer now or later?
