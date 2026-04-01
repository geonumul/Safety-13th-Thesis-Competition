# Data Preprocessing Process and Rationale

## 1. Data Source

This study utilized the raw data from the **10th Occupational Safety and Health Survey (2021)** conducted by the Occupational Safety and Health Research Institute of the Korea Occupational Safety and Health Agency (KOSHA, 2021). The construction industry survey covered **1,502 construction sites** selected as samples from sites with construction costs of 5 billion KRW or more.

---

## 2. Missing Values and Outlier Treatment: Sample Finalization

Multiple regression and machine learning models apply listwise deletion, excluding any case with even a single missing value across all input variables (independent, moderating, control, etc.). Accordingly, the original 1,502 sites were sequentially filtered by removing dependent variable missings, outliers, and key variable non-responses to finalize the analysis sample. This approach is consistent with the preprocessing methods adopted in Park (2023, 2024, 2025).

The sequential data cleaning process is as follows:

| Step | Target Variable | Removal Criteria | Removed | Remaining |
| :---: | :--- | :--- | :---: | :---: |
| Start | - | 10th OSH Survey (Construction) raw data | - | 1,502 |
| 1 | Dependent var. | Cannot determine accident occurrence (Q27_3_1~Q27_3_3 all NaN) | 16 | 1,486 |
| 2 | Dependent var. (outlier) | Suspected data entry error (Q27_3_3 fatality=30 recorded). 30 fatalities at a 5~12B KRW site with 10 workers is beyond permissible range | 1 | 1,485 |
| 3 | Independent var. (safety org.) | Non-response for dedicated dept. existence (Q6=9) | 21 | 1,464 |
| 4 | Independent var. (committee) | Non-response and unknown for committee operation (Q10=4 "don't know", Q10=9 non-response) | 62 | 1,402 |
| 5 | Independent var. (risk assess.) | Structural non-response (Q14=NaN). Sites reporting no hazard factors, thus skipping the risk assessment section entirely | 24 | 1,378 |
| 6 | Moderating var. (expert guidance) | Non-response for guidance status (Q9=9) | 3 | 1,375 |

**Final analysis sample: 1,375 construction sites**
(The operational definitions and descriptive statistics in Sections 3–6 below are based on the finalized sample of 1,375 sites.)

---

## 3. Dependent Variable Construction

### accident_occurred (Binary Variable)

Coded as 1 if any of Q27_3_1 (4–89 day recuperation), Q27_3_2 (90+ days), or Q27_3_3 (fatality) recorded at least 1 case; coded as 0 if all three equal zero.

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 0 | No accident | 984 | 71.6% |
| 1 | Accident occurred | 391 | 28.4% |
| | **Total** | **1,375** | **100.0%** |

The three variables were merged because 90+ day recuperation cases (25) and fatalities (14) are too few to serve as standalone dependent variables, compromising model stability. Park (2025) employed both count (negative binomial regression) and severity (ordered logit) approaches in parallel; however, this study adopts binary classification as the most stable dependent variable form for machine learning with SHAP.

---

## 4. Independent Variable Preprocessing

### 4.1 Merging Conditional (Skip Logic) Items

The OSH Survey contains numerous conditional items where eligibility for a sub-item depends on the response to its parent item. Directly inputting such sub-items into the model would generate massive structural missingness, making it impossible to retain the 1,375-case sample. To resolve this, parent and child items were merged into new single ordinal variables.

#### (1) safety_org_level (Q6 + Q7_1)

Q7_1 (existence of personnel whose primary duty is OSH) is a conditional item answered only when Q6=2 ("no dedicated department"). Sites with a dedicated department are structurally unable to respond. The two items were combined into a 3-level ordinal variable to resolve the missingness and capture qualitative differences in organizational capacity.

