# ExoDetect: Machine Learning Prioritization of Kepler Exoplanet Candidates

## Overview

**ExoDetect** is a machine learning project focused on the scientific prioritization of unresolved Kepler exoplanet candidates.

The project uses tabular data from the Kepler Objects of Interest catalogue to learn the difference between **confirmed exoplanets** and **false positive signals**. After training and evaluating several machine learning models, the best-performing model is applied to unresolved **candidate** objects in order to rank them by their probability of being planet-like.

The goal of this project is not to officially confirm new planets. Instead, it builds a careful and interpretable workflow that can help identify which Kepler candidates may be more interesting for further scientific investigation.

This project was developed as part of my final Machine Learning / AI project at ReDI School.

---

## Project Motivation

The Kepler Space Telescope discovered thousands of planet-like transit signals by measuring small drops in the brightness of stars. However, not every transit-like signal is caused by a real planet. Some signals can come from eclipsing binary stars, background objects, stellar variability, or instrumental noise.

Because of this, exoplanet discovery is not only a detection problem. It is also a prioritization problem.

Astronomers and researchers often need to decide which candidates are worth deeper follow-up analysis. Machine learning can support this process by learning patterns from already classified objects and applying those patterns to unresolved candidates.

The main question behind this project is:

> Can a machine learning model trained on known Kepler objects help prioritize unresolved exoplanet candidates in a scientifically meaningful way?

---

## Dataset

The project uses the **Kepler Objects of Interest cumulative table** from the NASA Exoplanet Archive.

The dataset contains:

* `CONFIRMED` objects: Kepler objects already classified as confirmed exoplanets
* `FALSE POSITIVE` objects: objects classified as non-planet signals
* `CANDIDATE` objects: unresolved objects that still need further investigation

In the notebook, the dataset contains:

| Data group                                 | Number of objects |
| ------------------------------------------ | ----------------: |
| Full KOI dataset                           |             9,564 |
| Confirmed objects                          |             2,747 |
| False positives                            |             4,839 |
| Unresolved candidates                      |             1,978 |
| Training data used for supervised learning |             7,586 |

The supervised model is trained only on objects with known labels: `CONFIRMED` and `FALSE POSITIVE`.

The unresolved `CANDIDATE` objects are kept separate and are used only after model evaluation, for candidate prioritization.

---

## Selected Features

The model uses physically meaningful transit, planetary, and stellar features, including:

| Feature         | Meaning                       |
| --------------- | ----------------------------- |
| `koi_period`    | Orbital period                |
| `koi_time0bk`   | Transit epoch                 |
| `koi_impact`    | Transit impact parameter      |
| `koi_duration`  | Transit duration              |
| `koi_depth`     | Transit depth                 |
| `koi_prad`      | Estimated planet radius       |
| `koi_teq`       | Equilibrium temperature       |
| `koi_insol`     | Insolation flux               |
| `koi_model_snr` | Transit signal-to-noise ratio |
| `koi_steff`     | Stellar effective temperature |
| `koi_slogg`     | Stellar surface gravity       |
| `koi_srad`      | Stellar radius                |
| `koi_kepmag`    | Kepler magnitude              |

A key part of the project is **leakage prevention**. I intentionally excluded columns that are too close to the final classification label, such as:

* `koi_score`
* `koi_pdisposition`
* false-positive flag columns
* object IDs and name columns

This makes the project more reliable because the model is not simply learning from columns that already contain the answer indirectly.

---

## Machine Learning Task

The project has two connected parts.

### 1. Binary Classification

The first task is to train a model that separates:

* `CONFIRMED` exoplanets
* `FALSE POSITIVE` signals

This is a supervised learning problem.

### 2. Candidate Prioritization

After the model is evaluated, it is applied to unresolved `CANDIDATE` objects.

Each candidate receives a predicted planet-like probability. These probabilities are then used to create a ranked list of candidates that may be more interesting for follow-up study.

---

## Workflow

The notebook follows a full machine learning workflow:

1. Load the Kepler Objects of Interest dataset
2. Explore the target labels and selected physical features
3. Remove leakage-related columns
4. Split known objects into train and test sets
5. Build preprocessing and modeling pipelines
6. Train multiple models
7. Compare model performance
8. Use cross-validation for stability checking
9. Test hyperparameter tuning
10. Select the final model
11. Evaluate probability-based performance
12. Interpret feature importance
13. Apply the model to unresolved candidates
14. Create candidate priority groups
15. Add unsupervised similarity checks
16. Compare candidates with selected reference planets
17. Build consensus candidate shortlists
18. Create a custom scientific priority score
19. Test ranking stability under different weighting choices

