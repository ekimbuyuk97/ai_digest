You are generating a recurring AI news digest email for readers curious about developments at Anthropic / Google DeepMind / OpenAI). This runs twice a day (~8am and ~8pm local time). Each run, produce TWO things in one email: a short top-level digest, and a longer tiered deep-dive section. Do the following:

STATE: Read `ai-digest/state.json` (relative to the repo root). Schema:
- `last_run`: ISO timestamp. Use it to scope "what's new since last time." If the file is missing or empty, treat this as the first run and just cover the last 12 hours.
- `seen_anthropic_urls`: array of Anthropic news/research URLs already covered. Keep at most the most recent ~200.
- `deep_dive_history`: array of `{topic, date_covered}` — every topic that has ever received a full deep-dive treatment. Check this before picking backlog topics so you never re-cover the same thing twice. Keep at most the most recent ~150.
- `open_threads`: array of `{topic, date_opened, last_update}` — ongoing, multi-part stories (e.g. a policy dispute, a legal fight, a talent-exodus story still unfolding) that are worth checking for follow-up developments each cycle even if no single new "headline" justifies a fresh top-level item. Add a thread here when a deep-dive topic is clearly still developing; remove it once it's been stale (no real update) for ~30 days or has clearly resolved.

CREDENTIALS: Read `RESEND_API_KEY` and `RECIPIENT_EMAIL` from environment variables (they are already set in the shell — do not read any .env file). Do not print the key value anywhere in your output.

## 1. GATHER SOURCES

Use WebFetch/WebSearch for each; skip gracefully and note in the email if a source fails — don't let one failure block the whole digest.

- Anthropic official: fetch `https://www.anthropic.com/news` and `https://www.anthropic.com/research`. Identify any post URLs not already in `seen_anthropic_urls`.
- Dario Amodei's and Boris Cherny's X/Twitter activity — best-effort search/fetch for recent posts or statements. Note clearly if blocked/unavailable rather than guessing content.
- Jack Clark's Import AI newsletter (importai.substack.com) — check for a new weekly issue.
- Chris Olah / interpretability research at transformer-circuits.pub — check for new papers.
- Broader AI industry news: WebSearch for "AI news" plus today's date, cross-checking against TLDR AI, Last Week in AI, The Information's AI coverage as quality filters for what's actually significant (skip routine/minor items).
- AI-for-PMs, sourced as **specific named interviews/podcast episodes/essays**, not generic blog-post scanning: check for new episodes of Lenny's Podcast and Aakash Gupta's Growth Podcast/Product Growth substack, specifically ones featuring people at Anthropic, OpenAI, or other AI-native companies talking about how product/eng actually operates there (the Fiona Fung, Cat Wu, and Aakash-Gupta-x-Rohan-Varma pieces from past cycles are the model — concrete, named, with operational details, not generic "AI is changing PM work" takes).
- For any `open_threads` entries, do a targeted search for follow-up developments specifically (e.g. "<topic> update", "<topic> response", "<topic> ruling") rather than relying on it surfacing from the general news sweep.

## 2. ASSESS VOLUME — decide what the deep-dive section covers

Count genuinely new, significant items found in step 1 (new Anthropic posts, notable industry developments, follow-ups on `open_threads`, etc.).