| Value | Definition | Note | N | % |
| :---: | :--- | :--- | :---: | :---: |
| 0 | Neither dedicated dept. nor designated personnel | Q6=2 and Q7_1=2 | 30 | 2.2% |
| 1 | No dedicated dept. but designated personnel exist | Q6=2 and Q7_1=1 | 392 | 28.5% |
| 2 | Dedicated department exists | Q6=1 | 953 | 69.3% |

Park (2025) used Q6 alone as a binary variable, whereas this study combines Q7_1 to capture the qualitative gradient of organizational safety management.

#### (2) committee_level (Q10 + Q10_1)

Q10_1 (implementation management of deliberated/resolved matters) is a sub-item answered only by sites that reported "operating" a committee/council. The survey uses the parallel notation "OSH Committee (or Labor-Management Council)," so both the OSH Committee (Q10=1) and Labor-Management Council (Q10=2) were merged as "operating." To distinguish substantive from nominal operation, Q10_1 values of 2 (status tracked but not continuously managed) and 3 (concluded upon deliberation) were classified as "poor follow-up."

| Value | Definition | Note | N | % |
| :---: | :--- | :--- | :---: | :---: |
| 0 | Committee not operating | Q10=3 | 175 | 12.7% |
| 1 | Operating but poor implementation follow-up | Q10=1 or 2, and Q10_1=2 or 3 | 191 | 13.9% |
| 2 | Operating + continuous implementation management | Q10=1 or 2, and Q10_1=1 | 1,009 | 73.4% |

Park (2023) confirmed that OSH committee operation significantly increases risk assessment participation.

#### (3) certification (Q12_1 + Q12_2)

Both KOSHA-MS (Q12_1) and ISO45001 (Q12_2) measure the same construct—occupational safety and health management systems—and a substantial number of sites hold both simultaneously. To prevent multicollinearity and enhance SHAP value interpretability, the two were merged into a single binary variable (1 if either certification is held).

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 0 | Neither certification held | 937 | 68.1% |
| 1 | At least one certification held | 438 | 31.9% |

#### (4) risk_assess_level (Q14 + Q14_2_2)

Q14_2_2 (worker participation in risk assessment) is a sub-item answered only by sites that reported "implemented." The distinction between irregular (Q14=2) and regular (Q14=3) implementation is irrelevant to the core purpose of this variable—whether workers participated—so both were merged into "implemented."

| Value | Definition | Note | N | % |
| :---: | :--- | :--- | :---: | :---: |
| 0 | Risk assessment not implemented | Q14=1 | 117 | 8.5% |
| 1 | Implemented but workers did not participate | Q14=2 or 3, and Q14_2_2=2 | 75 | 5.5% |
| 2 | Implemented + workers participated | Q14=2 or 3, and Q14_2_2=1 | 1,183 | 86.0% |

Park (2024) confirmed that risk assessment implementation is positively associated with construction site safety.

### 4.2 Original Variables Retained (On-site Safety Behavior)

The following variables were measured on a 5-point Likert scale (1=Strongly disagree to 5=Strongly agree) and were input without any preprocessing.

**training_effectiveness** (Q16_10): Perceived helpfulness of OSH education/training for accident prevention

| Value | 1 | 2 | 3 | 4 | 5 | Mean |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| N | 4 | 18 | 151 | 578 | 624 | 4.31 |
| % | 0.3% | 1.3% | 11.0% | 42.0% | 45.4% | |

**housekeeping** (Q16_14): Site housekeeping and cleanliness condition

| Value | 1 | 2 | 3 | 4 | 5 | Mean |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| N | 3 | 21 | 199 | 601 | 551 | 4.22 |
| % | 0.2% | 1.5% | 14.5% | 43.7% | 40.1% | |

**work_stoppage_right** (Q16_17): Degree of freedom for workers to refuse unsafe work

| Value | 1 | 2 | 3 | 4 | 5 | Mean |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| N | 4 | 25 | 133 | 542 | 671 | 4.35 |
| % | 0.3% | 1.8% | 9.7% | 39.4% | 48.8% | |

