# 01 — Problem Definition

## 1. Objective

Build a Natural Language Processing (NLP) system that reads an emergency report written in natural language and predicts the type of incident being described.

Example:

> Two cars collided on the highway and several people are injured.

Output:

`Accident`

The first version is a **multi-class text classification** problem.

---

## 2. What is Text Classification?

Text classification is the task of mapping a text document to one or more predefined categories.

For this project:

```
Emergency Report → Incident Class
```

Mathematically, given a text document (x), the classifier learns a function:

```
f(x) → y
```

where:

- (x) = input emergency report
- (y) = predicted incident class

For a probabilistic classifier:

```
P(y | x)
```

represents the probability of each possible class given the report.

The final prediction is usually:

```
ŷ = argmax_y P(y | x)
```

That means we select the class with the highest predicted probability.

---

## 3. Initial Classes

The first version will use six incident categories:

| Class | Example |
|---|---|
| Fire | Building is burning near the market |
| Flood | Water has flooded several streets |
| Accident | Two cars collided on the highway |
| Medical Emergency | A person collapsed and needs an ambulance |
| Crime | Robbery reported near the ATM |
| Earthquake | Strong shaking felt across the city |

A future version may add more classes if the dataset and experiments justify them.

---

## 4. Why This Is an NLP Problem

The input is not a structured numerical vector.

A human can understand:

> The road outside the hospital is flooded, but the hospital itself is safe.

A traditional ML algorithm cannot directly understand the sentence.

We therefore need a pipeline that converts language into a machine-readable representation:

```
Natural Language
      ↓
Text Representation
      ↓
Numerical Features
      ↓
Machine Learning Model
      ↓
Incident Class
```

This representation step is one of the central problems we will study throughout the project.

---

## 5. Why Emergency Reports Are Challenging

Emergency text contains information that simple word matching may fail to capture.

### 5.1 Synonyms and paraphrases

These reports can describe the same event:

> A building is burning.

> Flames have engulfed a structure.

The vocabulary is different, but the semantic meaning is similar.

### 5.2 Context

> There is a fire near the hospital.

and:

> There is no fire inside the hospital.

Both contain the word **fire**, but their meanings are different.

### 5.3 Negation

Words such as:

- no
- not
- never
- without

can fundamentally change the interpretation of a report.

### 5.4 Short reports

Real emergency reports may be extremely short:

> Huge smoke near station.

There may be insufficient context to confidently classify the incident.

### 5.5 Noisy language

Real users may produce:

- spelling mistakes
- abbreviations
- informal language
- incomplete sentences
- repeated words
- location-specific terminology

### 5.6 Ambiguity

Example:

> Explosion-like sound heard near the factory.

This may not contain enough evidence to confidently classify the event as an explosion, accident, or another incident.

This creates an important distinction:

**classification confidence is not the same thing as truth.**

---

## 6. What the Classifier Must Learn

The classifier should not merely memorize individual words.

Ideally, it should learn patterns such as:

```
"burning", "flames", "smoke" → Fire-related evidence

"water", "flooded", "overflow" → Flood-related evidence

"crashed", "collision", "vehicles" → Accident-related evidence

"collapsed", "injured", "ambulance" → Medical-related evidence

"robbery", "stolen", "burglary" → Crime-related evidence

"tremor", "shaking", "quake" → Earthquake-related evidence
```

But the project will deliberately investigate where this simple lexical view breaks down.

---

## 7. Learning Objective

Given a training dataset:

```
D = {(x₁,y₁), (x₂,y₂), ..., (xₙ,yₙ)}
```

we want to learn a model (f) that generalizes from known reports to unseen reports.

The important word is **generalizes**.

For example, if training data contains:

> Car crashed into a truck.

the model should ideally classify:

> Two vehicles collided on the highway.

as an accident even though the exact wording is different.

This is one reason semantic representations will become important later.

---

## 8. Project Progression

We will intentionally increase representational complexity:

```
Raw Text
   ↓
Bag-of-Words / TF-IDF
   ↓
Classical ML
   ↓
Word / Document Embeddings
   ↓
Neural NLP
   ↓
Contextual Embeddings
   ↓
Transformers
```

TF-IDF will be used as a **baseline**, not assumed to be the final solution.

Every major increase in complexity must be justified by experimental evidence.

---

## 9. Evaluation Philosophy

Accuracy alone will not determine whether a model is good.

We will track:

- Precision
- Recall
- F1-score
- Macro F1
- Weighted F1
- Confusion matrix
- Per-class performance

This is particularly important because different incident classes may have different error patterns.

We will also inspect individual misclassified reports.

---

## 10. Important Project Principle

> A more complex NLP model is not automatically a better model.

A transformer may perform worse than TF-IDF if:

- the dataset is too small
- labels are noisy
- classes are poorly defined
- training is inappropriate
- the evaluation setup is flawed

Therefore, this project is experiment-driven rather than architecture-driven.

---

## 11. Current Hypotheses

### Hypothesis H1

A TF-IDF representation combined with a linear classifier should provide a strong initial baseline.

### Hypothesis H2

Semantic embeddings should improve robustness to vocabulary variation and paraphrases.

### Hypothesis H3

Contextual transformer representations should handle context-dependent meaning better than purely lexical representations.

### Hypothesis H4

Dataset quality and label quality may become a larger bottleneck than model complexity.

These hypotheses will be tested rather than treated as facts.

---

## 12. Questions We Will Answer

By the end of the project, we should be able to answer:

1. How does a machine represent human language?
2. What information does TF-IDF preserve and lose?
3. Why do embeddings capture semantic relationships?
4. Why are contextual embeddings different from static embeddings?
5. Why does attention help NLP?
6. What does a transformer actually learn?
7. When does a simple classifier outperform a complex model?
8. Which errors are caused by the data and which by the model?
9. How should an NLP classifier be evaluated for an emergency domain?
10. What changes actually improve generalization?

---

## 13. Status

**Completed:** Problem definition

**Next:** Dataset Design

**Repository rule:** All future concepts, experiments, failures, bottlenecks, improvements, and conclusions will be added to the documentation as they are discovered.
