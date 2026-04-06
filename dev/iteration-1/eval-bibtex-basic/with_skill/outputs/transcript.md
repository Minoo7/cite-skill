# Transcript: cite "attention is all you need"

## Step 0: Read the skill file

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the two-step process:
1. Query CrossRef API to find the DOI
2. Use doi.org content negotiation to fetch BibTeX

## Step 1: Look up the DOI via CrossRef

**Command:**
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&rows=1" | jq -r '.message.items[0] | {title: .title[0], authors: [.author[]? | "\(.given) \(.family)"], DOI: .DOI}'
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

**Reasoning:** The title and authors match, but the DOI `10.65215/nxvz2v36` is from a 2025 re-registration by "Shenzhen Medical Academy of Research and Translation", not the original 2017 NeurIPS paper. I queried with `rows=5` and all top results were 2025 re-registrations or reprints. The original NeurIPS proceedings version is not in CrossRef, but the arXiv version (DOI `10.48550/arXiv.1706.03762`) is the well-known canonical reference for this paper.

## Step 2: Fetch BibTeX from the first DOI

**Command:**
```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/10.65215/nxvz2v36"
```

**Output:**
```bibtex
@article{Vaswani_2025, title={Attention Is All You Need}, url={http://dx.doi.org/10.65215/nxvz2v36}, DOI={10.65215/nxvz2v36}, publisher={Shenzhen Medical Academy of Research and Translation}, author={Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and Uszkoreit, Jakob and Jones, Llion and N.Gomez, Aidan and Kaiser, Lukasz and Polosukhin, Illia}, year={2025}, month=aug }
```

**Reasoning:** This BibTeX entry has year=2025 and a dubious publisher. This is not the correct canonical citation. I decided to use the well-known arXiv DOI instead.

## Step 3: Fetch BibTeX from the arXiv DOI

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

**Reasoning:** This is the correct, canonical 2017 BibTeX entry for "Attention Is All You Need" by Vaswani et al. The arXiv DOI is the standard reference used across the ML community. I also attempted to find the NeurIPS proceedings DOI by filtering CrossRef by date (2017-2018) but it was not indexed there.

## Step 4: Save output

Saved the BibTeX entry to `output.txt`.
