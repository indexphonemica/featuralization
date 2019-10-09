# Why not existing featuralization scripts?

IPHON needs a featuralization script that:
- is easily modifiable
- gives intuitive results for novel phonemes
- 'type-checks' characters to ensure that invalid representations are rejected from the DB
- can normalize input to avoid dual representation of the same sound (e.g. w̥ vs. ʍ, or õ̞ vs. õ̞)
- operates at both the binary feature level (like PHOIBLE) and at the descriptive level (like UPSID), for ease of search
- (ideally; this will probably be difficult) can be used to represent allophonic rules, to enable fine-grained search thereof; although allophonic rule representation imposes requirements that are unnecessary for segment featuralization (e.g. representation of zero, syllable and word boundaries, stress, onset/nucleus/coda or initial/medial/rime segmentation)

As far as I know, no current featuralization package meets all these needs.

Currently we're using PHOIBLE's featuralization, but there are a number of bad fits:
- It's written in R, which none of the IPHON developers know
- Poor separation of code and data (e.g. there's a file to define custom featuralization of certain n>1-graphs, but the featuralization package doesn't read from it by default; instead, the n-graph has to be defined in the file *and* added to the list of n-graphs to read from that file in the code)
- PHOIBLE featuralizes directly to the binary level, so going back 'up' to the UPSID-like descriptive level is nontrivial, which imposes developer burden for chart generation

# Design goals 

## Separation of code and data

The featuralization package should be easily reconfigurable (for example, to support Americanist phonetic notation) without touching the code, or needing to know the implementation language. A DSL or some configuration convention will be needed, but it's easier to learn a simple, non-TC *configuration* language than to learn a large, general-purpose, TC *programming* language.

## Clear representation of the featural ontology

Feature trees and valid features should be designed as types. In cases where the presence of a feature depends on another (for example, ±anterior only being present if +coronal, or ±high only being present if +dorsal), this should be represented at the type level. Invalid or nonsensical combinations of features should be prohibited at the type level.

## Ability to represent both binary and descriptive features

This is necessary for search, and will make automatic chart generation a lot nicer. This will probably work in two steps: descriptive featuralization of phonetic representations of segments, and binary featuralization of descriptive feature bundles.

## Normalization

Each feature bundle should have one and only one representation, except for phonetic representations that are disregarded for featuralization purposes. (For example, do we need to have front velars distinct from dorsopalatals, or back velars distinct from velars, or interdentals distinct from dentals? Maybe not! Should the 'more rounded' diacritic on a rounded vowel be represented at a featural level? There's precedent for this: there are some diacritics that PHOIBLE disregards for featuralization.)

This will involve, for example, imposing a total ordering on diacritics, so that inputs of `õ̞` and `õ̞` will produce the same featuralization and the same normalization.

## Type awareness

Certain IPA characters may mean different things in different contexts. For example:
- The character `t` represents affricate occlusion when it appears before a coronal fricative, and represents a full coronal plosive when it appears alone or in (say) the representation of a 'harmonic cluster' like Georgian `/tk/`. (Georgian 'harmonic clusters' are occasionally argued to be units; this should be representable at the feature level.)
- The diacritic COMBINING MINUS SIGN BELOW, in some dialects of IPA, transforms plain velars into back velars (e.g. `k̠`) and alveolars into postalveolars (e.g. `t̠`), and represents 'laryngealization' on vowels (e.g. Standard Nuosu <y> /ɿ/ vs. <yr> /ɿ̠/).
- The diacritic COMBINING BRIDGE BELOW, in some dialects of IPA, transforms alveolars into dentals (e.g. `t̪`) and labials into linguolabials (e.g. `p̪`), and cannot validly appear on certain classes of character (e.g. *`k̪`, *`a̪`).

This should be incorporated into the featural representation. So, instead of defining characters and diacritics as static, invariant bundles of features, they should be aware of their environment; for example:

```
diacritic X̪
  | X place:alveolar = place:dental
  | X place:labial   = place:linguolabial
  | otherwise        = TypeError
```

