# 02 — Dataset Design

## 1. Why Dataset Design Matters

For NLP, the dataset is not just an input to the model.

The dataset defines:

- what the model is allowed to learn
- what each class means
- what counts as an error
- whether evaluation is trustworthy
- whether the model generalizes beyond memorized wording

A sophisticated model cannot reliably compensate for ambiguous labels, leakage, duplicates, or a poorly defined taxonomy.

## 2. Classification Target

Our initial target is:

    Emergency report → one incident class

Initial taxonomy:

1. Fire
2. Flood
3. Accident
4. Medical Emergency
5. Crime
6. Earthquake
7. UNDEFINED

The classes are currently mutually exclusive for the first classifier.

A future version may investigate multi-label classification if real reports regularly contain multiple independent incident types.

## 3. The UNDEFINED Class

UNDEFINED is deliberately different from an ordinary incident class.

It represents text that should not be forced into the supported incident taxonomy.

Examples:

> My uncle is playing with her girl in bed.

> What time does the movie start?

> The weather is beautiful today.

> Something strange happened near the road.

The last example may be emergency-related in spirit but lacks enough evidence for one of the supported classes.

### Why this matters

Real crisis datasets commonly include non-informative, irrelevant, or background categories. CrisisBench explicitly maps non-informative content as a separate category, supporting the decision to treat background/out-of-domain text explicitly rather than forcing every message into a crisis class.

## 4. Initial Dataset Schema

The canonical dataset will eventually use fields similar to:

| Field | Type | Purpose |
|---|---|---|
| report_id | string | Unique sample identifier |
| text | string | Original report |
| label | categorical | Ground-truth incident class |
| source | string | Dataset/source provenance |
| event_id | string | Crisis/event grouping |
| language | string | Language of the report |
| timestamp | datetime/null | Original report time if available |
| location | string/null | Location metadata if available |
| annotator_note | string/null | Reason for difficult labels |

Not every field will be used as a model feature.

For the first classifier, the primary input is text and the target is label.

Metadata such as event_id, timestamp, and source are primarily useful for analysis, splitting, provenance, and future extensions.

## 5. Provenance

Every real dataset sample should retain information about where it came from.

Possible sources include:

- CrisisNLP
- CrisisLex
- CrisisBench
- other properly licensed crisis datasets
- carefully curated project-specific examples

Existing crisis benchmarks combine multiple sources and perform label mapping and duplicate filtering, demonstrating why provenance and normalization matter when consolidating crisis datasets.

## 6. Real Data vs Controlled Learning Data

We will distinguish two purposes.

### A. Controlled learning dataset

A small, easy-to-inspect dataset can be used initially to understand:

- preprocessing
- tokenization
- feature extraction
- TF-IDF
- model mechanics

This dataset is for learning the pipeline, not for claiming real-world performance.

### B. Real-world benchmark dataset

For meaningful model comparisons, we should use real crisis/emergency data with documented provenance and labels.

We will not create a synthetic replacement dataset and then present its performance as evidence of real-world emergency classification ability.

## 7. Label Quality

A label is a ground-truth assumption about what a report means.

Bad label:

> “Smoke near the building” → Fire

This may be reasonable, but smoke could come from fire, machinery, vehicle exhaust, industrial activity, cooking, or controlled burning.

Therefore, ambiguous examples need either:

1. a defensible label with an annotation note, or
2. an UNDEFINED label when the taxonomy does not provide enough evidence.

We will avoid silently inventing labels.

## 8. Class Definitions

### Fire

Evidence that combustion/fire is occurring or being reported.

### Flood

Evidence of water accumulation, overflow, inundation, or flooding.

### Accident

An unintended harmful incident such as a vehicle collision or crash.

### Medical Emergency

A health-related emergency requiring urgent attention.

### Crime

A report describing an alleged criminal act or immediate criminal threat.

### Earthquake

Evidence of seismic activity or an earthquake event.

### UNDEFINED

Text that does not provide sufficient evidence for one supported class or is outside the supported taxonomy.

