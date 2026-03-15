Early-Match Advantage and Win Prediction in Professional League of Legends

DSC 259R Final Project – UC San Diego
Tristan Roman · David Lightfoot · Timothy Chang · Julia Klayman

Professional League of Legends is a game defined by momentum. Small advantages in gold, experience, and map control during the opening minutes often cascade into larger strategic advantages that determine the outcome of a match. Analysts and commentators frequently claim that games are “decided early,” but this statement is usually made qualitatively rather than supported with systematic evidence.

This project investigates that claim quantitatively using professional esports data. Our goal is to determine whether information available early in a match — such as gold, experience, farming efficiency, combat activity, and objective control — already contains enough signal to predict which team will ultimately win.

Table of Contents

Introduction

Data Cleaning and Exploratory Data Analysis

Assessment of Missingness

Hypothesis Testing

Framing a Prediction Problem

Baseline Model

Final Model

Fairness Analysis

Conclusion

Introduction

Our central research question is:

To what extent can the early state of a professional League of Legends match explain or predict its eventual outcome?

To investigate this question we use the 2022 Oracle’s Elixir professional esports dataset, which contains detailed records of professional matches played across major global leagues.

The raw dataset includes both player-level and team-level rows. For this project we transform the data so that each observation represents one team in one match, allowing us to frame the analysis as a predictive modeling problem.

The response variable is result, where:

1 = win

0 = loss

Key early-match statistics include:

goldat10

xpat10

csat10

killsat10

assistsat10

deathsat10

golddiffat10

xpdiffat10

csdiffat10

We also incorporate contextual signals including:

early objective control (firstblood, firstdragon, firstherald, firsttower)

champion composition for each role

Our analysis progresses through three stages:

Exploratory analysis to understand early-match patterns

Hypothesis testing to evaluate statistical relationships suggested by those patterns

Predictive modeling to quantify how well early-match information predicts match outcomes

This progression allows us to connect descriptive statistics, inferential testing, and machine learning into a single coherent analysis.

Data Cleaning and Exploratory Data Analysis
Data Cleaning

The Oracle’s Elixir dataset contains mixed row types including player rows, team rows, and incomplete matches. Because our prediction task focuses on team outcomes, the first step was transforming the dataset into a consistent team-level structure.

We filtered the dataset to matches played in 2022 and retained only rows where:

datacompleteness == "complete"

This removed partially recorded matches that could distort later analysis.

Next, we restricted the dataset to the five competitive roles:

top

jungle

mid

bot

support

Player rows were then aggregated into team-level observations, ensuring that each match contributes exactly two rows (one per team).

To capture additional structure in early gameplay, we engineered several derived features:

kda10 — early combat efficiency

gold_per_cs10 — economic efficiency

combat_intensity10 — overall skirmish activity

We also incorporated additional signals describing objective control and champion composition.

These features capture both mechanical performance and strategic context during the opening stages of the match.

Univariate Analysis

We first examined the distribution of gold difference at 10 minutes (golddiffat10), a widely cited indicator of early advantage.

<iframe src="assets/gold_diff_distribution.html" width="100%" height="500"></iframe>

The distribution shows substantial variation in early economic states across matches, indicating that teams reach the 10-minute mark with a wide range of competitive positions.

We also examined the distribution of early combat efficiency (kda10).

<iframe src="assets/kda_distribution.html" width="100%" height="500"></iframe>

This visualization highlights the variability in teams’ ability to convert early fights into favorable outcomes.

Bivariate Analysis

Next we examined how early-match statistics relate to match outcomes.

<iframe src="assets/gold_vs_result.html" width="100%" height="500"></iframe>

Teams that eventually win tend to hold stronger gold positions at the 10-minute mark. This visual separation suggests that early economic advantage may be strongly associated with match outcomes.

We also examined the most frequently played botlane champions.

<iframe src="assets/botlane_top10.html" width="100%" height="500"></iframe>

This visualization highlights the competitive meta and helped identify champions with sufficient sample sizes for hypothesis testing.

Interesting Aggregates

To summarize early-match performance numerically, we grouped observations by team side and computed average statistics.

Side	Avg Gold @10	Avg XP @10	Avg CS @10	Avg Gold Diff @10	Avg KDA10
Blue	(values from notebook)				
Red	(values from notebook)				

