# PropReel V1 — Product Requirements Document
**UGC-to-Lead Engine for a Single Real Estate Agent**
Version 1.8 · 8 October 2026 · Status: Ready for build
Companion doc: *PropReel Vision PRD v3.3* (Phases 2–4). This document defines V1 only.

### Changelog
| Version | Change |
|---|---|
| v1.0 | Initial V1 scope: organic + paid lead capture, approval-gated replies, compliance core. |
| v1.1 | Email nurture program (T4 triggers, WF-8/9/10, dual-sender architecture, Track D metrics, AC-16–18). |
| v1.2 | Content Production Pipeline (staged reel workflow, slash-command prompt library, per-stage gates, content kanban, AC-19). |
| v1.3 | Review fixes. **Approvals:** two approval types (1:1 + template-segment), approval TTL, reply-window countdowns. **Business model:** seller lead type split into `seller_listing` / `seller_direct_offer` with TREC principal disclosure. **Pilot clock:** Day 0 = capture live; full Meta permission list incl. `ads_read`; permissions-pending mode. **Attribution:** keyword-first CTAs, link-in-bio router, defined model and denominator. **Content:** brief volume raised to 7/week to support ≥4 posts/week. **Compliance:** fair-housing strict mode extended to all public content and all scoring; re-permission flow; Texas TDPSA via rule pack. **Platform & data:** inline data model (§12a); two roles (Agent + Helper); YouTube polling-only; push notifications in onboarding. **Budget & models:** separate infra budget; classifier calibration via eval set; models named. **Acceptance criteria:** AC-19 split, AC-20–23 added. |
| v1.4 | Content benchmarks (§04a): five Houston producers studied from public sources, figures verified or marked unverified. Weekly brief mix becomes seven named formats (§06 WF-7) backed by the companion *PropReel Reel Playbook*. Home-value request and neighborhood-guide lead magnets added to T1. Track A targets checked against benchmarks (unchanged, rationale added). SEO neighborhood pages added to §18. |
| v1.5 | Reel-level benchmark data added to §04a (Instagram reels for 7 accounts + 5 YouTube channels, collected 7 Oct 2026; source file `benchmarks/producer_benchmark.csv`). Format ranking by evidence in WF-7; F7 must carry a real estate angle or CTA. Optional collaborator accounts on post kits (§11, §12a). Diagnostic content metrics added to §16 (views ÷ followers, CTA rate, leads per reel). |
| v1.6 | Stack change (§12): Python 3.12 FastAPI backend with SQLAlchemy + Alembic and the Inngest Python SDK; Next.js PWA is the UI only and calls the API. PropReel moves to its own repository, separate from the Lead-to-Closing OS. |
| v1.7 | Single-stack Next.js (§12): replaces the Python FastAPI backend. Next.js API routes handle webhooks and the app API; Bolt Database cron handles scheduled jobs. No Inngest, no Python. Resend chosen for bulk/transactional email (replaces "Postmark/Resend"). Groq Whisper API chosen for transcription (replaces "Deepgram/Whisper"). Classification model updated to Claude Haiku 5.5 (`claude-haiku-5-5`); drafts stay on Claude Sonnet 5.5 (`claude-sonnet-5-5`). |
| v1.8 | Database and scheduled jobs move to Supabase (§12): managed Postgres with pgvector and row-level security; Supabase Cron (pg_cron) runs scheduled jobs by calling protected API routes. Replaces Bolt Database. |

---

## 01. Product Intent

**Why this product exists.** The Agent's business today runs almost entirely on referrals from past clients. That channel is saturated — it cannot grow. Buying leads from portals (Zillow, Realtor.com) is expensive and builds no brand. V1 exists to open two **owned** lead channels and prove they work for one agent:

1. **Organic short-form content** — Instagram Reels, Facebook video, YouTube Shorts, filmed on a phone, each piece carrying a tracked call-to-action (primarily a per-piece comment/DM keyword; see §09 Attribution).
2. **Facebook/Instagram lead ads** — Meta instant forms feeding the same lead engine (Housing Special Ad Category, always).

**The strategic bet (explicit):** authentic, consistent, CTA-driven content + fast, human-approved follow-up beats portal lead fees for a solo agent. V1 must prove or disprove this in 90 days (from Day 0, §16) on a real business.

**Anti-bets (explicit):** no portal lead buying; no cold calling/SMS; no automated publishing; no scraping.

---

## 02. User

**Primary user: one licensed US real estate agent ("the Agent")** — solo, no team, operating under a sponsoring broker and state advertising rules (e.g., TREC in Texas). First market: one US metro (recommended: Houston, TX — configurable).

**The Agent's two seller offers (both in V1):**
- **Listing** — the Agent lists the seller's home as their agent (`seller_listing`).
- **Direct offer** — the Agent (or her entity) makes an offer to buy the home as a principal (`seller_direct_offer`). Because she is a licensee buying for her own account, every direct-offer touchpoint carries the state-required license-status disclosure (Texas: TREC Rule 535.144) from the state rule pack, and direct-offer pages/replies cannot be approved until that pack is lawyer-reviewed.

**Decisions only the Agent makes:**
- Approve or edit every outbound message (replies, follow-ups, nurture templates) — enforced in code
- Approve post kits and UGC briefs before use
- Set property availability and owner-approved shareable numbers
- Qualify leads, move pipeline stages, override scores
- Confirm the state rule pack is lawyer-reviewed before seller-offer pages go live in that state

**Anti-user (V1):** unlicensed wholesalers; teams with dedicated ad-ops staff; anyone who wants the system to publish or send without approval.

**Helper (optional, V1):** the Agent's spouse/operator. V1 supports **exactly two roles in one workspace — Agent and Helper.** The Helper has their own login (MFA required) and may prepare drafts, tag content, and triage the inbox, but holds **zero approval or send rights** (enforced by role-based access control and by the Sender). The licensed Agent approves everything under her name.

---

## 03. Business Context

| Item | Decision |
|---|---|
| First customer | The Agent's own business (pilot brand). All metrics baselined against her current numbers. |
| Geography | ONE US metro + ONE state rule pack in V1. Expansion = new rule pack, not new code. |
| Commercialization gate | 90-day pilot results (§16) become the case study for selling to other solo agents. Indicative future price: $100–150/mo — **pricing/billing is NOT built in V1**. |
| Budget — AI + media | ≤ $100/month (hard stop on generation jobs). Covers LLM calls, transcription, media processing. |
| Budget — infrastructure | Separate line, target ≤ $75/month: hosting, Supabase, object storage, transactional/bulk email provider, sending domain. Not subject to the AI hard stop. |
| Ad spend | Separate, manually managed in Meta Ads Manager, NOT optimized by V1. V1 only reads spend for reporting. |
| Time zones | Market time zone (pilot: America/Chicago) drives report times and quiet hours. Each user's personal time zone is configurable and shown alongside market time (the operator may be UTC+5:30). |

---

## 04. User Job

> "When I give this tool 30 minutes a day, it turns my content and my ads into qualified seller and buyer/tenant conversations, tells me exactly which video or ad produced each person, drafts replies in my voice, and never sends anything I didn't approve."

The Agent is NOT buying a content app or a CRM. She is buying **answered buying signals with attribution**.

---

## 04a. Content Benchmarks

