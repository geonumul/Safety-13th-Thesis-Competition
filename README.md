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
| 1 | 탐색적 데이터 분석 (EDA) | 기술통계, VIF, 상관관계 히트맵 |
| 2 | 계층적 로지스틱 회귀 | statsmodels Logit, ΔR² 우도비 검정, 조절효과 시각화 |
| 3 | 머신러닝 모델 비교 | RF / XGBoost / LightGBM + SMOTENC + 5-Fold CV |
| 4 | SHAP 분석 | TreeExplainer, Summary/Bar/Dependence Plot |

## 주요 결과

- **독립변수**: 정리정돈상태(OR=0.792, p=0.031)만 유의미 — 형식적 관리체계보다 현장의 실질적 행동이 중요
- **통제변수**: 공사규모, 기성공정률, 공사종류, 외국인비율이 사고발생에 가장 강한 영향
- **조절효과**: 인증보유 × 고용노동부감독 상호작용항 유의미(OR=2.081, p=0.022) — 선택편향으로 해석
- **최적 ML 모델**: Random Forest (F1=0.535, ROC-AUC=0.714)
- **SHAP Top-3**: 기성공정률 > 외국인비율 > 공사종류

## 실행 방법

Google Colab에서 순서대로 실행:

1. `notebooks/01_전처리.ipynb` — 원자료 전처리 및 `전처리_최종_v3.csv` 생성
2. `notebooks/02_데이터분석.ipynb` — 전체 분석 파이프라인 실행

> **데이터 경로**: 노트북은 `/content/` 경로 기준으로 작성되어 있습니다.
> Colab 사용 시 원자료와 `data/전처리_최종.csv`를 `/content/`에 업로드하세요.

## 파일 구조

```
Safety-13th-Thesis-Competition/
├── README.md
├── .gitignore
├── data/
│   └── 전처리_최종.csv              # 전처리 완료 데이터 (1,375 × 17)
├── notebooks/
│   ├── 01_전처리.ipynb              # 원자료 → 전처리_최종_v3.csv
│   └── 02_데이터분석.ipynb          # Phase 1~4 전체 분석
├── docs/
│   ├── 데이터분석_과정_및_근거.md   # 분석 설계 및 결과 해석
│   ├── 전처리_근거_및_과정.md       # 전처리 단계별 근거
│   └── 참고논문_정리.md             # 참고문헌 정리
└── results/
    └── (분석 실행 후 생성되는 이미지/CSV 파일)
```

## 주요 참고문헌

- Thabtah, F., et al. (2020). Integrated statistical modeling and machine learning techniques with SHAP for epidemiological data analysis. *Elsevier*. [KEY PAPER]
- Chawla, N. V., et al. (2002). SMOTE: Synthetic Minority Over-sampling Technique. *JAIR*, 16.
- Lundberg, S. M., & Lee, S. I. (2017). A Unified Approach to Interpreting Model Predictions. *NeurIPS*, 30.
- Obasi, S. N., et al. (2026). Machine learning for occupational accident analysis. *Safety Science*, Elsevier.
