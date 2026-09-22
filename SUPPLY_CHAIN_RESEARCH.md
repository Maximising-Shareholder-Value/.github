# Supply Chain Visualization — Pilot Research (AI Infrastructure)

Researched 2026-09-22 for Pillar 2+3 (supply chain visualization). This
is the first step of that pillar's own scoping note in
[TODO.md](TODO.md): "curate 5-10 companies and their real upstream/
downstream relationships — this is a research task, likely LLM-assisted
but needs human verification per company, not a live feed."

**Status: live, 2026-09-22 — msv-web PR #26, the "Supply Chain" tab.**
Jozsua's instruction ("just make the supply-chain research data set live
in the app please") is treated as his sign-off on this research. The
OpenAI/Anthropic open question below was resolved by including them as
real nodes, clearly labeled "No ticker — private company." All 17 nodes
and 20 relationships here are transcribed verbatim into `supplyChain.js`
— every relationship card in the live UI shows its own source and
confidence rating rather than stating anything as flat fact, preserving
the caveat on the NVIDIA "Customer A/B/C/D" row specifically. This
document remains the source of truth for that data — if a relationship
here is ever corrected, update `supplyChain.js` to match.

## Sourcing method

Research prioritized, in order:
1. **SEC filings** (10-K/10-Q customer concentration disclosures, or
   named partnerships) — the most authoritative, since it's a legal
   disclosure.
2. **Official company statements** (earnings calls, investor day
   presentations, press releases).
3. **Reputable journalism** (Reuters, Bloomberg, CNBC, The Information,
   etc.), only included when corroborated by more than one independent
   source, or so widely and consistently reported it's effectively
   common knowledge (e.g. NVIDIA using TSMC as its primary foundry).

Anything the research pass wasn't confident about was either explicitly
flagged "medium confidence" below or left out entirely — the instruction
given was "better to return 8 well-sourced relationships than 15 where
some are guessed."

## Recommended pilot nodes (8-12 public companies)

| Company | Ticker | Role in stack |
|---|---|---|
| NVIDIA | NVDA | GPU/chip designer |
| Taiwan Semiconductor (TSMC) | TSM (NYSE ADR) | Chip foundry/manufacturer |
| AMD | AMD | GPU/chip designer |
| Intel | INTC | Chip designer + foundry |
| Microsoft | MSFT | Hyperscaler/cloud, model investor |
| Alphabet (Google) | GOOGL | Hyperscaler/cloud, custom silicon |
| Amazon | AMZN | Hyperscaler/cloud, custom silicon |
| Meta Platforms | META | Hyperscaler-scale AI infra buyer |
| Broadcom | AVGO | Custom ASIC co-design, networking |
| Micron | MU | Memory (HBM) supplier |
| Arista Networks | ANET | Data-center networking |
| Super Micro Computer | SMCI | GPU server systems integrator |
| ASML | ASML (NYSE ADR) | Lithography equipment |

**Known gap**: SK Hynix is NVIDIA's single largest HBM supplier by far,
but trades only in Korea (KOSPI 000660) with no clean US ADR — likely a
real gap in this app's free-tier US-ticker data coverage if it's
included as a node. Also relevant but **private, no ticker** (can't be a
standalone node with price data): OpenAI, Anthropic. CoreWeave (CRWV) is
public and could be a 13th/bonus node given its central role.

## Relationship dataset

