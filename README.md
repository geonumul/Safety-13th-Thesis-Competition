# 건설현장 산업재해 발생 영향 요인 분석

제13회 안전보건 논문공모전 출품작

## 연구 질문 (RQ)

> 건설현장의 내부 안전 관리와 실질적 안전 행동이 산업재해 발생에 미치는 영향 — 외부 기관의 조절효과를 중심으로

## 데이터

**제10차 산업안전보건 실태조사 (건설업, 2021)**

- 원자료: 1,502개 사업장
- 전처리 후: 1,375개 사업장 × 17개 변수
- 출처: 안전보건공단

## 분석 구조

| Phase | 내용 | 방법 |
|:---:|:---|:---|
| 1 | 탐색적 데이터 분석 (EDA) | 기술통계, 단변량 분석(변수별 사고발생률), VIF, 상관관계 히트맵 |
| 2 | 계층적 로지스틱 회귀 | statsmodels Logit, ΔR² 우도비 검정, 조절효과 시각화 |
| 2+ | LR 타당성 검증 | Hosmer-Lemeshow 검정, FDR 보정, 표준화 OR |
| 3 | 머신러닝 모델 비교 | RF / XGBoost / LightGBM + SMOTENC + 5-Fold CV |
| 3+ | ML 타당성 검증 | Bootstrap 95% CI, McNemar 검정, Sensitivity Analysis (5 seeds), Ablation Study |
| 4 | SHAP 분석 | TreeExplainer, Summary/Bar/Dependence Plot, LR-SHAP 교차검증 |
| 4+ | 변수중요도 삼중검증 | SHAP × Permutation Importance × LR p-value |

## 주요 결과

- **독립변수**: 정리정돈상태(OR=0.793, p=0.041) — A/B 갈래 탐색(16개 모형)에서 일관되게 유의하여 보호 효과 경향 robust 확인
- **통제변수**: 기성률_1(OR=0.089), 기성률_2(OR=0.408), 공사금액 더미, 공사종류 더미, 신체위험(OR=0.828, p=0.021)이 사고발생에 강한 영향
- **조절효과**: 인증보유 × 고용노동부감독 상호작용항 유의(OR=2.333, p=0.013) — 선택편향으로 해석
- **최적 ML 모델**: XGBoost — F1=0.541, Recall=0.808, ROC-AUC=0.710 (Bootstrap 95% CI: F1 [0.463, 0.609], AUC [0.649, 0.774])
- **모델 안정성**: Sensitivity Analysis (5 seeds) F1=0.533±0.015 — seed 의존성 없음
- **SMOTENC 검증**: Ablation Study에서 SMOTENC 적용 시 Recall 0.718 → 0.808 (+0.090) 향상 확인
- **LR 모형 적합**: Hosmer-Lemeshow p=0.5106 — Model 4 적합도 검증 통과
- **변수중요도 삼중검증**: 기성률_1·_2, 공사금액_1이 SHAP·Permutation Importance·LR p-value 전부 일치

## 파일 구조

```
Safety-13th-Thesis-Competition/
├── README.md
├── .gitignore
├── data/
│   ├── 전처리_최종.csv                         # 전처리 완료 데이터 (1,375 × 17)
│   ├── 전처리_최종_확장.csv                     # 박천수(2025) 추가변수 포함 확장 버전
│   ├── 제10차 산업안전보건 실태조사_raw data_건설업_230824.CSV  # 원자료
│   └── external/                              # 참고 논문 PDF
├── notebooks/
│   ├── 01_전처리.ipynb                         # 원자료 → 전처리_최종.csv
│   └── 02_데이터분석.ipynb                     # Phase 1~4 전체 분석
├── docs/
│   ├── 데이터분석_과정_및_근거.md               # 분석 과정·결과·방법론 근거 (자기완결)
│   ├── 전처리_근거_및_과정.md                  # 전처리 단계별 근거
│   ├── 참고논문_정리.md                        # 참고문헌 정리
│   ├── Discussion_재작성_가이드.md             # 논문 Discussion 섹션 작성 가이드
│   └── backup_20260410/                       # 이전 버전 문서 백업
└── results/
    ├── (02_데이터분석.ipynb 실행 시 자동 생성: 1_target_distribution.png ~ 17_shap_threshold_정리정돈.png 등)
    ├── 30_A갈래_탐색.csv, 31_B갈래_탐색.csv    # A/B 갈래 탐색 결과
    └── (supplementary) fig_death_type_50_120.png, fig_external_trend.png,
                         fig_housekeeping_x_guidance.png, fig_moderation_by_scale.png
                         — 조절효과 심층분석 추가 시각화 (참고용 보관)
```
