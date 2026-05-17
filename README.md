# dsa210project_beraacar                                                                                                                                                    
# NBA Player Performance & Salary Analysis
**DSA 210 — Introduction to Data Science | Spring 2026**<br> **Student: Bera Acar (33834)**

---

## Motivation

NBA teams spend hundreds of millions of dollars on player salaries every season. Yet it is not always clear whether these salaries reflect actual on-court performance, or whether other factors — such as a player's reputation, age, or past contract history — play a larger role.

This project aims to answer a straightforward but important question: **Do NBA player statistics accurately predict their salaries?** By analyzing the relationship between performance metrics and salary, this project seeks to identify which factors matter most, which players are being overpaid or underpaid relative to their performance, and whether players can be meaningfully grouped into archetypes based on how they play.

---

## Data Source

The dataset used in this project is `NBA_Advanced_Salary_Data.csv`, which contains advanced statistics and salary information for NBA players across two seasons: **2022-23** and **2023-24**.

The dataset includes 902 entries and 34 columns, covering:
- **Salary information** (actual annual salary in USD)
- **Advanced performance metrics**: PER (Player Efficiency Rating), Win Shares (WS), Box Plus/Minus (BPM), Value Over Replacement Player (VORP), and others
- **Usage and efficiency stats**: USG%, TS%, WS/48, AST%, TRB%, BLK%, STL%
- **Basic info**: Age, Games Played (G), Minutes Played (MP), Position, Season

**Preprocessing steps applied:**
- Removed players with missing salary, PER, WS, or BPM values
- Removed players with salary = 0 (inactive/two-way contracts)
- Removed players with fewer than 20 games played to reduce noise from injured or rarely-used players
- Applied log transformation to salary (`log1p`) to normalize the right-skewed salary distribution

After cleaning, **683 player-seasons** remained for analysis, with 16 features used across all models.

---

## Data Analysis

Three machine learning tasks were applied to the dataset, each addressing a different dimension of the research question.

### Task 1: Regression — Salary Prediction

The goal of this task was to predict player salary from performance statistics. Four regression models were trained and compared:

| Model | Test R² | CV R² | MAE |
|---|---|---|---|
| Linear Regression | 0.659 | 0.622 | $4.51M |
| Ridge (α=10) | 0.659 | 0.622 | $4.51M |
| Lasso (α=0.01) | 0.654 | 0.620 | $4.55M |
| Random Forest | 0.630 | 0.630 | $4.78M |

StandardScaler was applied to Linear Regression, Ridge, and Lasso since these models are sensitive to feature scale. Random Forest was not scaled, as tree-based models are scale-invariant.

A log transformation was applied to salary before training, since the salary distribution is highly right-skewed. Predictions were converted back to dollar values using the inverse transformation (`expm1`).

In terms of test R², Ridge Regression achieved the best result (0.659). However, when cross-validation scores are compared, the difference between models is small. Random Forest was preferred for downstream tasks due to its ability to provide feature importance analysis, making the results more interpretable.

The top three most important features identified by the Random Forest model were **MP (Minutes Played)**, **Age**, and **USG% (Usage Rate)**.

Feature engineering was also applied (AgePER, Efficiency, OffDefRatio, PrimeAge), but did not produce meaningful improvements in model performance. This suggests that the existing advanced statistics (PER, WS, BPM, VORP) already capture player performance effectively.

### Task 2: K-Means Clustering — Player Archetypes

K-Means clustering was applied to group players into archetypes based purely on their performance statistics, without using salary information.

StandardScaler was applied before clustering, as K-Means is a distance-based algorithm and is sensitive to feature scale.

The optimal number of clusters was determined using two methods:
- **Elbow Method**: Identifies the point where adding more clusters yields diminishing returns in inertia
- **Silhouette Score**: Measures how well each point fits its assigned cluster vs. neighboring clusters

Both methods agreed on **K = 4** as the optimal number of clusters. The four clusters were labeled based on their average statistics:

| Cluster | Role | Players | Avg Salary |
|---|---|---|---|
| 0 | Role Player | 374 | $6.9M |
| 1 | Rim Protector | 140 | $8.6M |
| 2 | Playmaker | 163 | $22.1M |
| 3 | Scorer | 6 | $29.4M |

Playmakers and Scorers earn significantly more than other archetypes, which aligns with the general understanding that ball-handlers and high-usage offensive players command premium contracts in the NBA.