- **≥3 new significant items**: normal cycle. Deep-dive covers these fresh items plus any `open_threads` follow-ups found.
- **1–2 new significant items, or only minor follow-ups on `open_threads`**: thin cycle. Cover what's genuinely new (don't omit it), but fill out the deep-dive with ONE backlog topic (see below) rather than stretching thin material across multiple sections.
- **Genuinely nothing new and no `open_threads` movement**: quiet cycle. Pull 1–2 topics from the backlog (below). Do not manufacture urgency or pad minor items into major ones to fill space.
- **Backlog is also exhausted** (every major topic from the last ~6 months is already in `deep_dive_history` and there's truly nothing new): send the short top-level digest only, explicitly note it's a quiet cycle, and skip the deep-dive section entirely for this run. A quiet, honest digest beats a padded one.

**Sourcing the backlog**: WebSearch for the biggest AI-relevant stories over roughly the last 6 months, cross-reference against `deep_dive_history` to find ones never covered, and pick the most significant 1–2. Prioritize stories with real substance for a product manager following Anthropic, DeepMind, and OpenAI (product launches, org/culture revelations, policy fights, research with product implications) over pure model-benchmark churn.

## 3. SYNTHESIZE THE SHORT DIGEST

4 sections, each 3-6 concise bullet points (skip a section entirely if genuinely nothing new):
- **Top AI Industry News** — the most significant happenings since last run
- **Anthropic Spotlight** — new Anthropic blog/research posts, plus anything new from Dario Amodei or Boris Cherny on X
- **AI for PMs** — specific named interviews/podcast episodes/essays (per the sourcing note above) with a one-line takeaway each, not generic trend commentary
- **Why it matters** — one or two lines connecting the day's news to the broader trajectory of AI product work at these labs and why it's interesting for someone in product

## 4. SYNTHESIZE THE DEEP DIVE

This is the longer section the recipient explicitly wants to be thorough, not condensed. For every topic the deep-dive covers (fresh, follow-up, or backlog), write a substantial paragraph or two per topic with:

- **What actually happened/what it is** — concrete specifics (dates, numbers, names), not vague summary.
- **Who's responsible/credited** — the team, PM, exec, or researcher behind it, when findable.
- **Multiple distinct viewpoints** — don't settle for one source's framing. Actively look for the strongest counter-argument or skeptical take, not just "some people disagree." Name the source of each viewpoint.
- **Pros/cons** where applicable, and **what's next**.
- **A "Further reading" list** of every distinct source URL used, as markdown links, so the recipient can keep digging on whichever thread interests them.
- If a claim is serious but uncorroborated by mainstream sources, say so explicitly rather than presenting it as settled fact — flag contested/unverified claims clearly and attribute them to their actual source.

**Assign a priority icon to every topic**, calibrated to relevance for a product manager who follows Anthropic, DeepMind, and OpenAI closely (not general newsworthiness):
- 🔴 **P0** — essential, drop-everything relevance (e.g. direct signals about how Anthropic, DeepMind, or OpenAI operate, hire, or are positioned; major policy/safety fights with multi-sided implications)
- 🟠 **P1** — just as much depth as P0, one tier down in urgency
- 🟡 **P2** — strong context, useful but not urgent
- ⚪ **P3** — lower relevance to this specific goal, still worth knowing

Order the deep-dive section by priority tier (P0 first), and label each tier with a one-line guide to the reading-time budget it represents (e.g. "read these no matter what" / "next 10 minutes" / etc.) so the recipient can stop at whatever tier matches the time they have.

Include ALL topics inline in the email — do not collapse or truncate P2/P3 behind a toggle. The recipient wants everything in one read.

## 5. SEND EMAIL via Resend API

Use the RESEND_API_KEY and RECIPIENT_EMAIL environment variables (already available in the shell). Run:

```
curl https://api.resend.com/emails \
  -H "Authorization: Bearer $RESEND_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"from\":\"AI Digest <onboarding@resend.dev>\",\"to\":[\"$RECIPIENT_EMAIL\"],\"subject\":\"AI Digest – $(date '+%Y-%m-%d %p')\",\"html\":\"<digest + deep-dive formatted as HTML, in that order>\"}"
```

Never hardcode or echo the key. If the send fails (e.g. domain/recipient restrictions on the free tier), report the error clearly so the user knows email delivery needs fixing — don't silently fail.

## 6. UPDATE STATE

Write `ai-digest/state.json` with:
- `last_run`: current ISO timestamp
- `seen_anthropic_urls`: updated list (most recent ~200)
- `deep_dive_history`: append every topic covered in this run's deep-dive, with today's date (most recent ~150)
- `open_threads`: add any new multi-part developing stories surfaced this run; remove any that resolved or have gone stale (~30 days with no real update)

Note: audio/TTS generation is not yet implemented for this digest — text/HTML email only for now.
