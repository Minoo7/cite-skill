# Transcript: APA citation for BERT paper

## Task
Generate an APA citation for "BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding."

## Step 1: Read the cite skill
Read `/private/var/folders/gm/7xp51xz5579_rhmm24m4j2l80000gn/T/tmp.w1a2azGo8d/.agents/skills/cite/SKILL.md` to understand the workflow: Title -> DOI via CrossRef, then DOI -> citation via citation-js.

## Step 2: Verify citation-js is installed
```bash
which citation-js && citation-js --version
```
Output: `/opt/homebrew/bin/citation-js`, version `0.7.22`. Confirmed installed.

## Step 3: Search CrossRef for the DOI
```bash
curl -s "https://api.crossref.org/works?query.title=BERT+pre-training+of+deep+bidirectional+transformers&rows=3" | jq -r '.message.items[:3][] | {title: .title[0], authors: [...], year, DOI, publisher}'
```
Output: CrossRef returned unrelated papers (FAD-BERT, Spectrum-BERT, Trajectory-BERT). The original BERT paper was not in the top results.

## Step 4: Try Semantic Scholar as fallback
```bash
curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=BERT+pre-training+of+deep+bidirectional+transformers&limit=3&fields=title,externalIds,venue,year,authors"
```
Output: 429 Too Many Requests. Rate-limited.

## Step 5: Try OpenAlex as fallback
```bash
curl -s "https://api.openalex.org/works?search=BERT+pre-training+of+deep+bidirectional+transformers&per_page=3" | jq '.results[:3][] | {title, doi, year}'
```
Output: Also returned unrelated papers. The original BERT paper was not found.

## Step 6: Search CrossRef with full title and author
```bash
curl -s "https://api.crossref.org/works?query.title=BERT+Pre-training+of+Deep+Bidirectional+Transformers+for+Language+Understanding&query.author=Devlin&rows=3" | jq ...
```
Output: Still did not return the BERT paper.

## Reasoning
The BERT paper (Devlin et al., 2019) was published at NAACL-HLT 2019 via the ACL Anthology. The known DOI is `10.18653/v1/N19-1423`. CrossRef's search index has trouble matching it, but the DOI itself is valid.

## Step 7: Generate APA citation using known DOI
```bash
citation-js -t "10.18653/v1/N19-1423" -f string -s citation-apa
```
Output:
```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). Proceedings of the 2019 Conference of the North, 4171–4186. https://doi.org/10.18653/v1/n19-1423
```

## Step 8: Inspect raw metadata
```bash
citation-js -t "10.18653/v1/N19-1423" -f json
```
Output showed that the `title` field is empty (`""`) in CrossRef's metadata for this DOI. This is a known issue with some ACL Anthology entries where the title is missing from the CrossRef registration.

## Step 9: Verify via doi.org content negotiation
```bash
curl -sLH "Accept: application/x-bibtex" "https://doi.org/10.18653/v1/N19-1423"
```
Output confirmed the DOI is correct (Devlin et al., 2019, pages 4171-4186, NAACL proceedings) but the title field is missing.

## Step 10: Construct correct APA citation
Since the DOI is verified as correct (`10.18653/v1/N19-1423`) and the paper metadata is well-established, I constructed the APA 7th edition citation manually using the confirmed metadata (authors, year, pages, publisher) from CrossRef combined with the well-known title and full proceedings name:

```
Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). BERT: Pre-training of deep bidirectional transformers for language understanding. In Proceedings of the 2019 Conference of the North American Chapter of the Association for Computational Linguistics: Human Language Technologies (pp. 4171–4186). Association for Computational Linguistics. https://doi.org/10.18653/v1/N19-1423
```

## Notes
- The CrossRef metadata for DOI `10.18653/v1/N19-1423` has an empty title field, which is why citation-js produced an incomplete citation.
- The DOI, authors, year, pages, and publisher were all confirmed from CrossRef; only the title needed to be supplied from known information about this widely-cited paper.
- The citation follows APA 7th edition format for a conference paper.
