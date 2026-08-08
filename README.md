# Behavioral Fraud Detection & Risk Analysis

## Overview

Financial fraud is not always defined by a single suspicious transaction. In many cases, fraudulent activity can be identified through **behavioral patterns** - unusual transaction velocity, new recipients, deviations from normal spending behavior, transaction timing, payment channels, and other contextual signals.

This project develops a **behavioral fraud detection and risk analysis system** designed to identify potentially fraudulent transactions while considering the operational cost of false alerts.

Rather than optimizing only for accuracy, the project focuses on a more realistic fraud-detection objective:

> **Detect as much fraud as possible while producing a manageable number of alerts for investigation.**

The project combines exploratory data analysis, feature engineering, machine learning, threshold optimization, cost-sensitive analysis, and model interpretation to translate transaction-level predictions into actionable risk insights.



## Business Problem

A financial institution processes millions of transactions, but only a very small proportion are fraudulent.

This creates a major challenge for conventional classification models:

* Fraudulent transactions are extremely rare.
* A model can achieve high accuracy simply by predicting almost every transaction as legitimate.
* Missing a fraudulent transaction can be significantly more costly than investigating a legitimate transaction.
* At the same time, flagging too many legitimate transactions creates alert fatigue and increases investigation costs.

Therefore, the objective was not simply to build a model with high accuracy.

The objective was to develop a model that could:

1. Identify behavioral patterns associated with fraud.
2. Rank transactions according to their fraud risk.
3. Capture a high proportion of fraudulent transactions.
4. Concentrate fraud cases within a manageable group of alerts.
5. Balance false positives against false negatives.
6. Provide insights that could support fraud investigation and risk-management decisions.



## Project Objectives

* Explore behavioral patterns associated with fraudulent transactions.
* Understand the severe class imbalance in the dataset.
* Prepare transaction-level behavioral features for modelling.
* Compare baseline and ensemble machine-learning approaches.
* Evaluate models using fraud-appropriate metrics such as **PR-AUC, ROC-AUC, precision, recall, and F1-score**.
* Optimize the classification threshold rather than relying blindly on the default 0.50 threshold.
* Quantify fraud-detection lift and alert concentration.
* Evaluate different fraud-loss cost scenarios.
* Interpret the model's most influential features.
* Translate model results into practical fraud-risk recommendations.



## Dataset

The dataset contains **8,401,675 transactions** and 15 predictive features after separating the target variable.

The target variable is:

* `is_fraud` - whether the transaction was fraudulent.

### Target Distribution

| Transaction Type |         Count | Percentage |
| ---------------- | ------------: | ---------: |
| Legitimate       |     8,395,623 |   99.9280% |
| Fraudulent       |         6,052 |    0.0720% |
| **Total**        | **8,401,675** |   **100%** |

The extreme imbalance is important because a model could achieve very high accuracy while performing poorly at detecting fraud.

For this reason, the project places greater emphasis on **recall, precision, PR-AUC, fraud concentration, and cost-sensitive threshold selection**.



## Behavioral Features

The model uses transaction and behavioral signals including:

* Transaction amount
* Time since last transaction
* Transaction velocity
* Spending deviation
* First-transaction indicator
* Transaction hour
* Day of week
* Weekend indicator
* Cross-bank transfer indicator
* Cross-currency transfer indicator
* New receiver indicator
* New bank indicator
* New payment format indicator
* Payment channel

Payment channels included:

`ACH`, `Bitcoin`, `Cash`, `Cheque`, `Credit Card`, `Reinvestment`, and `Wire`.



## Exploratory Data Analysis

The EDA focused on identifying behavioral differences between fraudulent and legitimate transactions.

One notable finding was the relationship between **new receivers and fraud**.

Approximately **22.45% of transactions involved a new receiver**, but the observed fraud rate for transactions involving a new receiver was approximately **23.72%**, compared with approximately **2.42% for existing receivers**.

This indicates that a transaction involving a new receiver represents a substantially different risk profile from a transaction involving an established receiver.

Other behavioral variables, including transaction timing, transaction velocity, spending deviation, transaction amount, and payment characteristics, were also examined as potential fraud signals.