Five Houston producers were studied, each for one lesson. Business facts come from public web pages read on 7 Oct 2026. Reel and YouTube performance comes from a one-off collection of public post data on 7 Oct 2026 (file: `benchmarks/producer_benchmark.csv`). That collection was market research; the product's no-scraping rule (§01) is unchanged. Figures marked *unverified* could not be confirmed against a primary source.

| Producer | Lesson | Verified facts | Source |
|---|---|---|---|
| **Julia Wang** — founder/broker, NextGen Real Estate | **Reach** | IG @juliawang_htx: 468K followers (FeedSpot, updated 16 Sep 2026); profile positioned as "Luxury Lifestyle · Travel · Home · Beauty · Fashion". Founded NextGen Nov 2020. Runs a monthly e-newsletter, lifestyle blog, virtual tours and weekly open houses. | feedspot.com; nextgen.realestate/blog/what-it-takes-to-build-an-empire |
| **Sahar Khatib** — Keller Williams Realty Southwest, solo | **Solo-agent model** | Houston Agent Magazine 2026 Social Media Influencer of the Year – Agent. Over $31M sold in 2024 and over $24M in 2025 (her site). Licensed 2017; Top 10 Solo Agent, KW South Texas Region 2022–24. Site calls to action: Schedule a Consultation, "What's My Home Worth?", Seller's Guide, neighborhood guides. Her site says she "works solely off referrals and her sphere of influence". IG handle is **@saharzkhatib**. | houstonagentmagazine.com (28 Sep 2026); saharkhatib.com |
| **Nancy Almodovar** — CEO, Nan & Company Properties | **Brand → business** | Founded 2014; nearly $1B annual volume; 200+ agents; in-house production division (Nan Studios) producing social, photo, video, blog and email. Describes the firm as "a marketing company that just happens to sell real estate". IG @nancy_almodovar: 40,667 followers (7 Oct 2026 collection). | nanproperties.com/blog; houstonchronicle.com; houston.culturemap.com |
| **Nicole Freer** — Nicole Freer Group, Corcoran Genesis | **Volume + video** | 1,166 homes and over $448M in 2024; $1B+ career since 2013; recognized by Tom Ferry as a top-10 all-around video influencer; #2 team in Houston metro. Active on YouTube (@nicolefreergroup), Instagram, Facebook and LinkedIn. | nicolefreergroup.com/agents/nicole-freer |
| **Paige Martin / Houston Properties Team** — Real Brokerage | **Conversion system** | 525+ leads a month; 36,000+ database contacts; #1 Houston large team by volume, $169.45M (RealTrends Verified 2026); $2B+ career. 60+ buyer guides and 10,000+ pages of Houston content; ~400K site visitors a year; free neighborhood-guide download with email capture; an owned network of neighborhood websites that routes inquiries to agents. | join.houstonproperties.com; houstonproperties.com/houston-top-realtor |

