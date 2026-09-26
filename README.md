# TaskLens — AI 과업 노출 탐색기

**직업 과업의 이론적 AI 노출과 Claude 대화에서 관측된 과업을 함께 살펴보는 탐색기**

🔗 https://pasmal0320-droid.github.io/

> AI는 직업이 아니라 과업에 닿는다. 노출은 대체가 아니고, 관측은 사용률이 아니다.

미국 직업 923개, O\*NET 과업 18,838개마다 다음 값을 과업 단위로 보여 주는 정적 웹페이지입니다.

| 지표 | 무엇인가 | 분모 · 출처 |
|---|---|---|
| **이론적 노출** | GPT-4가 평가한 "LLM이 이 과업 시간을 크게 줄일 수 있는가"(E0 / E2 / E1). 직업 값은 β = E1 비중 + 0.5 × E2 비중 | 그 직업의 과업 · Eloundou 외(2023) *GPTs are GPTs* |
| **Claude 대화 관측 비중** | Claude.ai 대화 전체 중 해당 과업으로 분류된 대화의 비중(%). 종사자의 AI 사용률이 **아님** | Claude 대화 표본 · Anthropic Economic Index 2025-03-27 |
| **자동화 / 보조** | 해당 과업 대화 중 AI에 맡기는 방식(지시·피드백 루프)과 함께 일하는 방식(반복·학습·검증)의 비율 | 분류 가능한 대화 · 같은 출처 |
| **관측 노출 (직업)** | Anthropic이 공개한 직업별 관측 노출 지표(`observed_exposure`)를 그대로 사용 | AEI labor_market_impacts 2026-03 |

과업 관측 비중은 직업 단위로 합산하지 않습니다. 같은 과업 문구가 여러 직업에 연결되기 때문입니다.

## 기능

- 직업 검색: 한국어·영문 직업명, SOC 코드, 보고 직함(예: "CPA")으로 찾고 자동완성 최대 8개
- 직업 상세: 과업마다 노출·관측 비중·자동화 막대. 값마다 분모·시점·출처 표기
- 과업 결합 상태 5분류: 관측 / 공유 과업 / 관측되지 않음 / 연결되지 않음 / 다중 매칭
- 검수 직업 5개(인사 담당자, 회계사, 소프트웨어 개발자, 간호사, 법률 보조원)는 "검수됨" 배지와 한국어 과업. 나머지는 "자동 결합·미검수"
- 공유 URL: `#occ=15-1252.00` 형식으로 같은 화면 열기
- (P1) 이론적 노출 vs 관측 노출 산점도, 직업 비교(`#compare=a,b,c`), 직군·숙련 수준 필터, 다크 모드 토글
- 모바일 대응, 시스템 다크 모드 자동 적용

## 파일 구성

```
index.html   페이지 본체 (HTML·CSS·JS 한 파일, 프레임워크 없음)
data.js      가공 데이터 (window.TASKLENS_DATA, 파일을 직접 열어도 동작)
data.json    같은 데이터의 JSON (data.js가 없을 때 fetch로 대체 사용)
README.md    이 문서
```

외부 리소스: Chart.js 4.4.1(cdnjs), Pretendard 웹폰트(jsDelivr). 서버·API 호출은 없습니다.

## 로컬에서 보기

`index.html`을 더블클릭해도 열립니다(`data.js` 사용). 로컬 서버로 확인하려면:

```bash
python -m http.server 8765
```

그다음 http://localhost:8765/ 를 엽니다.

## 데이터 가공 요약

가공 스크립트(`build.py`)와 원천 데이터는 저장소에 올리지 않고 로컬에 보관합니다. 규칙은 PRD v1.3 3.3절을 따릅니다.

1. 기준: O\*NET 31.0 과업(8자리 O\*NET-SOC 코드 + Task ID)
2. 이론적 노출: (코드, Task ID)로 GPT-4 라벨을 결합했습니다. `8823.0`처럼 소수형인 Task ID는 정수로 변환했습니다. 18,735개 과업이 연결됐습니다(99.5%).
3. Claude 대화 관측: AEI 동봉 과업표를 O\*NET-SOC 2010→2019 공식 대응표로 변환한 뒤 "직업 코드 + 정규화 문구"로 결합했습니다. 문구가 조금 바뀐 과업은 (코드, Task ID)로 보완했습니다(453건).

| 분류 | 과업 수 | 비율 |
|---|---|---|
| 관측 (exact) | 3,771 | 20.0% |
| 공유 과업 (shared) | 19 | 0.1% |
| 관측되지 않음 (unobserved) | 13,784 | 73.2% |
| 연결되지 않음 (unlinked) | 1,191 | 6.3% |
| 다중 매칭 (multi) | 73 | 0.4% |

4. 직업 단위: 관측 노출은 876/923개 직업, OEWS 고용·임금은 891/923개 직업에 연결됐습니다.

## 한계

1. 노출도는 GPT-4의 평가이며 일자리 대체를 뜻하지 않습니다.
2. 모든 데이터는 미국 직업 체계 기준입니다. 한국의 직무 구성과 다를 수 있습니다.
3. Claude 대화 관측 비중은 특정 서비스·기간의 대화 중 비중이며 직업 종사자의 AI 사용률이 아닙니다.
4. 일부 과업은 과업표 버전 차이로 연결되지 않았습니다(매칭률 위 표).
5. 옛 직업 코드(2010)는 공식 대응표로 변환했고, 나뉘거나 합쳐진 직업은 "코드 변경 대응"으로 표시했습니다.
6. 직업명·과업 한국어는 AI 기계 번역입니다. 검수 직업 5개 외에는 수동 검수를 거치지 않았습니다.
7. 교육 목적이며 커리어·투자·법률 조언이 아닙니다.

## 출처와 라이선스

| 데이터 | 버전 | 라이선스 |
|---|---|---|
| [O\*NET Database](https://www.onetcenter.org/database.html) | 31.0 (August 2026), 2010→2019 대응표 | CC BY 4.0. O\*NET 31.0 Database by U.S. Department of Labor, Employment and Training Administration (USDOL/ETA). 한국어 번역·가공은 작성자가 수행 |
| [GPTs are GPTs](https://github.com/openai/GPTs-are-GPTs) (Eloundou, Manning, Mishkin & Rock, 2023) | `full_labelset.tsv`, `occ_level.csv` — GPT-4 평가 기준 | MIT License, Copyright 2024 OpenAI |
| [Anthropic Economic Index](https://huggingface.co/datasets/Anthropic/EconomicIndex) | release 2025-03-27, labor_market_impacts | 데이터셋 카드 기재: 데이터 CC-BY, 코드 MIT |
| [BLS OEWS](https://www.bls.gov/oes/tables.htm) | May 2025 National | 미국 정부 공개 데이터 |

페이지 코드와 이 저장소의 가공 데이터는 위 원천 라이선스의 조건(출처 표기·변경 고지)을 따릅니다.

## 만든 사람

강병석 · KAIST 기술경영전문대학원 · ITM.690 인공지능 경영과 법 과제 1 (2026)
AI와 대화하며 데이터 결합부터 페이지 제작까지 진행한 바이브 코딩 프로젝트입니다.