### Key EDA Insight

The analysis suggests that fraud risk is not driven by transaction value alone.

**Context and behavioral change matter.**

A transaction may become more suspicious because of *who the money is being sent to, how quickly transactions are occurring, how long it has been since the previous transaction, and how unusual the transaction is relative to expected behaviour.*



## Data Preparation

The dataset was divided into:

| Dataset    | Transactions | Fraud Cases | Fraud Rate |
| ---------- | -----------: | ----------: | ---------: |
| Training   |    5,881,172 |       3,445 |    0.0586% |
| Validation |    1,260,251 |       1,341 |    0.1064% |
| Test       |    1,260,252 |       1,266 |    0.1005% |

Categorical payment-channel features were encoded before modelling.

Memory optimization was also performed to make the large dataset more manageable.

Memory usage was reduced substantially:

| Dataset    |    Before |     After |
| ---------- | --------: | --------: |
| Training   | 990.69 MB | 201.91 MB |
| Validation | 211.40 MB |  43.27 MB |
| Test       | 211.52 MB |  43.27 MB |

After encoding, the feature matrices contained 21 features.



# Modelling

## Baseline Model

A baseline classification model was first evaluated to establish a reference point.

The baseline produced:

* **ROC-AUC:** 0.4887
* **PR-AUC:** 0.0037
* **Fraud recall:** 99.93%
* **Fraud precision:** 0.11%

Although the baseline detected almost all fraud cases, it generated an extremely large number of false-positive predictions.

This demonstrated an important point:

> **High fraud recall alone does not make a fraud detection system operationally useful.**



## Random Forest Model

A Random Forest model was then trained to capture nonlinear relationships and interactions between behavioural features.

On the validation set, the Random Forest achieved:

* **ROC-AUC:** 0.9773
* **PR-AUC:** 0.3630
* **Fraud precision:** 0.90%
* **Fraud recall:** 94.33%
* **Fraud F1-score:** 0.0178

The model substantially improved the separation between fraudulent and legitimate transactions compared with the baseline.

However, the default 0.50 classification threshold still generated a large number of false-positive alerts.

This led to the next stage of the analysis: **threshold optimization**.



# Threshold Optimization

A fraud detection model does not necessarily need to classify transactions using the default probability threshold of 0.50.

Different thresholds create different trade-offs between:

* fraud captured,
* legitimate transactions flagged,
* investigation workload,
* and potential fraud losses.

Threshold analysis showed that increasing the threshold reduced false positives while gradually reducing fraud recall.

At a threshold of **0.70**, the validation results were:

| Metric               |     Result |
| -------------------- | ---------: |
| Precision            |      2.70% |
| Recall               |     81.36% |
| F1-score             |     0.0523 |
| False positives      |     39,305 |
| Fraud detection lift | **25.38×** |

The baseline fraud rate in the validation set was only **0.1064%**.

At the selected threshold, the fraud rate among flagged transactions increased substantially, creating a **25.38× fraud-detection lift** over the baseline.

This means the model concentrates fraudulent transactions into a much smaller population of transactions requiring investigation.



# Final Model Evaluation

The selected model and threshold were evaluated on the previously unseen test set.

### Test Performance

| Metric          |     Result |
| --------------- | ---------: |
| ROC-AUC         | **0.9738** |
| PR-AUC          | **0.3433** |
| Fraud Precision |  **2.39%** |
| Fraud Recall    | **75.75%** |
| Fraud F1-score  | **0.0463** |
| Accuracy        | **96.87%** |

The test set contained:

* **1,260,252 total transactions**
* **1,266 fraudulent transactions**

At the selected threshold:

* **40,119 transactions were flagged**
* **959 fraudulent transactions were detected**
* **307 fraudulent transactions were missed**
* **39,160 legitimate transactions were incorrectly flagged**

This produced an alert rate of approximately **3.18%**.

Most importantly, the fraud rate among flagged transactions increased to **2.39%**, compared with the baseline fraud rate of **0.1005%**.

That represents a:

### **23.80× fraud concentration lift**

In practical terms, the model reduced the population requiring investigation from the entire transaction stream to approximately 3.18% of transactions while capturing approximately 75.75% of observed fraud.



