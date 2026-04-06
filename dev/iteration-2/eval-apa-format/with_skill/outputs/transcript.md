# Citation Transcript: BERT paper in APA format

## Task
Generate an APA citation for "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding".

## Step 1: Read the cite skill
Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` which describes using CrossRef + citation-js to generate citations.

## Step 2: Check prerequisites and install citation-js

```bash
which citation-js
# NOT_INSTALLED

npm install -g @citation-js/cli
# Installed successfully
```

## Step 3: Look up DOI via CrossRef

```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+of+deep+bidirectional+transformers&rows=1" \
  | jq -r '.message.items[0] | {title: .title[0], authors: ..., DOI: .DOI}'
```

**Result:** Wrong paper returned — "FAD-BERT: Improved prediction of FAD binding sites..." (DOI: 10.1016/j.compbiomed.2021.104258). The original BERT paper was not the top CrossRef result.

## Step 4: Retry with author name

```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+...&query.author=Devlin&rows=3" | jq ...
```

**Result:** Still wrong papers returned. CrossRef did not surface the BERT paper.

## Step 5: Fallback to Semantic Scholar (per skill troubleshooting instructions)

```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=BERT+pre-training+deep+bidirectional+transformers+language+understanding&limit=3&fields=title,externalIds"
```

**Result:** Found the correct paper:
- Title: "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding"
- DOI: `10.18653/v1/N19-1423`
- ArXiv: 1810.04805

## Step 6: Generate APA citation with citation-js

```bash
citation-js -t "10.18653/v1/N19-1423" -f string -s citation-apa
```

**Result:** Citation was missing the paper title:
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). Proceedings of the 2019 Conference of the North, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

## Step 7: Investigate metadata issue

```bash
citation-js -t "10.18653/v1/N19-1423" -f json
```

**Result:** The `title` field in CrossRef's metadata for this DOI is empty (`"title":""`). This is a known metadata gap in CrossRef for some ACL conference papers.

## Step 8: Fix by providing corrected CSL-JSON input

Created a corrected CSL-JSON file with the proper title and full conference name, then ran:

```bash
citation-js -i /tmp/bert_csl.json -f string -s citation-apa
```

**Result (correct APA citation):**
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding. Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

## Reasoning
- The skill recommended CrossRef as the primary DOI lookup, but CrossRef failed to return the BERT paper for this query.
- The skill's troubleshooting section recommended Semantic Scholar as a fallback, which successfully found the paper and its DOI.
- CrossRef's metadata for this DOI had an empty title field, so citation-js produced an incomplete citation.
- To fix this, I constructed a corrected CSL-JSON record with the known title and conference name, then fed it to citation-js to produce a properly formatted APA citation.
