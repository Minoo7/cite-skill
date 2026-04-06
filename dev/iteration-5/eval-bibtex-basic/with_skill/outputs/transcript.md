# Transcript: cite "attention is all you need"

## Step 1: Read the skill file

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the citation workflow: Title -> DOI via CrossRef -> BibTeX via citation-js.

## Step 2: Check prerequisites

Confirmed `citation-js` is installed at `/opt/homebrew/bin/citation-js`.

## Step 3: Look up DOI via CrossRef

**Command:**
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&rows=3" | jq -r '.message.items[:3][] | {title: .title[0], author: ..., DOI: .DOI, year: ..., publisher: .publisher}'
```

**Output:** All 3 results were suspicious -- re-registered DOIs from "Shenzhen Medical Academy of Research and Translation" with year 2025. These are aggregator/reprint DOIs, not the original publisher DOI.

**Reasoning:** Per the skill instructions, CrossRef returned junk (re-registered DOIs, wrong publisher, suspiciously recent year). Need to follow the fallback lookup sequence.

## Step 4: Try Semantic Scholar (Step 1 of fallback)

**Command:**
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=attention+is+all+you+need&limit=3&fields=title,externalIds,venue,year"
```

**Output:** Rate limited (HTTP 429).

## Step 5: Try OpenAlex (Step 2 of fallback)

**Command:**
```bash
curl -s "https://api.openalex.org/works?search=attention+is+all+you+need&per_page=3" | jq ...
```

**Output:** First result also had the aggregator DOI (`10.65215/2q58a426`). No original publisher DOI found here either.

## Step 6: Retry Semantic Scholar with author name

**Command:**
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=attention+is+all+you+need+vaswani&limit=3&fields=title,externalIds,venue,year"
```

**Output:** Successfully found the paper:
- Title: "Attention is All you Need"
- Venue: Neural Information Processing Systems
- Year: 2017
- ExternalIds: ArXiv: 1706.03762, no publisher DOI field

**Reasoning:** The paper was published at NeurIPS 2017. NeurIPS did not assign individual DOIs for older proceedings. Semantic Scholar has no publisher DOI but does have an ArXiv ID. Per the skill's Step 3 of the fallback sequence, use the arXiv DOI (`10.48550/arXiv.1706.03762`) and explain why.

## Step 7: Generate BibTeX with citation-js

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

## Step 8: Post-process BibTeX key

Per the skill instructions, rewrote the BibTeX key from `Vaswani2017Attention` to `vaswani2017attention` (conventional lowercase format).

## Step 9: Save output

Saved the final BibTeX to `output.txt`.

## Notes

- The arXiv DOI was used because NeurIPS did not assign individual DOIs for the 2017 proceedings, so the arXiv DOI is the best available identifier discovered through API lookups.
- All DOIs used came from API responses (Semantic Scholar ArXiv ID -> arXiv DOI convention), not from training data.