---

## Models Tested

Several models were trained and compared:

* Logistic Regression
* Random Forest
* Gradient Boosting
* Support Vector Machine with RBF kernel

The purpose of testing multiple models was to avoid depending on only one modeling approach. Logistic Regression was used as a simpler baseline, while tree-based and nonlinear models were tested for stronger predictive performance.

---

## Final Model

The best-performing model was:

## Random Forest Classifier

The Random Forest model achieved the strongest overall performance on the test set, especially when considering macro F1-score and ROC-AUC.

| Metric          | Score |
| --------------- | ----: |
| Accuracy        | 0.934 |
| Macro Precision | 0.928 |
| Macro Recall    | 0.930 |
| Macro F1-score  | 0.929 |
| ROC-AUC         | 0.978 |

Macro F1-score was especially important because the dataset is imbalanced. It gives balanced attention to both confirmed planets and false positives.

ROC-AUC was also important because the final project goal depends on probability ranking, not only hard classification.

---

## Model Interpretation

Feature importance analysis showed that the model mainly relied on physically meaningful features.

The most important features included:

1. `koi_prad` — estimated planet radius
2. `koi_impact` — transit impact parameter
3. `koi_model_snr` — transit signal-to-noise ratio
4. `koi_duration` — transit duration
5. `koi_period` — orbital period

This result is scientifically reasonable because these features describe the shape, strength, geometry, and physical interpretation of transit signals.

Both built-in Random Forest importance and permutation importance were used. The two methods produced similar but not identical rankings, which is expected because some features are correlated or contain overlapping physical information.

---

## Candidate Prioritization

After selecting the final model, I applied it to the unresolved Kepler `CANDIDATE` objects.

Each candidate received a predicted probability of being planet-like. Based on this probability, candidates were grouped into three priority levels:

| Priority group  | Number of candidates |
| --------------- | -------------------: |
| High Priority   |                   58 |
| Medium Priority |                  214 |
| Low Priority    |                1,706 |

Examples of highly ranked candidates by model probability include:

| Candidate | Predicted probability |           Radius |  SNR | Priority |
| --------- | --------------------: | ---------------: | ---: | -------- |
| K01145.01 |                 99.3% | 2.32 Earth radii | 33.1 | High     |
| K00426.01 |                 98.7% | 3.84 Earth radii | 62.4 | High     |
| K01972.01 |                 98.7% | 2.88 Earth radii | 39.8 | High     |
| K01861.01 |                 98.7% | 3.56 Earth radii | 47.5 | High     |
| K04526.01 |                 98.3% | 2.72 Earth radii | 15.0 | High     |

These candidates are not confirmed planets. They are model-prioritized candidates that may be worth further investigation.

---

## Scientific Priority Score

In addition to probability ranking, I created a custom scientific priority score.

The score rewards:

* high predicted planet probability
* stronger transit signal-to-noise ratio
* smaller estimated planet radius

The main scoring formula is:

```python
scientific_priority_score =
    0.60 * planet_probability
    + 0.20 * normalized_signal_to_noise
    + 0.20 * smaller_radius_score
```

This score is not an official astronomical validation metric. It is a practical heuristic designed to support candidate ranking.

Top candidates under this scientific priority score included:

| Candidate | Predicted probability | Scientific priority score | Radius |  SNR |
| --------- | --------------------: | ------------------------: | -----: | ---: |
| K04032.05 |                 93.7% |                     0.674 |   0.82 | 15.2 |
| K04452.01 |                 96.0% |                     0.668 |   1.23 | 14.2 |
| K02159.01 |                 95.7% |                     0.664 |   1.32 | 24.5 |
| K01145.01 |                 99.3% |                     0.661 |   2.32 | 33.1 |
| K00935.04 |                 97.7% |                     0.657 |   1.94 | 18.2 |

This step adds a scientific-interest layer beyond pure classification probability.

---

## Ranking Stability Check

Because the scientific priority score uses custom weights, I tested different weighting scenarios:

* 60% model probability, 20% signal-to-noise, 20% radius score
* 70% model probability, 15% signal-to-noise, 15% radius score
* 80% model probability, 10% signal-to-noise, 10% radius score

This helped test whether the highest-ranked candidates were stable or strongly dependent on one arbitrary scoring formula.

Some candidates remained near the top across multiple scoring scenarios, which makes them more robust and interesting as follow-up candidates.

---

## Unsupervised Similarity Check

I also added an unsupervised learning section as an exploratory validation step.

The purpose was not to predict labels directly, but to check whether unresolved candidates naturally cluster with known confirmed planets or known false positives.

Methods tested included:

