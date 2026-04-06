# Transcript: cite "Attention Is All You Need"

## Step 1: Read the cite skill

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the workflow: Title -> DOI via CrossRef -> citation-js for formatting.

## Step 2: Look up DOI via CrossRef

**Command:**
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&rows=3" | jq -r '.message.items[:3] | .[] | {title: .title[0], authors: [.author[]? | "\(.given) \(.family)"] | join(", "), year: .issued."date-parts"[0][0], DOI: .DOI, publisher: .publisher}'
```

**Output:**
```json
{
  "title": "Attention Is All You Need",
  "authors": "Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N.Gomez, Lukasz Kaiser, Illia Polosukhin",
  "year": 2025,
  "DOI": "10.65215/nxvz2v36",
  "publisher": "Shenzhen Medical Academy of Research and Translation"
}
{
  "title": "Attention Is All You Need",
  "authors": "J. Mark Bishop, Gabriel Seiberth",
  "year": 2025,
  "DOI": "10.1201/9781003561460-19",
  "publisher": "Routledge"
}
{
  "title": "Attention Is All You Need",
  "authors": "Ashish Vaswani, Noam Shazeer, Niki Parmar, Jakob Uszkoreit, Llion Jones, Aidan N.Gomez, Lukasz Kaiser, Illia Polosukhin",
  "year": 2025,
  "DOI": "10.65215/pc26a033",
  "publisher": "Shenzhen Medical Academy of Research and Translation"
}
```

**Reasoning:** All three CrossRef results are suspicious. The original paper is Vaswani et al. (2017) from NeurIPS, but CrossRef returns 2025-dated DOIs from "Shenzhen Medical Academy of Research and Translation" -- these are clearly re-registered/secondary DOIs, not the original publisher DOI. Per the skill instructions, I need to prefer the original publisher DOI and use Semantic Scholar or OpenAlex as fallback.

## Step 3: Try Semantic Scholar

**Command:**
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=attention+is+all+you+need&limit=3&fields=title,externalIds,venue,year"
```

**Output:**
```json
{"message": "Too Many Requests. Please wait and try again or apply for a key for higher rate limits.", "code": "429"}
```

**Reasoning:** Semantic Scholar is rate-limited. Moving to OpenAlex.

## Step 4: Try OpenAlex

**Command:**
```bash
curl -s "https://api.openalex.org/works?search=attention+is+all+you+need&per_page=3" | jq '.results[:3] | .[] | {title: .title, year: .publication_year, doi: .doi, source: .primary_location.source.display_name}'
```

**Output:**
```json
{
  "title": "Attention Is All You Need",
  "year": 2025,
  "doi": "https://doi.org/10.65215/2q58a426",
  "source": null
}
```

**Reasoning:** OpenAlex also returns a suspicious 2025 DOI from the same aggregator. NeurIPS (via Curran Associates) doesn't assign individual DOIs for older proceedings, so the arXiv DOI (`10.48550/arXiv.1706.03762`) is the best available and widely used standard identifier for this paper.

## Step 5: Generate BibTeX with citation-js

**Command:**
```bash
citation-js -t "10.48550/arXiv.1706.03762" -f string -s bibtex
```

**Output:**
```bibtex
@article{Vaswani2017Attention,
	author = {Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and Uszkoreit, Jakob and Jones, Llion and Gomez, Aidan N. and Kaiser, Lukasz and Polosukhin, Illia},
	doi = {10.48550/ARXIV.1706.03762},
	year = {2017},
	publisher = {arXiv},
	title = {Attention {Is} {All} {You} {Need}},
	url = {https://arxiv.org/abs/1706.03762},
}
```

**Reasoning:** The BibTeX key `Vaswani2017Attention` is already in a reasonable format. Per the skill instructions about key naming conventions, I normalized it to lowercase `vaswani2017attention` to match the common convention (e.g., `vaswani2017attention`, `devlin2019bert`).

## Step 6: Save output

Saved the final BibTeX to `output.txt` with the conventional key `vaswani2017attention`.

## Notes

- The user did not specify a citation format, so BibTeX was used as the default per the skill instructions.
- NeurIPS doesn't assign individual DOIs for older proceedings, so the arXiv DOI is the best available identifier for this paper.
- CrossRef returned re-registered DOIs from aggregators (2025 dates, unfamiliar publishers), which were correctly identified and rejected in favor of the known arXiv DOI.