**Reel performance (Instagram).** Medians, because views are heavily skewed (Wang's mean is 143K against a median of 28K, after one reel reached 6M). "Views" are Instagram plays. Collab reels owned by other accounts are excluded. Windows marked *partial* are shorter than 12 months because the collection hit its cap.

| Account | Followers | Reels / week | Median views | Median ÷ followers | Lead-CTA rate | Top format (n ≥ 5) | Window |
|---|---|---|---|---|---|---|---|
| @saharzkhatib (Khatib) | 4,249 | 3.1 | 1,165 | **0.27** | **74%** | F3 Price-myth buster (n=12, median 1,492) | 12 mo |
| @nicolemfreer (Freer, personal) | 21,376 | 12.5 | 3,615 | 0.17 | 15% | F2 Playbook tip (n=5, median 4,455) | partial, from 17 Aug |
| @nancy_almodovar (Almodovar) | 40,667 | 7.2 | 5,279 | 0.13 | 5% | Lifestyle, no RE angle (n=52, median 6,951) | partial, from 30 May |
| @nicolefreergroup_ (Freer, team) | 12,398 | 1.8 | 1,270 | 0.10 | 11% | F5 Listing walkthrough (n=6, median 1,555) | 12 mo |
| @houstonpropertiesteam | 6,580 | 5.9 | 568 | 0.09 | 6% | F4 Neighborhood guide (n=96, median 658) | partial, from 23 Apr |
| @juliawang_htx (Wang) | 468,258* | 7.8 | 28,161 | 0.06 | 4% | Lifestyle, no RE angle (n=51, median 24,714) | partial, from 13 Aug |
| @nanproperties (Nan & Co) | 24,680 | 7.1 | 883 | 0.04 | 9% | Lifestyle, no RE angle (n=5, median 1,966) | partial, from 18 Aug |

\*Wang's profile is age-restricted; follower count is third-party (FeedSpot, Sep 2026). Formats were classified by keyword against the F1–F7 scheme.

**YouTube.** Medians come from a most-popular sort and are biased upward.

| Channel | Subscribers | Videos | Total views | Notes |
|---|---|---|---|---|
| Nancy Almodovar | 3,780 | 955 | 3.42M | One video holds 2.41M |
| Houston Properties Team | 1,280 | 690 | 1.89M | Long-form library; supports the search-led model |
| Nan and Company Properties | 7,930 | 1,349 | 454K | |
| Nicole Freer Group | 580 | 608 | 125K | |
| Sahar Khatib | 50 | 66 | 33K | |

**What the reel data shows.**
1. **CTA discipline beats reach.** Khatib has the smallest account but the highest views-to-followers ratio (0.27–0.31) and the only high CTA rate (74%; others 4–15%). She is the closest model to PropReel.
2. **The biggest reach is lifestyle content.** Wang's and Almodovar's top formats have no real estate angle and carry CTAs on 4–5% of reels. Their views do not show lead generation.
3. **Personal accounts outperform company accounts.** Almodovar 0.13 vs @nanproperties 0.04; Freer personal 0.17 vs team 0.10. PropReel runs on the Agent's personal account.
4. **Houston Properties Team wins on search and YouTube, not Instagram** (median 568 views; 1.89M YouTube views). This supports the §18 SEO path.
5. **4 reels a week is realistic** for a solo agent: Khatib posts 3.1, the larger accounts 6–12.
6. **Collab posts are common.** Excluded collab reels: Freer team 58, Almodovar 23, Houston Properties Team 20, Nan & Co 10, Freer personal 9, Khatib 5. Post kits support collaborator accounts (§11).

**What the benchmarks mean for PropReel.**
- **Reach is not the goal; capture is.** Wang's audience is mostly lifestyle content and took years to build. PropReel adopts the *consistent face and personal story* format (one flex slot a week) but every piece still carries a keyword CTA (§09).
- **Social works best as an amplifier of the sphere for a solo agent.** Khatib is the closest analogue to the Agent (KW, solo), and her own site credits referrals and sphere. PropReel therefore treats past clients and sphere as a core audience: past-client nurture (WF-9), re-permission (WF-11), and `referral` source tracking (WF-4) stay first-class.
- **A production system, not a production team.** Almodovar's edge is an in-house studio. The solo-scale equivalent is the WF-7 pipeline: briefs, filming cards, captions and kits produced on a fixed weekly cadence.
- **Video consistency at volume.** Freer shows that listing and market video can carry a high-volume business. PropReel's listing walkthrough and market-minute formats come from this model.
- **Content → owned asset → database → nurture.** The Houston Properties Team funnel is the long-term model: useful neighborhood content earns search traffic, a guide download captures the email, and the database is nurtured. V1 adopts the **neighborhood-guide lead magnet** (double opt-in, AC-16) and a **home-value request** form (from Khatib's site). SEO neighborhood pages are deferred to §18.

**Mapping to features.**
| Lesson | PropReel feature |
|---|---|
| Reach (Wang) | Weekly personal-story slot; hook library in the Reel Playbook; monthly market-update email (WF-9 past-client segment) |
| Solo model (Khatib) | Home-value request form (T1); referral source tracking; past-client nurture; per-piece keyword CTAs on Houston-topical reels |
| Brand → business (Almodovar) | WF-7 pipeline cadence; consistent brand voice file; kit templates |
| Volume + video (Freer) | Listing-walkthrough and seller-playbook formats; transcript-driven captions |
| Conversion system (Houston Properties Team) | Neighborhood-guide lead magnet with double opt-in; nurture sequences; database hygiene (WF-10); SEO pages in §18 |

**Patterns to avoid** (seen in the category generally, not attributed to any producer): describing who lives in a neighborhood or who it "suits"; "safe" or "family-friendly" area claims; school-quality rankings used to characterize an area; lifestyle content with no CTA; guarantee or investment-return language. All are blocked or flagged by §10.

**Sources (read 7 Oct 2026):**
- https://influencers.feedspot.com/houston_real_estate_instagram_influencers/
- https://nextgen.realestate/blog/what-it-takes-to-build-an-empire
- https://houstonagentmagazine.com/2026/09/28/social-media-influencer-of-the-year-agent-sahar-khatib/
- https://www.saharkhatib.com/
- https://nanproperties.com/blog/how-nan-and-company-became-a-top-houston-luxury-real-estate-firm
- https://www.houstonchronicle.com/business/texas-inc/article/Nan-Co-CEO-Nancy-Almodovar-on-tech-lifesyle-15086561.php (quote via search summary; article paywalled)
- https://houston.culturemap.com/news/real-estate/08-22-19-nan-and-co-properties-nancy-almodovar-christies/
- https://nicolefreergroup.com/agents/nicole-freer
- https://join.houstonproperties.com/
- https://www.houstonproperties.com/houston-top-realtor
- `benchmarks/producer_benchmark.csv` — Instagram reel and YouTube channel data collected 7 Oct 2026

---

## 05. Trigger

**T1 — Organic signals** (real-time where the platform supports webhooks):
- New comment or DM on connected Instagram / Facebook accounts (webhooks)
- New comment on connected YouTube channel (**polled** every 15 min — YouTube offers no comment webhooks and no DMs; daily API quota 10,000 units)
- Comment/DM keyword: base intents `SELL`, `VIEW`, `RENT`, `OFFER`, plus per-piece keywords (e.g., `SELL-HEIGHTS`) issued by the post kit (high-confidence intent + attribution)
- Link-in-bio router click or tracked-link click (engagement event, may precede a form)
- Landing-page form submission (listing consultation / direct-offer request / home-value request / availability-viewing)
- Lead-magnet request: Houston neighborhood guide (email capture, double opt-in per T4)
- Calendly booking (seller call/visit, buyer/tenant viewing)
- Manual paste capture (phone calls, in-person, offline ads)

**T2 — Paid signals** (real-time webhook):
- Meta lead-ad instant-form submission → `leadgen` webhook (delivers `leadgen_id` only) → fetch full lead via Graph API → Signal Agent

**T4 — Email events:**
- New inbound business email (Gmail API watch/webhook)
- Lead-magnet request (email capture → double opt-in confirmation before delivery)
- Nurture engagement events (open / click / reply) → score updates; **any reply stops all sequences** and routes the message to T1
- Mailing-list import (past clients, cold leads) — requires email, source, consent status, capture date

**T3 — Scheduled** (cron):
- Daily 06:30 market time: attribution recompute + morning report delivery 07:30 market time
- Weekly Monday 06:30: generate **7 UGC briefs** for the week (see WF-7)
- Every 15 min: YouTube comment poll; reply-window expiry check

---

## 06. Workflow

Master loop for every signal: **Trigger → Input → Retrieve → Analyze → Decide → Complete.**

**WF-1 · Paid ad lead (T2):**
`leadgen` webhook (`leadgen_id`) → idempotent fetch of full lead from Graph API (retry with backoff; `leadgen_id` is the idempotency key) → fields + consents + campaign/ad-set/form name + timestamp → retrieve property availability, KB answer, brand voice → classify lead type + intent → dedupe (email/phone) → create/update lead, tag source = `paid_ad` + campaign → score → draft reply (standard template for buyer/tenant; brand-voice for seller) → compliance check → **Agent approval (1:1)** → send → log + attribution.

**WF-2 · Organic comment/DM (T1):**
Webhook/poll → message text + author + source post (if any) → retrieve source post kit, property availability, KB → classify (lead type, intent, urgency, entities) → if buying signal: create lead, source = `organic_content` + post-kit ID (resolved per §09 Attribution) → score → draft reply → compliance check → **Agent approval (1:1)** → send within platform reply window → log. Each queued item shows a **reply-window countdown** (see §09 Reply windows).

**WF-3 · Keyword comment:** classified as high-confidence intent; a per-piece keyword also resolves the source content piece. Lead created immediately. The private-reply DM ("Thanks! Just DM'd you") is drafted but still approval-gated, and sorted to the top of the queue by reply-window expiry.

**WF-4 · Form/booking (T1):** as WF-1 minus ad metadata; source = `organic_content` (UTM / router piece ID) or `referral` (if referred by past client — the referral channel still runs through the same inbox and gets attributed). Direct-offer forms display the license-status disclosure from the state pack.

**WF-5 · Investor-intent message** ("How can I invest with you?"): routed to *Investor interest (on hold)* list; neutral thank-you draft; Agent alerted; **no scoring, no sequences, no offering details** in V1.

**WF-6 · Accommodation request** (assistance animals, accessibility): escalated to Agent immediately; **no refusal is ever drafted**; "no pets" templates are excluded for this lead.

**WF-7 · Content Production Pipeline (T3 + on-demand commands):** every reel moves through explicit stages, each with a gate:

| Stage | Slash command | What the system does | Gate to exit stage |
|---|---|---|---|
| 1 · Prep | `/reel-prep` | Generate brief: hooks, script/talking points, shot list, filming checklist. Weekly batch = **7 briefs, one per format** (see below); more on demand. | Fair-housing + claims pre-check passes → Agent approves brief |
| 2 · Film/Edit | `/reel-edit` | Agent films on phone → uploads → auto-transcribe → captions/SRT, on-screen text, hook variants from transcript | Asset ingested + tagged |
| 3 · Kit | `/reel-kit` | Build post kit: per-platform captions, hashtags, disclosure tags, **unique per-piece keyword CTA**, link-in-bio router entry, thumbnail text | Rights + compliance checks pass → Agent approves kit |
| 4 · Post | `/reel-post` | Agent publishes manually → pastes live URLs (or auto-detected via account sync) | Post URL registered → attribution live |
| 5 · Measure | (automatic) | Nightly: views, comments, keyword hits, router clicks, leads attributed to this content piece | Feeds "film this next" suggestion |

**Weekly format mix** (from §04a; full specs in the companion *PropReel Reel Playbook*):

| # | Format | Audience | Default CTA | Benchmark source |
|---|---|---|---|---|
| F1 | Houston Market Minute | Seller | `WORTH-[AREA]` → home-value request | Khatib, Houston Properties Team |
| F2 | Seller Playbook tip | Seller | `SELL-[TOPIC]` → listing consultation | Freer |
| F3 | Price-myth buster / "What's it worth?" | Seller | `WORTH-[AREA]` → home-value request; `OFFER` for direct-offer variant | Khatib |
| F4 | Neighborhood guide (property-focused) | Buyer/tenant | `GUIDE-[AREA]` → neighborhood-guide lead magnet | Houston Properties Team |
| F5 | Listing walkthrough | Buyer/tenant | `VIEW-[PROPERTY]` → viewing request | Freer |
| F6 | Answering your DMs (buyer/renter Q&A) | Buyer/tenant | `RENT` / `VIEW` → qualifying reply | Khatib |
| F7 | Flex: behind the deal / personal story / Houston event | Either | Piece keyword → link-in-bio | Wang, Almodovar |

The Agent may swap any format in a given week; the mix of 3 seller, 3 buyer/tenant, 1 flex remains the default.

**Evidence by format (§04a reel data).** Strongest: **F3** (Khatib's top format, median 1,492, n=12) and **F2** (Freer personal top format, median 4,455, n=5). Moderate: **F5** (Freer team top format, median 1,555, n=6). Weak on Instagram: **F4** (Houston Properties Team, median 658, n=96); keep it for the guide lead magnet and future SEO pages, not for reach. **F7** is optional each week and must carry either a real estate angle or a keyword CTA; pure lifestyle reels (the top format for Wang and Almodovar) get reach but carry CTAs on 4–5% of reels.

**Prompt library:** stages 1–3 are driven by saved, versioned prompt templates (`/reel-prep`, `/reel-edit`, `/reel-kit`) in the Agent's brand voice — reusable on demand (not only via the weekly schedule), editable by the Agent, with every output storing the template version used.

**WF-8 · Inbound email inquiry (T4):** business email arrives → thread + sender → retrieve KB, brand voice, property availability → classify (lead type + intent) → dedupe against existing contacts → create lead, source = `email` → score → draft reply → compliance check → **Agent approval (1:1)** → send from the Agent's mailbox (Gmail API) → log.

**WF-9 · Nurture sequence engine (T4):** nightly job → for each enrolled lead where `scheduled_for <= now` and no stop-rule has fired → render the stage's **Agent-approved template** (template-segment approval, §10.10) with allowed personalization fields → Sender per-recipient checks (consent, suppression, caps, quiet hours, approval validity) → send via the authenticated sending domain → log; open/click/reply webhook events write back to the lead timeline (engagement feeds the responsiveness score). **Seller sequences and any free-text personalized message use 1:1 approval instead.**

**WF-10 · Cold-list re-engagement (T4):** contacts with no opens in 90 days → single "still thinking of selling / still looking?" re-engagement email → any response routes to T1 as a hot signal → still silent after send + 14 days → suppress to `cold` list (no further bulk).

**WF-11 · Re-permission (T4):** for each `consent_unknown` contact (typically imported past clients) → draft a short personal 1:1 email from the Agent's mailbox asking whether they'd like to receive her monthly market update (single clear yes link) → **Agent approval (1:1, batch-reviewable list in queue)** → send → click "yes" = `email_opt_in` recorded with timestamp/source and contact moves to the matching nurture segment → no response after 21 days = stays `consent_unknown` (1:1 only, never bulk); one attempt per contact per 12 months.

---

## 07. Inputs

| Input | Required | Notes / Missing-data behavior |
|---|---|---|
| Lead/contact: name or handle | Required | Anonymous click without identity = engagement event only, no lead |
| Channel + external message ID | Required | Used for dedupe; missing → cannot send, escalate |
| Source: post-kit ID, ad campaign, UTM/router piece ID, or `referral` | Required for leads | Unknown source → lead still created, flagged `source_missing`, counted against the ≥90% attribution target if the lead is organic-content |
| Email and/or phone | One required | Neither → cannot enter sequence; Agent must obtain |
| Consent: email opt-in | Required for bulk/nurture email | Absent → reply via platform DM or 1:1 email only; email opt-in requested conversationally or via WF-11 |
| Consent: TCPA text consent | Optional, **unchecked by default** | Records exact text, timestamp, number, source. Never required in V1 (no SMS). |
| Mailing-list import (past clients, cold leads) | Required: email, source, consent status, capture date | Missing consent status → imported as `consent_unknown`: 1:1-style emails only, **never bulk marketing**, until consent is captured via WF-11 |
| Seller offer type | Required for seller leads | `seller_listing` or `seller_direct_offer`; unknown → `seller_listing` default + draft asks which the seller prefers |
| Property of interest / area | Optional | Missing → draft asks a qualifying question instead of guessing |
| Timeline, budget, condition, occupancy | Optional | Missing motivation data is neutral (never negative); rationale notes "insufficient data" |

---

## 08. Retrieval Rules

| Data | Source | Priority / freshness | Conflict handling |
|---|---|---|---|
| Property availability & approved shareable numbers | Property records (system of record) | Must be updated by Agent; staleness > 14 days → reply drafts must not promise availability; escalate to Agent | Property record wins over KB and drafts. Any change to a property record voids pending approvals that reference it. |
| Brand voice | Active Brand Voice File (versioned) | Always the active version; every draft stores the version used | Manual edits by Agent always win |
| Business facts (process, areas, FAQs) | Knowledge base | Agent-maintained | If KB conflicts with property record → property record wins; flag KB for Agent review |
| Seller/buyer qualification answers | Lead record timeline | Most recent answer wins | Contradictions → Agent sees both, drafts ask gently |
| Past approved replies | Top 20 approved replies, same lead type | Style reference only; never auto-reused verbatim without fresh approval | — |
| Ad campaign metadata | Meta Graph API lead fetch | Immutable, stored with lead | — |
| Ad spend | Meta Marketing API insights (`ads_read`) | Pulled nightly | Read-only; never modifies campaigns |

---

## 09. Business / Decision Rules

**Classification.** LLM classifier (Claude Haiku 5.5) + rules fallback. `lead_type` ∈ {seller_listing, seller_direct_offer, buyer_tenant, investor_hold, other}; `intent` ∈ {sell_request, offer_question, availability, viewing_request, application_question, price_question, accommodation_request, praise, complaint, general, spam}. Rules fire first on keywords (SELL, VIEW, OFFER, per-piece keywords, "still available", "how much", "do you buy"); LLM resolves the rest. "Do you buy houses" / `OFFER` → `seller_direct_offer`; `SELL` / "list my home" → `seller_listing`; ambiguous → `seller_listing` + clarifying question.
**Review threshold:** items below the calibrated confidence threshold go to the human-review tab and are never auto-drafted. The threshold starts at 0.6 and is **calibrated against a labelled eval set** (≥200 real or realistic messages, built in weeks 1–2 by the Agent + builder, extended with every Agent correction) so that ≥95% of above-threshold items are correctly classified.

**Fairness rule for ALL scoring (seller and buyer/tenant).** Scoring may use only the inputs listed below. It **never** uses name, photo, profile text, language, location of the person (as opposed to the property), or any protected characteristic or proxy. Same criteria for everyone.

**Seller scoring (allowed inputs only):** stated timeline · situation as volunteered (inherited, tenant-occupied, repairs, hardship — optional, seller's choice) · property fit vs Agent's listing/buy areas · price-expectation fit · responsiveness · decision-maker clarity. Grades: **A** = timeline ≤90 days + fit + responsive · **B** = any two strong signals · **C** = one signal or no data yet · **D** = unresponsive after 3 delivered touches. *Missing data is neutral; non-response to delivered messages is behaviour, not missing data.* Every grade shows a one-line rationale; Agent can override (logged).

**Buyer/tenant scoring (allowed inputs ONLY):** property match · stated timeline · stated budget vs published price/rent · responsiveness. A–D grades on the same logic. Source-of-income handling follows the state rule pack.

**Hot lead:** any A-grade lead, any accommodation request, any viewing request within 24h → instant alert (push notification + email within 5 min).

**Attribution model.**
- **Primary organic CTA = per-piece keyword** (e.g., `SELL-HEIGHTS`) issued with each post kit, because Reel captions do not support clickable links. Secondary: a **link-in-bio router** page listing live pieces, each link carrying the piece ID; tracked links in Stories / Facebook / YouTube descriptions.
- Resolution order for a lead's source piece: (1) per-piece keyword → (2) comment on a registered post → (3) router/UTM piece ID → (4) DM referencing a shared post → (5) Agent tag at triage → else `source_missing`.
- **Lead credit = first touch** (the piece that created the lead). **Appointment credit = last touch** before booking. Both are stored; reports show both.
- **"Known source" metric denominator = organic-content leads only** (excludes paid, referral, email, manual-capture leads, which are tracked by their own source).

**Reply windows.** Every queued outbound item to a Meta channel carries its deadline: Messenger/IG standard 24h window from the user's last message; Human Agent tag up to 7 days (used only for genuine human follow-up); Instagram private reply to a comment — one message within 7 days of the comment. The approval queue shows a countdown and sorts by soonest expiry; items within 2h of expiry trigger a push notification; expired items convert to an email draft (if consent) or a "window missed" log entry — never a policy-violating send.

**Follow-up sequences (per-message approval for seller; template approval for buyer/tenant standard stages):**
- Seller: same-day reply → +2 days value follow-up → +5 days check-in → stop on reply/booking/unsubscribe/stage-change. 1:1 approval per message.
- Buyer/tenant: **one standard template per stage for every lead** (availability reply → viewing reminder → application next steps). Personalization limited to name, property, times. Template-segment approval. Consistency log records template + response time per lead.
- Channels: platform DM (within the applicable reply window) and email (authenticated domain, CAN-SPAM footer). Quiet hours: recipient-local, no sends 21:00–08:00. Cap: max 3 sequence touches per lead per week.

**Email nurture cadence rules.** Max 2 marketing emails per contact per week · every email has exactly one CTA · engagement-adjusted: opened last 3 emails → maintain cadence; no opens in 90 days → one re-engagement email (WF-10), then suppress to `cold` · **any human reply stops every sequence within 5 minutes** and the thread becomes a T1 signal handled as a live conversation (WF-2) · past-client segment (opted-in only): monthly market update + home-anniversary note; never mixed with active-lead sequences for the same contact.

**Content stage rules.** A brief cannot move to Film until its compliance pre-check passes and the Agent approves it · a post kit cannot be approved until rights + compliance checks pass · a content piece is only "done" when its live post URL is registered (otherwise attribution is broken and it shows as stuck in Post) · stale rule: any piece sitting in one stage >7 days appears on the morning report as "stuck content" with a one-tap action (re-draft, re-shoot, or kill).

---

## 10. Compliance & Consent Rules

1. **Fair housing — strict mode on ALL public content, forms, replies, sequences, and scoring** (seller and buyer/tenant). Blocking (not flagging). No override except editing the text. Property-focused language only; no stated or implied preferences about who should live somewhere; no steering language (e.g., "great for young families", "safe neighborhood", school-quality claims used to characterize residents). The known-bad phrase test suite must pass 100%. **Phrase sets:** known-bad and known-good sets are built from HUD advertising guidance and the state pack, owned by the builder, reviewed by the Agent, and versioned.
2. **Housing Special Ad Category.** Every Meta campaign is created manually with the Housing category. Ad creative and copy pass the fair-housing checker before the Agent approves spend. V1 does not build ads — it only ingests the leads they produce, reads spend, and attributes them.
3. **State rule pack gate.** Seller-offer (listing or direct-offer) pages, post kits, or replies for a property in a state without a lawyer-reviewed rule pack **cannot be approved**. V1 ships one pack (Texas), placeholder until the Agent's lawyer marks it reviewed. The pack contains: advertising/broker-name rules, IABS notice, **licensee-as-principal disclosure text (TREC 535.144) for direct offers**, protected classes incl. state/local additions, source-of-income handling, privacy-law obligations (Texas: TDPSA), and required footer text.
4. **FTC endorsements.** Any paid/gifted/incentivized content requires a disclosure tag before approval. No fabricated or AI-generated testimonials; testimonial content = interview questions only.
5. **CAN-SPAM + mailbox-provider rules.** Every commercial email: accurate sender/subject, business postal address, working one-click unsubscribe (RFC 8058 `List-Unsubscribe-Post`). Opt-outs suppress within 24h (stricter than the 10-business-day legal limit), permanently, across all sequences. Bulk-sender thresholds: spam complaints kept <0.1% (provider limit 0.3%).
5a. **Marketing vs transactional split.** 1:1 replies, confirmations and re-permission emails send from the Agent's mailbox (Gmail API). Bulk nurture sends only from an authenticated sending subdomain (SPF/DKIM/DMARC, `List-Unsubscribe` header). Lead magnets use double opt-in. The two sender identities are never mixed in one thread.
6. **TCPA readiness.** No SMS or calls exist in V1 (no adapter). Forms record the optional, unchecked text-consent box (exact text, timestamp, number, source) for future phases.
7. **Brokerage disclosure.** Licensee name + broker name + state-required notice (e.g., Texas IABS link) auto-appended to landing pages, post kits, and email footers per the state pack. Direct-offer materials additionally carry the licensee-as-principal disclosure.
8. **Claims check.** Blocks guaranteed price, guaranteed closing date, guaranteed returns, or unverifiable "cash offer" promises in any draft. Direct-offer copy may state that the Agent makes offers, never that an offer amount or closing is guaranteed.
9. **Consent & privacy.** Privacy notice on every form; contact-level consent records; suppression list enforced by the Sender; data access/export/deletion on request per the state pack's privacy law (Texas: TDPSA; design is also CCPA/CPRA-ready for future states); US data hosting.
10. **Approvals & audit trail.** Every approval, send, override, and compliance decision is append-only, retained ≥1 year. Two approval types:
    - **1:1 approval** — SHA-256 hash of exact content + recipient + channel. Used for all replies, seller sequences, re-permission emails, and any free-text message.
    - **Template-segment approval** — SHA-256 hash of template version + segment rule snapshot + allowed personalization field list + channel. Used for buyer/tenant standard stages and nurture emails. Rendered output may differ only in the allowed fields; the Sender re-checks each recipient (consent, suppression, caps, quiet hours) at send time.
    - Any edit to content, template, segment rule, or field list voids the approval. **Approvals expire after 72 hours** if unsent (1:1) or after 30 days (template-segment), and immediately when a referenced property record changes.
11. **Investor content.** Any investment-return language is blocked everywhere in V1; investor inquiries → hold list (WF-5).

---

## 11. User Experience

Six screens (mobile-first PWA). Default landing screen: **Today**.

1. **Today** — morning report inline: hot leads, approvals waiting (with soonest reply-window expiry), today's appointments/viewings, new leads by type/grade, pipeline snapshot, top lead-producing content (7d), issues, ad spend, one recommended action.
2. **UGC Studio** — content kanban across the 5 pipeline stages (Prep → Film/Edit → Kit → Post → Measure) with per-stage counts and stuck-content badges; weekly plan view; brief editor (hooks, script, shot list, filming checklist); slash-command prompt library panel; compliance result inline; teleprompter card for filming.
3. **Post Kits** — per-platform captions, SRT, per-piece keyword CTA, link-in-bio router entry, disclosure tags, optional collaborator accounts (Instagram Collab), compliance + rights panel, approve → copy/download → "mark as posted" with live URL. A collab reel is registered by its URL like any other, so its comments and keyword hits count toward the source piece.
4. **Inbox + Approval Queue** — unified comments/DMs/emails/form leads with type/intent badges and reply-window countdowns; tabs: 1:1 approvals · template approvals · escalations (complaints, accommodation, low-confidence) · re-permission batch; approve/edit/regenerate/skip; kill switch. Helper sees the queue read/draft-only.
5. **Lead Inbox + Pipelines** — tabs: Sellers (Listing / Direct offer) · Buyers/Tenants · Investor hold; grade-sorted; lead detail with timeline, score rationale, consent state, sequence status, first-touch and last-touch source; Kanban pipelines with values and stale indicators.
6. **Settings** — connections (Meta, YouTube, Gmail, Calendly) with permission status; users & roles (Agent, Helper); notifications (push setup); brand voice; properties + availability; compliance center (rule-pack review status, screening criteria, suppression list, postal address); time zones; budget gauges (AI/media and infrastructure).

**Notifications.** Hot-lead and reply-window alerts are delivered by web push. On iOS, web push works only after the PWA is added to the Home Screen — onboarding (AC-1) walks the Agent through this and sends a test push. Email alert is always sent in parallel as fallback.

**States:** every list has explicit empty (with next action), loading, error (retry), and awaiting-approval states. Approval queue is optimized for one-tap mobile approval.

---

## 12. System Behavior

| Layer | Choice |
|---|---|
| App | Single full-stack Next.js app (React/TypeScript): PWA screens (Tailwind; mobile upload + filming cards; web push) and server code in one codebase |
| API | Next.js API routes: app API for the PWA and webhook receivers (Meta leadgen + IG/FB comments/messaging, Calendly, Gmail push, email provider events); link-in-bio router |
| Scheduled jobs | Supabase Cron (pg_cron, calling protected API routes): YouTube polling (15 min), reply-window checks, nightly sequences, 06:30 attribution + spend pull, 07:30 morning report, Monday 06:30 weekly briefs, and retry of failed work (e.g., Meta lead fetch, retried with backoff for 24 h) from a jobs table. Event-driven work (classify → score → draft on a new message) runs from the webhook's API route. LLM calls through the Anthropic TypeScript SDK with a provider abstraction — **Claude Haiku 5.5** (`claude-haiku-5-5`) for classification, **Claude Sonnet 5.5** (`claude-sonnet-5-5`) for drafts/briefs |
| Sender service | The only component able to send. Verifies role, approval hash + type + TTL, consent, suppression, quiet hours, caps, reply window, kill switch — refuses on any failure |
| Media | Direct upload → object storage (S3/R2, signed URLs) → FFmpeg preview → transcription (Groq Whisper API) → auto-tags → transcript search. **V1 scope: upload, transcribe, tag, search. No auto-editing.** |
| Integrations | Meta Graph API (comments, messaging, leadgen, read-only ads insights), YouTube Data API (comment polling + replies), Gmail API (owner mailbox), Resend (bulk + transactional), Calendly |
| Email sending | Dual-sender: Gmail API for 1:1 (owner mailbox); Resend for bulk nurture from an authenticated subdomain (e.g., `mail.brand.com`) with SPF/DKIM/DMARC; open/click/reply via webhooks; **domain warm-up starts in build week 2** (≤20/day, ramp ~3 weeks) so bulk is ready when the nurture engine ships |
| Database | Supabase (Postgres); SQL migrations; pgvector for transcript search; core tables in §12a; row-level security by workspace and role |
| Security | OAuth tokens KMS-encrypted; MFA required for both roles (Agent approval rights activate only after MFA); signed expiring links; append-only audit log |

**Meta permissions (submit for App Review + Business Verification in build week 1):** `leads_retrieval`, `pages_manage_metadata`, `pages_show_list`, `pages_read_engagement`, `pages_messaging`, `instagram_basic`, `instagram_manage_comments`, `instagram_manage_messages`, `ads_read`. Until approved: development-mode access on the Agent's own accounts + **manual lead capture fallback** (Meta instant-form leads CSV export/import → dedupe on import; manual paste for DMs/comments). Instant forms work without messaging permissions, so paid leads can flow before DM automation is approved.

### 12a. Core data model

| Table | Key fields | Notes |
|---|---|---|
| `workspace` | id, market_tz, state_pack_id | One per agent in V1 |
| `user` | id, workspace_id, role (`agent` / `helper`), tz, mfa_enabled | Only `agent` can approve |
| `contact` | id, name/handle, emails, phones, segment, status (`active`/`cold`/`suppressed`) | Dedupe on email/phone/platform ID |
| `consent` | contact_id, type (email_opt_in, tcpa_text, consent_unknown), text, source, timestamp | Append-only |
| `suppression` | address, reason, timestamp | Checked by Sender on every send |
| `lead` | id, contact_id, lead_type, intent, grade, grade_rationale, first_touch_source, last_touch_source, stage, value | One contact may have multiple leads |
| `message` | id, lead_id, channel, external_id (unique), direction, body, received_at, reply_window_expires_at | Idempotency on external_id |
| `draft` | id, message_id/enrollment_id, body, template_version, brand_voice_version, compliance_result | |
| `approval` | id, type (`one_to_one`/`template_segment`), hash, approved_by, approved_at, expires_at, voided_at | |
| `send` | id, draft_id, approval_id, sender_identity, status, sent_at, refusal_reason | |
| `property` | id, address, state, availability, shareable_numbers, updated_at | Change voids linked approvals |
| `content_piece` | id, stage, brief, keyword, post_urls, stage_entered_at, prompt_template_version | |
| `post_kit` | id, content_piece_id, per-platform captions, srt, router_slug, collaborator_accounts, compliance_result | |
| `sequence_enrollment` | id, lead_id/contact_id, sequence_id, step, scheduled_for, stopped_reason | |
| `prompt_template` | id, command, version, body | Versioned |
| `audit_event` | id, actor, action, entity, payload_hash, timestamp | Append-only, ≥1 yr |

**Readiness:** see Meta permissions above.

---

## 13. Human vs System Responsibility

| Action | Who | Enforcement |
|---|---|---|
| Classify, score, dedupe leads | System (Agent can correct; corrections feed eval set) | — |
| Generate briefs, scripts, captions, reply drafts | System (Helper may also draft/edit) | — |
| Triage inbox, tag content | Helper or Agent | Audit log |
| **Approve any outbound message or template** | **Agent only** | Hash-matched approval (1:1 or template-segment); Sender refuses otherwise |
| **Approve post kits / briefs** | Agent only | Same gate |
| **Set property availability, shareable numbers** | Agent | Stale availability blocks availability-promising drafts |
| Publish posts | Agent (manual, V1) | No publish tool exists |
| Move pipeline stages | Agent (system suggests from events) | Audit log |
| Mark state rule pack reviewed | Agent + her lawyer | Seller-offer and direct-offer approvals blocked until reviewed |
| Send anything | Sender service only | Verifies role + hash + approval type + TTL + consent + suppression + quiet hours + caps + reply window; kill switch stops all sends ≤60s |
| Override a score | Agent | Logged with reason |

---

## 14. Failure Modes

| Failure | Behavior |
|---|---|
| LLM provider down | Rules-based keyword classification continues; drafts pause; Agent alerted; nothing is silently skipped |
| Meta API down / app review pending | Manual paste capture + CSV lead import; queue-and-retry webhooks; dedupe by external ID prevents double-processing on replay; Settings shows permission status |
| Lead fetch after `leadgen` webhook fails | Retried with backoff for 24h, keyed on `leadgen_id`; then Agent alerted to import via CSV |
| YouTube API quota exhausted | Polling pauses until quota reset; Agent notified; no data lost (next poll catches up) |
| Duplicate webhook delivery | Idempotency keys on external message ID; second delivery updates, never duplicates |
| Stale property availability (>14 days) | Reply drafts avoid availability claims; lead flagged; Agent prompted to update |
| Approval expired or property changed | Approval void; item returns to queue with reason |
| Reply window about to expire / expired | Push alert at T-2h; on expiry, converts to email draft (if consent) or logs "window missed" — never sends outside policy |
| Missing consent for email | Platform-DM or 1:1 reply only; consent request drafted conversationally |
| Suppressed / unsubscribed contact | Sender hard-blocks; attempted enrollment surfaces an error, not a send |
| Message edited after approval | Approval void; returns to queue |
| Helper attempts approval or send | Refused by RBAC and Sender; logged |
| Budget exhausted (AI/media) | Hard stop on generation jobs; **approved sends continue**; gauge shows 80% warning |
| Kill switch activated | All pending sends halted ≤60 seconds; report shows what was paused |
| Email bounce rate >2% or spam complaints >0.1% | Campaign pauses automatically; affected segment flagged for list hygiene; Agent alerted before any resend |
| Sending-domain reputation damage | Bulk sending halts; 1:1 mailbox unaffected; domain requires re-warm-up before bulk resumes |
| Contact replies to a nurture email | Every sequence stops ≤5 minutes; the reply becomes a T1 signal and is treated as a live human conversation from then on |
| Bulk send attempted to a `consent_unknown` contact | Sender refuses; contact flagged for the re-permission flow (WF-11) |
| Push notification not set up / fails | Email alert always sent in parallel; Today screen banner prompts setup |
| Inbound prompt injection (malicious comment/DM text) | Inbound treated as untrusted data; agents hold no send tools; drafts never execute embedded instructions |

---

## 15. Non-Goals (V1)

- Automated publishing/scheduling (manual post kits only)
- Contributor portal, paid UGC creators, usage-rights system (owner-shot + Agent-cleared testimonial content only; releases tracked as simple consent records)
- Video auto-editing / repurposing (text-level variants only)
- SMS, calls, cold outreach of any kind
- Ad campaign management or AI ad optimization (ads are created manually; V1 ingests leads and reads spend only)
- Payments, invoicing, creator payouts
- Mass newsletter broadcasting / ESP-style campaigns (nurture sequences only)
- Teams beyond the two V1 roles (Agent + Helper), multi-workspace, billing
- Investor/capital-partner pipeline
- Real-estate CRM integrations beyond CSV/webhook export and Google Sheets
- Markets outside the pilot state/metro
- YouTube DMs (the platform has none) and YouTube real-time webhooks

---

## 16. Success Metrics

**Day 0 = the first day lead capture is live** (manual paste or CSV import counts). The 90-day pilot and all checkpoints run from Day 0. Baselines are measured in the 7 days before Day 0: current referral leads/mo, current response time to DMs (expected: often missed), current content output.

**Permissions-pending mode:** while Meta App Review is pending, organic DM/comment leads captured manually still count toward Track A; the report shows "permissions pending" alongside checkpoint results. If approval has not arrived by Day 30, the Agent decides whether to extend the pilot by the delay.

**Track A — Organic content engine:**
| Metric | Day 45 checkpoint | Day 60 checkpoint | Day 90 target |
|---|---|---|---|
| Inbound leads from content (cumulative) | ≥6 | ≥14 | **20–30** |
| Organic-content leads with known source piece (denominator: organic-content leads only) | ≥80% | ≥85% | ≥90% |
| Post kits published/week | ≥2 | ≥3 | ≥4 |
| Brief → Posted conversion (content throughput) | — | — | ≥60% of approved briefs reach Post within 14 days (7 briefs/week × 60% ≈ 4.2 posts/week) |
| Median first-response time (business hours) | ≤4h | ≤3h | ≤2h |

*Benchmark check (§04a):* the Houston Properties Team routes 525+ leads a month, but with 14 agents, operating since 2002, 10,000+ content pages and ~400K yearly site visitors. A solo agent starting from a referral base cannot be measured against that. The 20–30 lead target for 90 days stays unchanged; revisit it after Day 90 using the Agent's own lead-per-post rate.

**Diagnostic content metrics** (reported weekly, not pass/fail; follower count baselined in the week before Day 0):
| Metric | Day 45 | Day 90 | Benchmark |
|---|---|---|---|
| Median reel views ÷ followers (personal account) | ≥0.15 | ≥0.25 | Khatib 0.27–0.31; Freer personal 0.17 |
| Reels carrying a lead CTA (keyword) | ≥90% | ≥90% | Khatib 74%; others 4–15% |
| Leads per posted reel | tracked | 0.4–0.6 | 20–30 leads ÷ ~50 reels over 90 days |
| Reels posted per week | ≥2 | ≥4 | Khatib 3.1; Houston Properties Team 5.9 |

If views ÷ followers is on track but leads per reel is low, change the CTA and format mix before raising volume.

**Track B — Paid ads (tracked separately, never blended with organic):**
| Metric | Target |
|---|---|
| Ad-sourced leads with campaign attribution | 100% |
| Cost per lead | Within Agent's approved CPL band per campaign (Agent sets band; system reports using `ads_read` spend) |
| Note | Volume target set by Agent per budget; ads must not be used to mask organic underperformance |

**Track C — Outcome (the honest north star):**
| Metric | Day 90 target |
|---|---|
| Appointments booked (seller calls/visits + viewings), all sources | 15–20 |
| Qualified pipeline value created | Baseline in month 1, then trend |
| Seller split | Reported separately for listing vs direct-offer |

**Track D — Email nurture program** (list sized for 100–500 contacts — solo-agent scale; volume comes from consistency, not list size):
| Metric | Day 90 target |
|---|---|
| Open rate — personal-style 1:1 nurture | ≥35% |
| Open rate — broadcast-style updates | ≥20% |
| Reply rate on nurture emails | ≥3% |
| Unsubscribe rate per send | <0.5% |
| Bounce rate | <2% |
| Re-permission opt-in rate (WF-11) | ≥25% of `consent_unknown` contacts emailed |
| Past-client / cold-list reactivation | ≥5 re-engaged conversations |

**Hard requirements (pass/fail):** 0 sends without valid recorded approval · 0 sends to suppressed contacts · 0 public items or buyer/tenant/seller messages published/sent that fail the fair-housing check · 100% of commercial emails with postal address + working unsubscribe · 0 direct-offer materials without the licensee-as-principal disclosure · 0 SMS/calls sent (no adapter exists) · 0 approvals or sends by the Helper role.

**Owner time:** median ≤30 active min/day excluding filming.

**Go/no-go for V2 (commercialization build):** Track A day-90 target met OR clearly trending (≥10 leads in final 30 days) **and** Track C ≥8 appointments. Otherwise: content mix and CTAs change; V2 build does not start on hope.

---

## 17. Acceptance Criteria

- **AC-1** Given onboarding, when the Agent completes setup (lead types, brand voice, compliance settings, one property, connections, push notifications incl. iOS Home Screen install + test push, MFA), then the system is capture-ready in ≤60 minutes.
- **AC-2** Given the versioned known-bad fair-housing phrase set, when checked on any public content, reply, or sequence (seller or buyer/tenant), then 100% are blocked with no approval path other than editing; and ≤10% false positives on the known-good property-focused set.
- **AC-3** Given a Meta instant-form submission, when the webhook fires, then the full lead is fetched by `leadgen_id` and a lead exists with source=`paid_ad`, campaign/ad-set/form name, consent states, and a drafted reply within 5 minutes.
- **AC-4** Given an Instagram comment "Do you buy houses in [area]?", when processed, then a `seller_direct_offer` lead is created linked to the source post kit, graded, with a reply draft carrying the licensee-as-principal disclosure in the approval queue — all within 5 minutes of webhook delivery.
- **AC-5** Given any reply, follow-up, or sequence step, when send is attempted without a valid hash-matched approval (tested via API, worker, and agent paths), then the Sender refuses. Given the text, template, segment rule, or field list is edited after approval, or the approval is past its TTL, or a referenced property record changed, then approval is void and the item returns to the queue.
- **AC-6** Given a duplicate webhook delivery, when processed twice, then exactly one lead and one timeline entry exist.
- **AC-7** Given an accommodation request ("I have a service dog"), when classified, then it appears in the escalations tab with no auto-drafted refusal and an Agent alert; no "no pets" template is attached.
- **AC-8** Given an investor-intent message, when processed, then it lands on the Investor hold list with a neutral draft and no score/sequence exists for it.
- **AC-9** Given a lead at a buyer/tenant stage, when enrolled in follow-up, then the template-segment-approved standard template for that stage is used; the consistency log records template ID and response time; personalization is limited to name/property/times.
- **AC-10** Given the morning report, when delivered at 07:30 America/Chicago (tested across US daylight-saving changes and with a user time zone of Asia/Kolkata), then it contains hot leads, approvals waiting, appointments, new leads by type/grade, pipeline snapshot, top lead-producing content, spend, and one recommended action — one page.
- **AC-11** Given a 2-minute phone video uploaded over mobile, when processed, then it is transcribed and searchable within 5 minutes with auto-tags; the Agent can build an approved post kit with per-platform captions, SRT, a per-piece keyword CTA, and a link-in-bio router entry.
- **AC-12** Given a commercial email send, when sent, then it carries the postal address and working one-click unsubscribe; when a recipient unsubscribes, then all future sends to that address are blocked (automated test).
- **AC-13** Given a seller-offer page (listing or direct-offer) for a property in a state whose rule pack is not marked reviewed, when approval is attempted, then it is blocked.
- **AC-14** Given the LLM provider is unavailable (tested), when a buying-signal comment arrives, then rules-based classification still creates the lead and the Agent is alerted that drafting is paused.
- **AC-15** Given the kill switch is activated, when sends are pending, then all outbound halts within 60 seconds and the report lists what was paused.
- **AC-16** Given a lead-magnet request, when processed, then the delivery email sends only after double-opt-in confirmation, carries unsubscribe link and postal address, and is attributed to the source content piece (automated test).
- **AC-17** Given a contact replies to any nurture email, when the reply arrives, then all sequences for that contact stop within 5 minutes and the reply enters the T1 inbox as a hot signal linked to the original source.
- **AC-18** Given a bulk send to a `consent_unknown` contact, when attempted by any path, then the Sender refuses and flags the contact for re-permission.
- **AC-19a** Given a reel brief that fails the compliance pre-check, when the Agent tries to mark it ready to film, then the transition is blocked.
- **AC-19b** Given an approved kit whose post URL is never registered, when 7 days pass, then it appears as stuck content on the morning report with a one-tap action.
- **AC-20** Given a user with the Helper role, when they attempt to approve or send anything (UI, API, or worker path), then the action is refused and logged.
- **AC-21** Given a queued Meta DM reply whose reply window expires in <2h, then the Agent receives a push alert; and when the window expires unsent, then no platform send is attempted and the item converts to an email draft (if consent) or a "window missed" log entry.
- **AC-22** Given a comment containing a per-piece keyword on any platform or a DM with no source post, when processed, then the lead's first-touch source resolves to that piece (keyword) or follows the §09 resolution order, and the known-source metric counts only organic-content leads.
- **AC-23** Given a `consent_unknown` contact, when the re-permission email (WF-11) is approved, sent and the "yes" link clicked, then an `email_opt_in` consent record is written and the contact becomes eligible for its nurture segment; without a click after 21 days, the contact remains 1:1-only.

---

## 18. Future Considerations

Intentionally outside V1, in rough priority order: **SEO neighborhood pages** (property-focused area guides on the Agent's site that turn reel transcripts into indexed pages with guide-download capture — the Houston Properties Team model, §04a) → contributor portal + usage-rights gate (unlocks customer/UGC content at scale) → automated video repurposing (re-cuts, burned-in captions, carousels) → AI trend/hook research + weekly content calendar → scheduling/auto-publishing with the same approval gate → SMS follow-ups (requires TCPA prior express written consent) → ad creative generation + campaign optimization → Stripe contributor payouts → real-estate CRM integrations (Follow Up Boss, REsimpli, HubSpot) → investor/capital-partner module (after legal review) → multi-operator SaaS with billing (Phase 4 commercialization).

### Recommended build sequence (V1)
| Weeks | Scope | Unlocks |
|---|---|---|
| 1–2 | Data model, roles + MFA, Sender service (approval types, TTL, kill switch), manual paste + CSV import, morning report skeleton; submit Meta App Review + Business Verification; start eval set; start domain warm-up | **Day 0 possible** (AC-1, 5, 6, 15, 20) |
| 2–4 | Meta lead-ad ingest + fetch, classification + scoring, approval queue with reply windows, Gmail 1:1, fair-housing + claims checkers, push notifications | Paid leads flowing (AC-2, 3, 7, 8, 13, 14, 21) |
| 4–6 | UGC Studio pipeline, post kits, keywords + link-in-bio router, IG/FB comments + DMs (on approval), YouTube polling | Organic attribution (AC-4, 11, 19a/b, 22) |
| 6–8 | Nurture engine on warmed domain, template-segment approvals, WF-10, WF-11 re-permission, Track D reporting | Email program (AC-9, 12, 16, 17, 18, 23) |

*All compliance features flag risk; nothing in this document is legal advice. State rule packs, disclosure text and release templates ship as placeholders until reviewed by a licensed attorney in the target state.*