These aggregates complement the visual analysis by providing concise numerical summaries of early-game performance.

Assessment of Missingness

We analyzed missingness using the raw dataset, since filtering to complete matches would hide the underlying missingness structure.

A plausible candidate for missingness is the later-game variable goldat25, which may be unavailable when matches end early.

First, we tested whether missingness depends on match outcome.

<iframe src="assets/missingness_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a p-value of 1.0, so we fail to reject the null hypothesis that missingness is independent of match outcome.

Next, we tested whether missingness depends on game length.

<iframe src="assets/missingness_gamelength_perm_test.html" width="100%" height="500"></iframe>

This test produced a p-value < 0.001, providing strong evidence that missingness depends on match duration.

Together these results suggest the missingness of goldat25 is not Missing Completely At Random (MCAR) and is more consistent with a Missing At Random (MAR) mechanism.

Hypothesis Testing
Hypothesis 1 — Champion Impact

We tested whether the botlane champion Aphelios has a win rate different from other botlane champions.

Null hypothesis:
Aphelios’s win rate equals the pooled win rate of other botlane champions.

Alternative hypothesis:
Aphelios’s win rate differs from that of other botlane champions.

<iframe src="assets/aphelios_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a p-value of 0.3697, so we fail to reject the null hypothesis. This suggests that champion selection alone does not strongly explain match outcomes.

Hypothesis 2 — Early Gold Advantage

We tested whether teams with above-median gold difference at 10 minutes win more often.

Null hypothesis:
Teams with above-median golddiffat10 do not have higher win rates.

Alternative hypothesis:
Teams with above-median golddiffat10 win more often.

<iframe src="assets/gold_diff_perm_test.html" width="100%" height="500"></iframe>

The permutation test produced a p-value < 0.001, providing strong evidence that early gold advantage is associated with match victory.

Framing a Prediction Problem

We frame the modeling task as binary classification.

Each observation represents one team in one match, and the model predicts the variable result.

To avoid leakage, we split the dataset by game ID, ensuring that both teams from the same match appear in the same training or testing partition.

Model performance is evaluated using:

Accuracy

ROC-AUC

Baseline Model

Our baseline model is a logistic regression classifier trained using only raw 10-minute team statistics.

Features:

goldat10

xpat10

csat10

killsat10

assistsat10

deathsat10

golddiffat10

xpdiffat10

csdiffat10

Baseline performance:

Accuracy: 0.7064

ROC-AUC: 0.7641

Even this simple early-match snapshot predicts roughly 70% of match outcomes, indicating substantial predictive signal in early gameplay.

Final Model

The final model expands the baseline using three additional sources of information.

Engineered features

kda10

gold_per_cs10

combat_intensity10

Objective control

firstblood

firstdragon

firstherald

firstbaron

firsttower

Champion composition

top_champ

jng_champ

mid_champ

bot_champ

sup_champ

Best hyperparameters identified through GridSearchCV:

C = 0.1

solver = lbfgs

Final model performance:

Accuracy: 0.8396

ROC-AUC: 0.9061

This represents a substantial improvement over the baseline model.

Fairness Analysis

We evaluated whether model performance differs between blue side and red side teams.

Observed accuracies:

Blue side: 0.8329

Red side: 0.8462

Difference in accuracy:

−0.0133

<iframe src="assets/fairness_perm_test.html" width="100%" height="500"></iframe>

A permutation test produced a p-value of 0.2798, so we fail to reject the null hypothesis that performance differs between sides. The model therefore appears reasonably balanced across blue and red side teams.

Conclusion

Our analysis demonstrates that the early state of a professional League of Legends match contains substantial predictive information about its eventual outcome. Even a simple baseline model trained only on raw 10-minute statistics achieves strong performance, indicating that early economic and combat signals already encode meaningful structure.

However, the strongest predictive performance emerges when we incorporate additional strategic context. Engineered summaries of combat efficiency, indicators of early objective control, and champion composition together provide a richer representation of early-game dynamics.

These results reinforce a widely held competitive intuition. Victories in professional League of Legends rarely emerge from a single decisive moment late in the match. Instead, they typically arise from a sequence of advantages that begin much earlier.

In professional League of Legends, victory rarely begins at the final teamfight — it begins in the quiet accumulation of gold, experience, and map control during the opening minutes of the game.