| Company A | Relationship | Company B | Description | Source / citation | Confidence |
|---|---|---|---|---|---|
| TSMC | manufacturing-partner-of | NVIDIA | TSMC fabricates nearly all of NVIDIA's advanced GPUs on 5nm/4nm nodes; NVIDIA is projected to become TSMC's single largest customer in 2026, overtaking Apple. | CNBC, "Nvidia set to supplant Apple as TSMC's largest customer" (Jan 26, 2026); reported consistently for years across outlets | High |
| ASML | supplier-of | TSMC, Intel, Samsung | ASML is the sole global supplier of EUV lithography machines needed for leading-edge chips; TSMC, Intel and Samsung co-invested in ASML's R&D and historically held minority equity stakes. | ASML company statements; Reuters/Techzine/Yahoo Finance coverage of High-NA EUV orders (Aug 2025–2026) | High |
| SK Hynix (no US ticker) | supplier-of | NVIDIA | SK Hynix is NVIDIA's primary HBM (high-bandwidth memory) supplier, holding ~62% HBM share in 2025 and a projected ~70% share of HBM4 for NVIDIA's upcoming Rubin platform. | CNBC, "Nvidia-supplier SK Hynix readies production for cutting-edge HBM4" (Sept 12, 2025); SK Hynix Newsroom 2026 outlook | High |
| Micron | supplier-of | NVIDIA | Secondary HBM supplier to NVIDIA — shipped HBM4 samples meeting NVIDIA's specs mid-2025 but trails SK Hynix (and Samsung) in contracted volume/share. | Benzinga/TrendForce reporting, June–Dec 2025 | Medium (early-stage/sample supply, not yet at SK Hynix's scale) |
| Super Micro Computer | manufacturing-partner-of / customer-of | NVIDIA | Launch partner building NVIDIA-certified, rack-scale GPU server systems (e.g. GB200/GB300, Vera Rubin NVL4); revenue heavily dependent on NVIDIA GPU allocation. | Supermicro/NVIDIA joint press releases (2025–2026); Fortune, Apr 2026 | High |
| Arista Networks | supplier-of | Microsoft, Meta | Arista's data-center switches used extensively by both; each has historically represented more than 10% of Arista's annual revenue, disclosed as a customer-concentration risk. | Arista Networks Form 10-K filings (SEC EDGAR); Seeking Alpha/BofA note on 2025 revenue contribution | High |
| Broadcom | manufacturing-partner-of | Google (Alphabet) | Broadcom has co-designed Google's custom TPU AI chips for seven generations since 2014; extended the custom-silicon/networking partnership through 2031. | Tom's Hardware (May 2026); Broadcom investor 8-K / StockTitan filing summary | High |
| Broadcom | manufacturing-partner-of | OpenAI | OpenAI co-designing its own AI accelerator chips, Broadcom manufacturing/deploying; deal covers up to 10 gigawatts of custom silicon, production starting H2 2026 through 2029. | OpenAI/Broadcom joint press release (Oct 13, 2025); CNBC, Bloomberg | High |
| AMD | supplier-of | OpenAI | AMD supplying up to 6 gigawatts of Instinct MI450 GPUs to OpenAI 2026–2030 (1GW tranche starting H2 2026); AMD expects $100B+ revenue over 4 years from the deal. | AMD/OpenAI press release (Oct 6, 2025); CNBC, TechCrunch | High |
| OpenAI | major-investor-in | AMD | Part of the chip-supply deal: AMD issued OpenAI warrants for up to 160 million AMD shares (vesting tied to deployment milestones/share price, up to ~10% stake potential). | CNBC, "OpenAI looks to take 10% stake in AMD" (Oct 6, 2025) | High |
| NVIDIA | major-investor-in / infrastructure-provider-for | OpenAI | NVIDIA agreed to invest up to $100 billion in OpenAI, funded progressively as OpenAI deploys at least 10 gigawatts of NVIDIA systems (~4-5 million GPUs). | NVIDIA Newsroom joint release (Sept 22, 2025); Bloomberg, CNBC | High |
| Microsoft | major-investor-in / infrastructure-provider-for | OpenAI | Microsoft committed $13B total to OpenAI ($11.9B funded as of mid-2025); OpenAI's API runs exclusively on Azure. Microsoft's FY2026 10-Q disclosed $24.1B in revenue tied to the OpenAI relationship. | Microsoft FY2026 Form 10-Q/10-K; Microsoft Official Blog (Oct 28, 2025); CNBC, Neowin | High |
| Microsoft & NVIDIA | major-investor-in / infrastructure-provider-for | Anthropic | Microsoft (up to $5B) and NVIDIA (up to $10B) investing in Anthropic; Anthropic committed to purchase $30B of Azure compute plus up to 1GW additional capacity, valuing Anthropic near $350B. | Microsoft/NVIDIA/Anthropic joint announcement (Nov 18, 2025); Microsoft Official Blog, CNBC | High |
| NVIDIA | major-investor-in / customer-of | CoreWeave | NVIDIA holds an equity stake (~1.2% at IPO), invested a further ~$2B in 2025, and signed a $6.3B deal to buy unused CoreWeave cloud capacity through April 2032. Microsoft, Meta and OpenAI also among CoreWeave's largest customers. | CoreWeave IPO S-1 filing; CNBC, "CoreWeave's stock rallies on disclosure of $6.3 billion order from Nvidia" (Sept 15, 2025) | High |
| Amazon (AWS) | customer-of / co-engineering-partner-of | NVIDIA | AWS deploying 2 million additional NVIDIA GPUs 2027–2028; Amazon's Annapurna Labs co-designing next-gen Trainium4 chips to interoperate with NVIDIA's NVLink Fusion in shared server racks. | NVIDIA Newsroom / AWS joint announcement, re:Invent 2025 | High |
| Meta Platforms | customer-of | NVIDIA | Meta's AI buildout (2GW+ data-center program, ~$115–135B 2026 capex) relies heavily on NVIDIA GPUs (Blackwell, then Rubin) — one of NVIDIA's largest customers by volume. | Tom's Hardware (Meta-NVIDIA deal coverage); CNBC (Feb 17, 2026) | High |
| Meta Platforms | customer-of | AMD | Diversifying silicon supply by also deploying AMD MI450 chips alongside NVIDIA GPUs in FY2026 AI infrastructure. | Enki AI / industry capex analysis, 2026 | Medium (scale/contract terms less precisely reported than the NVIDIA relationship) |
| Intel (Foundry) | manufacturing-partner-of | Microsoft | Intel Foundry secured Microsoft as a customer for its 18A process node, reportedly to manufacture Microsoft's custom "Maia 2" AI accelerator chip. | Tom's Hardware, SemiWiki (2025) — trade-press reporting, not yet an explicit joint SEC/press disclosure of the Maia 2 tie | Medium |
| Intel (Foundry) | manufacturing-partner-of | Amazon | Reportedly smaller-scale foundry arrangement with Amazon for custom AI Fabric chips on the 18A node. | Electronics Weekly, SemiWiki (2025) | Medium — lower confidence, volumes/specifics thinly sourced |
| NVIDIA | customer-of | Microsoft / Meta / Google / Amazon (unnamed in filing) | NVIDIA's own SEC filings show customer concentration (4 direct customers = 61% of a recent quarter's revenue, top single customer ~22%), anonymized as "Customer A/B/C/D." NVIDIA does not officially name them; multiple outlets have identified the likely hyperscalers as Microsoft, Meta, Google and Amazon. | NVIDIA SEC filings (concentration language, high-confidence/SEC-sourced); CNBC, "Nvidia's top two mystery customers made up 39% of Q2 revenue" (Aug 28, 2025) — identities are analyst/journalist inference | Medium — see note below |

