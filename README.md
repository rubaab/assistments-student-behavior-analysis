# Does Student Behavior Predict Next-Problem Success? An Analysis of ASSISTments Interaction Logs

**Author:** Rabab Mushtaq
**Data:** ASSISTments 2009-2010 skill builder dataset
**Tools:** Python, pandas, NumPy, matplotlib, scikit-learn

## Motivation

Online learning platforms record what students do: how long they take, whether they ask for hints, how many attempts they make. This project asks how much those traces tell us about what happens next, and where they fall short. The broader interest is in how learners monitor their own understanding (metacognition) and how AI-supported feedback could help.

## Research question

Do a student's recent behaviors (hint use, number of attempts, response time, very fast responses) predict whether they answer the next problem correctly, beyond simply knowing whether the previous answer was correct?

## Data

- **Source:** ASSISTments 2009-2010 skill builder data. (https://sites.google.com/site/assistmentsdata/home/2009-2010-assistment-data/skill-builder-data-2009-2010)
- **Full file:** 525,534 records, 30 columns.
- **After keeping original problems only** (`original == 1`, which removes scaffolding sub-questions): 449,220 records.
- **After cleaning** (removing missing values and non-positive response times): 444,536 records from 4,217 students.
- **Modeling set** (first problem per student dropped because it has no previous problem): 440,319 records.

## Method

1. **Filter and clean.** Keep original problems, drop missing values, compute response time in seconds, and flag responses under 2 seconds as "fast".
2. **Order interactions** within each student by `order_id`.
3. **Build features from the previous problem:** whether it was correct, hints used, attempts, whether the response was fast, and log response time.
4. **Model.** Logistic regression predicting whether the next problem is correct.
   - **Baseline:** previous correctness only.
   - **Behavior model:** previous correctness plus the four behavior features.
5. **Evaluate.** AUC on held-out students. Splits are made **by student** (`GroupShuffleSplit`), so no student appears in both training and test data. The split was repeated with five random seeds.

## Results

| Measure | Value |
|---|---|
| Mean AUC, baseline (5 splits) | 0.765 (SD 0.011) |
| Mean AUC, with behavior (5 splits) | 0.773 (SD 0.014) |
| Gain per split | +0.009, +0.008, +0.018, -0.001, +0.010 |
| Share of fast responses (< 2 s) | 3.8% |
| Accuracy, fast responses | 59.9% |
| Accuracy, normal-speed responses | 69.7% |

**Interpretation**

- Behavioral features gave a small improvement over the baseline in four of five splits. The gain is small relative to the variation between splits, so it should not be described as significant.
- More hints on the previous problem were associated with lower odds of next-problem success (coefficient about -0.18 on a single split), but see the limitations on how hints and correctness are linked.
- Fast responses were less accurate on the same problem (59.9% vs. 69.7%), consistent with guessing, but their relationship to next-problem success was unclear once other behavior was included.
- Overall, the logs capture performance much more than the processes behind it.

## Limitations

- **Logs show what students do, not why.** A fast or hint-heavy response could reflect guessing, confidence, or problem difficulty, and the data cannot separate them.
- **Hints and correctness are mechanically linked.** In ASSISTments skill builder data, `correct` indicates first-attempt success without hints, so any problem where a hint was used is coded as incorrect. The hint-accuracy relationship is partly a property of the label, not a finding.
- **Difficulty is not controlled.** Problem and skill difficulty may explain part of the fast-response pattern.
- **Few splits.** Five splits are enough to see the direction of the effect but not to claim statistical significance.
- **Fast-response coefficient is hard to interpret.** `prev_fast` and `prev_log_rt` overlap in information, so their coefficients should not be read separately.
- **Single platform, subject, and school year** (math, one dataset).
- **Association, not causation.**
- **Plot note:** in the response-time histogram, responses of 60 seconds or more are clipped into the last bin.

## Possible next steps

- Control for problem and skill difficulty (for example with skill or problem fixed effects).
- Use richer sequence features (rolling windows over several previous problems) instead of only the previous one.
- Compare with a nonlinear model (random forest or gradient boosting) and test significance across more splits.
- Connect behavioral patterns to theory on self-regulated learning and metacognition, for example by asking which behaviors might indicate poor monitoring and how that could be tested with additional data.

## How to run

1. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib scikit-learn
   ```
2. Download the ASSISTments 2009-2010 skill builder file and place `skill_builder_data.csv` in the same folder as the notebook.
3. Run the notebook cells in order: load and filter, clean, explore, build features, model, repeat splits.

## Files

- `analysis.ipynb`: full analysis notebook
- `hints.png`, `rt.png`: exploratory plots
- `README.md`: this file

## Reference

   ASSISTments 2009-2010 skill builder data. Retrieved from
   https://sites.google.com/site/assistmentsdata/home/2009-2010-assistment-data/skill-builder-data-2009-2010
