---
name: platform-perf-agent
description: 'Generate Intel-branded single-file HTML performance reports for CPU/platform studies (DMR, GNR, SRF, SPR, EMR, etc.) including EMON TMA breakdowns, frequency sweeps, core scaling, linear-vs-balanced studies, host-vs-container, and multi-platform A/B comparisons. Produces a self-contained single .html file with Chart.js, KPI strip, system-config table, sectioned charts (TMA, line, grouped bar), data tables, and tagged insight boxes.'
---

# Intel Performance HTML Report Generator

## When to Use
Trigger on any of:
- "generate / create / build an HTML report" for perf data
- "compare <platform> vs <platform>" with CSV/Excel or EMON input
- Delegated call from `platform-perf-engineer` agent containing processed perf metrics
- Studies: EMON Auto-Analysis, Frequency sweeps, core scaling, linear-vs-balanced, SPEC CPU, MySQL/Sysbench
- User shares CSV/XLSX benchmark results and expects a deliverable

If the user only wants a chat answer or quick summary, **do not** invoke this skill.

---

## Input Contract (Calling Agent vs User)
The skill accepts data in two ways:
1. **From `platform-perf-engineer` (Preferred):** Pre-computed metrics, TMA hierarchy, anomaly tags, and system config passed directly as structured JSON or CSV.
2. **From Direct User File:** If given raw Excel/CSV directly, parse it programmatically using local Python (`pandas`/`openpyxl`). **Never ask the user to manually convert Excel to CSV.**

---

## Output Contract
A single, self-contained `.html` file saved in the current working directory:
- **File naming convention:** `<workload>_<platform>_<study-type>_report.html`
- **Dependencies:** Chart.js 4.4.0 + `chartjs-plugin-datalabels` 2.2.0 loaded via CDN (`cdn.jsdelivr.net`) or cached locally.
- **Self-contained:** Embedded CSS and JS. No external image assets or separate stylesheets.
- **Print-ready:** Includes `@media print` rules preserving background colors and page breaks.

---

## Incremental Streaming Lifecycle (Deep Mode Support)
For Deep Mode reports, work through these phases in the current agent session:
- **Draft:** Create the initial skeleton, header, host inventory, and preliminary platform model.
- **Update:** Add computed EMON metrics, Top-Down microarchitecture stacked bars, and scaling curves.
- **Finalize:** Add the executive summary, tagged insight boxes, recommendations, and complete data-table appendix.

---

## Report Section Hierarchy (Strict Order)

| # | Section | Content & Formatting |
|---|---|---|
| **1** | **Sticky Header** | Workload title, study type, host ID / SUT, run date, Intel Blue branding (`#0054ae`). |
| **2** | **KPI Strip** | 4–8 high-contrast cards: Peak throughput, peak/core, headline delta %, health status badge. |
| **3** | **Executive Summary** | 2–4 concise sentences referencing exact numbers and architectural root causes. |
| **4** | **System Configuration** | Side-by-side table: CPU model, stepping, sockets, NUMA, DDR, BIOS, TDP. |
| **5** | **Workload Parameters** | Execution command line, dataset size, concurrency parameters, warmup duration. |
| **6** | **TMA & Core Scaling Charts** | Horizontal stacked bars for TMA L1/L2; grouped bars for per-core IPC/frequency uniformity. |
| **7** | **Dimension Study Charts** | Line/bar charts for primary study dimensions (thread scaling, latency vs load). |
| **8** | **Key Insights** | Tagged callout boxes: `[STRENGTH]`, `[ANOMALY]`, `[OBSERVATION]`, `[RECOMMENDATION]`. |
| **9** | **Full Data Appendix** | Collapsible data tables containing complete raw values and calculated deltas. |
| **10**| **Footer** | SUT Hostname · Workload · Generating Agent · Timestamp. |

---

## Architectural & Platform Hard Rules

- **DMR Specifics:**
  - DMR has NO HyperThreading (`threads_per_core=1`). Never display HT sibling pairs or threads.
  - DMR is single-NUMA regardless of CBB count (1 / X1 / X4). Never label X4 as "4 NUMA nodes".
- **Color Palette & Semantics:**
  - Primary Platform / Series: Intel Blue (`#0054ae`)
  - Secondary / Comparison Series: Warm Orange (`#d35400`)
  - Positive Delta / Improvement: Forest Green (`#1b8a3e`)
  - Negative Delta / Regression: Dark Red (`#dc2626`)
  - Latency Inversion: A negative latency delta (`-15%`) is better → Render in **Green**.
- **Typography:** Use em-dashes (`—`) in titles, never standard hyphens.

---

## Anti-Patterns (Do NOT)

- ❌ Never ask the user to format, clean, or convert data files.
- ❌ Never split output into separate `.css` or `.js` files.
- ❌ Never use pie charts (use horizontal stacked bars for TMA breakdowns).
- ❌ Never render emojis in report headings (use `§` or bold uppercase tags).
- ❌ Never generate generic placeholder text in the summary; every sentence must cite real metrics.

---

## Embedded Reference Standards

### 1. Metric Decimal Formatting Rules
- **TPS / QPS / Ops/sec:** 0 to 2 decimal places (e.g., `124,530.2`).
- **Latency (ms / us):** 2 decimal places (e.g., `1.45 ms`).
- **Percentage Deltas:** 1 decimal place with explicit sign (e.g., `+12.4%`, `−5.2%`).
- **IPC / CPI:** 2 to 3 decimal places (e.g., `1.84 IPC`).

### 2. Insight Box Tagging Classes
- `[STRENGTH]`: Outperformance vs baseline or high efficiency.
- `[ANOMALY]`: Imbalances across cores/sockets or unexpected TMA category dominance.
- `[OBSERVATION]`: Neutral architectural behavior (e.g., memory bandwidth plateau at 64 threads).
- `[RECOMMENDATION]`: Specific, actionable tuning knob or affinity change.
