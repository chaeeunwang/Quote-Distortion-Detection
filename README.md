# Quote Distortion Detection

> 뉴스 기사 속 인용문이 원 발언의 의미를 얼마나 왜곡했는지를 정량적으로 분석하는 AI 기반 의미 검증 시스템

---

## 프로젝트 개요

본 프로젝트는 뉴스 기사 내 인용문이 실제 발언의 의미를 얼마나 왜곡하고 있는지를 자동으로 판단하는 시스템입니다.  
기존의 팩트체크가 사실 여부에 집중했다면, 본 연구는 **의미적 왜곡(Semantic Distortion)** 자체를 정량화합니다.

### 문제 정의

외신 번역 과정에서 원문의 의미가 왜곡되는 사례가 빈번히 발생합니다.  
예: "Seems like a purge or revolution" → "부정선거로 정통성 부정" (완전히 다른 의미로 해석)

---

## 프로젝트 목표

- 인용문과 원문 간 의미 차이를 수치화
- 번역·요약 과정에서 발생하는 왜곡 자동 탐지
- 언론·미디어 분석에 활용 가능한 자동화 도구 구축

---

## 핵심 기능

- 인용문 자동 추출
- 원문 탐색 및 정렬
- 의미 유사도 계산 (SBERT)
- 왜곡 점수 산출 (0~100)
- Chrome Extension 기반 실시간 시각화

---

## 시스템 아키텍처

```
[Chrome Extension]
   └─ 인용문 감지 & 하이라이트
        ↓
[FastAPI Backend]
   ├─ 엔티티 추출 (KoELECTRA NER)
   ├─ 번역 (MarianMT) & 검색 쿼리 생성
   ├─ 원문 후보 검색 (Google CSE)
   ├─ 의미 유사도 계산 (SBERT)
   └─ 왜곡 점수 산출 (MPNet 분류)
        ↓
[Side Panel UI]
   └─ 결과 시각화 (원문, 유사도, 왜곡 점수)
```

---

## 기술 스택

- **Backend**: Python, FastAPI
- **NLP**: KoELECTRA, MPNet, Sentence-BERT
- **Frontend**: React, TypeScript, Chrome Extension
- **Infra**: Google Custom Search API

---

## 모델 구성

- **입력**: 인용문 + 원문 후보 (Sentence Pair)
- **아키텍처**: MPNet Transformer + [CLS] Pooling + Linear Classifier
- **출력**: 왜곡 확률 (0~100점)
- **학습**: 이진 분류 (정상/왜곡)
- **하이퍼파라미터**: lr=2e-5, batch=16, epochs=5

---

## 성능 요약

| Model | Accuracy | F1 Macro | AUC |
|------|----------|---------|-----|
| RoBERTa | 0.8516 | 0.8472 | 0.9391 |
| DeBERTa | 0.8667 | 0.8661 | 0.9422 |
| **MPNet** | **0.8677** | **0.8667** | **0.9383** |

**최종 모델**: MPNet (문장 간 의미 비교 성능 최적화)

### 데이터셋
- 총 4,646개 샘플 (Train: 3,717 / Validation: 929)
- Label 0 (정상): 2,510개
- Label 1 (왜곡): 2,136개
- GPT 기반 데이터 증강으로 라벨 불균형 해소

---

## 프로젝트 구조

```
quote-origin-pipeline/
├── qdd2/                    # 백엔드 핵심 모듈
│   ├── backend_api.py       # FastAPI 서버
│   ├── pipeline.py          # 메인 파이프라인
│   ├── quote_mining.py      # 왜곡 탐지 모델
│   └── ...
├── chrome_extension/        # Chrome Extension
│   ├── src/
│   │   ├── content-script.ts
│   │   └── side-panel.tsx
│   └── ...
├── QuoteMiningDetection/    # 학습된 모델
│   └── model_result/
└── scripts/                 # 유틸리티 스크립트
```

---

## 실행 방법

### Prerequisites

- Python 3.10+
- Node.js 18+
- Google Custom Search API 키 및 CSE ID

### Backend

```bash
# 1. 프로젝트 클론
cd quote-origin-pipeline

# 2. 의존성 설치
pip install -r requirements-api.txt

# 3. 환경 변수 설정 (.env 파일 생성)
GOOGLE_API_KEY=your_api_key
GOOGLE_CSE_ID=your_cse_id

# 4. 서버 실행
python run_server.py --port 8000
```

서버 실행 후 `http://localhost:8000/docs`에서 API 문서를 확인할 수 있습니다.

## 모델 가중치 다운로드 (필수)
본 프로젝트는 학습된 모델 가중치를 GitHub에 포함하지 않습니다.
아래 링크에서 모델 파일을 다운로드한 후 지정된 경로에 배치해야 합니다.
**경로** : quote-origin-pipeline/QuoteMiningDetection/model_result/
(https://drive.google.com/file/d/1Z2lMIgg8wdcLCBYrWVxng396cYzP75no/view?usp=sharing)



### Chrome Extension

```bash
# 1. Extension 디렉토리로 이동
cd chrome_extension

# 2. 의존성 설치
npm install

# 3. 빌드
npm run build

# 4. Chrome에 로드
# - chrome://extensions 접속
# - 개발자 모드 ON
# - "압축 해제된 확장 프로그램 로드" → chrome_extension 디렉토리 선택
```

---

## 역할 및 기여 (Team Contribution)

본 프로젝트는 팀 단위로 진행되었으며, 기획 단계부터 구현·검증까지의 전반적인 흐름을 제가 주도적으로 설계하고 조율했습니다.  
팀원들과 역할을 분담하여 협업하되, 전체 구조와 기술 방향성은 제가 중심이 되어 정리했습니다.

### 기획 및 방향 설정

- 문제 정의 및 프로젝트 목표 수립 주도
- "인용 왜곡 탐지"라는 문제를 기술적으로 해결 가능한 형태로 구조화
- 데이터 구성 방식, 모델 적용 방향, 전체 파이프라인 설계

### 데이터 및 모델링

- 데이터 수집·라벨링 기준 설계 및 가이드라인 작성
- 팀원들이 수집한 데이터 검수 및 품질 개선
- 모델 구조 선정(MPNet) 및 학습 전략 수립
- 성능 저하 원인 분석 및 개선 방향 제시

### 구현 및 통합

- 핵심 로직(의미 비교, 점수 산출) 직접 구현
- 백엔드 API 구조 설계 및 프론트엔드 연동 구조 정의
- 전체 시스템 통합 및 동작 검증

---

## 활용 가능 분야

- **팩트체킹 시스템**: 의미 단위 검증으로 기존 팩트체크 한계 보완
- **미디어 신뢰도 분석**: 뉴스 아카이브 기반 신뢰도 스코어링
- **학술 연구**: 저널리즘 재맥락화 이론과 NLP 기술 융합

## 확장 가능성

- 다국어 지원 (중국어, 일본어 등)
- 왜곡 유형 분류 (맥락 제거, 의미 변경 등)
- 문단·기사 단위 왜곡 스코어링

---

## 라이선스

본 프로젝트는 연구 및 학습 목적의 오픈소스입니다.
