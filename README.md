# OpenALEX Collector v6.0 — English Edition

OpenALEX Collector v6.0 is a single-file, browser-based tool for collecting and analyzing scholarly paper metadata with the OpenAlex API. It supports paper retrieval, command-line style search expressions, author and institution disambiguation, citation analysis, funding intelligence, Topic intelligence, collaboration/co-authorship networks, and reference/citation-network analysis.

> Every paper stands on someone's shoulders.

![Powered by OpenAlex API](https://img.shields.io/badge/Powered%20by-OpenAlex%20API-1a73e8)
![Version](https://img.shields.io/badge/version-v6.0-1a73e8)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Table of contents

- [Overview](#overview)
- [What is included in v6.0](#what-is-included-in-v60)
- [Quick start](#quick-start)
- [Typical workflow](#typical-workflow)
- [Modules](#modules)
  - [1. Paper Search](#1-paper-search)
  - [2. Paper Data](#2-paper-data)
  - [3. Author / Institution Disambiguation](#3-author--institution-disambiguation)
  - [4. Charts](#4-charts)
  - [5. Citation Ranking](#5-citation-ranking)
  - [6. Collaboration Network by Institution](#6-collaboration-network-by-institution)
  - [7. Funding Intelligence](#7-funding-intelligence)
  - [8. Topic Intelligence](#8-topic-intelligence)
  - [9. Network Map: Collaboration / Co-authorship](#9-network-map-collaboration--co-authorship)
  - [10. Citation Network: References](#10-citation-network-references)
  - [11. Detail popups](#11-detail-popups)
  - [12. Export / import](#12-export--import)
- [Command-line query syntax](#command-line-query-syntax)
- [Disambiguation-map JSON format](#disambiguation-map-json-format)
- [Topic score definition](#topic-score-definition)
- [Network metrics](#network-metrics)
- [CSV output](#csv-output)
- [External APIs](#external-apis)
- [Troubleshooting](#troubleshooting)
- [Notes and limitations](#notes-and-limitations)
- [License](#license)

---

## Overview

OpenALEX Collector v6.0 helps analysts understand the structure of a research field from multiple perspectives:

- whether a field is growing or declining;
- which Topics are expanding;
- which papers, authors, institutions, journals, and references are influential;
- whether homonymous authors or institution-name variants are distorting rankings and networks;
- which institutions and authors collaborate with each other;
- which funders and award numbers are linked to which Topics, authors, and institutions;
- which references act as foundational works, shared intellectual bases, or internal citation hubs.

The tool runs as a local HTML file. No server setup is required. Open `OpenALEX_Collector_en.html` in a modern browser and start searching.

---

## What is included in v6.0

v6.0 keeps the core design of the Japanese v6.0 build and provides an English UI while preserving the same functionality.

### Core functions

| Area | Function | Description |
|---|---|---|
| Paper retrieval | OpenAlex search | Search by keywords, institution, author, publication year, paper type, open access, citation threshold, abstract availability, and language. |
| Search syntax | Command-line query mode | Use numbered command rows such as `#01`, `#02`, then combine them with `T=(#01 AND #02) OR #03`. Supports `TI`, `AB`, `TA`, `TX`, `FT`, Boolean logic, `nearN`, `adjN`, and wildcards. |
| Disambiguation | Author disambiguation | Uses OpenAlex Author ID and ORCID when available. Homonymous authors are separated by ID and can be manually merged in the workbench. |
| Disambiguation | Institution disambiguation | Uses OpenAlex Institution ID, ROR, and lineage when available. Supports exact-ID aggregation, parent-institution grouping, and user-defined merge maps. |
| Analytics | Charts | Yearly counts, CAGR, trend charts, author comparison, author ranking, and institution ranking. |
| Citations | Citation ranking | Highly cited papers, author citation ranking, institution citation ranking, journal ranking, quantity × quality scatter, and h-index ranking. |
| Collaboration | Institution collaboration | Institution-pair analysis, ego networks, breadth of collaboration, pair time series, and collaboration matrices. |
| Funding | Funding Intelligence | Funder rankings, funder × year analysis, award-number ranking, funder × institution, funder × author, and matrix exports. |
| Topics | Topic Intelligence | Topic ranking, Topic × year, emerging Topic candidates, Topic × institution, Topic × author, Topic × funder, and matrices. |
| Networks | Collaboration / co-authorship network map | Interactive vis-network graphs for institutions and authors. Nodes and edges can be clicked to show paper lists and metrics in popups. |
| References | Citation network | Top referenced works, co-citation, bibliographic coupling, and internal citation-network analysis. |
| Output | CSV / PNG / JSON | Export papers, charts, matrices, network nodes/edges, and identity maps. Import previously saved CSV files. |

### v6.0-specific improvements

- Author and institution identity resolution is integrated into the analytical pipeline.
- GUI-based merge and unmerge operations are stored as an identity map and immediately reapplied to analyses.
- The identity workbench clearly shows merged records, merged source keys, and merge targets.
- The identity workbench caches parsed author/institution entities, profile aggregations, diagnostics, and display-label maps; bulk merges are saved to localStorage in one batch, and heavier chart, Funding, Topic, and network redraws are deferred after the immediate workbench update.
- The identity workbench includes name-based sorting, same-name-first sorting, reverse-name sorting, and metric sorting by paper count, citation count, average citation count, or h-index, so same-name authors and institutions can be reviewed next to each other.
- Network-map labels hide raw OpenAlex IDs in the visible names while preserving IDs internally and in CSV exports.
- Network-map nodes and edges open detail popups, including related papers, metrics, and collaboration/co-authorship context.
- Topic Intelligence includes an Author × Topic matrix.
- Funding Intelligence includes Funder × Author analysis and Funder × Author matrix output.
- The Network Map card icon is `＊`.
- The version label remains `v6.0`.

---

## Quick start

```bash
# 1. Open the HTML file in your browser.
open OpenALEX_Collector_en.html

# 2. Enter command lines and a T expression.
# Example:
# #01  TA=((sulfide OR oxide OR polymer) near5 electrolyte)
# T=(#01)

# 3. Optionally set institution filters, author filters, year range, and advanced filters.

# 4. Click "Run search".

# 5. Review results in Paper Data.

# 6. Open Author / Institution Disambiguation and, when needed, merge or unmerge keys.

# 7. Render charts, funding analysis, Topic analysis, network maps, and citation networks.

# 8. Export CSV, PNG, or identity-map JSON as needed.
```

Recommended browsers: Chrome, Edge, Firefox, or Safari.

---

## Typical workflow

```text
Define search conditions
  -> Retrieve papers from OpenAlex
  -> Keep author IDs, ORCIDs, institution IDs, RORs, and lineage metadata
  -> Apply the disambiguation map
  -> Inspect authors and institutions in the disambiguation workbench
  -> Merge or unmerge keys where needed
  -> Review the paper table
  -> Render charts, citation rankings, collaboration analysis, Topic analysis, funding analysis, and networks
  -> Export CSV / PNG / JSON
  -> Re-import saved CSV and the identity map for continued analysis
```

---

## Modules

### 1. Paper Search

The search form defines the OpenAlex query and the post-retrieval filtering rules.

| Input | Description | Example |
|---|---|---|
| Command row | One condition per row. Each row must specify `TI=`, `AB=`, `TA=`, `TX=`, or `FT=`. | `TI=(solid adj3 electrolyte)` |
| T expression | Combines command rows by `#01`, `#02`, etc. | `T=(#01 AND #02) OR #03` |
| Institution filter | Semicolon-separated institution names or OpenAlex Institution IDs. Values are ORed. | `Meijo University; I136199984` |
| Author filter | Semicolon-separated author names or OpenAlex Author IDs. Values are ANDed. | `Akira Yoshino; A123456789` |
| Year range | Publication-year start and end. | `2020` to `2026` |
| Maximum records | Maximum number of records per query or per year. | `200`, `10000` |
| Fetch by year | Splits retrieval by year, useful for large fields. | On / Off |

In command-line query mode, the Advanced Filters "Search scope" control is disabled because each command row defines its field explicitly. In standard OpenAlex search mode, the search terms are sent directly to OpenAlex and the Advanced Filters search scope is active.

### 2. Paper Data

The Paper Data table shows the retrieved papers.

| Function | Description |
|---|---|
| Columns | Title, year, authors, institutions, venue, citations, search match, and DOI. |
| Sorting | Click a column header to sort ascending or descending. |
| Pagination | Results are shown in pages of 50 records. |
| DOI links | DOI values link to `doi.org`. |
| CSV export | Exports the paper table with v6.0 identity and reference columns. |

Displayed author and institution names use the current disambiguation map. Original names and IDs are retained in CSV columns.

### 3. Author / Institution Disambiguation

The disambiguation module is a core feature of v6.0. It uses structured entities from OpenAlex and applies a user-editable merge map before analytical aggregation.

| Target | Primary key | Auxiliary key | Purpose |
|---|---|---|---|
| Author | OpenAlex Author ID | ORCID, normalized name | Separate homonymous authors and allow controlled manual merging. |
| Institution | OpenAlex Institution ID | ROR, lineage, normalized name | Control name variants, same-name institutions, parent-child grouping, and organizational groups. |

The workbench supports:

- switching between author and institution targets;
- filters for all keys, same-name/multiple-key candidates, name-only records, and merged records;
- text filtering by name, ID, ROR, Topic, co-author, or collaborator, with debounced redraws while typing;
- sorting by `Name A–Z (same names adjacent)`, `Same-name candidates first → name`, `Name Z–A`, paper count, citation count, average citation count, or h-index;
- detail popups showing papers, citations, h-index, source keys, display names, partners, Topics, and top papers;
- same-name candidate comparison;
- merging one key into another;
- merging checked keys, where the first checked key in display order becomes the merge target;
- undoing a merge;
- exporting and importing the identity map as JSON;
- clearing the identity map.

Merged records are explicitly marked in the workbench. Merged source keys and merge targets are shown so that users can verify what was merged.

The detail popup buttons pass author and institution keys safely through HTML click handlers, so keys containing punctuation or URL-derived characters can still open the merge/detail view reliably. Manual key-entry merging is not exposed; merge and split actions are performed from the list, detail popups, and checked-row merge workflow.

Internally, the workbench avoids repeated JSON parsing and repeated profile aggregation by caching parsed entities, canonical profiles, label maps, and diagnostics. When the merge map or institution aggregation mode changes, the relevant caches are invalidated and rebuilt. Filter input is debounced while typing. Bulk merge operations write the merge map once after all selected rules have been created. After merge or split operations, the table, diagnostics, workbench, and rule list update first, while heavier Chart, Funding, Topic, and Network redraws are deferred to browser idle time where available. These changes reduce overhead on large collections without changing the v6.0 feature set.

### 4. Charts

The Charts card provides basic trend and ranking views.

| Tab | Description |
|---|---|
| Yearly count | Annual publication counts. |
| Trend line | Total or author-specific time series. |
| Author ranking | Top authors by paper count. |
| Institution ranking | Top institutions by paper count. |

Clicking bars or line points opens a paper-list popup.

### 5. Citation Ranking

Citation Ranking analyzes influence within the retrieved collection.

| Tab | Description |
|---|---|
| Highly cited papers | Papers ranked by citation count. |
| Author citations | Author-level citation aggregation. |
| Institution citations | Institution-level citation aggregation. |
| Venue ranking | Venue-level aggregation. |
| Quantity × quality | Paper count versus average citations. |
| h-index ranking | h-index by author or institution within the collection. |

All author and institution aggregations use the current disambiguation map.

### 6. Collaboration Network by Institution

This module analyzes collaboration among institutions.

| Tab | Description |
|---|---|
| Institution pairs | Collaboration pairs ranked by co-authored paper count. |
| Ego network | Collaboration partners for a selected institution. |
| Collaboration breadth | Number and strength of collaboration partners. |
| Pair time series | Annual trend for a selected institution pair. |
| Collaboration matrix | Heatmap-style institution × institution matrix. |

### 7. Funding Intelligence

Funding Intelligence aggregates funding metadata from OpenAlex and optional enrichment sources.

| Tab | Description |
|---|---|
| Funder ranking | Top funders by paper count. |
| Funder × year | Time series by funder. |
| Award-number ranking | Frequent award IDs and grants. |
| Funder × institution | Links between funders and institutions. |
| Funder × institution matrix | Matrix export and heatmap-style view. |
| Funder × author | Links between funders and authors. |
| Funder × author matrix | Matrix export and heatmap-style view. |

Optional enrichment uses public APIs that do not require registration. External lookups may be limited by API availability and CORS behavior.

### 8. Topic Intelligence

Topic Intelligence uses OpenAlex Topic metadata to analyze research structure.

| Tab | Description |
|---|---|
| Topic ranking | Top Topics by paper count. |
| Topic × year | Annual Topic trends. |
| Emerging Topics | Simple-score ranking for recent growth and citation impact. |
| Topic × institution | Links between Topics and institutions. |
| Topic × author | Links between Topics and authors. |
| Topic × funder | Links between Topics and funders. |
| Topic × year matrix | Matrix export and heatmap-style view. |
| Topic × institution matrix | Matrix export and heatmap-style view. |
| Author × Topic matrix | Matrix export and heatmap-style view. |
| Funder × Topic matrix | Matrix export and heatmap-style view. |

All institution and author aggregations use the current disambiguation map.

### 9. Network Map: Collaboration / Co-authorship

The Network Map card draws interactive networks using vis-network.

| Network | Nodes | Edges |
|---|---|---|
| Institution network | Institutions | Co-authored papers between institutions. |
| Author network | Authors | Co-authored papers between authors. |

Controls include:

- maximum number of nodes;
- minimum edge weight;
- minimum paper count;
- minimum average citations;
- year range;
- Topic filter;
- funder filter;
- node-size metric;
- node-color metric.

Node labels are display names only. Raw IDs such as OpenAlex Author IDs or Institution IDs are not appended to visible names, but they are preserved internally and in CSV exports. Clicking a node or edge opens a detail popup with metrics and related papers.

### 10. Citation Network: References

The Citation Network card uses `referenced_works` from OpenAlex.

| Tab | Description |
|---|---|
| Top referenced works | External works most cited by the retrieved collection. |
| Top co-cited pairs | Pairs of references cited together by the same collection papers. |
| Top bibliographic coupling pairs | Pairs of collection papers sharing many references. |
| Internal citation network | Citation links among papers inside the retrieved collection. |

Reference coverage depends on publisher and field. Some papers may have no reference data.

### 11. Detail popups

The tool uses modal popups across charts, rankings, matrices, and networks. Popups are designed for copy-and-paste inspection and usually include:

- entity name and type;
- paper count;
- total and average citations;
- h-index where applicable;
- related Topics, authors, institutions, funders, or references;
- paper lists;
- DOI and OpenAlex links where available;
- copy-ready summaries.

### 12. Export / import

| Output | Description |
|---|---|
| Paper CSV | Full paper table with identity, funding, Topic, and reference columns. |
| Chart CSV | Aggregated values used in each chart. |
| Matrix CSV | Topic, funding, collaboration, and citation matrices. |
| Network CSV | Node and edge tables for institution and author networks. |
| PNG | Chart image export. |
| Identity-map JSON | Author and institution merge rules. |

Previously exported CSV files can be reloaded through the upload card. If an older CSV lacks v6.0 columns, the tool fills missing values where possible.

---

## Command-line query syntax

Command-line query mode uses two layers: command rows and a final T expression.

```text
#01  TI=(solid adj3 electrolyte)
#02  AB=((degradation AND capacity) near8 (mechanism OR fade*))
#03  TA=(review)
T=(#01 AND #02) AND NOT #03
```

### Fields

| Field | Meaning |
|---|---|
| `TI` | Title |
| `AB` | Abstract |
| `TA` | Title + abstract |
| `TX` | OpenAlex general search target; mainly used for candidate retrieval. |
| `FT` | OpenAlex full-text index; mainly used for candidate retrieval. |

### Operators

For ordinary `TI` / `AB` / `TA` keyword, Boolean, or wildcard searches, the tool accepts the OpenAlex candidate set directly. Internal distance filtering is applied only when `nearN` or `adjN` is used in `TI`, `AB`, or `TA`; `TX` and `FT` remain candidate-retrieval fields because the browser does not hold full-text content.


| Operator | Meaning |
|---|---|
| `AND` | Both conditions must match. |
| `OR` | Either condition may match. |
| `NOT` | Exclude matching records. |
| `nearN` | Unordered proximity within N words. |
| `adjN` | Ordered proximity: left term before right term within N words. |
| `*` | Wildcard for zero or more characters. |
| `?` | Wildcard for one character. |


---

## Disambiguation-map JSON format

The identity map stores user-defined merge rules. A simplified example is shown below.

```json
{
  "author": {
    "https://openalex.org/A111": "https://openalex.org/A999",
    "name:akira-yoshino": "https://openalex.org/A999"
  },
  "institution": {
    "https://openalex.org/I111": "https://openalex.org/I999",
    "ror:https://ror.org/12345": "https://openalex.org/I999"
  }
}
```

The left side is a source key. The right side is the canonical target key. The GUI is the recommended way to create and edit this map.

---

## Topic score definition

The Emerging Topics tab uses a simple configurable score:

```text
Score = w_recent × recent_3_year_count
      + w_delta × (recent_3_year_count - previous_3_year_count)
      + w_citation × recent_3_year_average_citations
```

The weights can be changed in the UI. The goal is exploratory ranking, not statistical forecasting.

---

## Network metrics

Network Map nodes and edges include the following metrics.

| Metric | Description |
|---|---|
| Paper count | Number of papers associated with the node or edge. |
| Citation count | Total citations of related papers. |
| Average citations | Mean citation count of related papers. |
| Degree | Number of connected partners. |
| Weighted degree | Sum of edge weights connected to the node. |
| Betweenness centrality | Approximate brokerage position in the network. |
| Edge weight | Number of co-authored or jointly affiliated papers connecting two nodes. |

---

## CSV output

The paper CSV includes standard bibliographic columns plus v6.0 extensions.

Common columns include:

- `paper_id`
- `title`
- `year`
- `authors`
- `institutions`
- `venue`
- `doi`
- `citation_count`
- `abstract`
- `topics`
- `primary_topic`
- `funders`
- `awards`
- `referenced_works`

Identity-related columns include:

- `author_ids`
- `author_orcids`
- `institution_ids`
- `institution_rors`
- `author_identity_keys`
- `author_identity_labels`
- `institution_identity_keys`
- `institution_identity_labels`
- `author_entities_json`
- `institution_entities_json`

Network node/edge CSV exports include both display names and internal keys so that labels remain readable while IDs remain available for downstream processing.

---

## External APIs

The tool is centered on OpenAlex. Optional enrichment may also use public endpoints.

| API | Use |
|---|---|
| OpenAlex API | Paper retrieval, metadata, authors, institutions, Topics, funders, references. |
| Crossref REST API | Optional DOI-based funding enrichment. |
| NIH RePORTER API | Optional grant/award enrichment where relevant. |

No OpenAlex API key field is implemented in v6.0. Semantic Scholar is not used.

---

## Troubleshooting

| Symptom | Possible cause and remedy |
|---|---|
| No papers are retrieved | Check the command syntax, year range, institution/author filters, and whether the query is too restrictive. Use the syntax preview before running. |
| Too many papers are retrieved | Add field-specific conditions, year constraints, institution filters, or minimum citation filters. |
| Search scope is disabled | This is expected in command-line query mode. Use `TI=`, `AB=`, `TA=`, `TX=`, or `FT=` inside command rows. |
| Name-only authors or institutions appear | OpenAlex did not provide stable IDs for those entities. Review them in the disambiguation workbench. |
| Parent institution grouping creates unexpected labels | The tool only aggregates to parents with retrievable names. If needed, use exact-ID aggregation or the GUI merge map. |
| Funding enrichment is incomplete | External APIs may not have data for the DOI/award or may be blocked by rate limits/CORS behavior. |
| Reference analysis is sparse | OpenAlex reference coverage varies by publisher, year, and field. |
| Browser becomes slow | Reduce maximum records, Top-N values, network node counts, or matrix dimensions. In the disambiguation workbench, use the 50-row limit, filter to same-name or merged records, and sort by name so same-name candidates stay adjacent with less rendering overhead. |
| CSV import lacks some columns | Older CSV files may not contain all v6.0 columns. The tool fills missing values where possible. |

---

## Notes and limitations

- OpenAlex metadata quality varies by publisher, field, and publication year.
- Abstracts and references may be unavailable for some records.
- Disambiguation suggestions are aids for analysis; final merge decisions should be made by the user.
- The tool runs entirely in the browser, so very large collections may be limited by browser memory and rendering performance.
- For very large identity workbenches, use the 50-row limit, filter to same-name or merged records, and sort by name to review candidates with less rendering overhead.
- Citation counts are OpenAlex citation counts and may differ from other databases.
- Network metrics are calculated within the retrieved collection, not the entire scholarly literature.

---

## License

MIT License.

©︎2026 Shibayama
