# Transcript: cite "Attention Is All You Need"

## Step 1: Read the skill file

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the citation workflow: Title -> DOI via CrossRef, then DOI -> BibTeX via citation-js (or doi.org fallback).

## Step 2: Look up the DOI via CrossRef

**Command:**
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&rows=1" | jq -r '.message.items[0] | {title: .title[0], authors: [.author[] | "\(.given) \(.family)"], DOI: .DOI}'
```

**Output:**
```json
{
  "title": "Attention Is All You Need",
  "authors": [
    "Ashish Vaswani",
    "Noam Shazeer",
    "Niki Parmar",
    "Jakob Uszkoreit",
    "Llion Jones",
    "Aidan N.Gomez",
    "Lukasz Kaiser",
    "Illia Polosukhin"
  ],
  "DOI": "10.65215/nxvz2v36"
}
```

**Reasoning:** The title and authors match, but the DOI `10.65215/nxvz2v36` is from "Shenzhen Medical Academy of Research and Translation" (a re-registration, not the original). Needed to find the original 2017 paper DOI.

## Step 3: Check if citation-js is installed

**Command:**
```bash
which citation-js
```

**Output:** Not found. Per the skill's troubleshooting section, falling back to doi.org content negotiation.

## Step 4: Try doi.org fallback with the CrossRef DOI

**Command:**
```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/10.65215/nxvz2v36"
```

**Output:** Returned a BibTeX entry but with year=2025 and publisher="Shenzhen Medical Academy of Research and Translation" -- clearly a re-registration, not the original NeurIPS 2017 paper.

## Step 5: Search CrossRef with author filter

**Command:**
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&query.author=vaswani&rows=3" | jq -r '.message.items[] | {title, DOI, publisher, year}'
```

**Output:** All 3 results were from the same re-registration publisher (year 2025). The original paper's DOI is not well-indexed in CrossRef under NeurIPS.

**Reasoning:** The well-known arXiv identifier for this paper is `1706.03762`, with DOI `10.48550/arXiv.1706.03762`. Used this directly.

## Step 6: Fetch BibTeX using the known arXiv DOI

**Command:**
```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/10.48550/arXiv.1706.03762"
```

**Output:**
```bibtex
@misc{https://doi.org/10.48550/arxiv.1706.03762,
  doi = {10.48550/ARXIV.1706.03762},
  url = {https://arxiv.org/abs/1706.03762},
  author = {Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and Uszkoreit, Jakob and Jones, Llion and Gomez, Aidan N. and Kaiser, Lukasz and Polosukhin, Illia},
  keywords = {Computation and Language (cs.CL), Machine Learning (cs.LG), FOS: Computer and information sciences, FOS: Computer and information sciences},
  title = {Attention Is All You Need},
  publisher = {arXiv},
  year = {2017},
  copyright = {arXiv.org perpetual, non-exclusive license}
}
```

**Reasoning:** This is the correct original 2017 paper by Vaswani et al. with all 8 authors. The BibTeX entry is complete and accurate.

## Step 7: Save outputs

- Saved BibTeX citation to `outputs/output.txt`
- Saved this transcript to `outputs/transcript.md`