**foreman_contribution** (Q17_4): Perceived practical contribution of foreman to accident prevention

| Value | 1 | 2 | 3 | 4 | 5 | Mean |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| N | 6 | 45 | 214 | 613 | 497 | 4.13 |
| % | 0.4% | 3.3% | 15.6% | 44.6% | 36.1% | |

---

## 5. Moderating Variable Preprocessing

All three variables measuring external institutional intervention were converted to binary (0/1). (Non-responses were removed during the sample finalization in Section 2.)

**expert_guidance** (Q9): Whether the site receives technical guidance from a construction disaster prevention expert agency

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 0 | Not receiving | 880 | 64.0% |
| 1 | Receiving | 495 | 36.0% |

**moel_inspection** (Q30): Whether the site received MOEL OSH inspection during 2019.6–2021.6

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 0 | No | 679 | 49.4% |
| 1 | Yes | 696 | 50.6% |

**kosha_support** (Q31): Whether the site received information/support from KOSHA during 2019.6–2021.6

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 0 | No | 298 | 21.7% |
| 1 | Yes | 1,077 | 78.3% |

Park (2025) confirmed that labor inspection (Q30) significantly reduces health accident frequency and severity. Park (2024) reported that KOSHA information support (Q31) significantly increases risk assessment participation, particularly in the building construction sector.

---

## 6. Control Variable Preprocessing

The following variables were constructed to control for site characteristics based on the finalized sample of 1,375 sites.

### 6.1 project_scale (SQ2)

Reclassified into 3 categories using the construction cost brackets from the survey instrument. This follows the safety manager appointment criteria in the Occupational Safety and Health Act Enforcement Decree (Table 3) and is identical to Park (2025).

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 1 | 5B–12B KRW | 414 | 30.1% |
| 2 | 12B–80B KRW | 634 | 46.1% |
| 3 | 80B+ KRW | 327 | 23.8% |

Total worker count (Q4_1) was excluded from the model due to high correlation with project scale (multicollinearity concern); scale effects are controlled via this categorical variable instead.

### 6.2 client_type (Q1_4)

Included as a control variable to prevent confounding between client characteristics (e.g., government) and the frequency of external oversight (moderating variables).

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 1 | Government + Public organizations | 501 | 36.4% |
| 2 | Private corporations / Individuals | 719 | 52.3% |
| 3 | Self-construction + Other | 155 | 11.3% |

### 6.3 completion_rate (Q1_5)

Original 6-category coding was retained without modification. Sites below 10% completion show low accident rates, while the 50–70% range exhibits the highest rates, necessitating control for stage-dependent accident patterns.

| Value | Definition | N | % |
| :---: | :--- | :---: | :---: |
| 1 | <10% | 234 | 17.0% |
| 2 | 10–30% | 342 | 24.9% |
| 3 | 30–50% | 261 | 19.0% |
| 4 | 50–70% | 202 | 14.7% |
| 5 | 70–90% | 189 | 13.7% |
| 6 | 90%+ | 147 | 10.7% |

### 6.4 construction_type (Q2)

The original 12 categories were merged into 7 by combining sparse categories with similar trades (e.g., bridge=20, rail=29).

| Value | Definition | Original codes | N | % |
| :---: | :--- | :---: | :---: | :---: |
| 1 | Apartment | 1 | 424 | 30.8% |
| 2 | Factory | 2 | 59 | 4.3% |
| 3 | Commercial/Office building | 3 | 287 | 20.9% |
| 4 | School/Hospital/Other building | 4+5 | 241 | 17.5% |
| 5 | Road/Rail/Bridge | 6+7+8 | 124 | 9.0% |
| 6 | Water supply/Civil engineering | 9+10 | 149 | 10.8% |
| 7 | Electrical/Telecom/Other | 11+12 | 91 | 6.6% |

### 6.5 foreign_worker_ratio (Q4_3 / Q4_1 × 100)

