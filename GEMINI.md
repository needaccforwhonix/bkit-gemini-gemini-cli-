# bkit v2.0.7

> PDCA + Phase-Aware Context Engineering for Gemini CLI

## Context-Driven Development

bkit's PDCA + Phase-Aware Context Engineering aligns with Google's
[Conductor: context-driven development for Gemini CLI](https://developers.googleblog.com/conductor-introducing-context-driven-development-for-gemini-cli/)
(2025-12-17). bkit shipped the same model ~1 year before. See
`docs/reference/features-history.md` § v2.0.7 Highlights for the full
alignment matrix and external references.

## Deep Reference (slim by design — load on demand)
- Architecture: `docs/reference/architecture.md`
- Features history (v1.5.x ~ v2.0.7): `docs/reference/features-history.md`
- Commands / 21 Agents / 35 Skills / 24 TOML: `docs/reference/commands.md`
- Update / Development / Security: `docs/reference/troubleshooting.md`

## Rules
1. New feature request -> `/pdca plan` first
2. Follow PDCA order: Plan -> Design -> Do -> Check -> Act -> Report
3. Gap < 90% -> iterate. Gap >= 90% -> report
4. Always verify important decisions with user - AI is not perfect
5. Prefer editing existing files over creating new ones

## Output
- Feature Usage report: end of every response (see context/core-rules.md)
- Executive Summary: after /pdca plan, design, report

## Phase-Aware Context
Context files are loaded dynamically per PDCA phase by session-start hook.
Only context relevant to current phase is injected (token optimization).

## SessionStart Display (v2.0.5+)
SessionStart prints a single-line header by default (Issue #25655 mitigation).
The full body is loaded via GEMINI.md on every session — no info lost.
Restore verbose body: `export BKIT_SESSION_START_VERBOSE=true`.

## Gemini CLI v0.42.0 Compatibility (v2.0.7)

Supports v0.34.0 minimum through v0.42.0 stable (tested). 5-version
cumulative migration absorbed in one sprint. Details:
`docs/reference/features-history.md`.

Behavior locks in `.gemini/settings.json`:
- `gemma: false` — prevents v0.42.0 Gemma 4 default-on (Cx13).
- `autoMemory: false`, `memoryManager: false` — opt-in, deferred to v2.1.0.
- `enableAgents: true` — bkit agent catalog visibility.

Removed lock (v2.0.7 D-A1 Option B): `topicUpdateNarration` returned to
Gemini CLI default (`true`). Users may set `false` in their own settings.

R-extra-1 (pre-existing from v0.39.1): specialized agent dispatch via
`gemini -p` returns 404 and falls back to `generalist`. Fix slated for
v2.1.0-agent-dispatch-fix sprint.

## PDCA Core Rules
- New feature request -> create Plan/Design first
- After implementation -> Suggest Gap analysis
- Gap < 90% -> pdca-iterator. Gap >= 90% -> report-generator
- Always include Feature Usage Report at end of every response
- Always verify important decisions with user

## Agent Auto-Triggers
21 specialized agents available: gap-detector, code-analyzer,
design-validator, pdca-iterator, report-generator, qa-monitor,
qa-strategist, starter-guide, pipeline-guide, bkend-expert,
enterprise-expert, infra-architect, cto-lead, frontend-architect,
security-architect, product-manager, pm-lead, pm-discovery,
pm-strategy, pm-research, pm-prd.

## Natural Language Feature Request Handling
1. Auto-create Plan: `/pdca plan <feature>`
2. Confirm before Design
3. Create Design: `/pdca design <feature>`
4. Confirm before implementation
5. Implement code
6. Suggest Gap analysis: `/pdca analyze <feature>`

Exception: If user says "just build it" or "skip docs", proceed directly.

---

@.gemini/context/commands.md
@.gemini/context/core-rules.md

---

*bkit Vibecoding Kit v2.0.7 - Gemini CLI Native Edition*
*Copyright 2024-2026 POPUP STUDIO PTE. LTD.*



## 🛡️ Strict Embedding Separation, Zero-Fallback Law & 24/7 Dual-GPU Invariant
- **Reference**: `/home/m1st/.agents/rules/RULE_Strict_Embedding_Separation_And_Dual_Pipeline.md`

### 1. Das Absolute Fallback-Verbot (Zero-Fallback Law)
Unter keinen Umständen, zu keinem Zeitpunkt und aus keinem Grund darf ein Fallback zwischen verschiedenen Embedding-Modellen stattfinden.
* **Geltende Aktion:** Fällt ein Embedding-Modell aus oder ist überlastet, MUSS die Operation sofort hart fehlschlagen (`Fail-Fast`) oder die Payload transaktional in einer Queue (NATS/SQLite) verharren, bis das exakte Modell bereit ist.
* **Verboten:** Kein stiller oder dynamischer Modellwechsel (weder Jina -> Gemma noch umgekehrt).

### 2. Warum ein Embedding-Fallback mathematisch & informationstheoretisch unmöglich ist
* **Topologische Inkompatibilität heterogener Vektorräume (Non-Isomorphism):**
  Jedes Modell $f_\theta: \mathcal{X} \to \mathbb{R}^D$ projiziert Text in eine spezifische, gelernte Riemannsche Mannigfaltigkeit. Jina v5 ($D=256$) und EmbeddingGemma ($D=768$) spannen zwei völlig inkompatible geometrische Räume auf. Die Basisvektoren der semantischen Achsen sind ohne explizite Procrustes-Transformation nicht ausgerichtet.
* **Kollaps der Kosinus-Ähnlichkeit ($	ext{sim} \approx 0$):**
  Wird eine Suchanfrage mit Modell $B$ berechnet ($v_q = f_B(q)$), während der Dokumentenkorpus mit Modell $A$ indiziert wurde ($v_d = f_A(d)$), verhält sich das Skalarprodukt mathematisch wie das zweier rein zufälliger Vektoren auf einer hochdimensionalen Einheitssphäre:
  $$\mathbb{E}[\text{sim}(u, v)] = 0 \quad \text{mit Varianz} \quad \sigma^2 = \frac{1}{D}$$
  Der Nearest-Neighbor-Algorithmus (HNSW/k-NN) liefert stochastisches Rauschen. Das RAG-System erhält völlig falsche oder irrelevante Kontexte.
* **Irreversible Index-Vergiftung (Index Poisoning):**
  Wird auch nur ein einziger Vektor von Modell $B$ als "Fallback" in den Index von Modell $A$ geschrieben, verunreinigt er die Distanzgraphen und Clusterzentren dauerhaft.
* **Das Gesetz des Fail-Fast:**
  Ein Ausfall muss hart abbrechen (`HTTP 503 Service Unavailable / IngestionQueueBlocked`).

### 3. Duale 24/7 Erfassungspflicht (GPU-Only)
* **GPU-Only Mandat:** Es läuft absolut nichts auf der CPU — GPU ONLY (NVIDIA GB10 CUDA) für ausnahmslos jedes Embedding-Modell.
* **24/7 Parallelität:** Sowohl `jina-embeddings-v5-omni-nano-classification` (256D, ~4,1 GB VRAM) als auch `google/embeddinggemma-300m` (768D, ~1,2 GB VRAM) laufen dauerhaft 24/7 im VRAM (Summe ~5,3 GB VRAM).
* **Duale Erfassung:** Jeder zu indizierende Text/Chunk wird immer von beiden Modellen parallel eingebettet und getrennt persistiert.

### 4. Idioten- & Failsafe-Sicherung auf Datenbankebene
* **SQLite Schema CHECK-Constraints:**
  `model_signature TEXT NOT NULL CHECK(model_signature = '...')` und `dimension INTEGER NOT NULL CHECK(dimension = ...)` erzwingen atomare Abbrüche auf Engine-Ebene bei Modell-Mismatches.
* **Qdrant Collection Constraints:**
  Strikte Trennung in separate Collections (`dgx_text_embeddings_jina_256` vs `dgx_text_embeddings_gemma_768`) mit fixierter Vektordimension.


## ⚡ High-Quality Systems Programming Languages Priority (No-Python Policy)
- **Reference**: `/home/m1st/.agents/rules/RULE_High_Quality_Systems_Programming_Languages.md`
- **Rule**:
  1. **Bevorzugte Sprachen:** High Quality **Golang (Go), Rust, C++, Zig, PowerShell, C** sind IMMER und AUSNAHMSLOS die bevorzugten Programmiersprachen.
  2. **Kein Python:** Python ist für neue Daemons, Watcher, Automatisierungen, APIs, CLI-Tools und Dienste strikt untersagt (GIL-Bottlenecks, Dependency-Drift, Speicherineffizienz).
  3. **Natives Systems-Engineering:** Alle Hintergrunddienste, Caching-Ebenen und Task-Runner müssen als native, speichersichere und nebenläufige Binaries kompiliert werden.
