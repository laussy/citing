# citing

Every work that cites a paper, from three open citation indexes, merged into one list and checked against your BibTeX files and your notes.

```
citing sanchezmunoz14a -b ~/bib/sci.bib -k notes.tex -m
```

lists the papers citing *Emitters of N-photon bundles* (key `sanchezmunoz14a` in `sci.bib`, DOI 10.1038/nphoton.2014.114) that `notes.tex` does not cite yet, each with its key in `sci.bib` when it has one.

## Why

No single index has every citation. OpenAlex, OpenCitations and Semantic Scholar each miss some that the others have, and each has its own way of listing a preprint and its published version. `citing` asks all three, joins what belongs together, and says which citing works you already have under a key, which you have in your bibliography but have not used, and which are new. Keeping such a list up to date (who cited a paper since you last looked) becomes one command.

## Install

A single Python 3 file, standard library only.

```
curl -O https://raw.githubusercontent.com/laussy/citing/main/citing
chmod +x citing
```

It was first released as `citers` (1.0.0); the old GitHub address redirects here.

## Sources

| Source | What it gives |
|---|---|
| [OpenAlex](https://openalex.org) | the works whose reference list contains the paper |
| [OpenCitations](https://opencitations.net) | the same from Crossref and other open references; some rows carry only an arXiv or PubMed id, which `citing` names through the arXiv API and PubMed |
| [Semantic Scholar](https://www.semanticscholar.org) | citations including preprints, with the sentences that cite (`--contexts`) |
| [Crossref](https://www.crossref.org) | its own count of citing works, printed as a check on coverage, and the reference lists that `--hunt` reads |

A preprint is joined to its journal version when arXiv records the journal doi. Works are also joined by DOI, PubMed id and normalised title. Records with no identifier at all, which come from badly parsed PDFs, are folded into the work their title is closest to.

## Usage

```
citing PAPER [options]

  PAPER                 a DOI, an arXiv id (arXiv:2604.24204), an OpenAlex id
                        (W2067552737), or a key of the --bib files
  -b, --bib FILES       BibTeX file(s), comma-separated: citing works get their key
  -k, --known FILE      a text holding the keys already accounted for (- = stdin):
                        \cite{..}, {{key}}, {{cite|a,b}} ... with --bib only real keys count
  -w, --wiki-page TITLE read the known keys from a MediaWiki page instead
      --wiki-api URL    that wiki's api.php (or env CITING_WIKI_API)
  -o, --own NAMES       surnames marking self-citations ("Smith,van der Berg")
  -s, --since YEAR      only citing works from YEAR on
  -m, --missing         only works not among the known keys
  -f, --format FMT      text, csv, json, keys, ids
      --contexts        the sentences that cite, under each work
      --hunt QUERY      look for citations the indexes failed to link (below)
      --hunt-since YEAR start of the hunt (default: the paper's year)
      --hunt-max N      candidates checked per query (default 300)
      --mailto EMAIL    contact for the OpenAlex/Crossref polite pools (or env CITING_MAILTO)
      --s2-key KEY      Semantic Scholar API key (or env S2_API_KEY)
      --openalex-key K  OpenAlex API key (or env OPENALEX_API_KEY)
  -q, --quiet           no progress or summary on stderr
```

Output, one line per citing work:

```
K  2014  strekalov14a             Strekalov      A bundle of photons, please                                  ACS
K* 2024  zubizarretacasalengua24b Zubizarreta Ca Two photons everywhere                                       ACS
b  2021  arXiv_cosacchi21a        Cosacchi       Suitability of solid-state platforms as sources of $N$-photo AS
N  2026  arXiv:2604.15605         Tang           Deterministic multiphoton bundle emission via interference-i S
```

### The paper by its BibTeX key

With `--bib`, the paper can be named by its key. Its DOI is read from the entry's `doi` field, or from `url` (`url = {doi:10.1038/nphoton.2014.114}` and `url = {https://doi.org/...}` both work); an arXiv-only entry is followed through its arXiv id (`eprint`, `journal = {arXiv:2604.24204}` or an arXiv URL). An entry with neither, an old paper for instance, is looked up in OpenAlex by its title, year and first author, and `citing` says which work it took:

```
$ citing hanle24a
  hanle24a has no DOI in the bib; OpenAlex W1970887935 by title: 10.1007/bf01331827
citing 2.0.0: works citing hanle24a (doi:10.1007/bf01331827 W1970887935)
...
```

A misspelt key gets the closest ones as suggestions. A DOI or arXiv id given directly is also matched to its key, which then names the paper in the messages.

### Defaults: ~/.citingrc

Options used every time go in `~/.citingrc` (or the file named by the environment variable `CITING_RC`), written as on the command line, one or more per line, `#` for comments:

```
# my bibliography, so that "citing somekey" works
--bib ~/bib/refs.bib,~/bib/preprints.bib
--wiki-api https://example.org/w/api.php
```

An option on the command line replaces the same option from the file: `-b other.bib` reads only `other.bib`.

### Output

The first column is the status: **K** known (its key is in your text), **b** in a bib file but not in your text, **N** new (no key anywhere). A `*` marks a self-citation (`--own`). The last column says who found it: **A** OpenAlex, **C** OpenCitations, **S** Semantic Scholar, **X** `--hunt`. The summary goes to stderr, so the list can be piped.

### Recipes

The new ones as identifiers, for whatever turns a DOI into BibTeX:

```
citing 10.1038/nphoton.2014.114 -b refs.bib -f ids
```

Keys of citing works already in the bibliography but not yet in a LaTeX file:

```
citing somekey14a -b refs.bib -k paper.tex -m -f keys
```

The same against a MediaWiki page that lists papers by template, `{{key}}`:

```
citing somekey14a -b refs.bib -w "My citing papers" --wiki-api https://example.org/w/api.php -m
```

A spreadsheet of everything since 2024, self-citations flagged:

```
citing 10.1038/nphoton.2014.114 -b refs.bib -o "Smith" -s 2024 -f csv > citing.csv
```

### --hunt

An index only knows a citation if it managed to match the reference. A garbled author name or a missing DOI in a reference list is enough to lose it, and then all three indexes may miss it together. `--hunt QUERY` searches OpenAlex for works on the subject (title and abstract) that are not in the list, reads each one's reference list as deposited with Crossref, and keeps those that cite the paper. A reference counts as a citation if it has the paper's DOI, its title, or its volume, page, year and first author. It is slow, about a second per candidate, so the default cap is 300 candidates per query.

```
citing 10.1038/nphoton.2014.114 -b refs.bib --hunt "photon bundle" --hunt "multiphoton emission"
```

## Limitations

- **Google Scholar is not used.** It has no API and refuses scripts. In physics it typically finds a third to a half more citing items than Crossref: theses, conference abstracts, book chapters, journals outside Crossref, and preprints that were never published.
- A preprint whose title changed on publication, and whose arXiv record has no journal doi, stays a separate line (status N, an `arXiv:` id). Check those against the published papers by hand.
- Citation contexts come from Semantic Scholar's text, often taken from the arXiv version.
- Without `--s2-key`, Semantic Scholar's rate limits are met by waiting, which can take a minute. The arXiv API may refuse after many calls in a short time; the preprints then stay unnamed and `citing` says so.
- OpenAlex is read page by page, which stops at 10,000 citing works. Without an API key OpenAlex gives each address a daily budget (1,000 credits in 2026: a citation query costs one, a title search about ten), ample for `citing` but shared by everything else that calls OpenAlex from the same address.

## See also

[vip](https://github.com/laussy/vip) grows, from one paper, the tree of the most important works that cite it, then of those that cite them, and so on, using the functions of `citing`.

## License

MIT