Converted from absolute count to percentage ratio to prevent scale imbalance when combined with other variables (Likert 1–5, binary 0/1).

| Statistic | Value |
| :--- | :---: |
| Mean | 13.30% |
| SD | 19.02% |
| Min | 0.00% |
| Median | 0.00% |
| 75th percentile | 22.04% |
| Max | 100.00% |
| Sites with 0% | 777 (56.5%) |

### 6.6 Excluded Control Variable: site_region (SQ3)

17 provinces were reclassified into 6 economic regions for review. However, chi-square testing showed no significant association with accident occurrence (p=0.137), and model AUC decreased by 0.0103 when included. Park (2025) also confirmed no significant regional variation in safety accident occurrence.

---

## 7. Final Analysis Dataset

The finalized dataset comprises **1,375 construction sites × 17 variables**, structured as follows:

| Category | # Vars | Variable Names |
| :--- | :---: | :--- |
| Independent A (Internal Mgmt.) | 3 | safety_org_level, committee_level, certification |
| Independent B (On-site Safety Behavior) | 5 | risk_assess_level, training_effectiveness, housekeeping, work_stoppage_right, foreman_contribution |
| Moderating (External Intervention) | 3 | expert_guidance, moel_inspection, kosha_support |
| Control (Site Characteristics) | 5 | project_scale, client_type, completion_rate, construction_type, foreign_worker_ratio |
| Dependent | 1 | accident_occurred |

### Summary of Descriptive Statistics

| Variable | Type | Range | Mean | SD | N |
| :--- | :---: | :---: | :---: | :---: | :---: |
| safety_org_level | Ordinal | 0–2 | 1.67 | 0.51 | 1,375 |
| committee_level | Ordinal | 0–2 | 1.61 | 0.71 | 1,375 |
| certification | Binary | 0–1 | 0.32 | 0.47 | 1,375 |
| risk_assess_level | Ordinal | 0–2 | 1.78 | 0.56 | 1,375 |
| training_effectiveness | Likert | 1–5 | 4.31 | 0.76 | 1,375 |
| housekeeping | Likert | 1–5 | 4.22 | 0.77 | 1,375 |
| work_stoppage_right | Likert | 1–5 | 4.35 | 0.76 | 1,375 |
| foreman_contribution | Likert | 1–5 | 4.13 | 0.83 | 1,375 |
| expert_guidance | Binary | 0–1 | 0.36 | 0.48 | 1,375 |
| moel_inspection | Binary | 0–1 | 0.51 | 0.50 | 1,375 |
| kosha_support | Binary | 0–1 | 0.78 | 0.41 | 1,375 |
| project_scale | Categorical | 1–3 | 1.94 | 0.73 | 1,375 |
| client_type | Categorical | 1–3 | 1.75 | 0.65 | 1,375 |
| completion_rate | Categorical | 1–6 | 2.96 | 1.56 | 1,375 |
| construction_type | Categorical | 1–7 | 3.03 | 1.82 | 1,375 |
| foreign_worker_ratio | Continuous | 0–100 | 13.30 | 19.02 | 1,375 |
| accident_occurred | Binary | 0–1 | 0.28 | 0.45 | 1,375 |

---

## References

- Park, C. (2023). A Study of Factors Affecting the Implementation of Workplace Safety and Health Risk Assessment: Focus on Manufacturing and Service Industries. *Health and Social Welfare Review*, 43(4), 158–178.
- Park, C. (2024). Factors Influencing Risk Assessment in Construction Workplaces. *Quarterly Journal of Labor Policy*, 24(4), 67–95.
- Park, C. (2025). A Study on Occupational Safety and Health Accidents and Their Determinants in Construction Sites. *Quarterly Journal of Labor Policy*, 25(4), 67–93.
- Korea Occupational Safety and Health Agency, Occupational Safety and Health Research Institute (2021). *10th Occupational Safety and Health Survey: User Guide*.
