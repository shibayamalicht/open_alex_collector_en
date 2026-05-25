# OpenALEX Collector v5.0

OpenALEX Collector v5.0 is a single-file HTML tool for collecting and analyzing scholarly paper metadata through the OpenAlex API. It supports paper search, topic analysis, funding analysis, citation analysis, collaboration analysis, coauthorship networks, and reference-network analysis directly in the browser. No server is required; open `OpenALEX_Collector_EN.html` in a modern browser.

> Every paper stands on someone's shoulders.

![Powered by OpenAlex API](https://img.shields.io/badge/Powered%20by-OpenAlex%20API-1a73e8)
![Version](https://img.shields.io/badge/version-v5.0-1a73e8)
![License](https://img.shields.io/badge/License-MIT-green)

---

## Table of contents

- [Overview](#overview)
- [Design principles](#design-principles)
- [Main features](#main-features)
- [Quick start](#quick-start)
- [Overall workflow](#overall-workflow)
- [Modules](#modules)
  - [1. Search form](#1-search-form)
  - [2. Data table](#2-data-table)
  - [3. Count charts](#3-count-charts)
  - [4. Citation rankings](#4-citation-rankings)
  - [5. Collaboration network, institution aggregation](#5-collaboration-network-institution-aggregation)
  - [6. Funding Intelligence](#6-funding-intelligence)
  - [7. Topic Intelligence](#7-topic-intelligence)
  - [8. Network graphs, collaboration and coauthorship](#8-network-graphs-collaboration-and-coauthorship)
  - [9. Citation network, references](#9-citation-network-references)
  - [10. Detail popups](#10-detail-popups)
  - [11. Export and import](#11-export-and-import)
- [Topic score definition](#topic-score-definition)
- [Network metric definitions](#network-metric-definitions)
- [CSV output columns](#csv-output-columns)
- [Search syntax](#search-syntax)
- [Use-case guide](#use-case-guide)
- [Technical specifications](#technical-specifications)
- [Notes](#notes)
- [Troubleshooting](#troubleshooting)
- [License](#license)

---

## Overview

**OpenALEX Collector v5.0** starts from OpenAlex paper metadata and helps users understand the structure of a research area from multiple angles.

It is designed to answer questions such as:

- Is this research area growing?
- Which Topics are increasing?
- Which papers, authors, institutions, and journals have influence?
- Which institutions collaborate with each other?
- Which authors coauthor with each other?
- Which funders and award IDs are linked to which Topics or institutions?
- Which references are foundational works, common foundations, or internal citation hubs?

In addition to the v4.0 search, citation, collaboration, funding, Topic, and network-analysis functions, v5.0 adds **command-line search expressions**. Search conditions can be defined as numbered command rows such as `#01`, `#02`, and then combined as a final logical expression such as `T=(#01 AND #02) OR #03`. OpenAlex is used to retrieve a broad candidate set, and Collector then performs internal Boolean, wildcard, and `nearN` / `adjN` proximity evaluation for titles and abstracts.

---

## Design principles

| Principle | Description |
|---|---|
| OpenAlex-centered | Paper population formation, Topics, references, institutions, authors, and funding information are centered on OpenAlex. |
| No Semantic Scholar dependency | Semantic Scholar is not used in v5.0. The tool is designed around OpenAlex to avoid large-scale retrieval constraints. |
| No API key field | v5.0 does not implement an OpenAlex API key input field. |
| Registration-free external APIs only | External enrichment is limited to the Crossref REST API and NIH RePORTER API. |
| Single-file HTML | No server is required. The tool runs in a browser. |
| English GUI | Interface labels and help text are provided in English. |
| CSV reuse | Previously downloaded CSV files can be reloaded to continue analysis without re-running the search. |

---

## Main features

| Category | Feature | Description |
|---|---|---|
| Paper collection | OpenAlex search | Search by keywords, institution, author, year range, publication type, search scope, open access, abstract availability, English-only filter, and more. |
| Paper collection | Command-line search expression | Supports numbered command rows such as `#01` and a combined expression such as `T=(#01 AND #02) OR #03`. Each row can use `TI` / `AB` / `TA` / `TX` / `FT`, `nearN` / `adjN`, Boolean logic, phrases, and wildcards. |
| Basic visualization | Count trends | Yearly paper counts, CAGR, trend label, author comparison, author ranking, and institution ranking. |
| Citation analysis | Citation rankings | Most-cited papers, authors, institutions, journals, count x average citation scatter, and h-index. |
| Collaboration analysis | Institution pair analysis | Collaboration pair ranking, ego network, collaboration breadth, pair time series, and collaboration matrix. |
| Funding analysis | Funding Intelligence | Funder ranking, funder x year, award ID ranking, funder x institution, funder x institution matrix, and external enrichment. |
| Topic analysis | Topic Intelligence | Topic ranking, Topic x year, emerging Topic candidates, Topic x institution, Topic x funder, and multiple matrix views. |
| Network graphs | Institution and author networks | vis-network-based institution and author networks with multiple node color and size metrics. |
| Citation network | Reference analysis | Top-N cited references, co-citation, bibliographic coupling, and internal citation network. |
| Detail display | Modal popups | Click bars, points, cells, nodes, or edges to view details. Paper lists include authors, institutions, and Topics. |
| Output | CSV / PNG | Supports paper CSV, chart CSV, matrix CSV, network node/edge CSV, and PNG export for Chart.js charts. |

---

## Quick start

```bash
# 1. Open the HTML file in a browser
open OpenALEX_Collector_EN.html

# 2. Enter command rows and the combined logical expression T
# Example command row:
# TA=((sulfide OR oxide OR polymer) near5 electrolyte)

# 3. Click "Run search"

# 4. Review the results and use "Render" buttons in each analysis card

# 5. Download CSV / PNG outputs as needed
```

Supported browsers include Chrome, Edge, Firefox, Safari, and other modern browsers.

---

## Overall workflow

```text
Specify search conditions
  ↓
Retrieve the paper set from OpenAlex
  ↓
Review the data table
  ↓
Visualize counts, citations, collaboration, Topics, funding, and references
  ↓
Click chart, matrix, and network elements to inspect details
  ↓
Export CSV / PNG
  ↓
Reload CSV later if needed and continue analysis
```

---

## Modules

### 1. Search form

The search form sets conditions for the OpenAlex API. It is also possible to leave the keyword field empty and search only with institution or author filters.

| Input | Description | Example |
|---|---|---|
| Command row | One condition per row. Each row must explicitly use `TI=` / `AB=` / `TA=` / `TX=` / `FT=`. | `#01  TI=(solid adj3 electrolyte)` |
| Combined logical expression T | References command rows as `#01`, `#02` and combines them with `AND` / `OR` / `NOT`. | `T=(#01 AND #02) OR #03` |
| Institution filter | Semicolon-separated institution names are OR-searched. | `Meijo University; MIT` |
| Author filter | Semicolon-separated author names are AND-searched. | `Akira Yoshino; John Goodenough` |
| Year range | Start year and end year. | `2020` to `2026` |
| Retrieval limit | Number of records per query or per year. | `200`, `10000` |
| Fetch by year | Retrieves records year by year to support result sets above 10,000 records. | ON/OFF |

Advanced filters include publication type, sort order, minimum citations, open-access-only, abstract-only, and English-language-only options.

In command-line search-expression mode, each command row explicitly defines the search scope. Therefore the Advanced Filter **Search scope** control is grayed out and is not used for internal evaluation. It is enabled only in Standard OpenAlex search mode. In Standard OpenAlex search mode, the command-line-only settings **Maximum width within proximity groups** and **OR split limit** are hidden.

### 2. Data table

The retrieved papers are displayed in a sortable table.

| Function | Description |
|---|---|
| Columns | Title, year, authors, institutions, journal, citation count, DOI, and Collector match information. |
| Sorting | Click column headers to switch ascending / descending order. |
| Pagination | Displays records in pages. |
| DOI links | Papers with DOI values link to `doi.org`. |
| CSV output | Exports paper data with v5.0 extended columns. |

### 3. Count charts

This basic chart card shows the quantitative trend of the research area.

| Tab | Description | Click behavior |
|---|---|---|
| Count trend, bar chart | Yearly paper counts. | Click a year to open the paper list for that year. |
| Count trend, line chart | Overall trend or author comparison. | Click a point to open the corresponding paper list. |
| Author ranking | Authors ranked by paper count. | Opens author details. |
| Institution ranking | Institutions ranked by paper count. | Opens institution details. |

### 4. Citation rankings

This card analyzes impact based on citations, not only paper volume.

| Tab | Description |
|---|---|
| Most-cited papers Top-N | Ranks individual papers by citation count. |
| Most-cited authors Top-N | Ranks authors by total citation count in the current collection. |
| Most-cited institutions Top-N | Ranks institutions by total citation count in the current collection. |
| Most-cited journals Top-N | Shows influential journals or venues. |
| Count x average citations | X = paper count, Y = average citations, bubble = total citations. |
| h-index Top-N | Computes h-index within the current collection. |

The h-index is calculated only from papers included in the current collection. It is not the OpenAlex-wide h-index for the author or institution.

### 5. Collaboration network, institution aggregation

Institutional collaboration is aggregated from papers. If a paper includes institutions A, B, and C, the pairs A x B, A x C, and B x C are counted.

| Tab | Description |
|---|---|
| Collaboration pair Top-N | Ranks institution pairs by coauthored paper count. |
| Ego network | Selects one central institution and shows its collaborators. |
| Collaboration breadth | Shows the number of unique collaborators for each institution. |
| Pair time series | Shows yearly coauthored-paper counts for top pairs. |
| Collaboration matrix | Heatmap for top institutions x top institutions. |

The collaboration matrix is an HTML table heatmap. It is exported as CSV rather than PNG.

### 6. Funding Intelligence

Funding Intelligence aggregates funding agencies, funders, award IDs, and their relationships with papers and institutions based primarily on OpenAlex funding information.

| Tab | Description | Main use |
|---|---|---|
| A. Funder ranking | Paper count, total citations, average citations, and number of award IDs by funder. | Identify major funders. |
| B. Funder x year | Yearly supported-paper counts for major funders. | Track funding trends over time. |
| C. Award ID ranking | Paper and citation counts by award ID / grant number. | Identify important grant programs or awards. |
| D. Funder x institution | Bar chart for funder-institution pairs. | See which funders are linked to which institutions. |
| E. Funder x institution matrix | Heatmap with funders as rows and institutions as columns. | Overview of funder-institution relationships. |
| F. External enrichment | Enrich funding data through Crossref / NIH RePORTER. | Add DOI-based and NIH award information. |

#### Funder x institution matrix

- Rows: funders
- Columns: institutions
- Cell value: number of matching papers
- Color: darker blue indicates more papers
- Cell click: opens the corresponding paper list
- CSV output: matrix-format CSV

#### External enrichment

Only registration-free APIs are used.

| API | Use |
|---|---|
| Crossref REST API | Enrich funder / award metadata by DOI. |
| NIH RePORTER API | Enrich NIH-style award numbers with project number, fiscal year, amount, PI, organization, and project title. |

Direct API integration for KAKEN / JST / AMED is not included as a standard feature in v5.0 because the standard tool is limited to registration-free APIs.

### 7. Topic Intelligence

Topic Intelligence uses OpenAlex `primary_topic` and `topics` metadata to analyze research themes and changes.

| Tab | Description | Main use |
|---|---|---|
| A. Topic ranking | Paper count, total citations, and average citations by Topic. | Identify major themes. |
| B. Topic x year | Yearly trend lines for top Topics. | Check growth or stagnation. |
| C. Emerging Topic candidates | Scores Topics by recent count, growth, and recent citation impact. | Explore growing themes. |
| D. Topic x institution | Bar chart for Topic-institution pairs. | Identify institutional strengths. |
| E. Topic x funder | Bar chart for Topic-funder pairs. | See where funding is directed. |
| F. Topic x year matrix | Heatmap with Topics as rows and years as columns. | Overview of thematic trends. |
| G. Topic x institution matrix | Heatmap with Topics as rows and institutions as columns. | Compare themes and players. |
| H. Topic x funder matrix | Heatmap with Topics as rows and funders as columns. | Compare themes and funding allocation. |

Clicking a matrix cell opens the corresponding paper list.

### 8. Network graphs, collaboration and coauthorship

vis-network is used to draw institution and author networks.

| Tab | Description |
|---|---|
| A. Institution network | Collaboration network among institutions. |
| B. Author network | Coauthorship network among authors. |

#### Filters

| Filter | Description |
|---|---|
| Number of nodes | Number of top nodes to show. |
| Minimum edge weight | Minimum number of collaborations or coauthored papers required for an edge. |
| Minimum papers | Minimum number of papers required for a node. |
| Minimum average citations | Minimum average citation count required for a node. |
| Start year / End year | Year range for network construction. |
| Topic filter | Keep only papers whose Topic name contains the specified text. |
| Funder filter | Keep only papers whose funder name contains the specified text. |

#### Node size and color

Node size and node color can be switched among:

- Betweenness centrality
- Degree
- Weighted degree
- Paper count
- Average citations

Click a node to open details for that author or institution. Click an edge to open the coauthored or collaborative papers for that pair.

### 9. Citation network, references

The citation-network card uses each paper's `referenced_works` to analyze reference structure.

| Tab | Description | Main use |
|---|---|---|
| A. Cited references Top-N | External papers most frequently cited by the collection. | Identify foundational or classic references. |
| B. Co-citation pairs Top-N | Pairs of references cited together in the same collection paper. | Identify schools, theoretical frameworks, or shared foundations. |
| C. Bibliographic coupling Top-N | Pairs of collection papers with many shared references. | Identify closely related papers or subthemes. |
| D. Internal citation network | Citation relationships among papers in the collection. | Understand knowledge flow inside the research area. |

In the internal citation network, an arrow starts from the citing paper and points to the cited paper. Clicking a node opens paper details and its cited / citing papers inside the collection.

### 10. Detail popups

Clicking chart bars, scatter points, matrix cells, network nodes, or network edges opens a detail popup.

Related paper lists include:

- Title
- Publication year
- Citation count
- DOI link
- Authors
- Institutions
- Topic

Popups also include copy-friendly summaries that can be pasted into reports or notes.

### 11. Export and import

| Function | Description |
|---|---|
| Paper CSV export | Exports retrieved paper data with v5.0 extended columns. |
| Chart CSV export | Exports not only the displayed Top-N but, in principle, all relevant records. |
| Matrix CSV export | Exports Topic x year, Topic x institution, Topic x funder, funder x institution, and other matrices. |
| Network CSV export | Exports node CSV and edge CSV for institution and author networks. |
| PNG export | Saves Chart.js charts as PNG. |
| CSV upload | Reloads a previously saved CSV and resumes analysis without re-searching. |

HTML-table matrices and vis-network graphs are primarily exported as CSV rather than PNG.

---

## Topic score definition

Emerging Topic candidates are scored using yearly data inside the current collection.

```text
Score =
  w_recent   x recent 3-year count
+ w_delta    x (recent 3-year count - previous 3-year count)
+ w_citation x recent 3-year average citations
```

Default weights are:

| Weight | Default | Meaning |
|---|---:|---|
| `w_recent` | 1.0 | Emphasis on volume in the recent 3-year period. |
| `w_delta` | 1.0 | Emphasis on increase from the previous 3-year period. |
| `w_citation` | 1.0 | Emphasis on citation impact of recent papers. |

The recent 3-year and previous 3-year windows are set automatically based on the maximum publication year in the current collection.

```text
Maximum publication year = Y
Recent 3 years           = Y-2, Y-1, Y
Previous 3 years         = Y-5, Y-4, Y-3
```

This score is an exploratory relative indicator. It should be used to rank candidate Topics within the same collection, not as an absolute cross-field metric.

---

## Network metric definitions

| Metric | Definition | Interpretation |
|---|---|---|
| Degree | Number of connected neighboring nodes. | Breadth of collaborators or coauthors. |
| Weighted degree | Sum of connected edge weights. | Total collaboration or coauthorship volume. |
| Betweenness centrality | Frequency of appearing on shortest paths. | Brokerage between clusters. |
| Paper count | Number of papers related to the node. | Output volume. |
| Average citations | Average citation count of papers related to the node. | Rough impact or quality proxy. |

Betweenness centrality is useful for finding authors or institutions that bridge multiple research communities.

---

## CSV output columns

The paper CSV outputs the following columns in v5.0:

```text
paper_id
title
abstract
year
authors
institutions
venue
citation_count
publication_date
doi
language
title_script
primary_topic
topics
referenced_works_count
referenced_works
collector_query
collector_candidate_query
collector_match
collector_match_field
collector_match_operator
collector_match_distance
collector_match_snippet
funder_names
funder_ids
award_ids
funding_sources
crossref_funder_names
crossref_award_ids
nih_project_nums
nih_award_amounts
nih_fiscal_years
nih_orgs
nih_pis
nih_project_titles
```

Main added columns are:

| Column | Description |
|---|---|
| `primary_topic` | Representative Topic in OpenAlex. |
| `topics` | Related Topics in OpenAlex. |
| `referenced_works` | OpenAlex Work IDs cited by the paper. |
| `collector_query` | Collector search expression used for the search. |
| `collector_candidate_query` | Candidate-retrieval query sent to OpenAlex. |
| `collector_match` | Result of Collector internal evaluation: `true` or `candidate_only`. |
| `collector_match_field` | Matched field, such as title or abstract. |
| `collector_match_operator` | Matched operator, such as `near5`, `adj3`, or `AND`. |
| `collector_match_distance` | Word distance for a near/adj match. |
| `collector_match_snippet` | Context around the matched location. |
| `funder_names` | Funder names from OpenAlex / Crossref / NIH RePORTER. |
| `award_ids` | Award IDs or grant numbers from OpenAlex / Crossref / NIH RePORTER. |
| `funding_sources` | Source of funding information, such as OpenAlex, Crossref, NIH RePORTER, or CSV. |
| `nih_*` | Project information enriched through NIH RePORTER. |

---

## Search syntax

### Search modes

| Mode | Description |
|---|---|
| Command-line search expression | Define numbered command rows such as `#01`, `#02`, and combine them as `T=(#01 AND #02) OR #03`. OpenAlex retrieves candidates, then Collector strictly evaluates `TI` / `AB` / `TA`. |
| Standard OpenAlex search | Sends the entered search terms directly to OpenAlex. |

### Basic command-line search expression

```text
#01  TI=(solid adj3 electrolyte)
#02  AB=((degradation AND capacity) near8 (mechanism OR fade*))
#03  TA=(review)
T=(#01 AND #02) AND NOT #03
```

Each command row is one search condition. Row numbers are assigned automatically in the UI. `#1` and `#01` refer to the same row. When multiple rows are used, enter the combined logical expression `T`. If only one row is entered and the T expression is blank, it is automatically treated as `T=(#01)`.

### Fields

| Field | Target | Strict internal evaluation |
|---|---|---:|
| `TI` | Title | Supported |
| `AB` | Abstract | Supported |
| `TA` | Title + abstract | Supported. near/adj does not cross the title-abstract boundary; it is evaluated within title or within abstract. |
| `TX` | OpenAlex general search target | Mainly candidate retrieval |
| `FT` | OpenAlex fulltext index | Mainly candidate retrieval |

In command-line search-expression mode, each row must explicitly contain one of `TI=` / `AB=` / `TA=` / `TX=` / `FT=`. A row without a field specification is a syntax error.

### Advanced Filter "Search scope"

In command-line search-expression mode, the Advanced Filter **Search scope** control is grayed out and cannot be clicked. Search scope is specified by each command row through `TI=` / `AB=` / `TA=` / `TX=` / `FT=`.

| State | Handling of Advanced Filter "Search scope" |
|---|---|
| Command-line search-expression mode | Disabled. Uses field specifications in each command row. |
| Standard OpenAlex search mode | Enabled. Applies to the entire entered search term. "Maximum width within proximity groups" and "OR split limit" are hidden. |

For OpenAlex candidate retrieval, if all fielded expressions use the same field, that scope is used. If `TI` / `AB` / `TA` are mixed, the candidate search is formed broadly around title and abstract, and Collector then strictly evaluates each field condition internally. If `TX` / `FT` are mixed, the candidate set is broadened further.

### Boolean operators

```text
A AND B
A OR B
NOT A
```

Uppercase operators are recommended. Lowercase operators are also interpreted.

### Proximity search

```text
A near5 B
A adj3 B
```

| Operator | Meaning |
|---|---|
| `nearN` | Within N words in any order. |
| `adjN` | The left term appears before the right term within N words. |

The left and right sides may contain terms, phrases, wildcards, or Boolean groups.

```text
TA=((sulfide OR oxide OR polymer) near5 electrolyte)
AB=((degradation AND capacity) near8 (mechanism OR fade*))
```

### Phrases and wildcards

```text
"solid electrolyte"
cathod*
wom?n
```

| Syntax | Meaning |
|---|---|
| `"..."` | Exact consecutive phrase. |
| `*` | Matches zero or more characters. |
| `?` | Matches one character. |

Leading wildcards such as `*electrolyte` are not allowed. Wildcards inside quoted phrases are not supported.

### OpenAlex candidate retrieval and internal filtering

Command-line search expressions are not passed to OpenAlex as-is. Collector first converts proximity conditions into broader Boolean candidate queries, retrieves candidate papers from OpenAlex, and then strictly evaluates `TI` / `AB` / `TA` internally.

The UI button **Syntax check / OpenAlex candidate query preview** shows command rows, the T expression, the expanded Collector expression, OpenAlex candidate queries, and the candidate-retrieval scope.

---

## Use-case guide

### Find growing subthemes

1. Search the target research area.
2. Open **Topic Intelligence -> Topic x year** to check major Topic trends.
3. Open **Topic Intelligence -> Emerging Topic candidates** to review recent growth and citation impact.
4. Adjust weights if needed to compare volume-oriented, growth-oriented, and citation-oriented rankings.
5. Click Topics of interest to inspect related papers, authors, and institutions.

### See which themes a funder supports

1. Search the target area.
2. Run **Funding Intelligence -> External enrichment** if needed.
3. Review **Topic Intelligence -> Topic x funder** or **Topic x funder matrix**.
4. Review **Funding Intelligence -> Funder x institution matrix** to see relationships between funders and major institutions.

### Find collaboration or coauthorship hubs

1. Search the target area.
2. Open **Network graphs -> Institution network** or **Author network**.
3. Set node color to **Betweenness centrality**.
4. Filter by Topic, funder, or year range if needed.
5. Click large or high-betweenness nodes to inspect related papers.

### Find foundational references or research streams

1. Search the target area.
2. Open **Citation network -> Cited references Top-N** to identify frequently cited external works.
3. Open **Co-citation pairs Top-N** to find pairs cited together.
4. Open **Bibliographic coupling Top-N** to find closely related papers within the collection.

### Compare strong themes across companies, universities, or research institutes

1. Search the target area.
2. Open **Topic Intelligence -> Topic x institution**.
3. Use **Topic x institution matrix** to overview the distribution of themes and institutions.
4. Use **Citation rankings -> Count x average citations** to compare output volume and citation impact.

---

## Technical specifications

| Item | Description |
|---|---|
| Format | Single HTML file. |
| Front end | HTML / CSS / JavaScript. |
| Chart rendering | Chart.js v4. |
| Network rendering | vis-network v9. |
| Heatmap | HTML table. |
| APIs | OpenAlex REST API, Crossref REST API, NIH RePORTER API. |
| External API registration | Only registration-free APIs are used in v5.0. |
| OpenAlex API key field | Not implemented. |
| CSV | UTF-8 BOM format, convenient for Excel. |
| Retry | Exponential backoff for 429 and related API errors. |
| CSV reload | Supports v5.0 extended CSV and older CSV files. |

### JavaScript modules

| Module | Role |
|---|---|
| `AppState` | Application-level state management. |
| `OpenAlexAPI` | OpenAlex API client, search, author/institution resolution, and metadata transformation. |
| `CollectorQuery` | Parser for Collector search syntax, OpenAlex candidate-query generation, TI/AB/TA internal filtering, and snippet generation. |
| `SearchController` | Search execution, progress display, and result rendering. |
| `TableRenderer` | Data table, sorting, and pagination. |
| `ChartRenderer` | Count, citation, collaboration, and reference-network charts. |
| `FundingRenderer` | Funding Intelligence aggregation, rendering, and external enrichment integration. |
| `TopicRenderer` | Topic Intelligence, Topic scoring, and matrix rendering. |
| `NetworkRenderer` | Institution and author networks, centrality calculation, and filters. |
| `FundingAPI` | Crossref / NIH RePORTER API client. |
| `Exporter` | CSV / PNG output. |
| `Importer` | CSV import. |
| `Modal` | Detail popups. |
| `UIController` | Tabs, toast notifications, and progress display. |

---

## Notes

- Topic analysis depends on OpenAlex `primary_topic` / `topics`. Papers without Topic metadata are excluded from Topic aggregation.
- Funding analysis is based primarily on OpenAlex funding metadata. A blank funding field does not necessarily mean that the paper had no funding.
- Crossref enrichment retrieves DOI-based metadata sequentially and is not suitable for very large batch retrieval.
- NIH RePORTER enrichment is useful when NIH-style award numbers are present.
- Direct API integration for KAKEN / JST / AMED is not included as a standard v5.0 feature.
- Emerging Topic scores are exploratory relative indicators and should not be used for absolute cross-field comparison.
- Network betweenness centrality is calculated within the current search collection and filter conditions.
- h-index is a simplified metric calculated within the current collection.
- OpenAlex reference coverage varies by publisher and field.
- Chart.js and vis-network are loaded from CDNs. In offline environments, graph rendering may not work.
- Because the browser calls external APIs directly, enrichment may fail due to CORS settings, rate limits, or API-side instability.

---

## Troubleshooting

| Symptom | Likely cause | Action |
|---|---|---|
| Too few search results | Filters are too restrictive: abstract-only, English-only, minimum citations, or Collector internal proximity filtering. | Loosen advanced filters. In command-line mode, increase `nearN` distance or maximum proximity-group width. |
| Collector syntax error | Missing closing parenthesis or quote, leading wildcard, or wildcard inside a quoted phrase. | Use **Syntax check / OpenAlex candidate query preview** to inspect the error. |
| No Topic-analysis data | OpenAlex has no Topic metadata for the papers, or the CSV has no Topic columns. | Run a new search or use a v5.0 CSV. |
| No Funding-analysis data | Papers do not contain funding metadata. | Try Crossref enrichment. If NIH-style numbers are present, run NIH enrichment. |
| Network graph is empty | Minimum edge weight, minimum papers, or Topic/funder filters are too strict. | Lower thresholds and clear filters. |
| Matrix is hard to read | Too many rows or columns are shown. | Reduce the number of Topics, funders, or institutions. |
| PNG export is unavailable | HTML-table matrices and vis-network graphs are not PNG-export targets. | Use CSV export. |
| External enrichment fails | CORS, rate limits, or unstable API responses. | Lower the retrieval limit and retry later. |

---

## License

MIT License

---

© 2026 Shibayama
