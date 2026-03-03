
# Early-Game Advantage and Win Prediction in Professional League of Legends

**Tristan Roman - David Lightfoot - Timothy Chang - Julia Klayman**

---

## Introduction

This project analyzes 2022 professional League of Legends matches to determine whether early-game performance metrics at the 10-minute mark can predict match outcomes.

---

## Data Cleaning and Exploratory Data Analysis

We filtered to complete matches and aggregated player-level data into team-level early-game statistics. Our focus was on gold, experience, CS, and combat statistics at 10 minutes.

---

## Assessment of Missingness

We examined missingness patterns in early-game features and confirmed that modeling variables did not exhibit non-trivial missingness that would bias our results.

---

## Hypothesis Testing

**Hypothesis 1: Champion Impact**

Null: Aphelios’ botlane winrate does not significantly differ from other botlane champions.  
Result: p ≈ 0.18 → fail to reject the null.

**Hypothesis 2: Early Gold Advantage**

Null: Teams with above-median gold difference at 10 minutes do not have higher winrates.  
Result: Statistically significant → early gold advantage strongly predicts win probability.

---

## Framing a Prediction Problem

We simulate predicting match outcome at the 10-minute mark using only information available at that time.

This is a binary classification problem where the response variable is `result`.

Each observation represents one team in one match.

---

## Baseline Model

We trained a logistic regression model using early-game team statistics.

Accuracy ≈ 71%  
ROC-AUC ≈ 0.78  

These results indicate strong predictive power from early-game performance.

---

## Final Model

We engineered additional features and performed hyperparameter tuning using GridSearchCV.

Performance remained stable, suggesting early gold and experience differentials capture most predictive signal.

---

## Fairness Analysis

We will evaluate whether predictive performance differs between Blue and Red side teams using a permutation-based fairness analysis.
