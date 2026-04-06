# Transcript: cite "attention is all you need"

## Step 0: Read the skill file

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the citation workflow: Title -> DOI via CrossRef, then DOI -> BibTeX via citation-js.

## Step 1: Look up DOI via CrossRef

### Command
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&rows=3" \
  | jq -r '.message.items[:3] | .[] | {title: .title[0], author, year, DOI, publisher}'
```

### Output
Three results returned:
1. "Attention Is All You Need" — Vaswani et al., 2025, DOI `10.65215/nxvz2v36`, publisher "Shenzhen Medical Academy of Research and Translation"
2. "Attention Is All You Need" — Bishop & Seiberth, 2025, DOI `10.1201/9781003561460-19`, publisher Routledge (a book chapter)
3. Same as #1 with different DOI `10.65215/2q58a426`

### Reasoning
All results are suspicious. The first and third are from an unfamiliar re-registrar ("Shenzhen Medical Academy of Research and Translation") with a 2025 year — clearly not the original 2017 NeurIPS paper. The second is a Routledge book chapter by different authors. Per the skill instructions, I should "prefer the original conference/journal DOI over an arXiv DOI" and watch for "suspiciously recent year" or "unfamiliar publisher."

## Step 2: Retry with author filter

### Command
```bash
curl -s "https://api.crossref.org/works?query.title=attention+is+all+you+need&query.author=vaswani&rows=5" \
  | jq -r '.message.items[:5] | .[] | {title, author, year, DOI, publisher, type}'
```

### Output
All 5 results were from the same suspicious "Shenzhen Medical Academy of Research and Translation" publisher with 2025 dates and `10.65215/` DOI prefixes. The original NeurIPS 2017 DOI is not appearing in CrossRef results.

### Reasoning
The original paper's NeurIPS proceedings DOI is not in CrossRef (NeurIPS proceedings are not always registered there). The well-known arXiv DOI `10.48550/arXiv.1706.03762` is widely used and accepted. Per the skill: "arXiv DOIs are fine as a fallback." Since no original conference DOI is available, I used the arXiv DOI.

## Step 3: Verify citation-js is installed

### Command
```bash
which citation-js && citation-js --version
```

### Output
```
/opt/homebrew/bin/citation-js
0.7.22
```

citation-js is installed and ready.

## Step 4: Generate BibTeX citation

### Command
```bash
citation-js -t "10.48550/arXiv.1706.03762" -f string -s bibtex
```

### Output
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

### Post-processing
The BibTeX key `Vaswani2017Attention` was already reasonable but per the skill convention (`authorYYYYfirstword`), I normalized it to lowercase: `vaswani2017attention`.

## Final output

```bibtex
@article{vaswani2017attention,
	author = {Vaswani, Ashish and Shazeer, Noam and Parmar, Niki and Uszkoreit, Jakob and Jones, Llion and Gomez, Aidan N. and Kaiser, Lukasz and Polosukhin, Illia},
	doi = {10.48550/ARXIV.1706.03762},
	year = {2017},
	publisher = {arXiv},
	title = {Attention {Is} {All} {You} {Need}},
	url = {https://arxiv.org/abs/1706.03762},
}
```

Saved to `output.txt`.
