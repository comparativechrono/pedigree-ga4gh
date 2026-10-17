# Changelog

## 1.0.0

The first release as a package of its own. Until now this code lived only in
the Pedigree workbench, which it continues to serve.

- One implementation of GA4GH output for family pedigrees, shared by
  studio2ga4gh (from 0.4.0), ga4gh2studio (from 0.4.0) and phenopacket-toolkit
  (from 0.4.0).
- Writes a GA4GH Pedigree in the Phenopackets form of the published standard,
  in its reduced form, and a Phenopackets v2 Family; reads both, and
  Phenopackets and Cohorts.
- `validate`: the checks of phenopacket-tools' JSON schemas and of its
  MetaDataValidator, and those of the pedigree.
- `upgrade`: rewrites files from studio2ga4gh up to 0.3 and abusua2ga4gh up to
  0.2, and retired KIN codes, once, listing every change.
- Recognises, and refuses with a message, files in the form of the draft of
  version 1 of the standard (ga4gh/pedigree pull request 50), which is not yet
  approved.
- KIN from `kin.owl` at commit 71616ea of
  ga4gh/pedigree_family_history_terminology (Apache License 2.0).
