# SERP Keyword Clustering Tool

This script fetches live SERP data for a list of keywords via the DataForSEO API, clusters them by search intent using shared URL overlap, and produces visual reports and exportable CSVs.

---

## What It Does

Queries that return the same URL in different SERPS cover the same intent, so they belong in the same cluster.

1. **Fetches live SERP data** for every keyword in your list using the DataForSEO Google Organic API
2. **Clusters keywords** by comparing which URLs appear across their search results — keywords that surface the same pages are grouped together
3. **Visualises** the clusters, SERP feature distribution, domain rankings, and domain overlap
4. **Exports** the results to CSV for use in spreadsheets or other tools


---

## How to Use

### 1. Install & configure
Run the first cell. It installs dependencies and sets your configuration:

| Setting | What it controls |
|---|---|
| `DFS_LOGIN` / `DFS_PASSWORD` | Your DataForSEO API credentials |
| `LOCATION_CODE` | Target country (default: `2826` = United Kingdom) |
| `LANGUAGE_CODE` | Language (default: `en`) |
| `SE_DOMAIN` | Search engine domain (default: `google.co.uk`) |
| `TOP_N_RESULTS` | How many organic results to fetch per keyword (default: `10`) |
| `MIN_COMMON_URLS` | Minimum shared URLs needed to connect two keywords (default: `2`) |
| `MIN_OVERLAP` | Minimum overlap coefficient to form a cluster edge (default: `0.3`) |

### 2. Upload your keywords
Upload a `.txt` file with one keyword per line.

### 3. Run all cells in order
Each cell builds on the previous one. Run them top to bottom.

---

## Cell-by-Cell Breakdown

| Cell | What it does |
|---|---|
| **Imports & config** | Installs packages, sets all configuration variables |
| **Keyword upload** | Loads and deduplicates your keyword list from a `.txt` file |
| **SERP scraper** | Calls DataForSEO for each keyword, captures organic URLs, SERP feature types, rank positions, and pixel positions (`rectangle_y`*). Also extracts the first reference URL from AI Overview items. |
| **Clustering** | Builds a URL-overlap graph between keywords, runs TF-IDF to name each cluster after its most representative keyword, and flags unconnected keywords as Noise |
| **SERP element analysis** | Charts the distribution of SERP feature types (organic, AI overview, people also ask, etc.) and shows what percentage of elements appear above the fold |
| **Domain heatmaps** | Shows which domains rank most frequently across your keywords, and which domains compete for the same keywords |
| **Export** | Saves three CSVs and auto-downloads them in Colab |

`rectangle_y` is the pixel distance from the top of the Google search results page to where that SERP element starts.
A threshold of 400px is used as a proxy for the bottom of the screen on a mobile device.

- **Above fold** (`rectangle_y < 400`) — the element appears in the first screenful without scrolling. A high above-fold count suggests queries are triggering AI Overviews or other SERP features that dominate the top of the page, leading to **higher impressions** for those features but **reduced visibility** for organic results.
- **Below fold** (`rectangle_y ≥ 400`) — the user must scroll to see it. A high below-fold count suggests organic results are being pushed down by AI Overviews or other snippets, resulting in **fewer impressions and lower CTR** for organic listings.

### What does it mean for your keywords?

| Signal | Implication |
|---|---|
| High above-fold count | Queries likely trigger AI Overview or rich SERP features — organic visibility is compressed |
| High below-fold count | Organic results are displaced — expect lower impressions and CTR despite strong rankings |


## Output Files

| File | Contents |
|---|---|
| `serp_clusters.csv` | Every keyword with its cluster ID, cluster name, and top shared URL |
| `serp_raw_urls.csv` | Every SERP item returned (all feature types) with URL, rank, and pixel position |
| `serp_cluster_summary.csv` | One row per cluster: name, size, keyword list, shared URL count, average overlap score |

---

## Requirements

- A [DataForSEO](https://dataforseo.com) account with API access
- Python packages
- Google Colab (recommended) or a local Jupyter environment

---

## How Clustering Works

TF-IDF similarity for SERP clustering measures how closely search results relate by comparing the weighted importance of terms across pages, grouping queries with similar ranking content.  
It does **not rely on semantic similarity**, but rather on **string-based matching of SERP URLs**.

For each pair of keywords, the script computes an **overlap coefficient** — the number of shared organic URLs divided by the size of the smaller URL set. If two keywords share at least `MIN_COMMON_URLS` URLs and have an overlap coefficient above `MIN_OVERLAP`, they are connected by an edge in a graph. Connected components in this graph form the clusters.

Each cluster is then labelled using **TF-IDF cosine similarity** across the keyword set: the keyword most similar to all others in the cluster is selected as the cluster name.

Keywords with no connections to any other keyword are labelled **Noise / Unclustered**.

---

## Cost

Each keyword costs **$0.002** on the DataForSEO live advanced endpoint. A list of 500 keywords costs approximately **$1.00**.