## 9. Data Leakage

Data leakage is one of the biggest risks in this project.

Suppose the same event produces:

> Major earthquake hits City X.

and:

> City X earthquake causes severe damage.

If nearly identical reports from the same event appear in both training and test sets, the measured score may look excellent even though the model has not truly generalized.

Crisis NLP research explicitly performs duplicate/retweet filtering and overlap checks to prevent test information from appearing during training.

Therefore:

> We must check duplicates before splitting and consider event-aware splitting when event identifiers are available.

## 10. Train / Validation / Test

We will eventually use:

    Training set → learn parameters
    Validation set → choose/tune model
    Test set → final unbiased evaluation

The test set should remain untouched until the evaluation stage.

A conventional starting point may be approximately 70% train, 15% validation, and 15% test.

The exact split will depend on dataset size and event structure.

If event IDs are available, an event-aware split may be preferable so that reports from the same crisis event do not leak across train and test.

## 11. Class Balance

We will inspect class counts before training.

A severely imbalanced dataset can make accuracy misleading.

We will therefore report:

- per-class precision
- per-class recall
- per-class F1
- macro F1
- weighted F1
- confusion matrix

## 12. UNDEFINED Is Also a Data Challenge

Adding UNDEFINED does not automatically solve out-of-domain detection.

A model may learn shortcuts such as:

    strange/rare wording → UNDEFINED
    no obvious emergency keyword → UNDEFINED

That could produce poor generalization.

We will therefore include diverse UNDEFINED examples:

- ordinary conversation
- questions
- unrelated news
- weather/general information
- ambiguous reports
- unsupported emergency types
- malformed/incomplete text

The class should represent the boundary of our taxonomy, not simply random text.

## 13. Duplicate Detection

Before model training we will investigate:

- exact duplicates
- normalized duplicates
- near-duplicates
- retweets/reposts where applicable
- repeated templates

Duplicates can inflate evaluation scores and make the model appear better than it is.

We will document the deduplication method and how many samples were removed.

## 14. Dataset Quality Checklist

Before training:

- [ ] Unique IDs
- [ ] No missing text
- [ ] No missing labels for supervised samples
- [ ] Valid class names
- [ ] Class distribution inspected
- [ ] Exact duplicates checked
- [ ] Near-duplicates considered
- [ ] Source/provenance retained
- [ ] Label definitions documented
- [ ] Ambiguous examples reviewed
- [ ] Train/validation/test strategy defined
- [ ] Leakage risks checked
- [ ] Test set isolated

## 15. Research Findings Relevant to Our Design

CrisisBench consolidated multiple crisis datasets and explicitly performed label mapping, filtering, and duplicate handling. This is a useful precedent for our own data pipeline.

Other crisis NLP work also shows that duplicate removal and overlap checks are important when constructing evaluation datasets.

A 2026 disaster corpus combines India and Nepal crisis posts and describes the data as noisy social-media text, reinforcing that realistic crisis NLP data differs substantially from clean textbook sentences.

## 16. Design Decisions

### Decision D1

Use UNDEFINED as an explicit initial class.

Reason: avoid forcing out-of-domain or insufficiently informative text into an emergency category.

### Decision D2

Retain dataset provenance.

Reason: different sources have different collection and annotation processes.

### Decision D3

Check duplicates before splitting.

Reason: prevent artificially inflated evaluation.

### Decision D4

Prefer event-aware splitting when event metadata is available.

Reason: test whether the model generalizes beyond wording from the same crisis event.

### Decision D5

Separate controlled learning data from real-world benchmark data.

Reason: learning experiments should not be confused with evidence of real-world performance.

## 17. Current Status

Completed: Dataset design specification

Next: Dataset acquisition and inspection

The next step is NOT model training.

We first inspect candidate datasets for:

- format
- language
- available labels
- class distribution
- duplicates
- missing values
- event information
- train/validation/test availability
- licensing/provenance
- compatibility with our seven-class taxonomy
