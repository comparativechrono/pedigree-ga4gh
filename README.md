# pedigree-ga4gh

Write, read, check and upgrade the GA4GH files that describe a family: the
**GA4GH Pedigree** and the **Phenopackets v2** Family, Phenopacket and Cohort.

It is the one GA4GH implementation behind the Hearn lab's pedigree tools.
studio2ga4gh, ga4gh2studio, abusua2ga4gh, ga4gh2abusua, the phenopacket
toolkit (phenotk) and the
[Pedigree workbench](https://github.com/comparativechrono/pedigree-workbench)
all read and write GA4GH files through it, so they agree on a single form. It
needs nothing beyond the Python standard library (Python 3.9 or later).

```bash
pip install pedigree-ga4gh
```

## What it writes

The GA4GH Pedigree Standard is still in development. Its published
documentation describes a conceptual model, carried either by Phenopackets or
by HL7 FHIR, and neither form has been approved. pedigree-ga4gh writes:

- **a GA4GH Pedigree in the Phenopackets form**, the form of the standard's
  published examples: every individual is a complete Phenopacket v2, and each
  relationship is `{individualId, relation, relativeId}` with a current term of
  KIN, the Kinship Ontology. Relationships follow the standard's *reduced
  form*: parents to their children, partners only when their children don't
  already say so, twins all relative to one twin, and no siblings, since
  shared parents imply them.
- **a Phenopackets Family** exactly to the Phenopackets v2 schema, with the
  whole family's pedigree in PED form and `consanguinousParents` worked out
  from the relationships: true when the proband's parents are recorded as
  consanguineous partners, or are related as second cousins or closer, as
  KIN defines it.

Every file it writes passes phenopacket-tools' JSON schemas. A condition
without an ontology term is left out with a warning rather than written with
an empty id, and so is a term from an ontology whose details it cannot write
in `metaData`: it knows the HPO, Mondo, OMIM, Orphanet, the Disease Ontology,
SNOMED CT, ICD-10-CM and the ontologies phenopacket-tools declares, and keeps
any other ontology a file it reads declares in full. Anything the standard
has no field for – a carrier, a pregnancy – goes into the pedigree's
narrative so that it isn't lost.

[docs/ga4gh.md](docs/ga4gh.md) sets out the decision, the sources behind it,
and the draft of version 1 of the standard that is now under review.

## On the command line

```bash
pedigree-ga4gh validate family.json             # check a file against the standard
pedigree-ga4gh validate family.json --json      # the findings as JSON
pedigree-ga4gh upgrade old.json -o new.json     # rewrite a file from an older tool
pedigree-ga4gh convert pedigree.json --to family -o family.json
pedigree-ga4gh convert family.json --to pedigree -o pedigree.json
pedigree-ga4gh kin                              # list the KIN relations
```

`validate` exits with status 1 when a file has errors, so it can be used in
scripts and checks.

## In Python

```python
from pedigree_ga4gh import (Kindred, Member, Relation, Term,
                            to_ga4gh_pedigree, to_phenopackets_family, validate, errors, dump)

trio = Kindred("FAM1", members=[
    Member("MOTHER", sex="FEMALE"),
    Member("FATHER", sex="MALE"),
    Member("CHILD", index=True, terms=[Term("cystic fibrosis", "MONDO:0009061")]),
], relations=[
    Relation("MOTHER", "KIN:027", "CHILD"),
    Relation("FATHER", "KIN:028", "CHILD"),
], versions={"MONDO": "2026-10-06"})

pedigree, warnings = to_ga4gh_pedigree(trio, created_by="my-tool 1.0")
family, warnings = to_phenopackets_family(trio, created_by="my-tool 1.0")
assert not errors(validate(pedigree)) and not errors(validate(family))
dump(pedigree, "FAM1_ga4gh-pedigree.json")
```

`from_document(doc)` reads any of the four kinds of file back into a
`Kindred`, `validate(doc)` returns the findings (errors, warnings and notes,
each with where in the file and how to fix it), and `upgrade(doc)` returns the
rewritten document with the list of changes and what is left for a person to
do.

## The checks

`validate` applies:

- the Phenopackets v2 JSON schemas of phenopacket-tools: required fields,
  types, enumerations, `oneof` groups, lengths and counts;
- phenopacket-tools' check that every ontology prefix used is declared in
  `metaData.resources`;
- the pedigree's own rules: everyone named in a relationship or as an index
  patient is in the file, relations are current KIN terms with KIN's labels,
  no one is related to themselves, a biological mother is female and a father
  male, no one has more than two genetic parents, and nobody is their own
  ancestor.

## Older files

studio2ga4gh up to 0.3 and abusua2ga4gh up to 0.2 wrote GA4GH files that the
standard does not accept, and earlier drafts of KIN used codes that have since
been retired. Those files are recognised (`identify_legacy`): the commands,
and the converters built on this package, refuse them with a message, and
`pedigree-ga4gh upgrade` rewrites them once, listing every change and what
still needs a person (usually conditions without an ontology term).
`from_document` itself reads whatever it is given, so check with
`identify_legacy` first if you call it directly.
A file in the form of the draft of version 1 is recognised too, and refused
with a message saying what it is: the draft isn't approved, and it can't be
converted yet.

## Licence

MIT (see [LICENSE](LICENSE)). The KIN terms in `pedigree_ga4gh/kin.py` come
from the GA4GH Pedigree Standard's `kin.owl`
([ga4gh/pedigree_family_history_terminology](https://github.com/ga4gh/pedigree_family_history_terminology),
commit 71616ea) and are used under the Apache License 2.0
(`pedigree_ga4gh/LICENSE-KIN.txt`).