* KMeans clustering
* Agglomerative clustering
* Gaussian Mixture Models

Most candidate objects were grouped into the cluster that was more similar to confirmed objects. However, the unsupervised clustering scores were modest, so this part should be interpreted as exploratory support rather than strong validation.

---

## Reference Planet Similarity Check

To add another scientific perspective, I compared unresolved candidates with selected known and interesting Kepler-related reference planets.

The reference comparison included:

* Kepler-186f
* Kepler-1649c
* KOI-5Ab

The goal was to ask:

> Which unresolved candidates are not only highly ranked by the model, but also physically similar to selected known exoplanets?

This was done using distance-based similarity over selected physical features.

This section does not prove habitability and does not confirm candidates. It only adds a reference-based similarity view to the ranking.

---

## Consensus Candidates

A final consensus step combined two independent ideas:

1. High model-based planet probability
2. High similarity to selected reference planets

This produced a stronger shortlist of candidates supported by both supervised learning and reference-based physical similarity.

The project found:

| Consensus group              | Number of candidates |
| ---------------------------- | -------------------: |
| Strong Consensus Candidate   |                   15 |
| Moderate Consensus Candidate |                   50 |
| Lower Consensus              |                1,913 |

These consensus candidates are the most interesting outputs of the project because they are supported by more than one ranking method.

---

## Main Results

The main results of this project are:

* A leakage-aware machine learning pipeline for Kepler exoplanet classification
* A Random Forest model with strong test performance
* A probability-based ranking of unresolved Kepler candidates
* A priority grouping system for candidate screening
* Feature-importance interpretation connected to physical transit features
* A custom scientific priority score
* Ranking stability checks under different score weights
* Unsupervised similarity analysis
* Reference planet similarity comparison
* A final consensus candidate shortlist

The project goes beyond a standard classification notebook by turning model predictions into a decision-support workflow for scientific prioritization.

---

## Important Limitations

This project does not officially confirm any new exoplanets.

The model is trained on tabular Kepler Object of Interest features. It does not use raw light curves, centroid analysis, astrophysical false-positive validation, or follow-up observation data.

Therefore, the output should be interpreted as:

> machine-learning-based candidate prioritization, not astronomical confirmation.

The results can help identify interesting candidates for further investigation, but they cannot replace professional astrophysical validation.

---

## Future Work

This project could be extended in several directions:

### 1. Raw Light Curve Analysis

Future versions could use Kepler light curves directly instead of only tabular features. This would allow the model to learn transit shape and signal quality more deeply.

### 2. External Validation with TESS

The ranking could be compared with candidates or confirmed planets from TESS or newer exoplanet catalogues.

### 3. SHAP Explainability

SHAP values could be added to explain individual candidate predictions and show why the model gives a high or low probability to each object.

### 4. More Detailed Habitability Filters

Future work could include more careful habitability-related filters, such as stellar luminosity, orbital distance, equilibrium temperature, and planet radius ranges.

### 5. Follow-up Candidate Reports

The top-ranked candidates could be exported into a structured report with physical parameters, model probability, similarity score, and scientific priority score.

---

## How to Run the Notebook

### Requirements

The notebook uses Python and the following main libraries:

```python
pandas
numpy
matplotlib
seaborn
scikit-learn
```

### Installation

```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

### Running

Open the notebook:

```bash
jupyter notebook ExoDetect_Redi_Final_Project_Oladzadeh.ipynb
```

or use JupyterLab:

```bash
jupyter lab
```

The notebook loads the Kepler dataset directly from the NASA Exoplanet Archive URL, so an internet connection is required when running it from the beginning.

---

## Repository Structure

```text
.
├── ExoDetect_Redi_Final_Project_Oladzadeh.ipynb
└── README.md
```

---

## Project Keywords

`machine-learning`
`exoplanets`
`kepler`
`astronomy`
`candidate-prioritization`
`random-forest`
`classification`
`model-interpretability`
`scientific-computing`
`data-science`

---

## Author

**Dalileh Oladzadeh AbbasAbadi**

Mathematics graduate with interests in machine learning, dynamical systems, geometry, and scientific data analysis.

This project reflects my interest in connecting mathematical thinking, interpretable machine learning, and real scientific datasets.

---

## Acknowledgement

This project was developed as a final project for the ReDI School Machine Learning / AI course.

The dataset is based on the Kepler Objects of Interest table from the NASA Exoplanet Archive.

---

## Final Note

ExoDetect is designed as a research-inspired educational project. It does not claim the discovery or confirmation of new planets. Its value is in showing how machine learning can be used responsibly as a scientific decision-support tool: not replacing domain experts, but helping focus attention on the most promising candidates.