# Cost-Sensitive Threshold Analysis

Fraud detection decisions depend on the relative cost of:

* investigating a legitimate transaction, and
* allowing a fraudulent transaction to pass undetected.

Therefore, several hypothetical fraud-loss scenarios were evaluated.

| Fraud-loss Cost | Best Threshold | False Positives | False Negatives | Relative Cost |
| --------------- | -------------: | --------------: | --------------: | ------------: |
| 10×             |           0.90 |          13,671 |             550 |        19,171 |
| 50×             |           0.90 |          13,671 |             550 |        41,171 |
| 100×            |           0.80 |          28,585 |             379 |        66,485 |

These results demonstrate that there is **no universally optimal threshold**.

The appropriate operating threshold depends on the institution's tolerance for:

* investigation workload,
* customer friction,
* fraud losses,
* and operational risk.

For an organization where missed fraud is extremely expensive, a lower threshold may be justified to capture more suspicious transactions.

For an organization where investigation capacity is constrained, a higher threshold may be more appropriate.



# Model Interpretation

Model interpretation was performed to understand which behavioural signals contributed most strongly to fraud predictions.

The model's feature-importance analysis highlighted several important variables.

### Top Features

| Feature                       | Importance |
| ----------------------------- | ---------: |
| `is_new_receiver`             |     0.3331 |
| `payment_channel_ACH`         |     0.3113 |
| `time_since_last_transaction` |     0.3046 |
| `amount`                      |     0.1813 |
| `payment_channel_Cheque`      |     0.1347 |
| `payment_channel_Credit Card` |     0.0911 |
| `velocity_score`              |     0.0507 |
| `is_new_payment_format`       |     0.0477 |
| `payment_channel_Cash`        |     0.0303 |
| `is_cross_bank_transfer`      |     0.0284 |

The consistency of several behavioural variables across the feature-importance analyses provides useful evidence that fraud detection is strongly connected to **transaction context and behavioural change**.

### Key Interpretation

`is_new_receiver`, transaction timing, transaction amount, transaction velocity, and payment characteristics emerged as important signals.

This supports the central premise of the project:

> **Fraud risk can be better understood by examining how a transaction fits within a user's behavioural context, rather than evaluating the transaction in isolation.**

Feature importance should not be interpreted as proof that a feature causes fraud. Instead, these variables indicate that the model found them useful for distinguishing fraudulent from legitimate transactions.



# Key Findings

### 1. Fraud is extremely rare

Only **0.072%** of all transactions in the dataset were fraudulent.

This makes accuracy an insufficient metric for evaluating the system.

### 2. Behavioural context provides meaningful fraud signals

Variables such as new receivers, transaction timing, velocity, amount, and payment characteristics were important to the model.

### 3. New receivers represent a significant risk signal

Transactions involving new receivers had a substantially higher observed fraud rate than transactions involving existing receivers.

### 4. Random Forest substantially outperformed the baseline

The Random Forest achieved a **0.9738 ROC-AUC** and **0.3433 PR-AUC** on the test set.

### 5. Threshold selection changes the operational value of the model

The default probability threshold was not sufficient for an operational fraud-screening system.

Threshold optimization allowed the model to trade some recall for a substantial reduction in false-positive alerts.

### 6. The model concentrated fraud into a smaller investigation population

At the selected threshold, approximately **3.18% of transactions were flagged**, while approximately **75.75% of observed fraud was detected**.

### 7. The system produced a 23.80× fraud concentration lift

Only 0.1005% of test transactions were fraudulent overall, but 2.39% of flagged transactions were fraudulent.

This makes the flagged population substantially more useful for targeted investigation.



# Business Recommendations

## 1. Use behavioural risk scoring rather than transaction amount alone

Fraud monitoring systems should incorporate behavioural signals such as:

* new recipients,
* transaction velocity,
* unusual spending patterns,
* transaction timing,
* payment method,
* and changes in transaction behaviour.



## 2. Introduce risk-based transaction triage

Rather than treating every flagged transaction equally, transactions can be grouped into risk tiers.

For example:

