Name: D Hari Rao
Register Number: 231FA04254
Section: 15

Problem Statement

The curriculum-industry skill-gap problem is to identify whether the skills possessed by Computer Science students are aligned with the skills expected by the IT industry. The dataset combines students' technical skills, soft skills, academic performance, experience, and industry expectations. By analyzing these factors, the machine-learning model can identify placement-related patterns and help measure where students may have skill gaps compared with industry requirements.

The main goal of this project is to create reusable features from the curriculum-industry skill data, store them using Feast, and use those features consistently for machine-learning training and prediction.

Dataset-
Number of Skills

The dataset contains 15 skill-related attributes:

10 Technical Skills-

Programming Skill
DSA Skill
DBMS Skill
OS Skill
CN Skill
Web Development
Cloud Computing
AI/ML
Cybersecurity
DevOps

5 Soft Skills-
Communication
Teamwork
Problem Solving
Critical Thinking
Leadership

Dataset Columns
The dataset contains 28 columns:

1.  Participant_ID
2.  Age
3.  Gender
4.  Degree_Year
5.  CGPA
6.  Programming_Skill
7.  DSA_Skill
8.  DBMS_Skill
9.  OS_Skill
10. CN_Skill
11. Web_Development
12. Cloud_Computing
13. AI_ML
14. Cybersecurity
15. DevOps
16. Communication
17. Teamwork
18. Problem_Solving
19. Critical_Thinking
20. Leadership
21. Internships
22. Projects
23. Certifications
24. Hackathons
25. Industry_Expectation
26. Employability_Score
27. Skill_Gap_Level
28. Placement_Status
// The dataset contains 50,000 student records.

Target
The original target column is:

Placement_Status
It contains the following categories:

Placed
Not Placed
Higher Studies

For the machine-learning model, it is converted into a binary target:
Placed                → 1
Not Placed            → 0
Higher Studies        → 0

Therefore, the ML model predicts whether a student is Placed or Not Placed.

Pipeline Architecture:


<img width="1202" height="647" alt="Screenshot 2026-08-17 120853" src="https://github.com/user-attachments/assets/2feabb77-60ab-4bb7-96a8-a040ced68358" />
<img width="1210" height="636" alt="Screenshot 2026-08-17 120920" src="https://github.com/user-attachments/assets/94a5914e-c813-4188-9b53-1dff92c37935" />
<img width="1196" height="310" alt="Screenshot 2026-08-17 120938" src="https://github.com/user-attachments/assets/4da9f60c-aa48-4ff8-bf4c-1ec3712d15a7" />

**Results**
Historical Feature Output

The engineered feature dataset was created from the original 50,000-row CSV and stored as:

data/skill_features.parquet

The historical feature data contains the Feast entity, timestamp, and engineered features required for model training.

Example historical feature output:

Participant_ID  event_timestamp        CGPA  technical_skill_avg  soft_skill_avg  experience_index  skill_strength_avg  Industry_Expectation  industry_alignment_score
1               2025-01-01 00:00:00    6.50        67.0                64.2              ...             66.16                  78                  69.71
2               2025-01-01 00:00:01    7.20        72.5                70.4              ...             71.87                  82                  74.91
3               2025-01-01 00:00:02    8.10        81.0                76.8              ...             79.74                  85                  81.32

The historical feature dataset is retrieved using Feast's historical feature retrieval mechanism and is used as the input for model training.

Model Accuracy

The dataset was divided into:

Training data = 80%
Testing data  = 20%

A stratified split was used because the placement classes are highly imbalanced.

Seven classification models were trained and compared.

Model	Accuracy	Precision	Recall	F1-Score	ROC-AUC
HistGradientBoosting	97.73%	98.24%	99.46%	98.85%	97.48%
Gradient Boosting	97.65%	98.32%	99.29%	98.80%	97.37%
Random Forest	96.71%	99.13%	97.50%	98.30%	97.38%
Extra Trees	94.87%	99.65%	95.09%	97.32%	97.22%
Decision Tree	93.44%	99.72%	93.56%	96.54%	92.24%
Logistic Regression	92.73%	99.93%	92.63%	96.14%	97.87%
Linear SVM	92.35%	99.94%	92.23%	95.93%	97.87%

The HistGradientBoosting model was selected as the best model because it achieved the highest F1-score.

The final test accuracy was:

97.73%

The confusion matrix of the selected model was:

[[  43   174]
 [  53  9730]]

This shows the model's performance on the two target classes:

0 = Not Placed / Higher Studies
1 = Placed

Because the dataset is highly imbalanced, F1-score, Recall and ROC-AUC were also considered rather than using accuracy alone.

Online Feature Output

After creating the Feast FeatureView, the historical feature data can be materialized into the local SQLite online store.

The online features are retrieved using Feast's online retrieval mechanism.

Example:

Participant_ID: 1


Age: 21
Degree_Year: 2026
CGPA: 5.71
Gender: Male
technical_skill_avg: 63.00
soft_skill_avg: 68.80
experience_index: 46.17
skill_strength_avg: 64.74
Industry_Expectation: 79.00
industry_alignment_score: 69.02

The online retrieval flow is:

Feast FeatureView
       ↓
Materialization
       ↓
SQLite Online Store
       ↓
get_online_features()
       ↓
Student Features
       ↓
Best ML Model
       ↓
Placement Prediction

For the final demonstration student used by the ML pipeline, the HistGradientBoosting model predicted:

Prediction: PLACED

with an estimated placement probability of approximately:

99.90%
Final Result Summary
Dataset
   ↓
Feature Engineering
   ↓
Feast Historical Features
   ↓
Train/Test Split
   ↓
7 ML Models
   ↓
Model Comparison
   ↓
Best Model: HistGradientBoosting
   ↓
Accuracy: 97.73%
   ↓
Feast Materialization
   ↓
Online Feature Retrieval
   ↓
Placement Prediction
