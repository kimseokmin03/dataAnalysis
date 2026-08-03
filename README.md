# Rate Cut Regime & Asset Behavior

금리 인하 국면에서 주요 자산군이 어떻게 움직이는지 살펴본 데이터 분석 프로젝트입니다.  
거시 지표로 다음 달 자산 수익률을 예측하는 ML 실험도 함께 진행했습니다.

## 질문

> 미국 기준금리 인하 구간에서 기술주·방어주·민감주·금·부동산·비트코인은 어떻게 다르게 반응하는가?  
> 거시 지표만으로 다음 달 자산 수익률을 얼마나 설명할 수 있는가?

## 요약

| 영역 | 내용 |
|------|------|
| 데이터 | Yahoo Finance (자산) + FRED (금리·거시지표) |
| 분석 | 인하기 구간 시각화, 자산–지표 상관관계 |
| 모델 | Ridge 회귀, LightGBM (next-month 예측) |
| 결론 | **국면·상관 분석이 더 유의미.** 단기 예측 R²는 대체로 음수 |


## 프로젝트 구조

```
├── notebooks/
│   ├── 01_collect_raw.ipynb      # 자산·금리 raw 수집
│   ├── 02_indicators.ipynb       # 거시 지표 수집·가공
│   ├── 03_eda.ipynb              # 금리 인하기 EDA
│   ├── 04_correlation.ipynb      # 자산–지표 상관·heatmap
│   ├── 05_regression.ipynb       # Ridge 회귀
│   └── 06_lgbm.ipynb             # LightGBM next-month 예측
├── data/                         # 분석용 CSV (일부)
├── .env.example
└── requirements.txt
```

## 데이터

**자산 (Yahoo Finance)**  
QQQ(기술주), XLP(방어주), XLY(민감주), GC=F(금), BTC-USD, 부동산 관련 시계열 등 - 종가, 일일 기준 병합

**거시 지표 (FRED)**  
기준금리(FEDFUNDS), CPI, GDP, 고용·심리·원자재·달러 인덱스 등 - 월 단위로 맞춰 병합

## 분석 흐름

1. **수집** — yfinance / FRED로 기간별 시계열 확보  
2. **EDA** — 금리 인하기를 구간으로 표시하고 자산 추이 비교  
3. **상관** — 지표 ↔ 자산 수익률 heatmap  
4. **모델** — 거시 지표 → 다음 달 자산 수익률 (Ridge / LightGBM / RandomForest)

### ML 실험 메모

- 타깃을 `shift(-1)` 해서 **다음 달 예측**으로 설정
- train/test는 시계열 순서 기준 분할 (셔플 없음)
- LightGBM test R² 예시: 대부분 **음수** (평균 예측보다 못함)

해석: 월별 샘플이 적고, 거시 정보가 이미 가격에 상당 부분 반영되어 있어 **단기 수익률 맞추기는 어렵다**에 가까움

## 스택

Python, pandas, numpy, matplotlib, seaborn, yfinance, fredapi, scikit-learn, LightGBM, Jupyter

## 한계

- 고빈도 데이터가 아니라 거시 공표 주기에 맞춰 데이터 수가 적음
- 상관분석에선 뚜렷한 관계는 공포지수 등 일부
- 머신러닝 다음달 예측의 R^2값이 대체로 음수 --> 예측 신호로는 부족