**Note on the last row**: the concentration disclosure itself is real
and SEC-sourced (high confidence). The specific company *identities*
behind "Customer A/B/C/D" are NOT confirmed by NVIDIA — only inferred by
journalists/analysts. If this ships, that distinction needs to stay
visible in the UI (e.g. "reported/inferred" label), not presented as
confirmed fact — this is exactly the kind of row that could quietly
violate the site's no-fabrication standard if the caveat gets lost
somewhere between this doc and the final visualization.

## Open questions before this becomes a real feature

- **OpenAI and Anthropic are private, no ticker.** A meaningful chunk of
  the most newsworthy 2025 relationships involve them (NVIDIA–OpenAI,
  Microsoft–OpenAI, AMD–OpenAI, Broadcom–OpenAI, Microsoft/NVIDIA–
  Anthropic). Decide up front: show them as unlisted nodes with no price
  data, or restrict the pilot strictly to entities with a real ticker
  (which would cut some of the most interesting, most current
  relationships).
- **SK Hynix** is arguably the single most important HBM relationship in
  this whole graph but has no clean US-listed free-tier data path —
  worth flagging before committing to the node list.
- **The strongest, most "visualizable" backbone**, per the research:
  ASML → TSMC/Intel → NVIDIA/AMD → Supermicro/Arista → Microsoft/
  Amazon/Google/Meta, with NVIDIA/Microsoft/AMD/Broadcom → OpenAI and
  NVIDIA/Microsoft → Anthropic as a clear "AI-model-company hub" layer.

## Next step

This needs Jozsua's own review — confirm which relationships look right,
flag anything that seems off, and decide the OpenAI/Anthropic
private-company question — before any visualization design work starts
on top of it.