### Task 3: Classification — Overpaid / Underpaid / Fair

The salary predicted by the Random Forest regression model was compared against each player's actual salary using a ratio:

```
SalaryRatio = Actual Salary / Predicted Salary
```

Players were then labeled using percentile-based thresholds to ensure balanced class representation:
- **Overpaid**: SalaryRatio above the 67th percentile
- **Underpaid**: SalaryRatio below the 33rd percentile
- **Fair**: Between the two thresholds

Two classifiers were trained to predict these labels:

| Model | Accuracy | CV Accuracy |
|---|---|---|
| Random Forest | ~0.42 | ~0.40 |
| Gradient Boosting | ~0.44 | ~0.41 |

Gradient Boosting performed slightly better and was selected as the final classifier.

**Most Overpaid Players (highest SalaryRatio):**

| Player | Season | Actual | Predicted | Ratio |
|---|---|---|---|---|
| Ben Simmons | 2022-23 | $35.4M | $5.9M | 5.97x |
| Duncan Robinson | 2022-23 | $16.9M | $3.4M | 4.99x |
| James Wiseman | 2023-24 | $12.1M | $2.7M | 4.42x |

**Most Underpaid Players (lowest SalaryRatio):**

| Player | Season | Actual | Predicted | Ratio |
|---|---|---|---|---|
| David Duke | 2022-23 | $0.5M | $2.8M | 0.19x |
| Sam Hauser | 2023-24 | $1.9M | $7.6M | 0.25x |
| Immanuel Quickley | 2023-24 | $4.2M | $15.1M | 0.28x |

---

## Findings

1. **Performance explains roughly two-thirds of salary.** With a best R² of 0.659, player statistics account for about 66% of salary variance. The remaining 34% is likely driven by factors outside of on-court performance, such as contract timing, market size, name recognition, and injury history.

2. **Minutes played, age, and usage rate are the strongest predictors of salary.** This suggests that availability and role within a team's offense matter more than pure efficiency metrics.

3. **Player archetypes have a large impact on salary.** Playmakers earn on average $22.1M, more than three times the average salary of Role Players ($6.9M), even when controlling for performance level.

4. **Some players are dramatically over or underpaid relative to their statistics.** Ben Simmons earns nearly 6x what his statistics predict, while players like Sam Hauser and Immanuel Quickley earn a fraction of what their performance would suggest.

5. **Classifying overpaid/underpaid players is inherently difficult.** The classification accuracy of ~44% indicates that the boundary between fair and unfair pay is not cleanly captured by statistics alone, which is consistent with finding #1.

---

## Limitations and Future Work

**Limitations:**

- The dataset covers only two seasons (2022-23 and 2023-24), which limits the ability to capture long-term trends or the effect of salary caps changing over time.
- Salary reflects contract value at signing, not current performance. A player signed to a max contract three years ago may now be overpaid or underpaid relative to their current form, but the model cannot account for this.
- The classification task uses the regression model's own predictions to generate labels, which introduces circularity and limits how much the classifier can learn independently.
- Injuries, team context, and market size are not included in the dataset and likely explain a significant portion of the unexplained salary variance.

**Future Work:**

- Incorporating multiple seasons of data would allow for longitudinal analysis and salary trajectory modeling.
- Adding contract year indicators (e.g., "is this a contract year?") could improve predictions, as players often perform differently when approaching free agency.
- Replacing the circular classification approach with an independently derived performance value metric (e.g., from a separate dataset) would produce a more robust overpaid/underpaid analysis.
- Applying more advanced models such as XGBoost or neural networks could improve regression accuracy.

---

## AI Usage Disclosure

In accordance with the course guidelines, the use of AI tools in this project is disclosed below.

**Tools used:** Google Gemini, Anthropic Claude (claude.ai)

**How they were used:**
- **Code assistance**: Claude was used to help structure the ML pipeline (regression, clustering, classification), debug code, and suggest appropriate sklearn implementations.
- **Conceptual explanations**: Both Gemini and Claude were used to clarify concepts such as log transformation, regularization (Ridge/Lasso), K-Means scaling requirements, and silhouette scoring.
- **Report writing**: Claude assisted in drafting and structuring sections of this report based on the analysis results produced during the project.

All outputs generated by AI tools were reviewed, verified, and adapted by the student. The analysis, interpretation of results, and final decisions regarding methodology were made by the student.
