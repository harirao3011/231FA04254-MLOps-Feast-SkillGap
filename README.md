[pipeline_diagram.txt](https://github.com/user-attachments/files/31131623/pipeline_diagram.txt)Name: D Hari Rao
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
CURRICULUM-INDUSTRY PLACEMENT ML PIPELINE

Original CSV
    |
    v
Load Dataset
    |
    v
Data Validation
    |
    +--> Check missing values / duplicates / target distribution
    |
    v
Feature Engineering
    |
    +--> technical_skill_avg
    +--> soft_skill_avg
    +--> experience_index
    +--> skill_strength_avg
    +--> industry_alignment_score
    |
    v
Select Model Features
    |
    +--> Numerical features --> median imputation --> StandardScaler
    |
    +--> Gender --> most-frequent imputation --> OneHotEncoder
    |
    v
ColumnTransformer
    |
    v
Logistic Regression
(class_weight='balanced')
    |
    +--> Train/Test split (80/20, stratified)
    |
    v
Evaluation
    |
    +--> Accuracy
    +--> Precision
    +--> Recall
    +--> F1-score
    +--> ROC-AUC
    +--> Confusion Matrix
    |
    v
Prediction

FEAST VERSION

Feature Engineering
    |
    v
skill_features.parquet
    |
    v
Feast Entity: Participant_ID
    |
    v
Feast FileSource
    |
    v
FeatureView: skill_gap_features
    |
    +--------------------------+
    |                          |
    v                          v
Historical Retrieval       Materialization
    |                          |
    v                          v
ML Model Training         SQLite Online Store
                               |
                               v
                       Online Feature Retrieval
                               |
                               v
                           Prediction



1. **What is the entity in your Feast implementation?**

The entity in the Feast implementation is Participant_ID.

Each row in the dataset represents one student/participant, and Participant_ID uniquely identifies that student. Feast uses this entity to associate the feature values with the correct student.

Entity:
Participant_ID
2. List the features stored in your FeatureView.

The skill_gap_features FeatureView stores the following features:

Feature	Meaning
age	Age of the student
degree_year	Year of the student's degree
cgpa	Student CGPA
technical_skill_avg	Average score of the student's technical skills
soft_skill_avg	Average score of the student's soft skills
experience_index	Combined score representing internships, projects, certifications and hackathons
skill_strength_avg	Combined technical and soft-skill strength
industry_expectation	Industry expectation score
industry_alignment_score	Overall alignment between student skills and industry expectations
gender	Student gender

The target variable placement_target is used for the machine-learning model but is not treated as a predictive Feast feature, because it is the label that the model is trying to predict.

3. Explain how one feature was calculated.

One important engineered feature is technical_skill_avg.

It is calculated as the average of the student's ten technical skill scores:

technical_skill_avg = (
    Programming_Skill +
    DSA_Skill +
    DBMS_Skill +
    OS_Skill +
    CN_Skill +
    Web_Development +
    Cloud_Computing +
    AI_ML +
    Cybersecurity +
    DevOps
) / 10

Therefore, technical_skill_avg provides a single overall measure of the student's technical ability.

Similarly, soft_skill_avg is calculated from Communication, Teamwork, Problem Solving, Critical Thinking and Leadership.

4. What is the difference between your original dataset and the feature dataset?

The original dataset contains the complete student and curriculum-industry information, including raw skill values, experience information, placement information and other descriptive columns.

The feature dataset is a smaller, ML-ready dataset created specifically for the Feast feature store.

The transformation is:

Original Dataset
        ↓
Feature Engineering
        ↓
Selected + Derived Features
        ↓
skill_features.parquet

The feature dataset contains the entity Participant_ID, a timestamp required for Feast historical retrieval, and the engineered features used by the model.

The target Placement_Status is converted into a binary placement_target:

Placed              → 1
Not Placed          → 0
Higher Studies      → 0

The original raw columns are therefore reduced into a consistent set of reusable ML features.

5. What is the purpose of the offline store?

The offline store contains historical feature data.

In this project, the feature data is stored in a Parquet file and used for historical feature retrieval and model training.

Its main purpose is to answer questions such as:

“What were the feature values for this student at a particular point in time?”

The offline data is therefore useful for:

historical training data
point-in-time feature retrieval
model training
avoiding manual reconstruction of features
6. What is the purpose of the online store?

The online store stores the latest feature values in a format optimized for fast retrieval.

In this project, Feast uses a SQLite-based online store for local development.

It is used when the model needs features for a new prediction.

For example:

Participant_ID
      ↓
Feast Online Store
      ↓
Current Feature Values
      ↓
ML Model
      ↓
Prediction

The online store therefore supports fast feature retrieval during prediction/inference.

7. What is the purpose of feast apply?

feast apply registers and applies the Feast definitions in the project.

It reads the Feast configuration and feature definitions and creates or updates objects such as:

Entity
Data Source
Feature View

In this project, it makes Feast aware of the Participant_ID entity, the Parquet data source and the skill_gap_features FeatureView.

In simple terms:

feast apply tells Feast what features and entities exist in the feature store and registers those definitions.

8. What does materialization do?

Materialization copies historical feature data from the offline store into the online store.

The flow is:

Parquet Offline Data
        ↓
   Materialization
        ↓
SQLite Online Store

After materialization, the latest feature values can be retrieved efficiently using Feast's online retrieval mechanism.

In this project, materialization makes the engineered student features available for online prediction.

9. What is the advantage of retrieving features through Feast instead of manually calculating them separately during training and prediction?

The main advantage is consistency.

Without Feast, the developer might calculate the features one way during training and differently during prediction.

For example:

Training:
Raw Data → Calculate Features A → Model


Prediction:
New Data → Calculate Features B → Model

This can create inconsistent results.

With Feast:

             Same Feature Definitions
                      ↓
        +-------------+-------------+
        ↓                           ↓
Historical Retrieval          Online Retrieval
        ↓                           ↓
 Model Training              Model Prediction

Therefore, Feast helps provide:

consistent feature definitions
reusable features
reduced duplicate feature-engineering code
less chance of training-serving mismatch
easier model deployment
centralized feature management
reproducible ML workflows

This is especially useful when the same features must be used repeatedly by multiple models.

10. State two limitations of your current dataset.

Limitation 1 – Highly imbalanced placement target

The dataset contains a very large number of Placed records compared with Not Placed and Higher Studies records. Therefore, a model can achieve high accuracy while still performing poorly on the minority class.

To address this, the ML pipeline uses techniques such as:

class_weight="balanced"

and evaluates precision, recall, F1-score and ROC-AUC rather than relying only on accuracy.

Limitation 2 – No real event timestamp

The original dataset does not contain a genuine timestamp representing when a student's skill or placement information was observed.

For the Feast demonstration, a deterministic timestamp is generated so that historical feature retrieval can be demonstrated.

A real production system should use actual timestamps from curriculum updates, assessments, internships, job-market observations or placement events.

11. State two ways your feature store could be improved when more curriculum and industry evidence becomes available.

Improvement 1 – Add time-based industry demand features

As more industry data becomes available, the FeatureView could include historical demand information such as:

monthly_skill_demand
skill_demand_growth
number_of_job_postings
skill_demand_trend

This would allow the model to understand how industry requirements change over time.

Improvement 2 – Add curriculum coverage and skill-gap features

More curriculum evidence could be incorporated to calculate features such as:

curriculum_skill_coverage
industry_skill_coverage
curriculum_industry_gap
skill_priority_score

For example:

Industry Demand
       +
Curriculum Coverage
       ↓
Skill Gap Score
       ↓
Student Skill Gap Prediction
