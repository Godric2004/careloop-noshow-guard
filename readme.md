# CareLoop Clinics — No-Show Guard

**GitHub Repository:** <https://github.com/Godric2004/careloop-noshow-guard/tree/main>

## Project Summary
* **Part 1 (Data Foundations):** Cleaned 1,020 raw booking records down to 990 valid appointments (218 no-shows, 22.0% overall no-show rate) and enriched all 8 clinics with live Open-Meteo temperature data.
* **Part 2 (SQL Analytics):** Verified database counts in MySQL and found that no-shows climb sharply with booking lead time—from 16.2% (1–7 days) to 60.5% (31+ days)—and peak at Gomti Nagar Care Point (33.3%) and Hadapsar Care Centre (27.3%).
* **Part 3 (Predictive Modeling):** Trained and cross-validated four classifiers on evening-before booking features, selecting Logistic Regression (CV ROC-AUC 0.7737, test ROC-AUC 0.7889) at a 0.35 probability threshold to flag 49 high-risk test appointments.
* **Part 4 (GenAI Follow-Up Review):** Extracted structured reasons from 10 call-centre notes using Gemini 3.5 Flash with Pydantic validation, finding that 2 of 2 `forgot_or_no_reminder` cases occurred when no reminder was sent (`reminder_sent = No`).

## Recommendation
Appointments booked far in advance—especially those with lead times over 14 days (30.0% no-show rate for 15–30 days and 60.5% for 31+ days)—patients with prior no-shows, and bookings where no reminder was sent are most likely to be missed, particularly at high-rate clinics like Hadapsar Care Centre (27.3%) and Gomti Nagar Care Point (33.3%). Call-centre notes confirm that missed appointments stem from preventable forgetfulness or missing reminders (20% of reviewed notes, 100% of which had `reminder_sent = No`), travel/distance friction (20%), and work schedule conflicts (20%). CareLoop should first ensure automated reminders are delivered to 100% of scheduled appointments (since reminders are associated with 69.1% lower no-show odds in our model) and deploy the Logistic Regression risk model at a 0.35 threshold each evening so front-desk staff prioritize confirmation calls to the ~25% riskiest next-day bookings—starting with appointments booked more than 14 days ahead—offering easy rescheduling or tele-consultation options.

## Model Choice & Threshold Justification
* **Selected Model:** **Logistic Regression** (wrapped in a `StandardScaler` + `LogisticRegression` pipeline) was chosen strictly from training data because it achieved the highest 5-fold cross-validated ROC-AUC (**0.7737**) compared to the tuned Random Forest (0.7673), untuned Random Forest (0.7466), and depth-3 Decision Tree (0.6784). On the held-out test set, it achieved an ROC-AUC of **0.7889**.
* **Selected Threshold (`0.35`):** Evaluated on out-of-fold training probabilities, lowering the decision threshold from `0.50` to **`0.35`** increased F1 score from `0.3471` to `0.4892` and nearly doubled recall from `0.2485` to `0.4675` (achieving `0.5714` precision and `0.5714` recall on the test set). High recall is prioritized operationally because missing a no-show costs an empty doctor slot, whereas an extra confirmation phone call costs only a minute of front-desk time.

## Limitations
1. **Small qualitative sample:** The 10 analysed call-centre notes serve as an operational illustration and are too small a sample to represent clinic-wide no-show reasons statistically.
2. **Observational data:** Because booking and reminder records are observational rather than from a randomized experiment, model coefficients (such as lower no-show odds when reminders are sent) reflect statistical associations rather than proven causal effects.
3. **Random split and pre-split imputation:** Using a random train/test split (instead of a past-to-future temporal split) allows the same patient to appear in both sets, and computing imputation medians and IQR outlier fences across all 990 rows prior to splitting introduced minor indirect leakage from test rows into preprocessing.
