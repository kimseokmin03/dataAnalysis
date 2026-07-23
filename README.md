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
QQQ(기술주), XLP(방어주), XLY(민감주), GC=F(금), BTC-USD, 부동산 관련 시계열 등

**거시 지표 (FRED)**  
기준금리(FEDFUNDS), CPI, GDP, 고용·심리·원자재·달러 인덱스 등

월 단위로 맞춰 병합한 뒤 `data/`에 저장해 두었습니다.  
대용량 raw(`finance_data.csv`)는 용량 때문에 git에 포함하지 않았습니다.

## 분석 흐름

1. **수집** — yfinance / FRED로 기간별 시계열 확보  
2. **EDA** — 금리 인하기를 구간으로 표시하고 자산 추이 비교  
3. **상관** — 지표 ↔ 자산 수익률 heatmap  
4. **모델** — 거시 지표 → 다음 달 자산 수익률 (Ridge / LightGBM)

### ML 실험 메모

- 타깃을 `shift(-1)` 해서 **다음 달 예측**으로 설정
- train/test는 시계열 순서 기준 분할 (셔플 없음)
- LightGBM test R² 예시: 대부분 **음수** (평균 예측보다 못함)

해석: 월별 샘플이 적고, 거시 정보가 이미 가격에 상당 부분 반영되어 있어 **단기 수익률 맞추기는 어렵다**에 가까움

## 실행 방법

```bash
# 1. 환경
python -m venv .venv
# Windows
.venv\Scripts\activate
pip install -r requirements.txt

# 2. (선택) 데이터 재수집 시 FRED 키
copy .env.example .env
# .env에 FRED_API_KEY 입력

# 3. 노트북
jupyter notebook
# notebooks/ 아래 01 → 06 순서 권장
```

이미 `data/`에 CSV가 있으면 **04~06**만으로도 상관·모델 재현이 가능합니다.  
01~03은 API로 다시 받을 때 사용합니다.

## 스택

Python, pandas, numpy, matplotlib, seaborn, yfinance, fredapi, scikit-learn, LightGBM, Jupyter

## 한계

- 월 데이터라 표본이 작음
- 인하기 구간 정의에 주관이 들어감
- 거래비용·레짐 전환 타이밍은 다루지 않음
- 예측 모델은 탐색용이며 실거래 신호가 아님

## 다음에 해볼 수 있는 것

- 인하기 안에서도 **집중하락기 vs 회복기** 수익률 표로 정리
- 동시 상관뿐 아니라 **lag 1~3** 상관 비교
- 예측 대신 **국면 분류**(인하기 여부) 문제로 바꾸기
