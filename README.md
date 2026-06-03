# OpenALEX Collector v7.1

OpenALEX Collector v7.1 is a single-file, browser-based tool for collecting and analyzing scholarly paper metadata with the OpenAlex API. It supports paper retrieval, Topic intelligence, funding intelligence, citation analysis, collaboration/co-authorship networks, author and institution disambiguation, technical-keyword extraction, must-read paper triage, technology-lineage visualization, journal intelligence, and prompt generation for AI-assisted analytical reports.

No server setup is required. Open `OpenALEX_Collector_en.html` in a browser and start searching. The tool itself does not call external AI APIs such as OpenAI, Anthropic, or Google. When you want to use AI, the tool generates prompts, you review and manually paste them into an external AI system, and then paste TSV results back into Collector.

> **Important (February 2026 change):** The OpenAlex API now **requires an API key** as of February 13, 2026 (the old `mailto` / polite-pool method has been retired). Before using the tool, get a key from a **free** OpenAlex account and set it on the in-app "OpenAlex API Key" screen (→ [Quick start](#quick-start)). This is separate from external AI APIs (OpenAI / Anthropic, etc.) — **no AI API key is needed**.

> Every paper stands on someone's shoulders.

![Powered by OpenAlex API](https://img.shields.io/badge/Powered%20by-OpenAlex%20API-1a73e8)
![Version](https://img.shields.io/badge/version-v7.1-1a73e8)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Table of contents

- [Overview](#overview)
- [Design principles](#design-principles)
- [Changelog](#changelog)
- [Major changes from v6.0 to v7](#major-changes-from-v60-to-v7)
- [Quick start](#quick-start)
- [Typical workflow](#typical-workflow)
- [Modules](#modules)
  - [1. Paper Search](#1-paper-search)
  - [2. Paper Data](#2-paper-data)
  - [3. Technical Keywords & AI Annotations](#3-technical-keywords--ai-annotations)
  - [4. Author / Institution Disambiguation](#4-author--institution-disambiguation)
  - [5. Charts](#5-charts)
  - [6. Citation Ranking](#6-citation-ranking)
  - [7. Collaboration Network by Institution](#7-collaboration-network-by-institution)
  - [8. Funding Intelligence](#8-funding-intelligence)
  - [9. Journal Intelligence](#9-journal-intelligence)
  - [10. Topic Intelligence](#10-topic-intelligence)
  - [11. Network Map: Collaboration / Co-authorship](#11-network-map-collaboration--co-authorship)
  - [12. Citation Network: References](#12-citation-network-references)
  - [13. Must-read Papers & Technology Lineage](#13-must-read-papers--technology-lineage)
  - [14. AI Analysis Report: Final Synthesis](#14-ai-analysis-report-final-synthesis)
  - [15. Detail Popups](#15-detail-popups)
  - [16. Export / Import](#16-export--import)
- [How AI assistance works](#how-ai-assistance-works)
- [TF-IDF technical-keyword extraction](#tf-idf-technical-keyword-extraction)
- [Must-read paper score](#must-read-paper-score)
- [Technology-lineage logic](#technology-lineage-logic)
- [AI prompt design](#ai-prompt-design)
- [AI paste-back TSV formats](#ai-paste-back-tsv-formats)
- [Applying AI Topic labels to Topic Intelligence](#applying-ai-topic-labels-to-topic-intelligence)
- [Disambiguation-map JSON format](#disambiguation-map-json-format)
- [Disambiguation storage and cache design](#disambiguation-storage-and-cache-design)
- [Topic score definition](#topic-score-definition)
- [Network metrics](#network-metrics)
- [CSV output columns](#csv-output-columns)
- [Search syntax](#search-syntax)
- [Use-case guide](#use-case-guide)
- [External APIs and data retrieval](#external-apis-and-data-retrieval)
- [How to interpret analytical outputs](#how-to-interpret-analytical-outputs)
- [How to validate AI outputs](#how-to-validate-ai-outputs)
- [Common setting patterns](#common-setting-patterns)
- [Performance (speed optimizations)](#performance-speed-optimizations)
- [Technical specifications](#technical-specifications)
- [Notes and limitations](#notes-and-limitations)
- [Troubleshooting](#troubleshooting)
- [Related tools](#related-tools)
- [Acknowledgements & data provenance (OpenAlex / OurResearch)](#acknowledgements--data-provenance-openalex--ourresearch)
- [License](#license)

---

## Overview

**OpenALEX Collector v7** helps analysts understand the structure of a research field from OpenAlex metadata. It inherits the v6.0 functionality for search, Topics, funding, citations, collaboration/co-authorship networks, reference networks, and author/institution disambiguation, and adds technical-keyword extraction, AI annotations, must-read paper triage, technology-lineage visualization, and prompt generation for AI-assisted synthesis.

The tool is intended to answer questions such as:

- Is this research field growing or declining?
- Which Topics are expanding?
- Which papers, authors, institutions, and journals are influential?
- Are homonymous authors or institution-name variants distorting rankings or networks?
- Which institutions and authors collaborate with each other?
- Which authors work on which Topics?
- Which funders and award numbers are linked to which Topics, institutions, and authors?
- Which references act as foundational works, shared intellectual bases, or internal citation hubs?
- From a large paper collection, which papers should be read first?
- How can abstract OpenAlex Topic labels be converted into human-readable technology labels?
- Which technical concepts can be extracted from titles and abstracts?
- How can the temporal and conceptual relationship among representative papers be visualized?
- What evidence should be prepared before sending anything to an external AI system?
- How can prompts for GPT, Claude, or another AI system be generated without embedding an AI API in the tool?

The AI-assistance features in v7 do not run AI inference inside Collector. Collector prepares structured evidence: collection summaries, technical keywords, must-read candidates, technology-lineage nodes, Topic-naming data, and final synthesis prompts. The user reviews the content, manually sends it to an external AI system, and pastes TSV results back into Collector. The pasted annotations are then reflected in must-read papers, technology lineage, Topic labels, final synthesis prompts, and CSV exports.

---

## Design principles

| Principle | Description |
|---|---|
| OpenAlex-centered | Paper collection, Topics, references, institutions, authors, and funding metadata are primarily based on OpenAlex. |
| Single HTML file | No server is required. The tool runs in a browser. |
| No AI API calls | Collector does not call external AI APIs. It only generates prompts and imports TSV outputs. |
| Analyst-led workflow | Scores, labels, and lineage relationships are aids for analysis, not final judgments. Users review weights, filters, and AI paste-back results. |
| Evidence first | Prompts include Work IDs, years, titles, Topics, citations, authors, institutions, journals, and technical keywords. |
| Explainable scoring | Must-read candidates are scored from explicit components: Topic growth, age-adjusted citations, bridging potential, technical-keyword novelty, author/institution signals, and funding signals. |
| Lineage nodes matter | Must-read lists can include papers that appear in the technology lineage even if they are not in the top score range. |
| AI labels are display overlays | AI Topic labels can replace the displayed Topic names while the original OpenAlex Topic remains the aggregation key. |
| Disambiguation is integrated | Author and institution disambiguation is applied to rankings, citations, collaboration, Topic × institution, Topic × author, funder × institution, funder × author, and network analyses. |
| IDs before labels | Author and institution names are display labels. Aggregation prefers Author ID, ORCID, Institution ID, ROR, and user-defined canonical keys. |
| Cache separation | Performance caches and user-defined disambiguation maps are separated. Saved maps are used only when explicitly loaded or stored for a dataset. |
| OpenAlex API key (free, required) | As of February 13, 2026 the OpenAlex API requires an API key (see [Quick start](#quick-start)). Set a free key on the in-app "OpenAlex API Key" screen. External enrichment via Crossref REST API and NIH RePORTER API is registration-free. No AI API key field is provided (prompt-generation + TSV paste-back). |
| CSV reuse | Saved CSV files can be imported to continue analysis. v7 includes AI annotation columns, Topic-label columns, and disambiguation-related columns. |

---

## Changelog

### v7.1 (performance fix)

v7.1 is functionally identical to v7; it is a **performance release that fixes freezes on large datasets (tens of thousands of papers).**

- **Removed an O(N²) in the "Technical Keywords & AI Annotations" card**: the overview-memo generation (`renderOverview`) that runs when you switch to the "B. AI prompt generation" tab and when you run "C. paste back AI results" was recomputing the min/max publication year across *all* papers once *per paper* (effectively N×N). It now computes them once. This removes the multi-second / freeze-level wait seen at tens of thousands of papers (the affected step dropped from ~640 ms to ~5 ms at 10,000 papers).
- **Lighter re-render after pasting back AI results**: instead of unconditionally re-rendering every card (Topic intelligence, technology lineage, etc.) on each paste-back, it now **updates only the currently open card immediately and marks the others to re-render the next time they are opened** (using the existing lazy-render mechanism). Paste-back feels much lighter (~1,300 ms → ~250 ms at 10,000 papers).
- **Fixed a freeze when uploading a saved CSV**: during CSV import, author/institution disambiguation labeling (`applyIdentityLabels`) was running **twice**, and it rebuilt author/institution data once per paper. We (1) removed the duplicate run and (2) reuse a per-dataset cache of the disambiguated entities. CSV import now takes roughly half the time (~2,380 ms → ~1,050 ms at 5,000 papers, with a larger gain at tens of thousands). The disambiguation results are verified to be identical to before.
- **Fixed crashes on large CSVs (tens to hundreds of MB) — memory-efficient CSV parser**: the old `parseCSV` had two serious memory problems. (1) It concatenated the text one character at a time (`current += ch`), causing massive string reallocation on big files. (2) It first held every line in an intermediate `lines[]` array and then re-parsed it, stacking up arrays roughly the size of the original text two or three times over. As a result it used about **5× the CSV file size** in memory, and at ~150 MB it exceeded the browser's JS heap limit (~4 GB) and **crashed the tab**. The new implementation slices fields with substring and builds one row object at a time, **without keeping a giant intermediate array.**
  - **Impact (measured)**: post-parse memory growth is about **1/3.5** (a 12 MB CSV went from +46 MB to +13 MB) and parsing is about **3.4× faster** (618 ms → 182 ms for the same file). As a result, **150 MB-class CSVs that used to crash now load** (the practical file-size ceiling is greatly raised).
  - **Bonus bug fix**: the old parser mis-split commas inside quoted fields (e.g. `"Smith, J."` was broken into two columns). The new parser handles quoting per RFC 4180. An export → import round-trip preserves all columns exactly, including commas, quotes, and embedded JSON.
- **Progress indicator during CSV upload**: while a large CSV is loading, a spinner overlay ("Loading CSV...") is shown and is dismissed automatically when done. The indicator is painted to the screen before the load begins, so "busy" is reliably visible even for heavy synchronous work.

> Results and displayed content are identical to v7; only speed and memory efficiency improve (the only behavioral change is the CSV quoting bug fix, which makes results more correct). After a paste-back, opening the Topic or technology-lineage card renders it with the latest annotations applied at that moment.

---

## Major changes from v6.0 to v7

v7 keeps the main v6.0 feature set and adds functions for reading, explaining, and synthesizing large paper collections.

| Area | v7 behavior |
|---|---|
| Technical keywords | Extracts TF-IDF keywords and phrases from titles and abstracts. Maximum phrase length, minimum characters, minimum document frequency, document limit, and additional stopwords can be adjusted. |
| AI annotations | Imports technical-point TSV, Topic-naming TSV, and must-read paper TSV. OpenAlex metadata is not overwritten; AI outputs are layered as annotations. |
| AI prompt generation | Generates prompts for technical points, Topic naming, must-read paper organization, and final synthesis reports. |
| Must-read papers | Scores papers using Topic growth, age-adjusted citations, bridging potential, technical-keyword novelty, author/institution signals, and funding signals. |
| Technology lineage | Prioritizes internal citations, and when citation links are absent, infers parent-child relationships from Topic overlap, keyword similarity, author/institution continuity, and bibliographic coupling. |
| Combined card | Must-read papers and technology lineage are placed in one card with tabs. |
| Lineage papers in reading list | Papers included in the technology lineage can be added to the must-read list even if they are outside the score-top-N range. |
| Detail popups | Must-read lists, lineage tables, and lineage nodes can open detail popups with AI annotations, reading categories, node explanations, and score components. |
| CSV export | Paper CSV, must-read paper CSV, and technology-lineage CSV can be exported separately. |
| AI Topic labels in Topic Intelligence | AI-generated Topic labels can be applied to Topic Intelligence as display labels while preserving the original OpenAlex Topic as the aggregation key. |
| Final synthesis report | A final card prepares a collection overview and a report-style prompt for interpreting the field and its technology lineage. |
| Search UI | Command-line query mode and standard OpenAlex search mode are separated. In command-line mode, each row's TI/AB/TA/TX/FT field controls the search scope. |
| Large retrieval | Year-by-year mode supports large collections by applying the retrieval limit per year. |
| Disambiguation | v6.0 author/institution disambiguation, diagnostics, JSON maps, and analysis-wide application are retained. |

---

## Quick start

### 0. Set your OpenAlex API key (first run only — required)

As of February 13, 2026, the OpenAlex API **requires an API key** (free). The old `mailto` (polite-pool) method has been retired.

1. Create a **free account** at [openalex.org](https://openalex.org) (email only, about 30 seconds).
2. Once signed in, open [openalex.org/settings/api](https://openalex.org/settings/api) (Settings → API).
3. Copy your **API key**.
4. Open the tool, go to the **"OpenAlex API Key"** screen at the top of the left sidebar, paste your key, and click **Save**. Use **Test connection** to verify the key works.

- The key is stored **only in your browser's `localStorage`** and is never sent anywhere except the OpenAlex API. On a shared computer, click "Clear" when you are done.
- Without a key, the tool runs on a **trial allowance** of about 100 requests/day and then fails with errors such as `403`.
- The free tier is **100,000 credits/day (about $1/day)**: unlimited single-work lookups, 10,000 list/filter calls, 1,000 full-text searches, 100 full-text downloads, etc. (a per-operation-type guideline). For higher academic limits, contact support@openalex.org.
  - Here, **"full-text downloads" means downloading a paper's PDF body**. **This tool fetches metadata only and never performs full-text downloads**, so the "100 full-text downloads/day" limit never applies — the allowances that matter are searches (~1,000/day) and list retrievals (~10,000/day). For normal research, the free tier is more than enough.
- This is separate from external AI APIs (OpenAI / Anthropic / Google, etc.). AI analysis uses a prompt-generation + TSV-paste-back flow, so **no AI API key is required**.

### Workflow

```bash
# 1. Open the HTML file in your browser.
open OpenALEX_Collector_en.html

# (first run only) Set a free API key on the "OpenAlex API Key" screen — see "0." above.

# 2. Enter a command-line query or use standard OpenAlex search.
# Example:
# #01  TA=((sulfide OR oxide OR polymer) near5 electrolyte)
# #02  AB=(degradation OR interface)
# T=(#01 AND #02)

# 3. Optionally set institution filters, author filters, years, and advanced filters.

# 4. Click "Run search".

# 5. Review the paper table.

# 6. Use Technical Keywords & AI Annotations to inspect technical terms and generate AI prompts.

# 7. Use Author / Institution Disambiguation to review homonyms, institution variants, and parent-institution grouping.

# 8. Render charts, citation rankings, collaboration analysis, funding analysis, Topic analysis, network maps, and reference networks.

# 9. Use Must-read Papers & Technology Lineage to review priority papers and lineage nodes.

# 10. Optionally send generated prompts to an external AI system and paste TSV results back.

# 11. Use AI Analysis Report for the final overview and technology-lineage synthesis prompt.

# 12. Export CSV, PNG, Markdown, or JSON files.
```

Recommended browsers: Chrome, Edge, Firefox, and Safari. The tool runs as a local HTML file, but Chart.js and vis-network are loaded from CDNs, so charts and network views require internet access.

---

## Typical workflow

```text
Define search conditions
  -> Retrieve papers from OpenAlex
  -> Preserve author IDs, ORCIDs, institution IDs, RORs, lineage, Topics, references, and funding metadata
  -> Apply a disambiguation map when needed
  -> Review the paper table
  -> Inspect technical keywords and prepare AI prompts
  -> Review author and institution keys in the disambiguation workbench
  -> Render charts, citations, collaboration, funding, Topics, networks, and reference analyses
  -> Calculate must-read paper scores
  -> Render technology lineage and inspect lineage nodes
  -> Optionally send prompts to an external AI system and paste TSV results back
  -> Reflect AI annotations in must-read papers, lineage, Topic labels, final prompts, and CSV exports
  -> Generate the final AI analysis prompt
  -> Export CSV / PNG / Markdown / JSON
  -> Reuse CSV and disambiguation maps in later analyses
```

---

## Modules

### Screen layout (left-sidebar tabbed UI)

The v7 UI is organized as a left sidebar of tabs. Each feature card is chosen from a sidebar group and shown **one at a time at full width**, so wide tables, network maps, technology lineages, and matrices fit without being cut off and the screen width is used generously.

| Sidebar group | Cards |
|---|---|
| Collect & Data | Paper Search / Paper Data / Disambiguation |
| Technical Keywords | Technical Keywords & AI |
| Quantitative Analysis | Charts / Citation Ranking / Topic Intelligence / Funding Intelligence / Journal Intelligence |
| Networks | Collaboration Network / Network Map / Citation Network |
| Technology Lineage & Key Papers | Must-read Papers & Technology Lineage |
| Report | AI Analysis Report |

- Collection metrics (count, period, etc.) live outside the sidebar and stay visible at the top on **every tab**.
- Running a search automatically switches to the Paper Data tab.
- Each card is computed **when it is shown** (lazy rendering), so large datasets do not freeze right after a search.
- The sub-tabs inside each card (e.g., the four chart tabs, six citation tabs) are unchanged.
- On narrow screens, the sidebar collapses into a horizontal menu at the top.

### 1. Paper Search

The Paper Search card defines the OpenAlex query and the post-retrieval filtering rules.

#### Search modes

| Mode | Description | Best used for |
|---|---|---|
| Command-line query | Define numbered command rows such as `#01`, `#02`, then combine them with `T=(#01 AND #02) OR #03`. | Patent-search-like or professional literature-search workflows requiring explicit title, abstract, proximity, and exclusion logic. |
| Standard OpenAlex search | Pass search strings directly to OpenAlex. One line per query; multiple lines are treated as OR queries. | Fast exploratory search using OpenAlex's native search behavior. |

In command-line query mode, every command row must explicitly use one of `TI=`, `AB=`, `TA=`, `TX=`, or `FT=`. The advanced-filter search-scope selector is disabled because field scope is controlled by each command row.

In standard OpenAlex search mode, the search-scope selector is active. Available scopes include all fields, title + abstract, title, abstract, and fulltext.

#### Command-line query syntax

Example:

```text
#01  TI=(solid adj3 electrolyte)
#02  AB=((degradation AND capacity) near8 (mechanism OR fade*))
#03  TA=(review)
T=(#01 AND #02) AND NOT #03
```

| Syntax | Meaning |
|---|---|
| `TI=(...)` | Search the title. |
| `AB=(...)` | Search the abstract. |
| `TA=(...)` | Search title + abstract. |
| `TX=(...)` | Use OpenAlex's general search target. Candidate retrieval only. |
| `FT=(...)` | Use OpenAlex's fulltext index. Candidate retrieval only. |
| `AND` | Both terms or subexpressions must match. |
| `OR` | Either term or subexpression may match. |
| `NOT` | Exclude a term or subexpression. |
| `nearN` | Unordered proximity within N words. |
| `adjN` | Ordered proximity: the left term must appear before the right term within N words. |
| `*` | Multi-character wildcard, such as `cathod*`. |
| `?` | Single-character wildcard, such as `wom?n`. |

For normal `TI` / `AB` / `TA` keyword search, Boolean logic, and wildcards, Collector accepts the OpenAlex candidate set. Internal distance validation is primarily applied when `nearN` or `adjN` is used inside `TI`, `AB`, or `TA`. `TX` and `FT` are candidate-retrieval fields and are not internally validated for strict proximity or snippet matching.

#### Institution filter

The institution filter is combined with the keyword query using AND. Multiple institutions separated by semicolons are treated as OR alternatives.

```text
The University of Texas at Austin; Meijo University
https://openalex.org/I136199984
I136199984
```

Institution names and OpenAlex Institution IDs can be used. Downstream analyses can aggregate institutions using Institution ID, ROR, lineage, and user-defined disambiguation maps.

#### Author filter

The author filter is combined with the keyword and institution filters using AND. Multiple authors separated by semicolons mean that the returned papers should include all specified authors.

```text
Akira Yoshino; John Goodenough
https://openalex.org/A123456789
A123456789
```

Author names and OpenAlex Author IDs can be used. Downstream analyses aggregate authors using Author ID, ORCID, and user-defined disambiguation maps.

#### Years and year-by-year retrieval

The publication-year range can be limited with start and end years. Year-by-year mode runs the OpenAlex search separately by year and applies the retrieval limit per year. Use it for large research fields that may exceed a single-query retrieval ceiling.

Year-by-year mode does not guarantee 10,000 papers. It raises the per-year maximum. The actual number depends on the true number of matching papers and on filters such as paper type, abstract availability, English-only filtering, OA-only filtering, and citation thresholds.

#### Advanced filters

| Filter | Description |
|---|---|
| Publication type | Article, Review, Book Chapter, Book, Dataset, Preprint, Dissertation, Editorial, Letter, and Report can be selected. |
| Search scope | Available only in standard OpenAlex search mode. Command-line query mode uses each row's field prefix. |
| Sort order | Relevance, citation count descending, newest first, or oldest first. |
| Minimum citations | Keep only papers whose citation count is at or above the threshold. |
| OA only | Keep open-access papers only. |
| Has abstract only | Uses `has_abstract:true`. |
| English only | Uses `language:en` and also removes titles containing CJK and similar scripts. |

---

### 2. Paper Data

The Paper Data card displays retrieved papers. It shows titles, years, authors, institutions, venues, citation counts, Topics, DOIs, and Collector internal match information.

Main functions:

| Function | Description |
|---|---|
| Pagination | Keeps large result sets usable. |
| Sorting | Sort by year, citation count, title, and other columns. |
| Detail view | Open a popup for paper metadata, authors, institutions, Topics, funding, references, and AI annotations. |
| CSV download | Export the current paper collection, including AI annotations and disambiguation-related IDs. |
| CSV upload | Load a saved CSV and continue analysis without running a new search. |

When a v7 CSV is re-imported, AI annotation columns, Topic-label columns, author/institution ID columns, and funding columns are restored when available. Older CSV files without ID columns are treated as name-only data where necessary.

---

### 3. Technical Keywords & AI Annotations

The Technical Keywords & AI Annotations card is placed directly below the Paper Data card. This makes it possible to inspect search results and immediately move into technical-keyword review, AI prompt generation, and TSV paste-back.

The card has three tabs.

| Tab | Description |
|---|---|
| A. Technical Keywords | Extract TF-IDF technical terms and phrases from titles and abstracts. |
| B. AI Prompt Generation | Generate prompts for technical-point TSV, Topic-naming TSV, and must-read paper TSV. |
| C. AI Result Paste-back | Import TSV results from an external AI system and apply them as annotations. |

#### Workflow

```text
1. Inspect technical keywords
  -> 2. Generate a task-specific prompt
  -> 3. Manually paste the prompt into an external AI system
  -> 4. Paste the AI's TSV output back into Collector
  -> 5. Reflect annotations in must-read papers, lineage, Topic labels, final prompts, and CSV exports
```

Collector does not call an AI API. The user reviews and controls what is sent to any external service.

#### Technical Keywords tab

The following parameters can be adjusted.

| Parameter | Description |
|---|---|
| Number of keywords | Number of terms or phrases to display. |
| Maximum phrase length | Maximum number of words in a phrase. |
| Minimum token length | Removes very short tokens. |
| Minimum document frequency | Keeps terms that appear in at least this many papers. |
| Document limit | Maximum number of papers used for keyword extraction. |
| Additional stopwords | User-defined terms to remove, such as search terms or overly broad field terms. |

#### AI Prompt Generation tab

Available prompt types:

| Type | Purpose | Expected output |
|---|---|---|
| Technical-point TSV | Add a short technology label, short summary, and importance reason to each paper. | TSV keyed by `work_id`. |
| Topic-naming TSV | Convert OpenAlex Topic names into human-readable technology labels. | TSV keyed by `topic_original`. |
| Must-read paper TSV | Add reading category, reading order, and reading reason to candidate papers. | TSV keyed by `work_id`. |

The technical-point prompt includes not only the top-scoring papers but also all current technology-lineage nodes. This ensures that lineage-important papers can receive AI annotations even if they are outside the top score range.

#### AI Result Paste-back tab

There are three paste-back forms.

| Form | Reflected in |
|---|---|
| Technical-point TSV | Lineage node labels, must-read paper technical points, overview/lineage prompts, and CSV exports. |
| Topic-naming TSV | Topic displays in the AI card, must-read papers, technology lineage, final synthesis prompts, and Topic Intelligence display labels. |
| Must-read paper TSV | Reading category, reading order, reading reason, and CSV exports. |

Pasted AI outputs do not overwrite OpenAlex metadata. They are layered as annotations.

---

### 4. Author / Institution Disambiguation

The Author / Institution Disambiguation card lets users handle authors and institutions by IDs, RORs, lineage metadata, and user-defined merge maps.

#### Basic policy

| Target | Preferred key | Supporting information |
|---|---|---|
| Author | OpenAlex Author ID | ORCID, display name, coauthors, Topics, institutions. |
| Institution | OpenAlex Institution ID | ROR, lineage, country, institution type, collaborators, Topics. |
| Author without ID | `name:` key | Name-only keys require caution because homonyms may be mixed. |
| Institution without ID | `name:` key | Name variants and homonymous organizations may be mixed. |
| User merge | `group:` key | Used for company groups, university + hospital grouping, or other analysis-specific units. |

Disambiguation affects rankings, citation analyses, collaboration networks, Topic × institution, Topic × author, funder × institution, funder × author, and network maps.

#### GUI workbench

Switch the target between Author and Institution, update the list, and open `Details / Disambiguation` for a key. The popup shows papers, coauthors or collaboration partners, Topics, and same-name candidates.

Main operations:

| Operation | Description |
|---|---|
| Merge candidate into this key | Merge another key into the current key. |
| Merge this key into candidate | Merge the current key into another key. |
| Bulk merge | Merge selected keys into the first selected key in display order. |
| Split | Remove a manually created merge rule and return to the original ID/ROR/name key. |
| Export JSON | Save the disambiguation map. |
| Import JSON | Reapply a saved disambiguation map. |

#### Institution aggregation mode

| Mode | Description |
|---|---|
| Exact ID | Treat OpenAlex Institution ID / ROR units separately. |
| Parent institution grouping | Use lineage to group under a parent institution. If the parent name is unavailable, the individual institution is retained. |
| Merge-map priority | Prefer user-defined merge maps. Useful for company groups or custom analytical units. |

#### Diagnostics

After search or CSV import, the following diagnostics are available.

| Diagnostic | Description |
|---|---|
| Author ID coverage | Share of author appearances with OpenAlex Author ID or ORCID. |
| Name-only authors | Author appearances without OpenAlex Author ID or ORCID. |
| Same-name multiple-author-key candidates | Same display name split across multiple keys. |
| Institution ID/ROR coverage | Share of institution appearances with OpenAlex Institution ID or ROR. |
| ROR-bearing affiliations | Affiliation appearances with ROR. |
| Same-name multiple-institution-key candidates | Same display name split across multiple institution keys. |

---

### 5. Charts

The Charts card provides basic quantitative views of the research field.

| Tab | Description |
|---|---|
| Yearly count bar chart | Shows publication counts by year. |
| Yearly count line chart | Shows the overall trend or selected author trends. |
| Author ranking | Ranks authors by paper count, with disambiguation applied. |
| Institution ranking | Ranks institutions by paper count, with disambiguation applied. |

Charts can be exported as PNG and CSV.

---

### 6. Citation Ranking

Citation Ranking shows citation-based influence for papers, authors, institutions, and venues.

| Analysis | Description |
|---|---|
| Highly cited papers | Papers ranked by citation count. |
| Author citation ranking | Total citations, average citations, and paper counts by author. |
| Institution citation ranking | Total citations, average citations, and paper counts by institution. |
| Journal / venue ranking | Paper counts and citation metrics by publication venue. |
| Quantity × quality scatter | Relationship between paper count and average citation count. |
| h-index ranking | h-index-like ranking for authors and institutions. |

Author and institution rankings reflect the current disambiguation map. Clickable tables and charts can open paper-list popups.

---

### 7. Collaboration Network by Institution

This card analyzes institutional coauthorship relationships.

| Analysis | Description |
|---|---|
| Institution pairs | Counts institution pairs that appear in the same papers. |
| Ego network | Shows collaborators of a selected institution. |
| Collaboration breadth | Shows how many partner institutions each institution has. |
| Pair time series | Shows yearly trends for a selected institution pair. |
| Collaboration matrix | Institution × institution matrix of coauthored papers. |

Use this card to study research communities, industry-academia relationships, international collaboration, and hub institutions.

---

### 8. Funding Intelligence

Funding Intelligence analyzes funders and award numbers using OpenAlex funding metadata and, when requested, external enrichment from Crossref REST API and NIH RePORTER API.

| Analysis | Description |
|---|---|
| Funder ranking | Paper counts, citations, and related Topics by funder. |
| Funder × year | Yearly trends by funder. |
| Award-number ranking | Counts award-number appearances. |
| Funder × institution | Shows which funders are linked to which institutions. |
| Funder × author | Shows which funders are linked to which authors. |
| Matrix output | Exports funder × institution and funder × author matrices. |
| External enrichment | Uses DOI and other metadata to query Crossref / NIH RePORTER for funding information. |

Funding analyses depend heavily on metadata completeness. Fields with sparse funding data may be underestimated.

---

### 9. Journal Intelligence

Analyzes the publication venue of each paper (OpenAlex `primary_location.source`): a journal ranking plus author × journal and institution × journal cross-tabs. Journals are identified by **source ID &gt; ISSN-L &gt; display name** to absorb name variants (authors and institutions inherit the "Disambiguation" settings).

| Tab | Description |
|---|---|
| A. Journal ranking | A per-journal table (papers / total citations / avg citations / h-index / OA rate / year range) plus a top-N bar chart. **Sort order toggles between "paper count" and "total citations".** |
| B. Author × Journal | Heatmap with rows = authors (after disambiguation), columns = journals. Shows which authors publish in which venues. |
| C. Institution × Journal | Heatmap with rows = institutions (after disambiguation, inheriting the institution aggregation mode), columns = journals. |

- **Metric toggle**: the "Metric" selector at the top switches both the ranking order and the B/C heatmap cell values between "paper count" and "total citations".
- **Type filter**: OpenAlex sources include journals as well as conference proceedings, preprint servers (e.g. arXiv), and repositories. The default is "Journals only", with options for "All sources", "Conference proceedings", "Repository / preprint", etc.
- **Click-through**: clicking a ranking row, or a heatmap cell / row header, opens the matching papers or the author / institution / journal detail.
- **Export**: ranking CSV (all journals), author × journal and institution × journal matrix CSVs (full), and a PNG of the ranking bar chart.
- h-index is a reference value computed within this collection only (under-estimates for small collections). OA rate is based on `open_access.is_oa`.
- If you re-import a CSV from an older version without source IDs, aggregation falls back to display names; use the "All sources" type filter in that case.

---

### 10. Topic Intelligence

Topic Intelligence uses OpenAlex Topics to summarize the structure of a research field.

| Analysis | Description |
|---|---|
| Topic ranking | Paper count, average citations, and active period by Topic. |
| Topic × year | Yearly Topic trends. |
| Emerging Topic candidates | Estimates growing Topics from recent counts, previous counts, growth, and average citations. |
| Topic × institution | Shows which institutions contribute to which Topics. |
| Topic × author | Shows which authors contribute to which Topics. |
| Topic × funder | Shows which funders are linked to which Topics. |
| Matrices | Exports Topic × year, Topic × institution, Author × Topic, Topic × funder, and related matrices. |

#### Applying AI Topic labels

When `Apply AI Topic labels in Topic Intelligence` is enabled, `technology_label` values imported through the Topic-naming TSV are used as display labels. The aggregation key remains the original OpenAlex Topic.

This preserves reproducibility while making charts and reports easier to read.

---

### 11. Network Map: Collaboration / Co-authorship

The Network Map card visualizes collaboration and coauthorship networks with vis-network.

| Network | Node | Edge |
|---|---|---|
| Institution network | Institution | Institution pair appearing in the same paper. |
| Author network | Author | Author pair appearing in the same paper. |

Node sizes and edge widths depend on metrics such as paper count, coauthorship count, and centrality. Clicking nodes or edges opens a detail popup with related papers and metrics. In addition to node/edge CSVs, the "Download PNG" button on each tab saves the displayed network diagram as an image.

---

### 12. Citation Network: References

The Citation Network card analyzes references used by the retrieved paper collection and internal citation relationships among retrieved papers.

| Analysis | Description |
|---|---|
| Top references | References frequently cited by the retrieved papers. |
| Co-citation | Pairs of references cited together by the same papers. |
| Bibliographic coupling | Pairs of papers sharing common references. |
| Internal citation network | Directed network among papers in the current collection. |

The in-collection citation network can be saved as an image via "Download PNG". The technology-lineage module also uses internal citation and bibliographic coupling information when inferring parent-child relationships.

---

### 13. Must-read Papers & Technology Lineage

This card combines paper triage and technology lineage in a single card. Use the tabs to switch between `A. Must-read Papers` and `B. Technology Lineage`.

#### A. Must-read Papers

The Must-read Papers tab calculates an explainable score from six components.

| Component | Meaning |
|---|---|
| Topic growth | Rewards papers in Topics that are growing recently. |
| Age-adjusted citations | Evaluates citation impact while reducing excessive advantage for older papers. |
| Bridging potential | Estimates cross-field or cross-community potential from authors, institutions, Topics, and references. |
| Technical-keyword novelty | Uses TF-IDF technical keywords to detect technically distinctive papers. |
| Author / institution signal | Uses major authors and institutions as signals. |
| Funding signal | Rewards papers with funding information. |

Weights can be adjusted in the UI. The top-N setting controls how many score-leading papers are displayed.

When `Include technology-lineage nodes` is enabled, papers included in the technology lineage are added to the must-read list even if they are outside the top-N score range. This helps avoid missing foundational nodes, branching nodes, convergence nodes, and recent frontier nodes.

Each row has a detail button. The detail popup shows paper metadata, AI technical points, short summaries, importance reasons, reading category, reading order, reading reason, score components, technical keywords, Topic, authors, institutions, and venue.

#### B. Technology Lineage

The Technology Lineage tab displays representative papers as a left-to-right tree/DAG. The horizontal position corresponds to time (older on the left, newer on the right), and nodes are packed vertically within each era column. This spreads the lineage horizontally from early to recent works so the diagram does not become extremely tall and narrow. Edges generally mean `earlier paper -> later paper`.

Parent-child relationships are inferred in this priority order:

1. Direct citation within the current collection
2. Topic overlap
3. Technical-keyword Jaccard similarity
4. Author / institution overlap
5. Bibliographic coupling through shared references

If a direct citation exists, it is prioritized. If no direct citation is available, the tool selects the closest earlier parent candidate using Topic, keyword, author/institution, and reference signals. Nodes whose relationships all fall below the minimum relation score are not left isolated; they are connected to their closest earlier paper with a dotted "weak relation" edge.

To avoid skewing toward only the highly-cited early papers, the displayed nodes are chosen by splitting the year range into era blocks and reserving high-score papers from each era. This keeps the lineage continuous from the roots / early works through to the recent frontier.

| Parameter | Description |
|---|---|
| Number of nodes | Maximum number of papers shown in the lineage. |
| Minimum score | Exclude nodes below this must-read score. |
| Minimum relation score | Inferred edges below this value are shown as dotted, faint "weak relation" edges (auxiliary links that keep otherwise-isolated nodes connected). Raising it reduces solid clear-relationship edges and increases dotted auxiliary edges. |
| Root candidate limit | Limit the number of root candidates. |
| Root citation weight | Controls how strongly citations influence root candidate selection. |
| Start / end year | Restrict the lineage to a year range. |
| Topic filter | Show lineage related to specific Topics or keywords. |

Clicking a node opens a paper-detail popup. Clicking an edge shows the relationship reason, such as direct citation, Topic match, keyword similarity, author/institution continuity, bibliographic coupling, or a dotted weak relation used to keep the lineage connected.

The lineage table also has detail buttons and displays node explanations, AI / reading annotations, parent nodes, and relationship reasons. The lineage can be exported as CSV, and the lineage diagram can be saved as a PNG image via "Download PNG".

---

### 14. AI Analysis Report: Final Synthesis

The AI Analysis Report card is the final synthesis area. It does not perform technical-point assignment or Topic naming; those tasks belong to the dedicated AI annotation card. This final card prepares a report-style prompt for interpreting the collection and its technology lineage.

Main outputs:

| Output | Description |
|---|---|
| Overview memo | Uses only Collector-side statistics to summarize paper count, year range, total citations, major Topics, growing Topics, technical keywords, and lineage clues. |
| Overview / lineage prompt | Generates a prompt for an external AI system to produce a readable analytical report. |

The overview / lineage prompt requests sections such as:

1. Executive summary
2. Reading of the collection
3. Reading of the technology lineage
4. Relationship between technical points and Topics
5. Major authors, institutions, journals, and funding signals
6. Must-read paper guide
7. Dataset limitations
8. Next checks

The prompt asks the external AI to separate `Layer 1: Facts`, `Layer 2: Interpretation`, `Layer 3: Insights`, and `Layer 4: Recommendations`. Layer 4 is constrained to next checks, additional research directions, and reading strategy rather than strong business claims.

---

### 15. Detail Popups

Detail popups are shared UI components for drilling down from analysis cards into the underlying papers or entities.

Depending on the target, a popup may show:

| Target | Examples |
|---|---|
| Paper | Title, year, authors, institutions, venue, DOI, citations, Topic, abstract, funding, references, AI annotations. |
| Author | Paper count, total citations, related Topics, coauthors, institutions, same-name candidates, disambiguation actions. |
| Institution | Paper count, total citations, collaboration partners, related Topics, lineage, ROR, disambiguation actions. |
| Topic | Paper count, yearly trend, representative papers, institutions, authors, funders. |
| Funder | Related papers, institutions, authors, Topics, award numbers. |
| Technology-lineage node | Technical point, node explanation, parent node, relationship reason, AI summary, reading reason. |

Popups are intended for human validation of rankings, disambiguation, AI annotations, and lineage relationships.

---

### 16. Export / Import

v7 supports several output types.

| Output | Description |
|---|---|
| Paper CSV | Full current paper collection, including AI annotations, identity keys, funding information, and Collector match information. |
| Chart PNG | Chart.js graph images (trend lines, rankings, scatter, etc.). |
| Network / lineage PNG | Images of vis-network diagrams: institution network, author network, citation network, and technology lineage (saved on a white background). Use the "Download PNG" button on each card. |
| Chart CSV | Source data for charts. |
| Matrix CSV | Topic × year, Topic × institution, Author × Topic, Funder × institution, and related matrices. |
| Network CSV | Node and edge tables. |
| Must-read paper CSV | Scores, candidate sources, technical points, AI summaries, reading categories, reading order, and reading reasons. |
| Technology-lineage CSV | Nodes, parent-child relationships, relationship reasons, AI annotations, Topics, authors, and institutions. |
| AI prompt Markdown | Saves generated prompts for external AI systems. |
| Overview Markdown | Saves the final overview memo. |
| Disambiguation-map JSON | Saves author and institution merge rules. |

Saved CSV files can be imported to resume analysis. When AI annotation columns are present, technical points, summaries, importance reasons, reading categories, reading order, reading reasons, and AI Topic labels can be restored.

---

## How AI assistance works

AI assistance in v7 is divided into three layers.

| Layer | Collector does | External AI does |
|---|---|---|
| Preparation | Extracts technical keywords, scores must-read papers, builds technology lineage, aggregates Topics, and generates prompts. | Nothing. |
| External AI processing | Outputs prompts for the user to copy. | Produces technical-point TSV, Topic-naming TSV, must-read TSV, or a narrative report. |
| Paste-back and reflection | Imports TSV and reflects annotations in papers, Topics, lineage, prompts, and CSV exports. | Nothing. |

Advantages of this design:

- No AI API key is entered into Collector.
- No automatic external transmission occurs.
- The user can inspect exactly what will be sent externally.
- AI outputs can be structured as TSV and matched by Work ID or Topic name.
- AI annotations do not overwrite source metadata and can be cleared.
- CSV export makes AI annotations reusable later.

---

## TF-IDF technical-keyword extraction

Technical-keyword extraction uses one- to N-word phrases from titles and abstracts and weights them with TF-IDF.

```text
TF-IDF = TF × (log((number of documents + 1) / (document frequency + 1)) + 1)
```

| Element | Description |
|---|---|
| TF | Frequency of the phrase. |
| DF | Number of documents in which the phrase appears. |
| IDF | Downweights terms that appear in too many documents. |
| Stopwords | Articles, prepositions, conjunctions, pronouns, auxiliary verbs, paper-generic words, research-generic words, broad adjectives, and unit expressions are removed. |
| Additional stopwords | User-defined terms can be removed, such as the search terms themselves or overly broad field terms. |

Technical keywords are used in must-read scoring, lineage node labels, AI prompts, and report preparation.

---

## Must-read paper score

The must-read paper score normalizes multiple signals to 0–1, applies user-defined weights, and converts the result to a 0–100 score.

```text
score = 100 × Σ(weight_i × component_i) / Σ(weight_i)
```

Default weights:

| Component | Default weight | Description |
|---|---:|---|
| Topic growth | 0.25 | Rewards papers in growing Topics. |
| Age-adjusted citations | 0.20 | Citation impact adjusted for publication age. |
| Bridging potential | 0.20 | Estimates cross-community potential from authors, institutions, Topics, and references. |
| Technical-keyword novelty | 0.15 | Distinctiveness based on TF-IDF technical keywords. |
| Author / institution signal | 0.10 | Signal from major authors and institutions. |
| Funding signal | 0.10 | Signal from funding metadata. |

Suggested adjustments:

| Goal | Suggested adjustment |
|---|---|
| Track recent growth themes | Increase Topic growth and technical-keyword novelty. |
| Find foundational papers | Increase age-adjusted citations. |
| Find bridge papers | Increase bridging potential. |
| Focus on major players | Increase author / institution signal. |
| Focus on policy or funding interest | Increase funding signal. |
| Reduce noise | Reduce candidate count and add stopwords. |

---

## Technology-lineage logic

Technology lineage is a hypothesis map built from a paper collection. It selects representative paper nodes and infers relationships from citations, Topics, keywords, authors/institutions, and references. It is not a definitive historical reconstruction.

| Element | Description |
|---|---|
| Node | Representative paper. The label comes from AI technical points, TF-IDF keywords, or a shortened title. |
| Edge | Relationship from an earlier paper to a later paper. Direct citation is prioritized. |
| Root candidate | Older paper with strong citation or internal-citation signals. |
| Branch | Direction where Topics or technical keywords diverge. |
| Convergence | Point where multiple Topics, keywords, or references connect. |
| Recent frontier | Recent high-score papers or papers in growing Topics. |

When the view becomes too dense, reduce the node count, increase the minimum relation score, apply a Topic filter, or restrict the year range.

---

## AI prompt design

v7 generates several prompt types.

| Prompt | Purpose | Output |
|---|---|---|
| Technical-point TSV | Add technology label, short summary, and importance reason to papers. | TSV |
| Topic-naming TSV | Convert OpenAlex Topic labels into human-readable technology labels. | TSV |
| Must-read paper TSV | Assign reading category, reading order, and reading reason. | TSV |
| Overview / lineage prompt | Produce a narrative report on the field and technology lineage. | Markdown or another narrative format. |

Prompts may include:

- Paper count, year range, and total citations
- Major Topics and growing Topics
- Technical-keyword candidates
- Must-read paper candidates
- Technology-lineage nodes
- Work IDs, DOIs, years, titles, venues, citation counts
- Authors, institutions, funders
- Pasted AI technical points and Topic labels when available

The prompts instruct the external AI to:

- Avoid unsupported market predictions or deployment claims.
- Attach Work ID, year, and title to important claims.
- Distinguish evidence from inference.
- Follow the requested TSV schema when TSV output is required.
- Keep original OpenAlex Topics distinct from AI-generated labels.

Note: the four-layer model (`Layer 1: Facts / Layer 2: Interpretation / Layer 3: Insight / Layer 4: Recommendation`) is used only in the overview / lineage (final synthesis) prompt. The technical-point, Topic-naming, and must-read TSV prompts do not use the four-layer model; each field is produced as a concise statement without layer labels or embedded statistics.

---

## AI paste-back TSV formats

### Technical-point TSV

```text
work_id	technical_point	short_summary	importance_reason
```

| Column | Description |
|---|---|
| `work_id` | OpenAlex Work ID used to match the paper in Collector. |
| `technical_point` | Short technology label for tables and nodes. |
| `short_summary` | Short explanation of the paper. |
| `importance_reason` | Why the paper is important or worth reading. |

Reflected in must-read papers, technology-lineage nodes, final synthesis prompts, and CSV exports.

### Topic-naming TSV

```text
topic_original	technology_label	short_definition	representative_keywords	representative_work_ids
```

| Column | Description |
|---|---|
| `topic_original` | Original OpenAlex Topic label used as the matching key. |
| `technology_label` | A concise, specific technology name (noun phrase) usable as a survey heading. Do not include statistics (counts, citations, years, work_ids) or subjective wording such as "mainstream". |
| `short_definition` | A neutral 1–2 sentence definition of the research subject and methods the name covers (not analysis or statistics). |
| `representative_keywords` | Representative keywords. |
| `representative_work_ids` | Representative Work IDs. |

Reflected in Topic displays, must-read papers, technology lineage, final synthesis prompts, and Topic Intelligence display labels. The aggregation key remains the original OpenAlex Topic.

### Must-read paper TSV

```text
work_id	must_read_category	reading_order	must_read_reason	technical_point	short_summary
```

| Column | Description |
|---|---|
| `work_id` | OpenAlex Work ID. |
| `must_read_category` | Example: foundation, bridge paper, rising topic, recent must-read. |
| `reading_order` | Suggested reading order. |
| `must_read_reason` | Why the paper should be read. |
| `technical_point` | Optional; also reflected as a technical point. |
| `short_summary` | Optional; also reflected as a short summary. |

Work IDs not present in the current collection are skipped. Collector reports matched and skipped counts after import.

---

## Applying AI Topic labels to Topic Intelligence

AI Topic labels are treated as display overlays for OpenAlex Topics.

```text
OpenAlex Topic: Batteries, Fuel Cells, and Energy Storage
AI label: Solid-state battery electrolyte technology
```

When the AI-label mode is enabled in Topic Intelligence, charts and tables display the AI label where available. The original OpenAlex Topic remains the internal aggregation key and is preserved in CSV exports.

This allows:

- Reproducibility against the source data
- Human-readable labels for reports
- CSV-based auditing
- Easier interpretation in AI prompts

---

## Disambiguation-map JSON format

A disambiguation map is a JSON file. It can be exported from the GUI and re-imported later.

```json
{
  "version": 1,
  "updated_at": "2026-05-30T00:00:00.000Z",
  "institutionAggregationMode": "exact",
  "author": {
    "aliases": {
      "https://openalex.org/A222": {
        "canonical": "https://openalex.org/A111",
        "label": "Akira Yoshino",
        "decision": "manual_merge"
      }
    },
    "labels": {
      "https://openalex.org/A111": "Akira Yoshino"
    }
  },
  "institution": {
    "aliases": {
      "https://openalex.org/I123": {
        "canonical": "group:toyota",
        "label": "Toyota Group",
        "decision": "manual_merge"
      }
    },
    "labels": {
      "group:toyota": "Toyota Group"
    }
  }
}
```

Examples of usable keys:

| Type | Example |
|---|---|
| OpenAlex Author ID | `https://openalex.org/A123456789` |
| OpenAlex Institution ID | `https://openalex.org/I136199984` |
| ROR | `https://ror.org/057zh3y96` |
| Name key | `name:john smith` |
| User-defined group | `group:toyota` |

---

## Disambiguation storage and cache design

v7 separates performance caches from user decisions.

| Type | Storage | Purpose |
|---|---|---|
| Computation cache | Memory | Speeds up rendering and aggregation. Cleared on page reload. |
| Dataset-specific disambiguation | Browser storage | Reuses decisions for the same dataset. |
| JSON disambiguation map | User-saved file | Transfers decisions to another browser, computer, or future analysis. |
| CSV IDs and AI annotations | CSV file | Reuses paper data, identity keys, and AI annotations. |

To avoid mixing decisions from a previous analysis, reset the current map and import only the JSON file intended for the current dataset.

---

## Topic score definition

Topic Intelligence uses metrics such as:

| Metric | Description |
|---|---|
| Paper count | Number of papers assigned to the Topic. |
| Average citations | Average citation count for papers in the Topic. |
| Active period | First and last publication year. |
| Recent count | Number of papers in the latest three-year window. |
| Previous count | Number of papers in the three years before the recent window. |
| Delta | Recent count minus previous count. |
| Emerging Topic score | Convenience score combining recent count, delta, and citation information. |

The emerging Topic score is a heuristic for detecting growth. Topics with small paper counts may fluctuate strongly.

---

## Network metrics

Network maps and collaboration analysis use metrics such as:

| Metric | Description |
|---|---|
| Node | Author or institution. |
| Edge | Coauthorship or collaboration relationship. |
| Edge weight | Number of papers shared by the pair. |
| Degree | Number of directly connected nodes. |
| Weighted degree | Connection strength including edge weights. |
| Betweenness centrality | Extent to which a node bridges other parts of the network. |
| Component | Connected group of nodes. |

For large datasets, adjust minimum edge weight, node limit, year range, and Topic filters to improve readability.

---

## CSV output columns

The main paper CSV includes columns such as:

| Column group | Examples |
|---|---|
| Paper metadata | `paper_id`, `title`, `abstract`, `year`, `publication_date`, `doi`, `language`, `title_script` |
| Authors | `authors`, `raw_authors`, `author_names`, `author_ids`, `author_orcids`, `author_identity_keys`, `author_identity_labels`, `author_entities_json` |
| Institutions | `institutions`, `raw_institutions`, `institution_names`, `institution_ids`, `institution_rors`, `institution_country_codes`, `institution_types`, `institution_lineage_ids`, `institution_identity_keys`, `institution_identity_labels`, `institution_entities_json` |
| Venue and citations | `venue`, `citation_count`, `referenced_works_count`, `referenced_works` |
| Topics | `primary_topic`, `topics` |
| Collector search metadata | `collector_query`, `collector_candidate_query`, `collector_match`, `collector_match_field`, `collector_match_operator`, `collector_match_distance`, `collector_match_snippet` |
| Funding | `funder_names`, `funder_ids`, `award_ids`, `funding_sources`, `crossref_funder_names`, `crossref_award_ids`, `nih_project_nums`, `nih_award_amounts`, `nih_fiscal_years`, `nih_orgs`, `nih_pis`, `nih_project_titles` |
| AI annotations | `ai_technical_point`, `ai_short_summary`, `ai_importance_reason`, `ai_must_read_category`, `ai_reading_order`, `ai_must_read_reason`, `ai_topic_label`, `ai_topic_definition` |
| Disambiguation | `identity_version` |

The must-read paper CSV includes score, candidate source, Work ID, DOI, year, venue, citations, Topic, AI Topic label, AI technical point, AI summary, reading category, reading order, reading reason, title, authors, institutions, keywords, and lineage membership.

The technology-lineage CSV includes rank, year, priority score, root score, internal citations, Work ID, technical point, node explanation, AI annotations, Topic, AI Topic label, venue, authors, institutions, parent Work ID, relation, relation score, title, and reason.

---

## Search syntax

### Search modes

v7 provides two search modes:

1. Command-line query mode
2. Standard OpenAlex search mode

Command-line query mode uses numbered conditions and a final T expression.

```text
#01  TI=(solid adj3 electrolyte)
#02  AB=((degradation AND capacity) near8 (mechanism OR fade*))
#03  TA=(review)
T=(#01 AND #02) AND NOT #03
```

Standard OpenAlex search passes terms directly to OpenAlex.

```text
"solid state battery"
"sulfide electrolyte"
```

Multiple lines are treated as OR queries.

### Fields

| Field | Target | Notes |
|---|---|---|
| `TI` | Title | Internal near/adj validation is available. |
| `AB` | Abstract | Internal near/adj validation is available. |
| `TA` | Title + abstract | Internal near/adj validation is available. |
| `TX` | OpenAlex general search | Candidate retrieval. |
| `FT` | OpenAlex fulltext index | Candidate retrieval. |

### Operators

| Operator | Example | Meaning |
|---|---|---|
| `AND` | `A AND B` | Both A and B. |
| `OR` | `A OR B` | A or B. |
| `NOT` | `A AND NOT B` | Exclude B. |
| `nearN` | `A near5 B` | A and B within N words in any order. |
| `adjN` | `A adj3 B` | A before B within N words. |
| `*` | `cathod*` | Multi-character wildcard. |
| `?` | `wom?n` | Single-character wildcard. |

---

## Use-case guide

### Build a reading order from a large paper collection

1. Enable year-by-year retrieval and collect a broad dataset.
2. Adjust paper type, year range, abstract availability, English-only, and other filters to reduce noise.
3. In Technical Keywords & AI Annotations, add search terms and overly broad words to stopwords.
4. Adjust must-read score weights.
5. Enable inclusion of technology-lineage nodes.
6. Export the must-read paper CSV.
7. Optionally generate a must-read paper TSV prompt, send it to an external AI system, and paste back reading categories and reading order.

### Make abstract OpenAlex Topic names easier to read

1. Review major Topics in Topic Intelligence.
2. Generate a Topic-naming TSV prompt in Technical Keywords & AI Annotations.
3. Paste the returned TSV into the Topic-naming form.
4. Enable AI Topic labels in Topic Intelligence.
5. Re-read Topic ranking, Topic × year, Topic × institution, Topic × author, and Topic × funder views.

### Create a technology-lineage report

1. Recalculate must-read scores.
2. Adjust node count, minimum relation score, root candidate limit, year range, and Topic filter in the Technology Lineage tab.
3. Inspect node and edge details in popups.
4. Export the technology-lineage CSV.
5. Generate an overview / lineage prompt in the AI Analysis Report card.
6. Check that the external AI output attaches Work IDs, years, and titles to important claims.

### Analyze a field with many author homonyms

1. Open Author / Institution Disambiguation after retrieval.
2. Check author ID coverage and same-name multiple-key candidates.
3. Merge keys if they refer to the same person; otherwise keep them separate.
4. Export the disambiguation map as JSON.
5. Re-render author rankings, Author × Topic, and author networks.

### Analyze institutions at company-group level

1. Open institution disambiguation.
2. Inspect subsidiaries, laboratories, hospitals, and related entities.
3. Merge them into a user-defined group such as `group:company` when appropriate for the analysis.
4. Re-render collaboration, Topic × institution, funder × institution, and institution-network analyses.
5. Export the disambiguation map for reuse.

### Validate a search population

1. Run syntax check / OpenAlex candidate preview.
2. Check candidate count, internal match count, and dropped count.
3. If `nearN` or `adjN` is used, inspect internal match snippets.
4. If too many papers are dropped, adjust proximity distance, proximity group width, or OR split limit.

---

## External APIs and data retrieval

v7 uses OpenAlex as the central source for building the paper collection and obtaining core metadata. Other APIs are used only to enrich the analysis.

| API / library | Purpose | Notes |
|---|---|---|
| OpenAlex API | Papers, authors, institutions, Topics, references, and funding metadata. | The foundation of search and analysis. **An API key is required as of February 13, 2026 (free)** — set it on the in-app "OpenAlex API Key" screen ([Quick start](#quick-start)); it is sent as the `api_key` parameter. Use year-by-year mode for large retrievals and avoid unnecessary repeated queries. |
| Crossref REST API | Funding enrichment using DOI metadata. | Funding metadata is not available for every paper. |
| NIH RePORTER API | NIH-related funding enrichment. | Most useful for biomedical and NIH-funded fields. |
| Chart.js | Chart rendering. | Loaded from a CDN. |
| vis-network | Network maps and technology-lineage graphs. | Loaded from a CDN. |
| External AI services | Interpret prompts, generate TSV, and write narrative reports. | Collector never sends data automatically. The user manually controls external use. |

For large searches, broad queries can increase candidate counts, retrieval time, and browser-side processing load. A practical workflow is to start with a small retrieval limit, validate the query and noise level, and only then increase limits or enable year-by-year retrieval.

### OpenAlex candidates and Collector internal matches

In command-line query mode, Collector retrieves candidates from OpenAlex and performs internal validation only when needed.

| Stage | Description |
|---|---|
| OpenAlex candidates | Candidate papers returned by OpenAlex. |
| Collector internal matches | Papers that pass Collector-side validation such as strict proximity checks. |
| Collector internal drops | OpenAlex candidates that fail internal validation. |

Internal dropping primarily occurs when proximity operators such as `nearN` or `adjN` are used in `TI`, `AB`, or `TA`. For normal keyword, Boolean, and wildcard searches, Collector generally accepts the OpenAlex candidate set.

### Practical guidelines for large retrievals

| Situation | Suggested action |
|---|---|
| Validate the search population | Start with a retrieval limit of about 200–500 papers. |
| Review yearly trends broadly | Set a year range and enable year-by-year retrieval. |
| Browser becomes slow | Narrow by year, publication type, abstract availability, English-only, or minimum citations. |
| Test only Topic analysis or AI prompts | Create a smaller CSV first and validate the workflow. |
| Avoid repeating the same search | Save the paper CSV, disambiguation-map JSON, and AI-annotated CSV. |

---

## How to interpret analytical outputs

v7 produces many rankings, scores, and networks. Treat them as maps for investigation, not as final conclusions. Final judgment should be based on detail popups, original papers, search conditions, and disambiguation status.

| Output | How to read it | Caution |
|---|---|---|
| Yearly trend | Indicates quantitative growth or decline. | Affected by search query and OpenAlex coverage. |
| Citation ranking | Shows influential papers, authors, institutions, or venues. | Older papers often have an advantage. |
| Topic ranking | Shows the topical structure of the collection. | OpenAlex Topics may not match analyst-defined technology categories. |
| Emerging Topics | Suggests Topics that may be growing. | Small-count Topics can be unstable. |
| Funding Intelligence | Shows relationships between funding and research activity. | Sparse funding metadata can lead to underestimation. |
| Collaboration network | Shows coauthorship and institutional collaboration structure. | Large multi-author papers can create strong edges. |
| Reference network | Shows shared intellectual bases and internal citation hubs. | Depends on availability of reference IDs. |
| Must-read score | Creates a candidate reading order. | Rankings change when weights change. |
| Technology lineage | Provides a hypothesis map of technical development. | Non-citation edges are inferred and require review. |
| AI labels | Improve human readability. | They are annotations, not source metadata. |

### What to verify in detail popups

- Which papers support a ranking item or lineage node.
- Whether authors and institutions are disambiguated correctly.
- Whether Topic labels fit the analytical purpose.
- Whether a citation-based ranking is reasonable after considering publication year and paper type.
- Whether AI annotations are consistent with the title and abstract.
- Whether a lineage edge is a direct citation or an inferred relationship.

---

## How to validate AI outputs

Review external AI outputs before and after pasting them back into Collector. AI systems can produce incorrect Work IDs, Topic labels, technology names, or reading order.

| Check | What to verify |
|---|---|
| Work ID | `work_id` exists in the current paper collection. |
| Topic name | `topic_original` matches an OpenAlex Topic in Collector. |
| TSV columns | Required column names are present and tab-separated. |
| Technology label | Not too abstract and not mixing too many concepts. |
| `short_summary` | Does not assert facts absent from title or abstract. |
| `importance_reason` | Supported by citations, Topic position, lineage role, or other evidence. |
| `reading_order` | Moves reasonably from foundations to developments to recent frontiers. |
| `must_read_category` | Not overly vague or inconsistent. |
| Narrative report | Important claims include Work ID, year, and title. |
| Recommendations | Do not overreach into unsupported market or deployment claims. |

After paste-back, inspect must-read paper rows and technology-lineage detail popups to confirm that annotations are attached to the intended papers. Annotations can be cleared. Save an AI-annotated CSV if you want to reuse them later.

---

## Common setting patterns

### Get a quick overview of a field

| Setting | Suggested value / action |
|---|---|
| Search mode | Standard OpenAlex search, or a broad `TA=(...)` query. |
| Retrieval limit | Around 200–1,000 papers. |
| Year range | Last 5–10 years. |
| Advanced filters | Enable abstract-only and English-only; optionally select Article / Review. |
| Cards to inspect | Yearly trends, Topic Intelligence, Citation Ranking, Technical Keywords. |

### Build a systematic search population

| Setting | Suggested value / action |
|---|---|
| Search mode | Command-line query mode. |
| Fields | Explicitly use `TI`, `AB`, or `TA`. |
| Query | Group synonyms with OR and remove noise with NOT. |
| Proximity | If using `nearN` / `adjN`, check syntax and snippets. |
| Year-by-year retrieval | Enable when the collection is large. |
| Save | Save paper CSV, query notes, and disambiguation map. |

### Make technology lineage easier to read

| Setting | Suggested value / action |
|---|---|
| Node count | Start around 30–80 nodes. |
| Minimum relation score | Increase if the graph is too dense; decrease if branches are missing. |
| Root candidate limit | Start around 5–15. |
| Topic filter | Use when focusing on a specific theme. |
| AI annotations | Paste back technical-point TSV to make node labels more readable. |
| Export | Save technology-lineage CSV and overview / lineage prompt. |

### Generate a report with external AI

| Setting | Suggested value / action |
|---|---|
| Preparation | Review technical keywords, must-read papers, and technology lineage first. |
| Topic naming | If Topic names are too abstract, paste back Topic-naming TSV before final prompting. |
| Prompt paper count | Start around 20–40 papers. |
| Instruction | Separate facts, interpretation, insights, and next checks. |
| Validation | Use only claims supported by Work IDs, years, and titles. |

---

## Performance (speed optimizations)

With a large collection (thousands to tens of thousands of papers), post-retrieval aggregation and network computations can get heavy and briefly freeze the browser. v7 includes five measures to keep it responsive.

### A. Lazy rendering (compute only when needed)

Instead of computing every card at once right after a search, **each analysis card is computed only when you open its sidebar tab**. After a search, the metrics and paper table appear first; Funding, Topic, Technical Keywords, Must-read / Lineage, and disambiguation diagnostics are computed when you open that tab. This avoids the "compute everything the instant you search" freeze. Once computed, a card is cached and re-displays instantly.

### B. Memoization (don't repeat the same work)

- **Shared technology-lineage snapshot**: Must-read scoring and the three AI prompt builders all assemble the technology lineage internally. This used to run about seven times per update; now it is **computed once and reused** (the lineage relationship inference is quadratic in node count, so this matters a lot).
- **Cached aggregations**: TF-IDF keywords, must-read scores, and topic aggregation are cached by (dataset fingerprint + parameters + extra stopwords). Switching tabs, or changing only display-side parameters, reuses the cache instead of recomputing. The key does not depend on AI annotations, so the display still updates correctly after paste-back.

### C. Algorithmic improvement (removing O(N²))

- **Bibliographic coupling**: previously compared every pair of papers (O(N²); about 12.5M pairs at N=5,000). It now uses an **inverted index** (reference → list of papers citing it) and generates **only the pairs that actually share a reference**. Verified to produce the same results as the brute-force method.
- **Bound guards**: co-citation caps the number of references considered per paper, and bibliographic coupling skips references cited by an extreme number of papers (hub references), to bound the worst case. **Because of these caps, a small number of pairs may be left out of the aggregation on very large datasets (an approximation). We document this here rather than truncating silently.** It rarely matters at the usual scale of thousands to tens of thousands of papers.

### D. Web Worker (off the main thread)

The heavy reference analyses (co-citation, bibliographic coupling) run in a **Web Worker (a separate thread)**, so the UI stays responsive during computation.

- **Robust fallback**: where the browser blocks Workers (e.g., some local `file://` contexts), it automatically **falls back to synchronous computation**, with a timeout safeguard so it never hangs. Even without a Worker, it is fast thanks to C (the inverted index).

### E. Large-N optimization of identity aggregation (tens of thousands of papers)

Author/institution aggregation (charts, citation ranking, Journal Intelligence, etc.) extracts author/institution entities per paper, applies disambiguation, and aggregates. At the scale of tens of thousands of papers this repeatedly re-scanned the whole dataset and became very slow. Three changes fix it:

- **O(1) dataset fingerprint**: the internal cache-validity check used to recompute a "sort + hash of all paper IDs" on every call; it is now memoized while the paper array is unchanged. (That check was invoked from many loops, so at large N it was effectively O(N²) — a multi-second freeze source.)
- **Reused disambiguation labels**: the display-name canonicalization map is cached per (dataset + aggregation mode) and shared across charts, citation ranking, and Journal Intelligence.
- **One-time entity build**: each paper's disambiguated authors/institutions are built once per dataset and cached, then reused by every analysis. Changing the disambiguation map or aggregation mode rebuilds it automatically.

As a result, even with tens of thousands of papers only the **first** pass after retrieval takes a short time (building the disambiguated entities); **subsequent card views, tab switches, and metric toggles are near-instant**.

### Overall effect

Combining A (no bulk compute), B (no redundant recompute), C (no O(N²)), D (off-thread), and E (identity-aggregation optimization) substantially reduces both the "freeze right after a search" and the "stall when operating a card" on collections of tens of thousands of papers. If something is still heavy, narrowing the scope with the in-card filters (minimum edge weight, displayed node count, year range, Topic filter, minimum document frequency, the Journal Intelligence row/column caps, etc.) makes it lighter still. Performance also depends on browser memory and machine specs, so for extremely large collections, splitting retrieval into smaller batches helps.

---

## Technical specifications

| Item | Description |
|---|---|
| Runtime | Single HTML file. |
| Primary API | OpenAlex API. |
| Enrichment APIs | Crossref REST API, NIH RePORTER API. |
| AI API | Not used. |
| Charts | Chart.js. |
| Networks | vis-network. |
| Outputs | CSV, PNG, Markdown, JSON. |
| Disambiguation | Author ID, ORCID, Institution ID, ROR, lineage, and user-defined canonical keys. |
| AI annotations | Kept in memory and reusable through CSV export/import. |
| Browsers | Chrome, Edge, Firefox, Safari. |

---

## Notes and limitations

- OpenAlex metadata quality, Topic assignment, references, and funding data vary by field and time period.
- Citation counts depend on the OpenAlex snapshot available at retrieval time.
- OpenAlex Topics may not match the categories an analyst would use for technology research.
- AI Topic labels are display aids and do not replace source Topics.
- Technology lineage is a hypothesis map inferred from the current paper collection; it is not a definitive history of a technology.
- Must-read scores should be adjusted according to the research purpose.
- Prompts sent to external AI systems may contain titles, abstracts, authors, institutions, and other metadata. Follow the policies of your organization and the external AI service.
- The tool runs locally, but CDN libraries and external API features require internet access.
- To stay responsive on large datasets, each analysis card (Funding, Topic, Technical Keywords & AI, Must-read & Lineage, disambiguation diagnostics) is computed when you scroll it into view. Right after a search, the metrics and paper table appear first, avoiding a freeze from computing everything at once.
- The heavy reference analyses (co-citation, bibliographic coupling) run in a Web Worker on a separate thread, with automatic fallback to synchronous computation where Workers are unavailable (e.g., some local-file contexts). They cap the references considered per paper and skip references cited by an extreme number of papers, so results may be approximate at very large scale.
- Use large retrieval responsibly and avoid unnecessary load on OpenAlex.

---

## Troubleshooting

| Symptom | Action |
|---|---|
| Search fails with `403` / `401` / rate-limit errors | Your OpenAlex API key may be missing, invalid, or over its limit. A key is required as of February 13, 2026. Set it on the "OpenAlex API Key" screen at the top of the left sidebar and use "Test connection" to verify (→ [Quick start](#quick-start)). |
| Search returns no result | First confirm your OpenAlex API key is set. Then check query syntax, years, publication types, abstract-only, English-only, and minimum-citation filters. In command-line mode, every row needs TI/AB/TA/TX/FT. |
| Search scope cannot be selected | This is expected in command-line mode. Use TI/AB/TA/TX/FT in each row. Switch to standard OpenAlex search to use the scope selector. |
| Year-by-year mode does not retrieve 10,000 papers | It raises the per-year maximum; actual results depend on matching papers and active filters. |
| Unsure whether publication type filtering is applied | Select the chips in Advanced Filters and rerun the search. Multiple types can be selected. |
| The same author name appears multiple times | Different Author IDs or ORCIDs are treated as separate keys. Use diagnostics and merge only if they are the same person. |
| Homonymous authors are mixed | Name-only keys can mix different people. Use detail popups to check papers, affiliations, and Topics. |
| The same university or company appears under multiple names | Use institution disambiguation and inspect Institution ID, ROR, lineage, collaborators, and Topics. |
| Parent-institution grouping does not behave as expected | Lineage may be incomplete or parent names may be unavailable. Use a custom merge map if needed. |
| Disambiguation changes do not affect analysis | Confirm that the merged key is present in the current dataset and re-render the analysis card. |
| Previous disambiguation decisions are mixed in | Reset the current map and import only the intended JSON file. |
| The disambiguation workbench is slow | Reduce displayed rows and filter to same-name candidates or merged keys. |
| Technical keywords are too generic | Add search terms, generic words, and overly broad field terms to additional stopwords. |
| Too few technical keywords are shown | Lower minimum document frequency, increase document limit, or remove extra stopwords. |
| Must-read scores feel wrong | Adjust the weights. Increase citation weight for foundational papers; increase Topic growth and keyword novelty for recent trends. |
| The lineage graph is too dense | Reduce node count, increase minimum relation score, or filter by Topic and year. |
| A lineage paper is missing from the must-read list | Enable inclusion of technology-lineage nodes. |
| AI prompt is too long | Reduce the prompt paper count and, if needed, the candidate count. |
| AI output cannot be pasted back | Ensure the output is TSV with the required column names. Free-form prose cannot be imported. |
| AI Topic labels do not appear in Topic Intelligence | Import Topic-naming TSV and enable the AI-label option in Topic Intelligence. |
| AI annotations are missing after CSV import | Check whether the saved CSV contains the `ai_` columns. |
| Charts are not rendered | The dataset may be empty, no data may match the active tab, or CDN libraries may not be available. |
| Network view is unreadable | Increase minimum edge weight, reduce node limit, and filter by year or Topic. |

---

## Related tools

There are already several excellent open-source tools for bibliometrics on OpenAlex. This tool does not aim to replace them; it occupies a different niche: **everything runs in the browser, with no installation and no coding.**

| Tool | Form | License | Focus |
|---|---|---|---|
| **pyalex** | Python library | MIT | OpenAlex API wrapper (requires coding) |
| **openalexR** | R package | MIT | Bibliographic data collection in R (requires coding) |
| **litstudy** | Python library | Apache-2.0 | Literature review + network analysis (requires coding) |
| **bibliometrix / biblioshiny** | R package + GUI | MIT/GPL-3 | Rich scientometrics; GUI, but **needs an R environment** |
| **VOSviewer** | Desktop app (Java) / web | Freeware (web version MIT) | Network **visualization**; requires installation |
| **Local Citation Network** | Web app | GPL-3.0 | **Literature discovery** via citation networks |
| **OpenAlex Explorer** | Web app (Flask/Python server) | MIT | Data exploration/visualization; needs a running server |
| **This tool (OpenALEX Collector)** | **Single HTML (browser-only, no server/install, no-code)** | MIT | End-to-end: search → metrics → **author/institution disambiguation** → topic/funding/journal analysis → networks |

Key points:

- The library tools (pyalex / openalexR / litstudy) are powerful, but **assume you can write Python/R**.
- Among GUI tools, **bibliometrix needs an R environment and VOSviewer needs Java**, and some web apps (e.g. OpenAlex Explorer) require **running a server**.
- **This tool just needs you to open an HTML file** to search, analyze, disambiguate, visualize, and export CSV/PNG. Its **GUI-based author/institution disambiguation** and **fully client-side** design (data never leaves your browser) are uncommon among the tools above.

> Each of these is a fine OSS project in its own right. Use whatever fits the task, or combine them (e.g. VOSviewer for deep visualization, Local Citation Network for literature discovery, pyalex/openalexR for scripted automation).

---

## Acknowledgements & data provenance (OpenAlex / OurResearch)

This tool would not exist without the bibliographic data provided by **OpenAlex**. We are deeply grateful to OpenAlex and to its parent nonprofit, **OurResearch**.

- **OpenAlex** is a free, open catalog of scholarly data — works, authors, institutions, sources, Topics, and citation relationships (launched in 2022 as the de facto successor to Microsoft Academic Graph).
- **OurResearch**, which operates OpenAlex, is a US **501(c)(3) nonprofit** known for building open scholarly infrastructure such as Unpaywall.
- OpenAlex is committed to **[POSI (Principles of Open Scholarly Infrastructure)](https://openscholarlyinfrastructure.org/)**, keeping its data, code, and governance open. This tool is one open application built on top of that open, freely usable infrastructure.

### Data license (CC0)

OpenAlex data is released under **[CC0 1.0 (public-domain equivalent)](https://creativecommons.org/publicdomain/zero/1.0/)**. This is what lets users freely use and redistribute the data this tool retrieves, displays, and exports to CSV. Unlike proprietary, paywalled bibliographic databases, it can be used without a license agreement — which is the foundation of this project.

> Note: CC0 applies to the bibliographic data. The **copyright of each paper's full text / PDF remains with its rights holders**; handle full text within the terms and copyright that apply to you.

### Citation & credit

If you use this tool or OpenAlex data in research, please credit OpenAlex:

- Priem, J., Piwowar, H., & Orr, R. (2022). *OpenAlex: A fully-open index of scholarly works, authors, venues, institutions, and concepts.* arXiv:2205.01833.
- OpenAlex: <https://openalex.org> · OurResearch: <https://ourresearch.org>

OpenAlex is run by a nonprofit. If you rely on it, please consider [supporting it (donations / membership)](https://openalex.org/pricing).

---

## License

MIT License

©︎2026 しばやま

The tool itself (HTML/JS/docs) is MIT-licensed. The bibliographic data it retrieves comes from OpenAlex and is provided under CC0 1.0 (see "Acknowledgements & data provenance" above).
