---
name: cite
description: >
  Look up a paper by title and return a formatted citation (BibTeX, APA, Chicago, IEEE, Vancouver, RIS, etc.).
  Use this skill whenever the user wants to cite a paper, get a BibTeX entry, look up a DOI,
  generate a reference, or convert a paper title into any citation format. Also trigger when
  the user mentions CrossRef, citation-js, doi.org, or asks "how do I cite X".
---

# Cite

Turn a paper title into a properly formatted citation using CrossRef and citation-js. No API keys, no accounts needed.

## Prerequisites

- `curl` and `jq` (for DOI lookup)
- `citation-js` CLI: `npm install -g @citation-js/cli`

If citation-js is not installed, tell the user to run `npm install -g @citation-js/cli` first.

## How it works

1. **Title → DOI**: Query the CrossRef API to find the paper's DOI
2. **DOI → Citation**: Use `citation-js` to format the citation in any style

## Step 1: Look up the DOI

```bash
curl -s "https://api.crossref.org/works?query.title=QUERY&rows=1" | jq -r '.message.items[0]'
```

Replace spaces in the title with `+`. Use `query.title` (not `query.bibliographic`) for better title matching.

Before proceeding, confirm with the user that the top result is the right paper. Show them the title, authors, and DOI. If it looks wrong, try `rows=3` and show the top 3 results so the user can pick.

### Handling ambiguous titles

If multiple results share the same or very similar title, do NOT just pick one and cite it. Instead, list the candidates (title, authors, year, DOI) and ask the user which one they mean. Only proceed to Step 2 once the user has confirmed a specific paper.

### Prefer original publisher DOIs

CrossRef may return re-registered or secondary DOIs (e.g. from aggregators or reprint services) instead of the original publisher's DOI. Look for clues like a suspiciously recent year, an unfamiliar publisher, or a DOI prefix that doesn't match the expected venue.

**DOI preference order** (use the first one you can find through API lookups):
1. Original conference/journal publisher DOI (ACL, IEEE, ACM, Springer, etc.)
2. arXiv DOI — only as a last resort, and only if discovered through an API (not from your own knowledge)

**Important**: Do NOT fall back to a DOI you "just know" from training data. Every DOI used must come from an API response in this session. If all APIs fail to return a usable DOI, tell the user you couldn't find one and suggest they check the paper's page directly.

**Lookup sequence when CrossRef returns junk** (re-registered DOIs, wrong papers, suspicious publishers):

**Step 1** — Try DBLP, which is excellent for CS papers and reliably has correct venue info:
```bash
curl -s "https://dblp.org/search/publ/api?q=TITLE+FIRST_AUTHOR_LASTNAME&format=json&h=3" | jq '.result.hits.hit[] | {title: .info.title, venue: .info.venue, year: .info.year, doi: .info.doi, ee: .info.ee}'
```
DBLP's `doi` field (if present) is always the original publisher DOI. The `ee` field has the direct link to the paper (proceedings page, publisher URL). If DBLP has a `doi`, use it. If `doi` is null but `ee` points to the proceedings, note that for the user.

