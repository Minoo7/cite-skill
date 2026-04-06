# Transcript: APA Citation for BERT Paper

## Step 1: Read the cite skill

Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the workflow: Title -> DOI via CrossRef, then DOI -> formatted citation via citation-js.

## Step 2: Verify citation-js is installed

```bash
which citation-js
```
Output: `/opt/homebrew/bin/citation-js` — confirmed available.

## Step 3: Look up the DOI via CrossRef

```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+of+deep+bidirectional+transformers&rows=3" | jq -r '.message.items[:3][] | {title, author, DOI, year, publisher}'
```

**Result:** The top 3 results were unrelated papers (FAD-BERT, Spectrum-BERT, Trajectory-BERT). The original BERT paper was not returned.

### Refined search with author name

```bash
curl -s "https://api.crossref.org/works?query.bibliographic=BERT+Pre-training+Deep+Bidirectional+Transformers+Language+Understanding+Devlin+Chang+Lee+Toutanova&rows=5" | jq ...
```

**Result:** Found the paper at position 4:
- Authors: Jacob Devlin, Ming-Wei Chang, Kenton Lee, Kristina Toutanova
- DOI: `10.18653/v1/n19-1423`
- Year: 2019
- Publisher: Association for Computational Linguistics

Also confirmed by direct DOI lookup:
```bash
curl -s "https://api.crossref.org/works/10.18653%2Fv1%2FN19-1423" | jq ...
```

**Reasoning:** This is the correct paper — the right authors (Devlin et al.), published at NAACL 2019 by ACL. The DOI `10.18653/v1/n19-1423` is the original publisher DOI (ACL Anthology), which is preferred over an arXiv DOI per the skill instructions.

## Step 4: Generate APA citation with citation-js

```bash
citation-js -t "10.18653/v1/n19-1423" -f string -s citation-apa
```

**Output:**
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). Proceedings of the 2019 Conference of the North, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

**Problem:** The title field is empty in CrossRef's metadata for this DOI. The citation-js output is missing the paper title entirely.

### Verification via doi.org

```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/10.18653/v1/n19-1423"
```

Confirmed: the title field is missing from the CrossRef/doi.org record itself. This is a known metadata issue for this particular paper.

### Verification via CSL-JSON

```bash
citation-js -t "10.18653/v1/n19-1423" -f json
```

Confirmed `"title": ""` in the JSON output.

## Step 5: Manual correction

Since the CrossRef metadata is incomplete (missing title), I manually constructed the APA citation using:
- The verified metadata from CrossRef (authors, year, pages, DOI, conference)
- The well-known paper title: "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"
- The full conference name: "Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers)"

**Reasoning:** The skill instructions say to use citation-js for formatting, but citation-js cannot produce a correct citation when the upstream metadata is incomplete. The paper's title is universally known and unambiguous. I corrected the citation to include the title and the full conference proceedings name, following APA 7th edition formatting for conference papers.

## Final APA Citation

```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of deep bidirectional transformers for language understanding. Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, Volume 1 (Long and Short Papers), 4171–4186. https://doi.org/10.18653/v1/n19-1423
```
