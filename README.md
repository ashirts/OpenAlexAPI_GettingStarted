# OpenAlex API Starter Notebook

A beginner-friendly Jupyter notebook for exploring the [OpenAlex API](https://developers.openalex.org/) and exporting scholarly metadata to CSV.

This repository is designed as a companion resource for a quick-start guide on **OpenAlex for research data discovery**. It is meant for researchers, librarians, repository managers, and data support teams who want a practical first step into using OpenAlex programmatically.

## What this notebook does

`OpenAlexApi.ipynb` retrieves scholarly work metadata from OpenAlex for a selected institution and date range, then writes the results to a CSV file.

By default, the notebook is configured to pull works associated with **Utah State University** and export fields such as:

- Title
- Journal/source name
- Publisher
- Publication date
- Work type
- Open access status
- Version / accepted / published status
- License
- DOI
- Author names
- Author affiliations
- Author ORCID IDs

The notebook also includes a simple interactive JSON viewer so users who are new to APIs can inspect a real OpenAlex response and understand how nested fields work.

## Why this matters

OpenAlex is useful because it treats research outputs as connected metadata objects rather than isolated records. A publication, dataset, thesis, or software record can be connected to authors, institutions, funders, topics, repositories, identifiers, and related works.

For research data support, this helps with questions like:

- What research outputs are associated with an institution?
- Which works have DOIs or open access metadata?
- Which authors and affiliations are represented in a set of records?
- What metadata fields are available for enrichment or repository workflows?
- How can researchers move from a web search to a repeatable API-based workflow?

## Who this is for

This repository is especially useful for:

- Researchers who are new to APIs
- Librarians and repository staff working with publication or dataset metadata
- Research data support teams
- Students learning how JSON responses become tabular data
- Anyone who wants a small, editable OpenAlex starter script

No prior OpenAlex experience is required. Basic Python familiarity is helpful.

## Repository contents

```text
.
├── OpenAlexApi.ipynb    # Main Jupyter notebook
└── README.md            # Project overview and setup guide
```

## Requirements

The notebook uses a small Python stack:

```bash
pip install requests
```

If you are running the notebook locally, you will also need Jupyter:

```bash
pip install jupyter
```

Or open the notebook in an environment that already supports notebooks, such as:

- JupyterLab
- VS Code
- Google Colab
- Anaconda Navigator

## Getting started

### 1. Open the notebook

Open:

```text
OpenAlexApi.ipynb
```

Read the first markdown cell before running the code. It explains which variables you are expected to customize.

### 2. Choose an institution

The notebook starts with:

```python
institution_id = "https://openalex.org/i121980950"
```

This is currently set for Utah State University.

To use a different institution, search OpenAlex for the institution and replace this value with the appropriate OpenAlex institution ID.

Official guide:  
https://developers.openalex.org/quickstart

You can search institutions with an API call like:

```text
https://api.openalex.org/institutions?search=stanford
```

### 3. Choose an output filename

Update:

```python
output_file = "Title.csv"
```

Use a descriptive filename, for example:

```python
output_file = "openalex_usu_works_2019_2025.csv"
```

### 4. Review the API parameters

The notebook defines a request like this:

```python
params = {
    "filter": f"institution.id:{institution_id},from_publication_date:2019-01-01,to_publication_date:2025-07-15",
    "per_page": "200",
    "mailto": "your.email@usu.edu",
    "cursor": "*"
}
```

Update:

- `from_publication_date`
- `to_publication_date`
- `mailto`
- `institution_id`

For current OpenAlex API usage, you may also want to add an API key:

```python
"api_key": "YOUR_API_KEY"
```

OpenAlex API keys are free and recommended for higher-volume use. See the official authentication docs:  
https://developers.openalex.org/api-reference/authentication

### 5. Run the export cell

The main code cell:

1. Sends a request to the OpenAlex `/works` endpoint
2. Reads the JSON response
3. Extracts selected metadata fields
4. Writes one row per work to a CSV file
5. Uses cursor pagination to continue fetching results

Official paging guide:  
https://developers.openalex.org/guides/page-through-results

## Understanding the JSON viewer

The notebook includes a second section called **Exploring JSON Objects**.

This section is for users who are new to JSON. It fetches one OpenAlex Work record:

```python
url = "https://api.openalex.org/works/W3103145119"
```

Then it displays the nested JSON response in a collapsible format.

This helps users understand fields such as:

- `title`
- `publication_date`
- `doi`
- `open_access`
- `primary_location`
- `source`
- `authorships`
- `raw_affiliation_strings`

This is useful because OpenAlex responses are nested. For example, open access metadata lives inside `open_access`, while journal/source metadata lives inside `primary_location → source`.

Official Works documentation:  
https://developers.openalex.org/api-reference/works

## Customizing the CSV output

To add or remove fields, update two places:

### 1. The CSV header row

Example:

```python
writer.writerow([
    "Title", "Journal", "Publisher", "Date", "in_scopus", "type", "is_oa",
    "version", "is_accepted", "is_published", "license", "DOI"
] + author_headers)
```

### 2. The row-writing block

Example:

```python
writer.writerow([
    title, journal, publisher, date, in_scopus, work_type,
    is_oa, version, is_accepted, is_published, license, doi
] + authors_data)
```

If you add a field to the header, make sure you also extract it from the JSON and add it to the row in the same order.

## Example: adding `oa_status`

Current code:

```python
is_oa = work.get('open_access', {}).get('is_oa', '')
```

To also capture `oa_status`, add:

```python
oa_status = work.get('open_access', {}).get('oa_status', '')
```

Then add `"oa_status"` to the header row and `oa_status` to the row-writing block.

## Notes on author fields

The notebook captures up to 200 authors per work. For each author, it records:

- Author name
- Raw affiliation string
- ORCID

The code creates fixed-width author columns so the CSV has a consistent structure:

```text
Author1Name, Author1Affiliation, Author1ORCid, Author2Name, ...
```

If a work has fewer than 200 authors, the remaining author columns are left blank.

## Best practices and caveats

This notebook is intended as a learning tool and starting point, not a production-grade harvester.

Recommended practices:

- Start with a small date range while testing.
- Inspect the CSV before running a large export.
- Use `select` in API calls when you only need specific fields.
- Validate important metadata before using it for reporting or repository decisions.
- Treat OpenAlex as a discovery and enrichment layer, not the only source of truth.
- For very large exports, use the OpenAlex snapshot instead of paging through the entire API.

Official snapshot documentation:  
https://developers.openalex.org/download/overview

## Official OpenAlex documentation

Helpful links:

- OpenAlex Web: https://openalex.org/
- Developer docs overview: https://developers.openalex.org/
- API quickstart: https://developers.openalex.org/quickstart
- API overview: https://developers.openalex.org/api-reference/introduction
- Works documentation: https://developers.openalex.org/api-reference/works
- Searching guide: https://developers.openalex.org/guides/searching
- Pagination guide: https://developers.openalex.org/guides/page-through-results
- Authentication and pricing: https://developers.openalex.org/api-reference/authentication
- Data snapshot overview: https://developers.openalex.org/download/overview

## Suggested next steps

After running the starter notebook, try:

1. Searching for a different institution.
2. Changing the publication date range.
3. Adding fields such as `oa_status`, `cited_by_count`, `topics`, or `locations`.
4. Filtering for datasets with `type:dataset`.
5. Comparing GUI exploration in OpenAlex Web with API results.
6. Using the exported CSV for repository cleanup, metadata enrichment, or basic analysis.