**Step 2** — Try Semantic Scholar, which separates publisher DOIs from arXiv IDs:
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=TITLE&limit=3&fields=title,externalIds,venue,year"
```
The `externalIds` field will have `DOI` (publisher) and `ArXiv` separately. Prefer the `DOI` field. Only use the `ArXiv` field (as `10.48550/arXiv.XXXX`) if no publisher DOI exists.

**Step 3** — If Semantic Scholar is rate-limited, try OpenAlex:
```bash
curl -s "https://api.openalex.org/works?search=TITLE&per_page=3"
```

**Step 4** — If no API returned a publisher DOI, check if DBLP or Semantic Scholar had an ArXiv ID. If so, use the arXiv DOI (`10.48550/arXiv.XXXX`) and explain to the user why (e.g. "NeurIPS doesn't assign individual DOIs for older proceedings, so the arXiv DOI is the best available"). Also include the proceedings URL from DBLP's `ee` field if available, so the user has the direct link.

**Step 5** — If no API returned any usable identifier at all, tell the user you couldn't find a verified DOI and suggest they look it up on the paper's venue page or Google Scholar.

## Step 2: Format the citation with citation-js

citation-js accepts a DOI directly and outputs in any format/style:

### BibTeX (default)
```bash
citation-js -t "DOI_HERE" -f string -s bibtex
```

### Named citation styles (APA, IEEE, Vancouver, etc.)
```bash
citation-js -t "DOI_HERE" -f string -s citation-apa
citation-js -t "DOI_HERE" -f string -s citation-ieee
citation-js -t "DOI_HERE" -f string -s citation-vancouver
citation-js -t "DOI_HERE" -f string -s citation-chicago-author-date
citation-js -t "DOI_HERE" -f string -s citation-harvard-cite-them-right
citation-js -t "DOI_HERE" -f string -s citation-modern-language-association
citation-js -t "DOI_HERE" -f string -s citation-nature
citation-js -t "DOI_HERE" -f string -s citation-science
```

The pattern is `citation-` followed by the CSL style ID. Thousands of styles are supported — the style name matches the [CSL Style Repository](https://github.com/citation-style-language/styles) filename without `.csl`.

### Other output formats
```bash
citation-js -t "DOI_HERE" -f string -s ris       # RIS format
citation-js -t "DOI_HERE" -f json                  # CSL-JSON
citation-js -t "DOI_HERE" -f html -s citation-apa  # HTML formatted
```

### Language support
```bash
citation-js -t "DOI_HERE" -f string -s citation-apa -l fr-FR  # French
citation-js -t "DOI_HERE" -f string -s citation-apa -l de-DE  # German
```

## Other input types

citation-js also accepts inputs beyond DOIs — if the user provides any of these, skip the CrossRef lookup and pass directly to citation-js:

- **DOI**: `citation-js -t "10.xxxx/xxxxx"`
- **ISBN**: `citation-js -t "978-3-16-148410-0"`
- **PMID/PMCID**: `citation-js -t "PMID:12345678"`
- **Wikidata ID**: `citation-js -t "Q12345"`
- **BibTeX string**: `citation-js -i input.bib -f string -s citation-apa` (converts between formats)
- **ORCID profile**: `citation-js -t "https://orcid.org/0000-0002-1234-5678"`

## Choosing a format

- If the user doesn't specify a format, use **BibTeX**
- If they say "APA", "Chicago", "IEEE", etc., use the matching `citation-*` style
- If they ask for a journal-specific style (e.g. "Nature", "Science", "Elsevier"), use `citation-nature`, `citation-science`, etc.
- If they want multiple formats, just run citation-js once per format

## Output

Present the citation in a code block so it's easy to copy. If BibTeX, use a `bibtex` code fence. For plain-text styles (APA, Chicago, etc.), just use a plain code block.

### BibTeX key naming

If citation-js or doi.org returns a BibTeX key that's a URL or DOI string (like `@misc{https://doi.org/10.48550/...}`), rewrite it to a conventional key like `authorYYYYfirstword` (e.g. `vaswani2017attention`, `devlin2019bert`). BibTeX keys should be short, readable identifiers — not URLs.

If the user seems like they want to copy it somewhere specific (clipboard, file, LaTeX project), suggest the appropriate next step — but don't over-assume.

## Troubleshooting

- **Wrong paper from CrossRef**: Try `rows=3` or `rows=5` and show options. Adding author names to the query can help: `query.title=TITLE&query.author=AUTHOR`
- **citation-js not found**: User needs to run `npm install -g @citation-js/cli`
- **Unknown style**: The style ID must match a CSL repository filename. Common ones: `apa`, `ieee`, `vancouver`, `chicago-author-date`, `harvard-cite-them-right`, `nature`, `science`, `modern-language-association`
- **Paper not in CrossRef**: Some papers (especially very recent preprints) may not be indexed yet. Try Semantic Scholar API as a fallback: `curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=TITLE&limit=1&fields=title,externalIds"`
- **Fallback to doi.org**: If citation-js is unavailable, you can fall back to doi.org content negotiation: `curl -sLH "Accept: application/x-bibtex" "https://doi.org/DOI_HERE"`