* **Low risk:** monitor automatically.
* **Medium risk:** additional verification.
* **High risk:** manual investigation or transaction hold.

This allows investigation resources to be concentrated where they are most valuable.



## 3. Monitor new-recipient transactions more closely

Because new-recipient behaviour emerged as one of the strongest fraud signals, financial institutions could apply additional verification or monitoring to high-risk transactions involving newly added recipients.



## 4. Tune thresholds according to operational capacity

The threshold should not be treated as a permanent technical constant.

It should be adjusted based on:

* investigation team capacity,
* fraud-loss exposure,
* customer experience,
* acceptable false-positive rates,
* and changing fraud patterns.



## 5. Use cost-sensitive decision-making

Organizations should estimate the real financial cost of:

* missed fraud,
* manual investigation,
* customer friction,
* and false alerts.

These costs can then be incorporated into threshold selection.



## 6. Continuously monitor model performance

Fraud patterns evolve.

A production system should therefore monitor:

* precision,
* recall,
* PR-AUC,
* fraud concentration,
* alert volume,
* false-positive rate,
* and changes in feature behaviour.

The model should be retrained or recalibrated when fraud patterns shift.



# Limitations

This project demonstrates a behavioural fraud-detection workflow, but several limitations should be considered.

### Dataset limitations

The analysis is based on the available dataset and therefore may not capture every behavioural pattern present in real-world financial transactions.

### Feature limitations

The model can only learn from the behavioural signals available in the dataset. Additional signals such as device information, location, account history, authentication behaviour, and network relationships could potentially improve fraud detection.

### Class imbalance

Fraud represents a very small proportion of transactions, making precision relatively low even when the model captures a large proportion of fraud.

### Model interpretability

Random Forest feature importance helps identify influential variables, but feature importance does not establish causation.

### Operational assumptions

The cost-sensitive analysis uses hypothetical relative fraud-loss costs. A production implementation would require actual business costs to determine the most appropriate threshold.



# Conclusion

This project demonstrates that effective fraud detection is not simply a classification problem.

It is a **risk-management problem**.

The analysis moved from identifying behavioural patterns in millions of transactions to building a predictive model, evaluating its performance under severe class imbalance, optimizing the decision threshold, examining the cost of different operating strategies, and translating the results into business recommendations.

The final model achieved a **0.9738 ROC-AUC** and **0.3433 PR-AUC** on the unseen test set.

At the selected operating threshold, the system flagged approximately **3.18% of transactions**, captured **75.75% of observed fraud**, and achieved a **23.80× fraud concentration lift**.

The most important takeaway is that the value of a fraud model is not determined by accuracy alone.

A useful fraud-detection system must answer a more practical question:

> **Can we identify a manageable group of transactions where fraudulent activity is significantly more concentrated, so that limited investigation resources can be directed toward the transactions with the greatest risk?**

This project demonstrates one approach to answering that question using behavioural analytics, machine learning, threshold optimization, and risk-based decision-making.



## Project Structure

```text
behavioral-fraud-detection/
│
├── README.md
├── behavioral_fraud_detection.ipynb
├── .gitignore
└── data/
    └── README.md
```



## Tools & Technologies

* Python
* Pandas
* NumPy
* Scikit-learn
* Matplotlib
* Seaborn
* Google Colab
* Git & GitHub



## Skills Demonstrated

* Exploratory Data Analysis
* Behavioural Analytics
* Fraud Detection
* Risk Analysis
* Feature Engineering
* Data Preprocessing
* Imbalanced Classification
* Random Forest
* Model Evaluation
* Precision-Recall Analysis
* Threshold Optimization
* Cost-Sensitive Analysis
* Model Interpretation
* Business Intelligence
* Translating Machine Learning Results into Business Recommendations



## Notebook

The complete analysis, including data preparation, EDA, modelling, threshold optimization, cost-sensitive analysis, model interpretation, and recommendations, is available in the project notebook.

**[Open the notebook in Google Colab](YOUR_COLAB_LINK_HERE)**



## Author

**Mercy Sunday**

Data Scientist | Economics Graduate

Using data, analytics, and technology to solve practical business and financial problems.

