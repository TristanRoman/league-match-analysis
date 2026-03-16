# Early-Match Advantage and Win Prediction in Professional League of Legends

**DSC 259R Final Project – UC San Diego**  
Tristan Roman · David Lightfoot · Timothy Chang · Julia Klayman

Professional *League of Legends* is a game defined by momentum. Small advantages in gold, experience, and map control during the opening minutes often cascade into larger strategic advantages that determine the outcome of a match. Analysts and commentators frequently claim that games are “decided early,” but this statement is usually made qualitatively rather than supported with systematic evidence.

Professional League of Legends is also one of the most widely viewed esports in the world, with major tournaments drawing millions of concurrent viewers. Because of this massive audience, fans, analysts, and commentators are constantly trying to understand which factors most strongly influence whether a team wins or loses. By analyzing early-match statistics from professional games, our project investigates whether advantages accumulated during the first ten minutes — such as gold, experience, combat efficiency, and objective control — already contain predictive information about eventual match outcomes.

This project investigates that claim quantitatively using professional esports data. Our goal is to determine whether information available early in a match — such as gold, experience, farming efficiency, combat activity, and objective control — already contains enough signal to predict which team will ultimately win.

---

## Table of Contents

- [Introduction](#introduction)
- [Data Cleaning and Exploratory Data Analysis](#data-cleaning-and-exploratory-data-analysis)
- [Assessment of Missingness](#assessment-of-missingness)
- [Hypothesis Testing](#hypothesis-testing)
- [Framing a Prediction Problem](#framing-a-prediction-problem)
- [Baseline Model](#baseline-model)
- [Final Model](#final-model)
- [Fairness Analysis](#fairness-analysis)
- [Conclusion](#conclusion)

---

## Introduction

Our central research question is:

**To what extent can the early state of a professional League of Legends match explain or predict its eventual outcome?**

To investigate this question, we use the **2022 Oracle’s Elixir professional esports dataset**, which contains detailed records of professional matches played across major global leagues.

The dataset contains **144,780 rows**. Because the raw dataset includes both player-level and team-level rows, we transform the data so that each observation in our modeling table represents **one team in one match**, allowing us to frame the problem as a prediction task centered on team victory.

The response variable is **`result`**, where:

- 1 = win
- 0 = loss

Key early-match statistics include:

- `goldat10`
- `xpat10`
- `csat10`
- `killsat10`
- `assistsat10`
- `deathsat10`
- `golddiffat10`
- `xpdiffat10`
- `csdiffat10`

We also incorporate contextual signals including:

- early objective control (`firstblood`, `firstdragon`, `firstherald`, `firsttower`)
- champion composition for each role

Our analysis progresses through three stages:

1. Exploratory analysis to understand early-match patterns
2. Hypothesis testing to evaluate statistical relationships suggested by those patterns
3. Predictive modeling to quantify how well early-match information predicts match outcomes

This progression allows us to connect descriptive statistics, inferential testing, and machine learning into a single coherent analysis.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The Oracle’s Elixir dataset contains mixed row types including player rows, team rows, and incomplete matches. Because our prediction task focuses on **team outcomes**, the first step was transforming the dataset into a consistent team-level structure.

We filtered the dataset to matches played in **2022** and retained only rows where `datacompleteness == "complete"`.

This removed partially recorded matches that could distort later analysis.

Next, we restricted the dataset to the five competitive roles:

- top
- jungle
- mid
- bot
- support

Player rows were then aggregated into team-level observations, ensuring that each match contributes exactly two rows, one per team.

To verify that the cleaning and aggregation steps produced the intended team-level structure, we examine the first few rows of the cleaned dataset.

| Game ID | Side | Result | Gold @10 | XP @10 | CS @10 | Kills @10 | Assists @10 | Deaths @10 | Gold Diff @10 | Bot | Jungle | Mid | Support | Top | First Blood | First Dragon | First Herald | First Baron | First Tower |
|--------|------|-------|---------|-------|------|---------|-----------|----------|--------------|------|--------|-----|--------|-----|------------|-------------|-------------|------------|------------|
| ESPORTSTMNT01_2690210 | Blue | 0 | 16218 | 18213 | 322 | 3 | 5 | 0 | 1523 | Samira | Xin Zhao | LeBlanc | Leona | Renekton | 1 | 0 | 1 | 0 | 1 |
| ESPORTSTMNT01_2690210 | Red | 1 | 14695 | 18076 | 330 | 0 | 0 | 3 | -1523 | Jinx | Viego | Viktor | Alistar | Gragas | 0 | 1 | 0 | 0 | 0 |
| ESPORTSTMNT01_2690219 | Blue | 0 | 14939 | 17462 | 317 | 1 | 1 | 3 | -1619 | Jhin | Lee Sin | Orianna | Rakan | Gragas | 0 | 0 | 1 | 0 | 0 |
| ESPORTSTMNT01_2690219 | Red | 1 | 16558 | 19048 | 344 | 3 | 3 | 1 | 1619 | Syndra | Nidalee | Renekton | Leona | Gangplank | 1 | 1 | 0 | 1 | 1 |
| ESPORTSTMNT01_2690227 | Blue | 1 | 15466 | 19600 | 368 | 0 | 0 | 1 | -103 | Aphelios | Talon | Zoe | Yuumi | Renekton | 0 | 1 | 0 | 1 | 1 |

To capture additional structure in early gameplay, we engineered several derived features:

- `kda10` — early combat efficiency
- `gold_per_cs10` — economic efficiency
- `combat_intensity10` — overall skirmish activity

We also incorporated additional signals describing objective control and champion composition.

These features capture both mechanical performance and strategic context during the opening stages of the match.

### Univariate Analysis

We first examined the distribution of gold difference at 10 minutes (`golddiffat10`), a widely cited indicator of early advantage.

<iframe src="assets/gold_diff_distribution.html" width="100%" height="500"></iframe>

The distribution shows substantial variation in early economic states across matches, indicating that teams reach the 10-minute mark with a wide range of competitive positions.

We also examined the distribution of early combat efficiency (`kda10`).

<iframe src="assets/kda_distribution.html" width="100%" height="500"></iframe>

This visualization highlights the variability in teams’ ability to convert early fights into favorable outcomes.

### Bivariate Analysis

Next, we examined how early-match statistics relate to match outcomes.

<iframe src="assets/gold_vs_result.html" width="100%" height="500"></iframe>

Teams that eventually win tend to hold stronger gold positions at the 10-minute mark. This visual separation suggests that early economic advantage may be strongly associated with match outcomes.

We also examined the most frequently played botlane champions.

<iframe src="assets/botlane_top10.html" width="100%" height="500"></iframe>

This visualization highlights the competitive meta and helped identify champions with sufficient sample sizes for hypothesis testing.

### Interesting Aggregates

To summarize early-match performance numerically, we grouped observations by team side and computed average statistics.

| Side | Avg Gold @10 | Avg XP @10 | Avg CS @10 | Avg Gold Diff @10 | Avg KDA10 |
|------|-------------:|-----------:|-----------:|------------------:|----------:|
| Blue | 15,736.54 | 18,222.02 | 315.36 | 85.58 | 2.21 |
| Red  | 15,650.96 | 18,198.40 | 315.01 | -85.58 | 2.10 |

These aggregates complement the visual analysis by providing concise numerical summaries of early-game performance.

---

## Assessment of Missingness

We analyzed missingness using the raw dataset, since filtering to complete matches would hide the underlying missingness structure.

A plausible candidate for missingness is the later-game variable **`goldat25`**, which may be unavailable when matches end early.

First, we tested whether missingness depends on match outcome.

<iframe src="assets/missingness_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a **p-value of 1.0**, so we fail to reject the null hypothesis that missingness is independent of match outcome.

Next, we tested whether missingness depends on game length.

<iframe src="assets/missingness_gamelength_perm_test.html" width="100%" height="500"></iframe>

This test produced a **p-value < 0.001**, providing strong evidence that missingness depends on match duration.

Together, these results suggest the missingness of `goldat25` is **not Missing Completely At Random (MCAR)** and is more consistent with a **Missing At Random (MAR)** mechanism.

---

## Hypothesis Testing

### Hypothesis 1 — Champion Impact

We tested whether the botlane champion **Aphelios** has a win rate different from other botlane champions.

**Null hypothesis:** Aphelios’s win rate equals the pooled win rate of other botlane champions.

**Alternative hypothesis:** Aphelios’s win rate differs from that of other botlane champions.

We chose this test because Aphelios appeared frequently in the cleaned botlane data, which made him a natural candidate for a champion-specific question. Since the comparison is between one champion and the rest of the botlane pool, we used the **absolute difference in win rates** as the test statistic. That statistic directly measures how far Aphelios’s observed performance is from the rest of the field. We use a **significance level of 0.05**, which is a conventional threshold in statistical analysis that balances the risk of false positives with the ability to detect meaningful effects in the data.

<iframe src="assets/aphelios_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a **p-value of 0.3697**, so we fail to reject the null hypothesis. This suggests that champion selection alone does not strongly explain match outcomes.

### Hypothesis 2 — Early Gold Advantage

We tested whether teams with above-median gold difference at 10 minutes win more often.

**Null hypothesis:** Teams with above-median `golddiffat10` do not have higher win rates.

**Alternative hypothesis:** Teams with above-median `golddiffat10` win more often.

We use the **difference in win rate** as the test statistic because it directly measures whether teams with stronger early gold positions win at a meaningfully different rate than teams without that advantage. This statistic is appropriate for our question since we are asking whether early economic position is associated with eventual victory. We again use a **significance level of 0.05** as the threshold for statistical significance.

<iframe src="assets/gold_diff_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a **p-value < 0.001**, providing strong evidence that early gold advantage is associated with match victory.

---

## Framing a Prediction Problem

We frame the modeling task as **binary classification**.

Each observation represents **one team in one match**, and the model predicts the variable **`result`**. We chose `result` because the central goal of the project is to determine whether early-match information can predict whether a team ultimately wins or loses.

To avoid leakage, we split the dataset by **game ID**, ensuring that both teams from the same match appear in the same training or testing partition.

Model performance is evaluated using:

- **Accuracy**
- **ROC-AUC**

We report accuracy because it provides a simple and interpretable measure of the proportion of match outcomes predicted correctly. We also report ROC-AUC because it captures how well the model separates wins from losses across classification thresholds. Since the data are not extremely imbalanced, accuracy is a reasonable primary metric, while ROC-AUC provides an additional view of ranking performance.

---

## Baseline Model

Our baseline model is a logistic regression classifier trained using only raw 10-minute team statistics.

Features:

- `goldat10`
- `xpat10`
- `csat10`
- `killsat10`
- `assistsat10`
- `deathsat10`
- `golddiffat10`
- `xpdiffat10`
- `csdiffat10`

The baseline feature set contains **9 quantitative variables**, representing early-game numeric statistics such as gold, experience, farming, and combat participation. No ordinal variables are used in this baseline model, and no nominal variables are included because the baseline intentionally restricts the feature set to raw numeric early-match statistics.

Baseline performance:

- **Accuracy:** 0.7064
- **ROC-AUC:** 0.7641

Even this simple early-match snapshot predicts roughly 70% of match outcomes, indicating substantial predictive signal in early gameplay.

---

## Final Model

The final model expands the baseline using three additional sources of information.

**Engineered features**

- `kda10`
- `gold_per_cs10`
- `combat_intensity10`

**Objective control**

- `firstblood`
- `firstdragon`
- `firstherald`
- `firstbaron`
- `firsttower`

**Champion composition**

- `top_champ`
- `jng_champ`
- `mid_champ`
- `bot_champ`
- `sup_champ`

These added features are useful because they better reflect the way early advantages are generated and converted in professional play. Combat-efficiency features summarize how well teams turn skirmishes into favorable outcomes, objective-control indicators reflect whether early pressure becomes map-wide strategic advantage, and champion composition captures draft structure that can shape how teams create and use early leads. From the perspective of the game’s data-generating process, these features provide a richer representation of how early-match state develops into eventual victory.

The modeling algorithm used for the final model is **logistic regression**. We chose logistic regression because it is well suited for binary classification, produces interpretable coefficients, and provides a strong benchmark for understanding how early-game variables contribute to win probability.

Best hyperparameters identified through GridSearchCV:

- `C = 0.1`
- `solver = lbfgs`

Final model performance:

- **Accuracy:** 0.8396
- **ROC-AUC:** 0.9061

This represents a substantial improvement over the baseline model.

---

## Fairness Analysis

We evaluated whether model performance differs between **blue side** and **red side** teams.

Observed accuracies:

- Blue side: **0.8329**
- Red side: **0.8462**

Difference in accuracy:

- **−0.0133**

For the fairness analysis, the evaluation metric is **classification accuracy**, measured separately for blue-side and red-side teams. The **null hypothesis** states that the model performs equally well for both groups, meaning any difference in accuracy is due to random variation. The **alternative hypothesis** states that the model performs differently between the two groups.

We use the **difference in accuracy between the two sides** as the test statistic and perform a permutation test to determine whether the observed difference is statistically significant. The significance level is set to **0.05**.

<iframe src="assets/fairness_perm_test.html" width="100%" height="500"></iframe>

A permutation test produced a **p-value of 0.2798**, so we fail to reject the null hypothesis that performance differs between sides. The model therefore appears reasonably balanced across blue and red side teams.

---

## Conclusion

Our analysis demonstrates that the early state of a professional League of Legends match contains substantial predictive information about its eventual outcome. Even a simple baseline model trained only on raw 10-minute statistics achieves strong performance, indicating that early economic and combat signals already encode meaningful structure.

However, the strongest predictive performance emerges when we incorporate additional strategic context. Engineered summaries of combat efficiency, indicators of early objective control, and champion composition together provide a richer representation of early-game dynamics.

These results reinforce a widely held competitive intuition. Victories in professional League of Legends rarely emerge from a single decisive moment late in the match. **Instead, they typically arise from a sequence of advantages that begin much earlier, defined by the strategic accumulation of gold, experience, and map control during the opening minutes of the game.**