# Transcript: APA Citation for BERT Paper

## Step 0: Read the Skill File

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the citation workflow:
1. Use CrossRef API to find DOI from paper title
2. Use doi.org content negotiation to get formatted citation

## Step 1: Look Up the DOI via CrossRef

### Attempt 1: Basic title search
```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+of+deep+bidirectional+transformers&rows=1" | jq -r '.message.items[0] | {title, author, DOI}'
```
**Result:** Returned wrong paper — "FAD-BERT: Improved prediction of FAD binding sites using pre-training of deep bidirectional transformers" (DOI: 10.1016/j.compbiomed.2021.104258).

### Attempt 2: Fuller title with more results
```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+of+deep+bidirectional+transformers+for+language+understanding&rows=3" | jq -r '.message.items[] | {title, author, DOI}'
```
**Result:** Top 3 results were FAD-BERT, Spectrum-BERT, and Trajectory-BERT. Still not the original BERT paper.

### Attempt 3: Add author name
```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+deep+bidirectional+transformers+language+understanding&query.author=Devlin&rows=3" | jq -r '.message.items[] | {title, author, DOI}'
```
**Result:** Returned unrelated papers by people named Devlin.

### Attempt 4: Broader bibliographic search with all author names
```bash
curl -s "https://api.crossref.org/works?query.bibliographic=BERT+pre-training+deep+bidirectional+transformers+language+understanding+Devlin+Chang+Lee+Toutanova&rows=5" | jq '.'
```
**Result:** Found the correct paper as the 4th result:
- **Authors:** Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina Toutanova
- **DOI:** `10.18653/v1/n19-1423`
- **Venue:** Proceedings of the 2019 Conference of the North (NAACL 2019)
- **Pages:** 4171-4186
- **Citations:** 6048

**Reasoning:** The title field in CrossRef is empty for this entry (a known metadata issue), but the authors (Devlin, Chang, Lee, Toutanova), venue (NAACL 2019), and page range (4171-4186) all confirm this is the correct BERT paper.

## Step 2: Fetch APA Citation via doi.org Content Negotiation

The user requested APA format, so used the `text/x-bibliography; style=apa` Accept header:

```bash
curl -sLH "Accept: text/x-bibliography; style=apa" "https://doi.org/10.18653/v1/n19-1423"
```

**Raw output from doi.org:**
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). Proceedings of the 2019 Conference of the North, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

**Issue:** The citation returned by doi.org is missing the paper title because the CrossRef metadata has an empty title field for this DOI. The container title is also truncated.

## Step 3: Construct Corrected APA Citation

Since the doi.org response had a missing title due to incomplete CrossRef metadata, I manually corrected the citation using the known paper details:
- Full title: "BERT: Pre-training of deep bidirectional transformers for language understanding"
- Full conference name: "Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies"

**Final APA citation:**
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of deep bidirectional transformers for language understanding. Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

This follows APA 7th edition format for a conference proceedings article.
