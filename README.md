# OpenALEX Collector v4.0

**OpenALEX Collector v4.0** is a single-file, browser-based tool for collecting and analyzing scholarly papers with the OpenAlex API. It focuses on paper discovery, topic intelligence, funding intelligence, citation analysis, collaboration networks, co-authorship networks, and reference-based network analysis.

> Every paper stands on someone’s shoulders.

![Powered by OpenAlex API](https://img.shields.io/badge/Powered%20by-OpenAlex%20API-1a73e8)
![Version](https://img.shields.io/badge/version-v4.0-1a73e8)
![License](https://img.shields.io/badge/license-MIT-green)

Author: **Shibayama**

---

## Table of contents

- [Overview](#overview)
- [v4.0 design principles](#v40-design-principles)
- [Quick start](#quick-start)
- [Main workflow](#main-workflow)
- [Feature modules](#feature-modules)
  - [1. Search form](#1-search-form)
  - [2. Paper data table](#2-paper-data-table)
  - [3. Basic charts](#3-basic-charts)
  - [4. Citation rankings](#4-citation-rankings)
  - [5. Institutional collaboration analysis](#5-institutional-collaboration-analysis)
  - [6. Funding Intelligence](#6-funding-intelligence)
  - [7. Topic Intelligence](#7-topic-intelligence)
  - [8. Network graphs](#8-network-graphs)
  - [9. Reference / citation network](#9-reference--citation-network)
  - [10. Detail popups](#10-detail-popups)
  - [11. Export functions](#11-export-functions)
- [Emerging Topic score](#emerging-topic-score)
- [Network metrics](#network-metrics)
- [CSV output](#csv-output)
- [Use cases](#use-cases)
- [Technical notes](#technical-notes)
- [Limitations and cautions](#limitations-and-cautions)
- [License](#license)

---

## Overview

OpenALEX Collector v4.0 is designed for researchers, analysts, IP strategists, R&D planners, and science-policy professionals who need to explore a research field from multiple perspectives.

It can help answer questions such as:

- How has this research field grown over time?
- Which authors, institutions, journals, and papers are most influential?
- Which topics are emerging?
- Which funders are associated with which topics or institutions?
- Which institutions collaborate with each other?
- Which authors form co-authorship networks?
- Which references are foundational to the collected paper set?
- Which papers are close to each other through shared references?

The tool runs as a **single HTML file**. No server installation is required. Open `OpenALEX_Collector_EN.html` in a modern browser and start searching.

---

## v4.0 design principles

v4.0 follows these principles:

1. **OpenAlex-first**  
   OpenAlex is the primary source for paper metadata, authors, institutions, topics, references, funders, and awards.

2. **No Semantic Scholar dependency**  
   Semantic Scholar is not used in this version. The design avoids dependence on APIs that are not suitable for browser-based large-scale retrieval.

3. **No OpenAlex API key field in this build**  
   This version does not implement an OpenAlex API-key input field.

4. **Only registration-free external funding APIs**  
   External enrichment is limited to Crossref and NIH RePORTER, which can be used without user registration in this browser-based workflow.

5. **Browser-only and portable**  
   The tool is distributed as a standalone HTML file. It can be shared as a file and used without a backend.

6. **Interactive analytics**  
   Bars, points, matrix cells, and network nodes can be clicked to open detailed popups with copy-ready summaries.

---

## Quick start

```bash
# 1. Open the HTML file in a browser
open OpenALEX_Collector_EN.html

# 2. Enter a search query, for example:
"lithium-ion battery"

# 3. Set year range and retrieval limit

# 4. Click Run search

# 5. Explore charts, topic analysis, funding analysis, and networks

# 6. Export paper data, chart data, matrices, or network tables as CSV
```

Recommended browsers:

- Google Chrome
- Microsoft Edge
- Firefox
- Safari

For larger datasets, Chrome or Edge is recommended.

---

## Main workflow

```text
Search form
  ↓
Paper collection from OpenAlex
  ↓
Paper data table
  ↓
Charts / rankings / funding / topics / networks
  ↓
Detail popups
  ↓
CSV or PNG export
```

The tool supports both keyword-based search and filter-based search. You may search by keyword, institution, author, publication year, publication type, citation count, open-access status, abstract availability, and language.

---

## Feature modules

### 1. Search form

The search form builds OpenAlex queries using the following inputs.

| Input | Description |
|---|---|
| Search keywords | One query per line. Multiple lines are OR-searched and merged with duplicate removal. |
| Institution filter | Institution names. Multiple institutions separated by semicolons are OR-searched. |
| Author filter | Author names. Multiple authors separated by semicolons are AND-searched. |
| Year range | Start year and end year. |
| Retrieval limit | Maximum number of papers to retrieve per query. |
| Year-by-year retrieval | Runs searches by publication year to handle larger result sets. |
| Publication types | Article, review, book chapter, dataset, preprint, dissertation, report, etc. |
| Search scope | All indexed text, title + abstract, title only, abstract only, or full text only. |
| Sort order | Relevance, citation count, newest first, or oldest first. |
| Minimum citations | Filters papers by citation threshold. |
| Open access only | Retrieves only open-access papers. |
| Abstract required | Uses OpenAlex `has_abstract:true`. Enabled by default. |
| English papers only | Uses `language:en` and excludes titles containing CJK or other non-Latin scripts. Enabled by default. |

Institution and author names are resolved through OpenAlex. If an author name is ambiguous, the tool selects the most-cited matching author and displays the resolved name.

---

### 2. Paper data table

The paper table displays the retrieved records.

Columns:

- Title
- Year
- Authors
- Institutions
- Journal / venue
- Citation count
- DOI

The table supports sorting and pagination. The paper data can be exported as CSV.

---

### 3. Basic charts

The basic chart card provides four views.

| Tab | Description |
|---|---|
| Publication trend (bar) | Annual paper counts as a bar chart. |
| Publication trend (line) | Annual trend as a line chart. It also supports author comparison. |
| Author ranking | Top authors by paper count. |
| Institution ranking | Top institutions by paper count. |

Clicking a bar or point opens a detail popup with relevant papers.

---

### 4. Citation rankings

This module ranks papers and entities by citation impact.

| Tab | Description |
|---|---|
| Most-cited papers Top-N | Individual papers ranked by citation count. |
| Most-cited authors Top-N | Authors ranked by total citations in the collection. |
| Most-cited institutions Top-N | Institutions ranked by total citations in the collection. |
| Most-cited journals Top-N | Journals / venues ranked by total citations. |
| Volume × average citations | Bubble chart: X = paper count, Y = average citations, bubble size = total citations. |
| h-index Top-N | h-index calculated within the collected dataset. |

The h-index shown here is calculated only from papers in the current collection. It is not the full OpenAlex h-index for an author or institution.

---

### 5. Institutional collaboration analysis

This module analyzes institutional co-authorship relationships.

| Tab | Description |
|---|---|
| Collaboration pairs Top-N | Top institution pairs by number of co-authored papers. |
| Ego network | Select one institution and view its main collaboration partners. |
| Collaboration breadth | Institutions ranked by number of unique collaboration partners. |
| Pair time series | Annual trends for top institution pairs. |
| Collaboration matrix | Heatmap of institution × institution collaboration counts. |

For papers with multiple institutions, every pair of institutions is counted as a collaboration pair. For example, institutions A, B, and C generate A×B, A×C, and B×C.

---

### 6. Funding Intelligence

Funding Intelligence aggregates funding-related metadata from OpenAlex and optional external enrichment.

Main data sources:

- OpenAlex funders / awards / grants-related fields
- Crossref DOI metadata, optional enrichment
- NIH RePORTER grant metadata, optional enrichment

Tabs:

| Tab | Description |
|---|---|
| Funder ranking | Funders ranked by paper count, total citations, average citations, or award count. |
| Funder × year | Annual paper counts by funder. |
| Award / grant ranking | Grant or award IDs ranked by output papers and citations. |
| Funder × institution | Funder–institution pairs ranked by paper count. |
| Funder × institution matrix | Heatmap of funders × institutions. |
| External enrichment | Manual enrichment using Crossref and NIH RePORTER. |

The Funder × Institution matrix is useful for identifying which funding bodies appear in papers from which research institutions. Clicking a matrix cell opens the corresponding paper list.

External enrichment is intentionally manual and bounded by a user-defined limit. This avoids turning the browser tool into a large-scale API crawler.

---

### 7. Topic Intelligence

Topic Intelligence uses OpenAlex `primary_topic` and `topics` fields to analyze research themes.

Tabs:

| Tab | Description |
|---|---|
| Topic ranking | Topics ranked by paper count, total citations, or average citations. |
| Topic × year | Annual trend lines for high-volume topics. |
| Emerging Topic candidates | Topics ranked by a configurable emerging-topic score. |
| Topic × institution | Topic–institution pairs. |
| Topic × funder | Topic–funder pairs. |
| Topic × year matrix | Heatmap of topics × years. |
| Topic × institution matrix | Heatmap of topics × institutions. |
| Topic × funder matrix | Heatmap of topics × funders. |

Matrices are useful where bar charts are not enough. For example, Topic × Funder reveals which funders are associated with which research themes.

---

### 8. Network graphs

v4.0 includes interactive network graphs powered by vis-network.

Networks:

- Institution collaboration network
- Author co-authorship network

Available filters:

- Number of nodes
- Minimum edge weight
- Minimum paper count
- Minimum average citations
- Start year
- End year
- Topic filter
- Funder filter

Node size can be mapped to:

- Paper count
- Weighted degree
- Degree
- Average citations
- Betweenness centrality

Node color can be mapped to:

- Betweenness centrality
- Degree
- Weighted degree
- Paper count
- Average citations

Clicking a node opens entity details. Clicking an edge opens the shared paper list for the corresponding pair.

---

### 9. Reference / citation network

This module uses each paper’s `referenced_works` field.

| Tab | Description |
|---|---|
| Referenced works Top-N | External works most frequently cited by the collected papers. Useful for finding foundational literature. |
| Co-citation pairs Top-N | Pairs of references that are cited together by the same papers. Useful for identifying intellectual lineages. |
| Bibliographic coupling Top-N | Pairs of collected papers that share many references. Useful for finding topical proximity. |
| Internal citation network | Directed network of citation links among papers in the current collection. |

Reference coverage in OpenAlex can vary by publisher and field. Some papers may have empty `referenced_works`.

---

### 10. Detail popups

Most charts, matrices, and networks are interactive.

Clicking a bar, point, node, edge, or matrix cell opens a detail popup containing:

- Paper title
- Publication year
- Citation count
- DOI
- Authors
- Institutions
- Topic
- Related papers
- Copy-ready summary

The popup text is selectable, and the copy button can be used to copy summaries into reports or notes.

---

### 11. Export functions

The tool supports:

- Paper CSV export
- Chart data CSV export
- Matrix CSV export
- Network node CSV export
- Network edge CSV export
- PNG export for Chart.js charts

HTML heatmaps and vis-network graphs are not exported as PNG by default. Use CSV exports for those modules.

---

## Emerging Topic score

The Emerging Topic module uses a simple, adjustable score.

Default formula:

```text
Score =
  1 × recent 3-year paper count
+ 1 × (recent 3-year paper count − previous 3-year paper count)
+ 1 × recent 3-year average citations
```

The three weights can be adjusted in the UI:

- Weight: recent count
- Weight: increase
- Weight: average citations

Interpretation:

| Component | Meaning |
|---|---|
| Recent 3-year paper count | Current activity level. |
| Increase from previous 3 years | Growth momentum. |
| Recent 3-year average citations | Early impact of recent papers. |

This is a lightweight heuristic, not a normalized bibliometric indicator. For rigorous comparisons across fields, consider normalizing by field, year, and document type.

---

## Network metrics

| Metric | Meaning |
|---|---|
| Degree | Number of connected nodes. |
| Weighted degree | Sum of edge weights connected to the node. |
| Betweenness centrality | How often a node lies on shortest paths between other nodes. High values indicate broker or bridge positions. |
| Paper count | Number of papers associated with the node. |
| Average citations | Average citation count of papers associated with the node. |

For institution networks, an edge means institutional collaboration. For author networks, an edge means co-authorship.

---

## CSV output

Main paper CSV columns include:

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

CSV files are exported with UTF-8 BOM for better compatibility with spreadsheet software.

---

## Use cases

### Research landscape analysis

Use publication trends, Topic ranking, Topic × Year, and Emerging Topic candidates to understand how a field is changing.

### Funding strategy analysis

Use Funder ranking, Funder × Year, Funder × Institution, Topic × Funder, and Funder × Institution matrix to see which funders are associated with specific themes and institutions.

### Collaboration analysis

Use institutional collaboration rankings, collaboration matrices, and network graphs to identify central institutions, collaboration clusters, and bridge players.

### Author network analysis

Use the author network to identify co-authorship communities and highly connected researchers.

### Foundational literature discovery

Use Referenced works Top-N and Co-citation pairs to discover foundational papers, classics, and intellectual lineages.

### Topic proximity analysis

Use Bibliographic coupling to identify collected papers that are close because they share many references.

---

## Technical notes

- The tool is implemented as a standalone HTML file.
- Charts are rendered with Chart.js.
- Network graphs are rendered with vis-network.
- Data retrieval is performed in the browser with `fetch`.
- Paper metadata is collected primarily from OpenAlex.
- Crossref and NIH RePORTER are optional external enrichment sources.
- No backend server is required.

---

## Limitations and cautions

1. **OpenAlex coverage varies**  
   Reference, funding, topic, and institution metadata coverage may vary by publisher, field, and year.

2. **Funding metadata is incomplete**  
   Absence of funding metadata does not mean that a paper had no funding.

3. **Full counting is used in many modules**  
   If one paper has multiple authors, institutions, funders, or topics, it may be counted for each entity.

4. **Browser performance has limits**  
   Very large datasets can slow down rendering, especially matrices and network graphs.

5. **External enrichment is not bulk harvesting**  
   Crossref and NIH RePORTER enrichment should be used with moderate limits.

6. **Network metrics are collection-specific**  
   Degree, weighted degree, and betweenness centrality are calculated only within the current dataset.

---

## License

MIT License.

