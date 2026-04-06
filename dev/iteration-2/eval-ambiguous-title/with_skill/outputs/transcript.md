# Transcript: Citing "learning to rank"

## Step 1: Read the skill file

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the citation workflow:
1. Query CrossRef API for Title -> DOI
2. Use citation-js to format DOI -> citation
3. If ambiguous, show top results for disambiguation

## Step 2: Query CrossRef for "learning to rank"

### Command:
```bash
curl -s "https://api.crossref.org/works?query.title=learning+to+rank&rows=3" | jq -r '.message.items[] | {title: .title[0], authors: [.author[]? | "\(.given // "") \(.family // "")"], DOI: .DOI, year: (.published["date-parts"][0][0] // "N/A")}'
```

### Output:
```json
{
  "title": "Learning to Rank",
  "authors": ["Hang Li"],
  "DOI": "10.1007/978-3-031-02141-1_1",
  "year": 2011
}
{
  "title": "Learning to Rank",
  "authors": ["Hang Li"],
  "DOI": "10.1007/978-1-4899-7502-7_893-1",
  "year": 2016
}
{
  "title": "Learning to Rank",
  "authors": ["Hang Li"],
  "DOI": "10.1007/978-1-4899-7687-1_893",
  "year": 2017
}
```

### Reasoning:
The title "learning to rank" is ambiguous -- multiple papers share this exact title. Per the skill instructions, when the top result may not be the right paper, I should use `rows=3` or more and show disambiguation. I expanded to `rows=5` to get a fuller picture.

## Step 3: Expanded search with rows=5

### Command:
```bash
curl -s "https://api.crossref.org/works?query.title=learning+to+rank&rows=5" | jq -r '.message.items[] | {title: .title[0], authors: [.author[]? | "\(.given // "") \(.family // "")"], DOI: .DOI, year: (.published["date-parts"][0][0] // "N/A"), type: .type}'
```

### Output:
5 results found, all titled "Learning to Rank":
1. Hang Li (2011) - DOI: 10.1007/978-3-031-02141-1_1 - book-chapter
2. Hang Li (2017) - DOI: 10.1007/978-1-4899-7687-1_893 - book-chapter
3. Hang Li (2016) - DOI: 10.1007/978-1-4899-7502-7_893-1 - book-chapter
4. Unknown author (2014) - DOI: 10.1007/978-1-4614-6170-8_100897 - book-chapter
5. Unknown author (2018) - DOI: 10.1007/978-1-4939-7131-2_100591 - book-chapter

### Reasoning:
All 5 results have the identical title "Learning to Rank." The top result by Hang Li (2011) in Synthesis Lectures on Human Language Technologies is the most prominent and well-known work with this title -- it is Hang Li's seminal monograph on learning to rank for information retrieval. Since I cannot interactively ask the user to disambiguate, I selected result #1 as the most likely intended paper and documented all alternatives.

## Step 4: Check citation-js availability

### Command:
```bash
which citation-js
```

### Output:
citation-js not found

### Action:
Installed citation-js:
```bash
npm install -g @citation-js/cli
```

## Step 5: Generate BibTeX citation

### Command:
```bash
citation-js -t "10.1007/978-3-031-02141-1_1" -f string -s bibtex
```

### Output:
```bibtex
@inbook{Li2011Learning,
	author = {Li, Hang},
	booktitle = {Synthesis {Lectures} on {Human} {Language} {Technologies}},
	doi = {10.1007/978-3-031-02141-1_1},
	isbn = {9783031021411},
	issn = {1947-4040},
	year = {2011},
	pages = {1--9},
	publisher = {Springer International Publishing},
	title = {Learning to {Rank}},
	url = {http://dx.doi.org/10.1007/978-3-031-02141-1_1},
}
```

## Step 6: Save outputs

Saved disambiguation and citation to `output.txt` and this transcript to `transcript.md`.
