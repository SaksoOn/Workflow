# 🤖 사내 RAG 기반 AI 문서 검색 챗봇 시스템 - 프로젝트 기획서

**버전**: 1.2  
**작성일**: 2024-01-15  
**최종 수정일**: 2024-01-15 (문서 버전 관리 기능 추가)  
**프로젝트 타입**: 사내 전용 시스템 (On-Premise)

---

## 🆕 v1.2 주요 업데이트

이번 버전에서는 **문서 버전 관리 시스템**이 추가되었습니다:

✨ **새로운 기능:**
- **자동 버전 감지**: 동일 파일명 업로드 시 자동으로 버전 번호 부여
- **이전 버전 보존**: 모든 버전을 아카이브 폴더에 안전하게 보관
- **버전 히스토리 추적**: 각 문서의 변경 이력 완전 추적
- **롤백 기능**: 이전 버전으로 복구 가능
- **검색 결과 최적화**: 항상 최신 활성 버전만 검색

📝 **업데이트된 섹션:**
- 1.3 주요 기능 (문서 버전 관리 추가)
- 5.5 문서 버전 관리 워크플로우 (신규)
- 6.2 파일 시스템 구조 (archives 폴더 추가)
- 6.3.1 문서 메타데이터 (version 정보 추가)
- 8.1.3 관리자 페이지 - 버전 관리 UI (신규)
- FAQ 버전 관리 관련 질문 3개 추가 (Q17-Q19)

---

## 🆕 v1.1 주요 업데이트

**세션 기반 대화 관리 기능**이 추가되었습니다:

✨ **새로운 기능:**
- **멀티턴 대화 지원**: 사용자가 "그럼", "또", "그건" 등으로 이전 대화를 참조하는 후속 질문 가능
- **세션 자동 관리**: 브라우저 세션 ID를 통해 대화 맥락 1시간 자동 유지
- **서버 파일 기반 저장**: Redis 없이 간단한 파일 시스템으로 세션 관리
- **자동 정리**: 작업 스케줄러로 1시간 경과 세션 자동 삭제

📝 **업데이트된 섹션:**
- 5.4 세션 기반 대화 관리 워크플로우
- 6.2 파일 시스템 구조 (sessions 폴더 추가)
- 6.3.3 세션 데이터 구조
- 8.2.3 세션 관리 구현
- 9.2.1 일일 운영 (세션 정리 추가)
- FAQ 세션 관련 질문 3개 추가 (Q11-Q13)

---

## 📑 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [시스템 아키텍처](#2-시스템-아키텍처)
3. [하드웨어 스펙 (권장 사양)](#3-하드웨어-스펙-권장-사양)
4. [기술 스택](#4-기술-스택)
5. [워크플로우 상세 설계](#5-워크플로우-상세-설계)
6. [데이터베이스 설계](#6-데이터베이스-설계)
7. [보안 및 접근 제어](#7-보안-및-접근-제어)
8. [사용자 인터페이스](#8-사용자-인터페이스)
9. [배포 및 운영 계획](#9-배포-및-운영-계획)
10. [예상 성능 지표](#10-예상-성능-지표)
11. [비용 분석](#11-비용-분석)
12. [리스크 관리](#12-리스크-관리)
13. [FAQ](#13-faq)

---

## 1. 프로젝트 개요

### 1.1 프로젝트 배경
회사 내부 문서(매뉴얼, 정책, 가이드 등)가 산재되어 있어 직원들이 필요한 정보를 찾는 데 시간이 소요되는 문제를 해결하기 위한 AI 기반 검색 시스템 구축

### 1.2 프로젝트 목표
- **주요 목표**: RAG(Retrieval-Augmented Generation) 기술을 활용하여 사내 문서를 학습한 AI 챗봇 구축
- **부가 목표**: 
  - 관리자가 쉽게 문서를 등록/관리할 수 있는 웹 인터페이스 제공
  - 사내 모든 직원이 사내 네트워크에서 접근 가능한 챗봇 서비스
  - 질의응답 히스토리 관리 기능

### 1.3 주요 기능

#### 관리자 기능
- 문서 업로드 (PDF, DOCX, TXT, Markdown, Excel, PPT, CSV)
- **문서 버전 관리** ⭐ NEW
  - 동일 파일명 자동 버전 번호 부여
  - 이전 버전 아카이브 보존
  - 버전 히스토리 조회
  - 이전 버전 복구/활성화
- 문서 메타데이터 관리
- 벡터화 진행 상태 확인
- 질의응답 히스토리 조회
  - 세션 ID별 연속 대화 추적
  - 키워드/IP/날짜 검색
  - CSV 다운로드
- 통계 대시보드 (선택사항)

#### 사용자 기능
- 자연어 질문 입력
- 실시간 AI 답변 수신
- 답변 출처 문서 확인
- **세션 기반 연속 대화** (멀티턴 지원)
  - "그럼", "그건", "또" 등 후속 질문 가능
  - 이전 대화 맥락 자동 유지 (1시간)
- 간단한 질문 기록 조회

### 1.4 프로젝트 범위
- **포함**: n8n 워크플로우 개발, Docker 기반 배포, 웹 UI, 문서 처리 파이프라인
- **제외**: 모바일 앱, 외부 네트워크 접근, 다국어 지원 (초기 버전)

---

## 2. 시스템 아키텍처

### 2.1 전체 시스템 구성도

```
┌─────────────────────────────────────────────────────────────┐
│                    사내 네트워크 (192.168.x.x)                 │
│                                                               │
│  ┌──────────────┐      ┌──────────────┐      ┌──────────────┐│
│  │ 관리자 PC    │      │  직원 PC 1   │      │  직원 PC N   ││
│  │              │      │              │      │              ││
│  └──────┬───────┘      └──────┬───────┘      └──────┬───────┘│
│         │                     │                     │        │
│         └─────────────────────┼─────────────────────┘        │
│                               │                              │
│                               ▼                              │
│         ┌─────────────────────────────────────────┐          │
│         │     Windows Server (192.168.x.x)       │          │
│         │                                         │          │
│         │  ┌────────────────────────────────┐    │          │
│         │  │     Docker Environment         │    │          │
│         │  │                                │    │          │
│         │  │  ┌──────────────────────────┐  │    │          │
│         │  │  │   n8n (:5678)            │  │    │          │
│         │  │  │   - Workflow Engine      │  │    │          │
│         │  │  │   - Webhook Endpoints    │  │    │          │
│         │  │  └──────────┬───────────────┘  │    │          │
│         │  │             │                  │    │          │
│         │  │  ┌──────────▼───────────────┐  │    │          │
│         │  │  │   Ollama (:11434)        │  │    │          │
│         │  │  │   - exaone3.5 7B (LLM)  │  │    │          │
│         │  │  │   - nomic-embed-text     │  │    │          │
│         │  │  └──────────┬───────────────┘  │    │          │
│         │  │             │                  │    │          │
│         │  │  ┌──────────▼───────────────┐  │    │          │
│         │  │  │   Qdrant (:6333)         │  │    │          │
│         │  │  │   - Vector Database      │  │    │          │
│         │  │  │   - 768차원 벡터 저장    │  │    │          │
│         │  │  └──────────────────────────┘  │    │          │
│         │  │                                │    │          │
│         │  └────────────────────────────────┘    │          │
│         │                                         │          │
│         │  ┌────────────────────────────────┐    │          │
│         │  │   파일 시스템                   │    │          │
│         │  │   C:\n8n-data\                │    │          │
│         │  │   ├── uploads\  (원본 문서)   │    │          │
│         │  │   └── history\  (질의응답)    │    │          │
│         │  └────────────────────────────────┘    │          │
│         │                                         │          │
│         └─────────────────────────────────────────┘          │
│                                                               │
└───────────────────────────────────────────────────────────────┘
```

### 2.2 데이터 흐름

#### 문서 업로드 플로우
```
관리자 → 웹 페이지 → n8n Webhook → 파일 저장 → 텍스트 추출 
→ 청크 분할 → Ollama 임베딩 → Qdrant 저장 → 완료 알림
```

#### 질의응답 플로우 (세션 관리 포함)
```
사용자 → 질문 입력 → n8n Webhook 
→ 세션 ID 확인/생성 → 이전 대화 불러오기 (선택)
→ Ollama 질문 벡터화 → Qdrant 유사도 검색 
→ 컨텍스트 구성 (이전 대화 + 검색 문서 + 현재 질문)
→ Ollama LLM 추론 → 답변 생성 
→ 세션 데이터 업데이트 (1시간 TTL)
→ 영구 히스토리 저장 (감사용)
→ 답변 반환
```

---

## 3. 하드웨어 스펙 (권장 사양)

### 3.1 선택된 시나리오: **시나리오 2 - 권장 사양 (CPU 전용)**

#### 3.1.1 하드웨어 구성

| 구성요소 | 스펙 | 용도 | 예상 가격 |
|---------|------|------|----------|
| **CPU** | AMD EPYC 7313P (16코어/32쓰레드) 또는<br>Intel Xeon Silver 4314 (16코어/32쓰레드) 또는<br>Intel Core i9-12900 (16코어/24쓰레드) | LLM 추론, 임베딩 생성 | 60-90만원 |
| **RAM** | 64GB DDR4 ECC (3200MHz)<br>32GB x 2 듀얼 채널 | 모델 로딩, 캐시, Docker | 30-40만원 |
| **SSD** | 500GB NVMe Gen3<br>Samsung 980 Pro / WD Black SN850 | OS, Docker, 모델, 문서 | 8-10만원 |
| **HDD** | 2TB SATA 7200RPM<br>Seagate IronWolf | 백업, 장기 저장 | 10만원 |
| **메인보드** | Supermicro X12STL-IF (Xeon용)<br>또는 ASUS Pro WS (Ryzen용) | 서버급 안정성 | 35-40만원 |
| **전원** | 550W 80+ Gold<br>Seasonic Focus GX-550 | 안정적 전력 공급 | 10만원 |
| **쿨링** | 타워형 CPU 쿨러<br>Noctua NH-U12S | CPU 발열 관리 | 7만원 |
| **케이스** | 서버/워크스테이션 케이스<br>Fractal Design Define 7 | 소음 차단, 확장성 | 20만원 |

**총 예상 비용**: 약 **220-260만원**

#### 3.1.2 소프트웨어 라이선스

| 항목 | 라이선스 | 가격 |
|------|---------|------|
| **Windows Server 2022 Standard** | 유료 | 100만원 |
| **Windows 10/11 Pro** (대체) | 유료 | 20만원 |
| **Docker Desktop** | 무료 (소규모 비상업적) | 0원 |
| **n8n** | 자체 호스팅 무료 | 0원 |
| **Ollama** | 오픈소스 | 0원 |
| **Qdrant** | 오픈소스 | 0원 |

**소프트웨어 비용**: 20-100만원 (OS 선택에 따라)

#### 3.1.3 리소스 사용량 예측

```yaml
CPU 사용률:
  - 유휴 시: 10-15%
  - 질의응답 처리 시: 80-95%
  - 문서 업로드/임베딩 시: 90-100%

RAM 사용량:
  - OS + Docker: 6GB
  - n8n: 1GB
  - Ollama (exaone3.5 로딩): 8-10GB
  - Qdrant: 2-3GB
  - 버퍼: 10GB
  - 총 사용량: 27-30GB / 64GB (여유 34GB)

디스크 사용량:
  - OS: 40GB
  - Docker 이미지: 10GB
  - Ollama 모델: 5GB
  - Qdrant 데이터: 5GB (문서 100개 기준)
  - 원본 문서: 500MB
  - 로그/히스토리: 1GB
  - 총 사용량: 61.5GB / 500GB (여유 438GB)

전력 소비:
  - 평균 사용: 150-200W
  - 최대 부하: 300W
  - 월 전기료 (24시간 운영): 약 3-4만원
```

### 3.2 성능 예측

#### 3.2.1 응답 속도

| 작업 | 예상 소요 시간 |
|------|---------------|
| 문서 업로드 (100페이지 PDF) | 30-60초 |
| 벡터 임베딩 생성 (1,000 청크) | 2-3분 |
| 질문 벡터화 | 0.5초 |
| Qdrant 유사도 검색 | 0.3-0.5초 |
| exaone3.5 답변 생성 | 4-6초 |
| **총 질의응답 시간** | **5-8초** |

#### 3.2.2 동시 처리 능력

```
동시 사용자 수: 10-15명
처리 방식: 큐 기반 순차 처리
대기 시간: 
  - 1명 사용 중: 0초
  - 5명 대기 중: 평균 15-20초
  - 10명 대기 중: 평균 40-50초

권장 사용 패턴: 피크 시간대 분산 권장
```

---

## 4. 기술 스택

### 4.1 핵심 기술

#### 4.1.1 워크플로우 엔진
- **n8n** (v1.x)
  - 노코드/로우코드 워크플로우 자동화
  - 525+ 통합 노드 지원
  - 자체 호스팅 가능
  - Webhook 기반 API 제공

#### 4.1.2 대규모 언어 모델 (LLM)
- **Ollama** (v0.x)
  - 로컬 LLM 실행 프레임워크
  - **모델**: exaone3.5 7B
  - **특징**: 한국어 지원, 상업적 사용 가능
  - **API**: REST API (OpenAI 호환)

#### 4.1.3 임베딩 모델
- **nomic-embed-text** (via Ollama)
  - **차원**: 768차원
  - **컨텍스트 길이**: 8,192 토큰
  - **특징**: 다국어 지원, 고품질 임베딩

#### 4.1.4 벡터 데이터베이스
- **Qdrant** (v1.x)
  - 고성능 벡터 검색 엔진
  - 완전 무료 오픈소스
  - 웹 UI 대시보드 제공
  - **검색 방식**: Cosine Similarity
  - **인덱싱**: HNSW 알고리즘

### 4.2 지원 문서 형식

| 형식 | 처리 라이브러리 | 추출 방법 |
|------|----------------|----------|
| **PDF** | pdf-parse (Node.js) | 텍스트 레이어 추출 |
| **DOCX** | mammoth.js | HTML 변환 후 텍스트 추출 |
| **TXT** | 내장 | 직접 읽기 |
| **Markdown** | 내장 | 직접 읽기 |
| **Excel** | xlsx (Node.js) | 시트별 데이터 추출 |
| **PPT** | python-pptx (Python) | 슬라이드별 텍스트 추출 |
| **CSV** | 내장 | 직접 읽기 |

### 4.3 개발 환경

```yaml
컨테이너 플랫폼: Docker Desktop for Windows
오케스트레이션: Docker Compose
버전 관리: Git
문서화: Markdown
모니터링: n8n 내장 로그, Qdrant 대시보드
```

---

## 5. 워크플로우 상세 설계

### 5.1 관리자 문서 업로드 워크플로우

#### 5.1.1 워크플로우 다이어그램

```
[Webhook Trigger] 
  ↓ (POST /webhook/admin-upload)
[IP 검증 노드]
  ↓ (사내 IP만 허용)
[파일 저장 노드]
  ↓ (C:\n8n-data\uploads\{doc_id}\)
[파일 형식 감지]
  ↓
[텍스트 추출 노드] (형식별 분기)
  ├─ PDF → pdf-parse
  ├─ DOCX → mammoth
  ├─ Excel → xlsx
  ├─ PPT → python-pptx
  └─ TXT/MD/CSV → 직접 읽기
  ↓
[청크 분할 노드] (Code)
  ↓ (512 토큰, 오버랩 50)
[Loop 노드] (각 청크 반복)
  ↓
[Ollama 임베딩 노드]
  ↓ (POST /api/embeddings)
[Qdrant 저장 노드]
  ↓ (PUT /collections/documents/points)
[완료 알림]
  ↓
[Webhook Response] "✅ 업로드 완료"
```

#### 5.1.2 주요 노드 설정

**1) Webhook Trigger**
```javascript
// 설정
Method: POST
Path: /webhook/admin-upload
Authentication: None (IP로 제한)
Response Mode: When Last Node Finishes
```

**2) IP 검증 노드 (Code)**
```javascript
const allowedSubnets = ['192.168.1.', '192.168.2.', '10.0.0.'];
const clientIP = $request.headers['x-forwarded-for'] || $request.ip;

if (!allowedSubnets.some(subnet => clientIP.startsWith(subnet))) {
    $response.status(403).json({
        error: '⛔ 사내 네트워크에서만 접속 가능합니다.'
    });
    return [];
}

return [{ json: { ip: clientIP, allowed: true } }];
```

**3) 파일 저장 노드 (Code)**
```javascript
const fs = require('fs');
const path = require('path');

const file = $input.item.binary.file;
const docId = `doc_${Date.now()}`;
const uploadDir = `C:\\n8n-data\\uploads\\${docId}`;

fs.mkdirSync(uploadDir, { recursive: true });
const filePath = path.join(uploadDir, file.fileName);
fs.writeFileSync(filePath, file.data);

return [{
    json: {
        doc_id: docId,
        filename: file.fileName,
        filepath: filePath,
        size: file.data.length,
        upload_date: new Date().toISOString()
    }
}];
```

**4) 청크 분할 노드 (Code)**
```javascript
const text = $input.item.json.text;
const maxTokens = 512;
const overlap = 50;

// 간단한 토큰화 (공백 기준)
const words = text.split(/\s+/);
const chunks = [];

for (let i = 0; i < words.length; i += maxTokens - overlap) {
    const chunk = words.slice(i, i + maxTokens).join(' ');
    chunks.push({
        chunk_id: chunks.length + 1,
        text: chunk,
        start_index: i,
        end_index: i + maxTokens
    });
}

return chunks.map(chunk => ({ json: { ...chunk, ...($input.item.json) } }));
```

**5) Ollama 임베딩 노드 (HTTP Request)**
```yaml
Method: POST
URL: http://localhost:11434/api/embeddings
Headers:
  Content-Type: application/json
Body:
  {
    "model": "nomic-embed-text",
    "prompt": "={{$json.text}}"
  }
Response: 
  vector: {{$json.embedding}}
```

**6) Qdrant 저장 노드 (HTTP Request)**
```yaml
Method: PUT
URL: http://localhost:6333/collections/documents/points
Headers:
  Content-Type: application/json
Body:
  {
    "points": [
      {
        "id": "={{$json.doc_id}}_{{$json.chunk_id}}",
        "vector": "={{$json.vector}}",
        "payload": {
          "doc_id": "={{$json.doc_id}}",
          "chunk_id": "={{$json.chunk_id}}",
          "filename": "={{$json.filename}}",
          "text": "={{$json.text}}",
          "upload_date": "={{$json.upload_date}}"
        }
      }
    ]
  }
```

---

### 5.2 사용자 챗봇 워크플로우

#### 5.2.1 워크플로우 다이어그램

```
[Webhook Trigger]
  ↓ (POST /webhook/chat)
[IP 검증 노드]
  ↓
[질문 수신]
  ↓
[Ollama 질문 벡터화]
  ↓ (nomic-embed-text)
[Qdrant 유사도 검색]
  ↓ (상위 5개 청크)
[컨텍스트 구성 노드] (Code)
  ↓ (프롬프트 생성)
[Ollama LLM 추론]
  ↓ (exaone3.5)
[답변 후처리]
  ↓
[히스토리 저장] (Code)
  ↓
[Webhook Response] (JSON)
```

#### 5.2.2 주요 노드 설정

**1) Qdrant 검색 노드 (HTTP Request)**
```yaml
Method: POST
URL: http://localhost:6333/collections/documents/points/search
Headers:
  Content-Type: application/json
Body:
  {
    "vector": "={{$json.question_vector}}",
    "limit": 5,
    "with_payload": true,
    "with_vector": false
  }
```

**2) 컨텍스트 구성 노드 (Code)**
```javascript
const searchResults = $input.all();
const question = $('Webhook').item.json.question;

// 검색된 문서 컨텍스트 구성
const context = searchResults
    .map((item, idx) => `[문서 ${idx + 1}]\n${item.json.payload.text}`)
    .join('\n\n');

// 프롬프트 구성
const prompt = `당신은 회사 문서 전문가 AI 어시스턴트입니다.
아래 문서를 참고하여 질문에 정확히 답변해주세요.

[규칙]
1. 문서에 정보가 있으면 정확히 답변하세요
2. 문서에 없는 정보는 "해당 정보를 찾을 수 없습니다"라고 답변하세요
3. 답변 마지막에 출처 문서명을 표시하세요
4. 친절하고 전문적인 톤을 유지하세요

[참고 문서]
${context}

[질문]
${question}

[답변]`;

return [{
    json: {
        prompt: prompt,
        question: question,
        source_docs: searchResults.map(r => r.json.payload.filename)
    }
}];
```

**3) Ollama LLM 노드 (HTTP Request)**
```yaml
Method: POST
URL: http://localhost:11434/api/generate
Headers:
  Content-Type: application/json
Body:
  {
    "model": "exaone3.5",
    "prompt": "={{$json.prompt}}",
    "stream": false,
    "options": {
      "temperature": 0.3,
      "top_p": 0.9,
      "top_k": 40
    }
  }
```

**4) 히스토리 저장 노드 (Code)**
```javascript
const fs = require('fs');
const path = require('path');

const historyDir = 'C:\\n8n-data\\history';
const timestamp = new Date().toISOString().replace(/[:.]/g, '-');
const filename = `${timestamp}.json`;

const record = {
    timestamp: new Date().toISOString(),
    user_ip: $('Webhook').item.json.headers['x-forwarded-for'],
    question: $json.question,
    answer: $json.response,
    source_docs: $json.source_docs,
    response_time_ms: $json.elapsed_time
};

fs.mkdirSync(historyDir, { recursive: true });
fs.writeFileSync(
    path.join(historyDir, filename),
    JSON.stringify(record, null, 2)
);

return [{ json: record }];
```

---

### 5.3 관리자 히스토리 조회 워크플로우

#### 5.3.1 워크플로우 다이어그램

```
[Webhook Trigger]
  ↓ (GET /webhook/admin-history)
[IP 검증 노드]
  ↓
[히스토리 파일 읽기] (Code)
  ↓ (C:\n8n-data\history\*.json)
[정렬 및 필터링]
  ↓ (최신순, 페이지네이션)
[HTML 테이블 생성]
  ↓
[Webhook Response] (HTML)
```

#### 5.3.2 히스토리 읽기 노드 (Code)

```javascript
const fs = require('fs');
const path = require('path');

const historyDir = 'C:\\n8n-data\\history';
const files = fs.readdirSync(historyDir)
    .filter(f => f.endsWith('.json'))
    .sort()
    .reverse(); // 최신순

const limit = 50; // 최근 50개
const records = files
    .slice(0, limit)
    .map(file => {
        const content = fs.readFileSync(path.join(historyDir, file), 'utf8');
        return JSON.parse(content);
    });

return records.map(record => ({ json: record }));
```

---

### 5.4 세션 기반 대화 관리 워크플로우 ⭐ NEW

#### 5.4.1 세션 관리 개요

세션 기반 대화 관리는 사용자가 **연속적인 질문**(멀티턴)을 할 수 있도록 이전 대화의 맥락을 유지하는 기능입니다.

**작동 원리:**
```
👤: "2024년 휴가 정책은?"
🤖: "15일이 부여됩니다..."

👤: "그럼 신청은 어떻게 해?" ← "휴가 신청" 맥락 자동 유지
🤖: "인사시스템에서 신청하시면 됩니다..."

👤: "사용 기한은?" ← "휴가 사용 기한" 맥락 유지
🤖: "해당 연도 말까지 사용해야 합니다..."
```

#### 5.4.2 세션 관리 방식

**옵션 2: 서버 파일 기반 세션 관리** (선택됨)

| 항목 | 설명 |
|------|------|
| **저장 위치** | `C:\n8n-data\sessions\` |
| **파일 형식** | `session_{timestamp}_{random}.json` |
| **세션 유지** | 1시간 (마지막 접근 시간 기준) |
| **정리 방법** | 자동 정리 스크립트 (매시간 실행) |
| **최대 메시지** | 최근 10개 메시지 (5턴) |

#### 5.4.3 워크플로우 다이어그램 (업데이트됨)

```
[Webhook Trigger]
  ↓ (POST /webhook/chat)
[IP 검증 노드]
  ↓
[세션 관리 노드] ⭐ NEW
  ├─ 세션 ID 확인 (요청 헤더/쿠키)
  ├─ 없으면 새 세션 생성
  └─ 세션 파일 읽기 (C:\n8n-data\sessions\)
  ↓
[질문 수신]
  ↓
[Ollama 질문 벡터화]
  ↓ (nomic-embed-text)
[Qdrant 유사도 검색]
  ↓ (상위 5개 청크)
[컨텍스트 구성 노드] ⭐ 업데이트
  ├─ 이전 대화 (세션에서, 최근 3턴)
  ├─ 검색된 문서 (Qdrant에서)
  └─ 현재 질문
  ↓
[Ollama LLM 추론]
  ↓ (exaone3.5)
[답변 후처리]
  ↓
[데이터 저장] ⭐ 업데이트
  ├─ 세션 데이터 업데이트 (임시, 1시간)
  └─ 영구 히스토리 저장 (감사용)
  ↓
[Webhook Response] (JSON)
```

#### 5.4.4 주요 노드 설정

**1) 세션 관리 노드 (Code)**

```javascript
const fs = require('fs');
const path = require('path');

// 세션 ID 확인/생성
let sessionId = $json.session_id || $request.headers['x-session-id'];
if (!sessionId) {
    sessionId = `session_${Date.now()}_${Math.random().toString(36).substring(7)}`;
}

const sessionDir = 'C:\\n8n-data\\sessions';
const sessionFile = path.join(sessionDir, `${sessionId}.json`);

// 세션 디렉토리 생성
fs.mkdirSync(sessionDir, { recursive: true });

// 기존 세션 불러오기
let sessionData = { messages: [], created_at: new Date().toISOString() };
if (fs.existsSync(sessionFile)) {
    try {
        sessionData = JSON.parse(fs.readFileSync(sessionFile, 'utf8'));
    } catch (error) {
        console.error('Session read error:', error);
        // 손상된 세션 파일은 새로 시작
    }
}

// 세션 메타데이터 업데이트
sessionData.last_access = new Date().toISOString();

return [{
    json: {
        session_id: sessionId,
        question: $json.question,
        conversation_history: sessionData.messages.slice(-6), // 최근 3턴(6개 메시지)
        session_data: sessionData
    }
}];
```

**2) 컨텍스트 구성 노드 (세션 포함) (Code)**

```javascript
const searchResults = $input.all();
const question = $('Webhook').item.json.question;
const conversationHistory = $json.conversation_history || [];

// 이전 대화 컨텍스트 구성
const previousConversation = conversationHistory.length > 0
    ? conversationHistory
        .map(msg => `${msg.role === 'user' ? '사용자' : 'AI'}: ${msg.content}`)
        .join('\n')
    : '';

// 검색된 문서 컨텍스트 구성
const documentContext = searchResults
    .map((item, idx) => `[문서 ${idx + 1}]\n제목: ${item.json.payload.filename}\n내용: ${item.json.payload.text}`)
    .join('\n\n');

// 프롬프트 구성
const prompt = `당신은 회사 문서 전문가 AI 어시스턴트입니다.
아래 정보를 참고하여 질문에 정확히 답변해주세요.

[규칙]
1. 문서에 정보가 있으면 정확히 답변하세요
2. 문서에 없는 정보는 "해당 정보를 찾을 수 없습니다"라고 답변하세요
3. 이전 대화의 맥락을 고려하여 자연스럽게 답변하세요
4. 답변 마지막에 출처 문서명을 표시하세요
5. 친절하고 전문적인 톤을 유지하세요

${previousConversation ? `[이전 대화]\n${previousConversation}\n` : ''}
[참고 문서]
${documentContext}

[현재 질문]
${question}

[답변]`;

return [{
    json: {
        prompt: prompt,
        question: question,
        session_id: $json.session_id,
        source_docs: searchResults.map(r => r.json.payload.filename),
        conversation_history: conversationHistory
    }
}];
```

**3) 세션 데이터 업데이트 노드 (Code)**

```javascript
const fs = require('fs');
const path = require('path');

const sessionId = $json.session_id;
const sessionDir = 'C:\\n8n-data\\sessions';
const sessionFile = path.join(sessionDir, `${sessionId}.json`);

// 세션 데이터 읽기
let sessionData = JSON.parse(fs.readFileSync(sessionFile, 'utf8'));

// 새 메시지 추가
sessionData.messages.push({
    role: 'user',
    content: $json.question,
    timestamp: new Date().toISOString()
});

sessionData.messages.push({
    role: 'assistant',
    content: $json.answer,
    timestamp: new Date().toISOString()
});

// 최근 10개 메시지만 유지 (5턴)
sessionData.messages = sessionData.messages.slice(-10);

// 마지막 접근 시간 업데이트
sessionData.last_access = new Date().toISOString();

// 세션 저장
fs.writeFileSync(sessionFile, JSON.stringify(sessionData, null, 2));

return [{
    json: {
        session_id: sessionId,
        message_count: sessionData.messages.length
    }
}];
```

#### 5.4.5 세션 정리 스크립트

**자동 정리 스크립트 (PowerShell)**

```powershell
# cleanup-sessions.ps1
# 작업 스케줄러로 매시간 실행

$sessionDir = "C:\n8n-data\sessions"
$maxAge = (Get-Date).AddHours(-1)  # 1시간 이상 된 세션 삭제

if (Test-Path $sessionDir) {
    Get-ChildItem $sessionDir -Filter "*.json" | ForEach-Object {
        try {
            $session = Get-Content $_.FullName | ConvertFrom-Json
            $lastAccess = [DateTime]$session.last_access
            
            if ($lastAccess -lt $maxAge) {
                Remove-Item $_.FullName -Force
                Write-Host "✅ Deleted expired session: $($_.Name)"
            }
        } catch {
            # 손상된 파일 삭제
            Remove-Item $_.FullName -Force
            Write-Host "⚠️ Deleted corrupted session: $($_.Name)"
        }
    }
    
    $remainingSessions = (Get-ChildItem $sessionDir -Filter "*.json").Count
    Write-Host "📊 Active sessions: $remainingSessions"
} else {
    Write-Host "❌ Session directory not found: $sessionDir"
}
```

**작업 스케줄러 등록 (PowerShell)**

```powershell
# 관리자 권한으로 실행
$action = New-ScheduledTaskAction -Execute "PowerShell.exe" `
    -Argument "-File C:\n8n-scripts\cleanup-sessions.ps1"

$trigger = New-ScheduledTaskTrigger -Once -At (Get-Date) `
    -RepetitionInterval (New-TimeSpan -Hours 1) `
    -RepetitionDuration ([TimeSpan]::MaxValue)

$principal = New-ScheduledTaskPrincipal -UserId "SYSTEM" `
    -LogonType ServiceAccount -RunLevel Highest

Register-ScheduledTask -TaskName "n8n-SessionCleanup" `
    -Action $action `
    -Trigger $trigger `
    -Principal $principal `
    -Description "n8n 챗봇 세션 정리 (1시간마다)"

Write-Host "✅ 작업 스케줄러 등록 완료"
```

#### 5.4.6 세션 관리 장점

| 장점 | 설명 |
|------|------|
| **자연스러운 대화** | "그럼", "그건", "또" 등 후속 질문 가능 |
| **맥락 유지** | 이전 대화를 기억하여 더 정확한 답변 |
| **사용자 경험 향상** | 매번 전체 맥락을 다시 설명할 필요 없음 |
| **서버 부담 최소화** | 파일 기반으로 간단, Redis 불필요 |
| **디버깅 용이** | 세션 파일을 직접 열어 확인 가능 |

---

### 5.5 문서 버전 관리 워크플로우 ⭐ NEW

#### 5.5.1 버전 관리 개요

문서 버전 관리는 동일한 파일명으로 문서를 재업로드할 때 **자동으로 버전을 관리**하는 기능입니다.

**작동 원리:**
```
첫 업로드:
📄 회사정책.pdf (v1) → uploads/doc_xxx/

재업로드:
📄 회사정책.pdf → 
  ├─ 기존 v1 → archives/doc_xxx_v1/ (보관)
  └─ 신규 v2 → uploads/doc_yyy/ (활성)

세 번째 업로드:
📄 회사정책.pdf → 
  ├─ 기존 v2 → archives/doc_yyy_v2/ (보관)
  └─ 신규 v3 → uploads/doc_zzz/ (활성)
```

#### 5.5.2 버전 관리 워크플로우 다이어그램

```
[Webhook Trigger]
  ↓ (POST /webhook/admin-upload)
[IP 검증 노드]
  ↓
[파일 수신]
  ↓
[버전 체크 노드] ⭐ NEW
  ├─ 동일 파일명 검색
  ├─ 기존 버전 있음?
  │   ├─ YES → 버전 번호 증가 (v2, v3, ...)
  │   └─ NO  → 신규 문서 (v1)
  ↓
[기존 버전 아카이브] ⭐ NEW (버전이 있을 경우)
  ├─ 파일 이동: uploads → archives
  ├─ Qdrant 업데이트: active = false
  └─ 메타데이터 기록
  ↓
[새 버전 저장]
  ↓ (uploads 폴더에)
[텍스트 추출]
  ↓
[청크 분할]
  ↓
[Loop 노드]
  ↓
[Ollama 임베딩]
  ↓
[Qdrant 저장] ⭐ 업데이트
  └─ payload에 { active: true, version: X } 추가
  ↓
[완료 알림]
```

#### 5.5.3 주요 노드 설정

**1) 버전 체크 노드 (Code)**

```javascript
const fs = require('fs');
const path = require('path');

const newFilename = $json.filename;
const uploadDir = 'C:\\n8n-data\\uploads';

// 기존 문서 검색
let existingDoc = null;
let currentVersion = 0;

const existingDocs = fs.readdirSync(uploadDir);
for (const docDir of existingDocs) {
    const metadataPath = path.join(uploadDir, docDir, 'metadata.json');
    if (fs.existsSync(metadataPath)) {
        const metadata = JSON.parse(fs.readFileSync(metadataPath, 'utf8'));
        
        // 파일명 일치 확인
        if (metadata.filename === newFilename && metadata.active === true) {
            existingDoc = metadata;
            currentVersion = metadata.version || 1;
            break;
        }
    }
}

const newVersion = currentVersion + 1;

return [{
    json: {
        ...($json),
        existing_doc: existingDoc,
        new_version: newVersion,
        is_update: existingDoc !== null
    }
}];
```

**2) 기존 버전 아카이브 노드 (Code)**

```javascript
const fs = require('fs');
const path = require('path');

// 업데이트인 경우만 실행
if (!$json.is_update) {
    return [{ json: { message: 'No archiving needed (new document)' } }];
}

const existingDoc = $json.existing_doc;
const uploadDir = 'C:\\n8n-data\\uploads';
const archiveDir = 'C:\\n8n-data\\archives';

// 아카이브 디렉토리 생성
fs.mkdirSync(archiveDir, { recursive: true });

// 기존 문서 경로
const oldDocPath = path.join(uploadDir, existingDoc.doc_id);
const archivePath = path.join(archiveDir, `${existingDoc.doc_id}_v${existingDoc.version}`);

// 파일 이동
fs.renameSync(oldDocPath, archivePath);

// 메타데이터 업데이트 (아카이브 시간 기록)
const metadataPath = path.join(archivePath, 'metadata.json');
const metadata = JSON.parse(fs.readFileSync(metadataPath, 'utf8'));
metadata.active = false;
metadata.archived_date = new Date().toISOString();
fs.writeFileSync(metadataPath, JSON.stringify(metadata, null, 2));

// Qdrant 업데이트 (active = false로 설정)
const updateUrl = 'http://qdrant:6333/collections/documents/points/payload';
const updatePayload = {
    filter: {
        must: [
            { key: "doc_id", match: { value: existingDoc.doc_id } }
        ]
    },
    payload: {
        active: false,
        archived_date: new Date().toISOString()
    }
};

await $http.post(updateUrl, updatePayload);

return [{
    json: {
        message: 'Archived successfully',
        archived_doc_id: existingDoc.doc_id,
        archived_version: existingDoc.version,
        archive_path: archivePath
    }
}];
```

**3) 새 버전 저장 노드 (Code) - 업데이트**

```javascript
const fs = require('fs');
const path = require('path');

const file = $input.item.binary.file;
const docId = `doc_${Date.now()}`;
const uploadDir = `C:\\n8n-data\\uploads\\${docId}`;

// 디렉토리 생성
fs.mkdirSync(uploadDir, { recursive: true });

// 파일 저장
const filePath = path.join(uploadDir, file.fileName);
fs.writeFileSync(filePath, file.data);

// 메타데이터 생성 (버전 정보 포함)
const metadata = {
    doc_id: docId,
    filename: file.fileName,
    original_filename: file.fileName,
    file_size: file.data.length,
    file_type: file.mimeType,
    upload_date: new Date().toISOString(),
    uploaded_by: $json.uploaded_by || 'admin@company.com',
    category: $json.category || 'General',
    tags: $json.tags || [],
    version: $json.new_version,
    previous_version: $json.existing_doc ? $json.existing_doc.doc_id : null,
    active: true,
    version_history: []
};

// 버전 히스토리 업데이트
if ($json.existing_doc) {
    // 이전 문서의 히스토리 가져오기
    const prevHistory = $json.existing_doc.version_history || [];
    
    // 이전 버전 정보 추가
    prevHistory.push({
        version: $json.existing_doc.version,
        doc_id: $json.existing_doc.doc_id,
        upload_date: $json.existing_doc.upload_date,
        archived_date: new Date().toISOString(),
        uploaded_by: $json.existing_doc.uploaded_by
    });
    
    metadata.version_history = prevHistory;
}

// 메타데이터 저장
const metadataPath = path.join(uploadDir, 'metadata.json');
fs.writeFileSync(metadataPath, JSON.stringify(metadata, null, 2));

return [{
    json: {
        ...metadata,
        filepath: filePath
    }
}];
```

**4) Qdrant 저장 노드 - 업데이트**

```javascript
// Qdrant에 벡터 저장 시 payload에 버전 정보 추가
{
  "points": [
    {
      "id": "={{$json.doc_id}}_{{$json.chunk_id}}",
      "vector": "={{$json.vector}}",
      "payload": {
        "doc_id": "={{$json.doc_id}}",
        "chunk_id": "={{$json.chunk_id}}",
        "filename": "={{$json.filename}}",
        "text": "={{$json.text}}",
        "upload_date": "={{$json.upload_date}}",
        "version": "={{$json.version}}",
        "active": true
      }
    }
  ]
}
```

#### 5.5.4 검색 시 버전 필터링

**사용자 챗봇 워크플로우에서 Qdrant 검색 시**

```javascript
// Qdrant 검색 노드 (HTTP Request)
const searchUrl = 'http://qdrant:6333/collections/documents/points/search';
const searchPayload = {
    vector: $json.question_vector,
    limit: 5,
    with_payload: true,
    with_vector: false,
    filter: {
        must: [
            { key: "active", match: { value: true } }  // 활성 버전만 검색 ⭐
        ]
    }
};

const response = await $http.post(searchUrl, searchPayload);
```

#### 5.5.5 버전 관리 장점

| 장점 | 설명 |
|------|------|
| **완전한 이력 추적** | 모든 문서 변경 기록 보존 |
| **롤백 가능** | 이전 버전으로 즉시 복구 |
| **검색 정확도** | 항상 최신 버전만 검색되어 중복 없음 |
| **감사 대응** | 언제 누가 수정했는지 완전 추적 |
| **디스크 효율** | archives 폴더를 정기 정리 가능 |

---

## 6. 데이터베이스 설계

### 6.1 Qdrant 컬렉션 스키마

#### 6.1.1 Collection: `documents`

```json
{
  "name": "documents",
  "vector_size": 768,
  "distance": "Cosine",
  "hnsw_config": {
    "m": 16,
    "ef_construct": 100
  },
  "payload_schema": {
    "doc_id": "keyword",
    "chunk_id": "integer",
    "filename": "keyword",
    "text": "text",
    "upload_date": "keyword",
    "page": "integer",
    "category": "keyword",
    "version": "integer",
    "active": "bool"
  }
}
```

#### 6.1.2 데이터 예시

```json
{
  "id": "doc_1705123456789_001",
  "vector": [0.123, -0.456, 0.789, ...], // 768차원
  "payload": {
    "doc_id": "doc_1705123456789",
    "chunk_id": 1,
    "filename": "2024_휴가정책.pdf",
    "text": "2024년 연차 휴가는 입사일 기준으로 산정되며...",
    "upload_date": "2024-01-13T09:30:00Z",
    "page": 3,
    "category": "인사",
    "version": 3,
    "active": true
  }
}
```

### 6.2 파일 시스템 구조

```
C:\n8n-data\
│
├── uploads\                    # 원본 문서 저장소 (현재 활성 버전만)
│   ├── doc_1705123456789\
│   │   ├── 2024_휴가정책.pdf (v3)
│   │   └── metadata.json       # 문서 메타데이터
│   ├── doc_1705123567890\
│   │   └── 업무매뉴얼.docx
│   └── ...
│
├── archives\                   # 이전 버전 문서 보관 (버전 관리) ⭐ NEW
│   ├── doc_1705100000000_v1\
│   │   ├── 2024_휴가정책.pdf (v1)
│   │   └── metadata.json
│   ├── doc_1705120000000_v2\
│   │   ├── 2024_휴가정책.pdf (v2)
│   │   └── metadata.json
│   └── ...
│
├── sessions\                   # 세션 기반 대화 임시 저장 (1시간 TTL)
│   ├── session_1705123456789.json
│   ├── session_1705124567890.json
│   └── ...
│
├── history\                    # 질의응답 히스토리 (영구 보관)
│   ├── 2024-01-13T09-30-00.json
│   ├── 2024-01-13T10-15-30.json
│   └── ...
│
├── config\                     # 시스템 설정
│   ├── allowed_ips.json
│   ├── embedding_config.json
│   └── system_settings.json
│
└── logs\                       # 로그 파일
    ├── upload_log.txt
    ├── query_log.txt
    └── error_log.txt
```

### 6.3 메타데이터 구조

#### 6.3.1 문서 메타데이터 (metadata.json)

```json
{
  "doc_id": "doc_1705123456789",
  "filename": "2024_휴가정책.pdf",
  "original_filename": "2024_휴가정책.pdf",
  "file_size": 2457600,
  "file_type": "application/pdf",
  "upload_date": "2024-01-13T09:30:00Z",
  "uploaded_by": "admin@company.com",
  "category": "인사",
  "tags": ["휴가", "연차", "정책"],
  "total_pages": 15,
  "total_chunks": 45,
  "embedding_status": "completed",
  "embedding_date": "2024-01-13T09:32:15Z",
  "version": 3,
  "previous_version": "doc_1705120000000",
  "active": true,
  "version_history": [
    {
      "version": 1,
      "doc_id": "doc_1705100000000",
      "upload_date": "2024-01-11T09:00:00Z",
      "archived_date": "2024-01-12T10:00:00Z",
      "uploaded_by": "admin@company.com"
    },
    {
      "version": 2,
      "doc_id": "doc_1705120000000",
      "upload_date": "2024-01-12T10:00:00Z",
      "archived_date": "2024-01-13T09:30:00Z",
      "uploaded_by": "admin@company.com"
    }
  ]
}
```

#### 6.3.2 히스토리 레코드 구조 (영구 보관)

```json
{
  "timestamp": "2024-01-13T10:15:30Z",
  "session_id": "session_1705123456789",
  "user_ip": "192.168.1.105",
  "question": "2024년 휴가 정책은?",
  "answer": "2024년 연차 휴가는 입사일 기준으로...",
  "source_docs": [
    "2024_휴가정책.pdf",
    "인사규정_2024.docx"
  ],
  "response_time_ms": 5432,
  "similarity_scores": [0.89, 0.82, 0.78, 0.71, 0.68]
}
```

#### 6.3.3 세션 데이터 구조 (1시간 TTL)

```json
{
  "session_id": "session_1705123456789",
  "created_at": "2024-01-13T10:15:00Z",
  "last_access": "2024-01-13T10:30:00Z",
  "messages": [
    {
      "role": "user",
      "content": "2024년 휴가 정책은?",
      "timestamp": "2024-01-13T10:15:30Z"
    },
    {
      "role": "assistant",
      "content": "2024년 연차 휴가는 15일이 부여됩니다...",
      "timestamp": "2024-01-13T10:15:35Z"
    },
    {
      "role": "user",
      "content": "그럼 신청은 어떻게 해?",
      "timestamp": "2024-01-13T10:16:10Z"
    },
    {
      "role": "assistant",
      "content": "인사시스템에서 신청 가능합니다...",
      "timestamp": "2024-01-13T10:16:16Z"
    }
  ]
}
```

---

## 7. 보안 및 접근 제어

### 7.1 네트워크 보안

#### 7.1.1 IP 기반 접근 제어

```javascript
// 허용된 IP 대역 설정
const ALLOWED_IP_RANGES = [
    '192.168.1.',    // 본사 1층
    '192.168.2.',    // 본사 2층
    '10.0.0.',       // 연구소
    '172.16.0.'      // 지점
];

function isAllowedIP(clientIP) {
    return ALLOWED_IP_RANGES.some(range => clientIP.startsWith(range));
}
```

#### 7.1.2 방화벽 규칙 (Windows Firewall)

```powershell
# n8n 포트 (5678) - 사내망만 허용
New-NetFirewallRule -DisplayName "n8n Web Access" `
    -Direction Inbound `
    -LocalPort 5678 `
    -Protocol TCP `
    -RemoteAddress 192.168.0.0/16,10.0.0.0/8,172.16.0.0/12 `
    -Action Allow

# Qdrant 포트 (6333) - 로컬호스트만 허용
New-NetFirewallRule -DisplayName "Qdrant API" `
    -Direction Inbound `
    -LocalPort 6333 `
    -Protocol TCP `
    -RemoteAddress 127.0.0.1 `
    -Action Allow

# Ollama 포트 (11434) - 로컬호스트만 허용
New-NetFirewallRule -DisplayName "Ollama API" `
    -Direction Inbound `
    -LocalPort 11434 `
    -Protocol TCP `
    -RemoteAddress 127.0.0.1 `
    -Action Allow
```

### 7.2 데이터 보안

#### 7.2.1 암호화
- **전송 중 암호화**: HTTPS (선택사항, 내부망이므로 HTTP 가능)
- **저장 암호화**: Windows BitLocker로 디스크 암호화 (권장)

#### 7.2.2 접근 권한
```
관리자:
  - 문서 업로드/삭제
  - 히스토리 조회
  - 시스템 설정 변경

일반 사용자:
  - 챗봇 질의
  - 자신의 질문 기록 조회 (선택)
```

### 7.3 로깅 및 감사

#### 7.3.1 로그 수집 항목
```yaml
접근 로그:
  - 타임스탬프
  - 사용자 IP
  - 요청 URL
  - 요청 방법 (GET/POST)
  - 응답 코드

질의 로그:
  - 세션 ID ⭐ NEW
  - 질문 내용
  - 답변 내용
  - 소요 시간
  - 사용된 문서

시스템 로그:
  - 문서 업로드 이벤트
  - 임베딩 처리 이벤트
  - 오류 및 경고
```

---

## 8. 사용자 인터페이스

### 8.1 관리자 페이지 (옵션 C - 모던 디자인)

#### 8.1.1 페이지 구조

```
┌────────────────────────────────────────────────────────┐
│  [로고]  사내 AI 문서 검색 - 관리자                       │
├────────────────────────────────────────────────────────┤
│                                                        │
│  📁 문서 관리    📊 히스토리    ⚙️ 설정                  │
│                                                        │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌───────────────────────────────────────────────┐   │
│  │  파일 업로드                                    │   │
│  │  ┌─────────────────────────────────────────┐ │   │
│  │  │  파일을 드래그하거나 클릭하여 선택      │ │   │
│  │  │  지원 형식: PDF, DOCX, TXT, MD, Excel   │ │   │
│  │  │             PPT, CSV                     │ │   │
│  │  └─────────────────────────────────────────┘ │   │
│  │                                                │   │
│  │  카테고리: [인사 ▼]  태그: [#휴가 #정책]     │   │
│  │                                                │   │
│  │  [업로드 시작]                                  │   │
│  └───────────────────────────────────────────────┘   │
│                                                        │
│  ┌───────────────────────────────────────────────┐   │
│  │  최근 업로드 문서                               │   │
│  │  ─────────────────────────────────────────── │   │
│  │  📄 2024_휴가정책.pdf       2024-01-13 09:30 │   │
│  │     15 페이지 | 45 청크 | ✅ 임베딩 완료        │   │
│  │                                                │   │
│  │  📄 업무매뉴얼.docx          2024-01-12 14:20 │   │
│  │     32 페이지 | 98 청크 | ✅ 임베딩 완료        │   │
│  └───────────────────────────────────────────────┘   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

#### 8.1.2 기술 스택
```yaml
Frontend:
  - HTML5
  - CSS3 (Tailwind CSS)
  - JavaScript (Vanilla / React 스타일)
  - 파일 업로드: Dropzone.js
  - 차트: Chart.js (히스토리 통계)
```

#### 8.1.3 버전 관리 UI ⭐ NEW

**문서 목록 - 버전 정보 표시**

```
┌────────────────────────────────────────────────────────┐
│  📁 문서 관리                                           │
├────────────────────────────────────────────────────────┤
│                                                        │
│  📄 2024_휴가정책.pdf  [v3]  📅 2024-01-15 09:30     │
│     ├─ 15페이지 | 45청크 | ✅ 활성                    │
│     ├─ 👤 admin@company.com                           │
│     └─ [📜 버전 히스토리] [🔄 복구] [🗑️ 삭제]         │
│                                                        │
│  📄 업무매뉴얼.docx  [v2]  📅 2024-01-14 14:20       │
│     ├─ 32페이지 | 98청크 | ✅ 활성                    │
│     ├─ 👤 admin@company.com                           │
│     └─ [📜 버전 히스토리] [🔄 복구] [🗑️ 삭제]         │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**버전 히스토리 팝업**

```
┌────────────────────────────────────────────────────────┐
│  📜 버전 히스토리: 2024_휴가정책.pdf                    │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ✅ v3 (현재 활성)                                     │
│     📅 2024-01-15 09:30                                │
│     👤 admin@company.com                               │
│     📦 15페이지 | 45청크                                │
│     [📥 다운로드]                                       │
│                                                        │
│  📦 v2 (아카이브)                                      │
│     📅 2024-01-13 10:00                                │
│     👤 admin@company.com                               │
│     📦 15페이지 | 43청크                                │
│     [📥 다운로드] [🔄 이 버전 활성화]                   │
│                                                        │
│  📦 v1 (아카이브)                                      │
│     📅 2024-01-11 09:00                                │
│     👤 hr@company.com                                  │
│     📦 14페이지 | 40청크                                │
│     [📥 다운로드] [🔄 이 버전 활성화]                   │
│                                                        │
│  [닫기]                                                │
└────────────────────────────────────────────────────────┘
```

**업로드 시 버전 충돌 알림**

```
┌────────────────────────────────────────────────────────┐
│  ⚠️ 파일 버전 확인                                      │
├────────────────────────────────────────────────────────┤
│                                                        │
│  "2024_휴가정책.pdf" 파일이 이미 존재합니다.           │
│                                                        │
│  현재 버전: v3                                          │
│  업로드 날짜: 2024-01-15 09:30                         │
│  업로드자: admin@company.com                           │
│                                                        │
│  ────────────────────────────────────────────────────  │
│                                                        │
│  새로 업로드하는 파일:                                  │
│  크기: 2.5 MB                                          │
│  수정 날짜: 2024-01-16 11:20                           │
│                                                        │
│  ────────────────────────────────────────────────────  │
│                                                        │
│  이 파일을 새 버전(v4)으로 저장하시겠습니까?            │
│                                                        │
│  ✅ 기존 버전은 아카이브로 이동됩니다.                  │
│  ✅ 검색 시 새 버전만 사용됩니다.                       │
│  ✅ 언제든지 이전 버전으로 복구할 수 있습니다.          │
│                                                        │
│  [확인 (v4로 저장)]  [취소]                            │
└────────────────────────────────────────────────────────┘
```

### 8.2 사용자 챗봇 페이지 (옵션 C - 모던 디자인)

#### 8.2.1 페이지 구조

```
┌────────────────────────────────────────────────────────┐
│  [로고]  사내 AI 문서 검색 챗봇                          │
├────────────────────────────────────────────────────────┤
│                                                        │
│  ┌───────────────────────────────────────────────┐   │
│  │                                                │   │
│  │  🤖: 안녕하세요! 회사 문서에 대해 질문해주세요. │   │
│  │      예: "2024년 휴가 정책은?"                 │   │
│  │                                                │   │
│  │  👤: 2024년 휴가 정책은?                       │   │
│  │                                                │   │
│  │  🤖: 2024년 연차 휴가는 입사일 기준으로 산정  │   │
│  │      되며, 1년 근무 시 15일이 부여됩니다.      │   │
│  │      추가로 3년 이상 근무 시 매년 1일씩        │   │
│  │      가산되어 최대 25일까지 부여됩니다.        │   │
│  │                                                │   │
│  │      📄 출처: 2024_휴가정책.pdf (3페이지)      │   │
│  │                                                │   │
│  │  👤: ___________________________________       │   │
│  │      [전송 →]                                  │   │
│  │                                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                        │
│  💡 추천 질문:                                         │
│  • 연차 사용 절차는?                                   │
│  • 업무 시간은 어떻게 되나요?                          │
│  • 출장 신청 방법은?                                   │
│                                                        │
└────────────────────────────────────────────────────────┘
```

#### 8.2.2 UI/UX 특징
```yaml
디자인:
  - 깔끔한 카드 기반 레이아웃
  - 부드러운 애니메이션 (타이핑 효과)
  - 다크/라이트 모드 지원 (선택)
  - 반응형 디자인 (모바일 대응)

사용자 경험:
  - 실시간 로딩 인디케이터
  - 출처 문서 링크 (클릭 시 원본 다운로드)
  - 키보드 단축키 (Enter 전송, Shift+Enter 줄바꿈)
  - 질문 히스토리 (세션 내)
  - **세션 기반 연속 대화** ⭐ NEW
    - 이전 대화 자동 참조
    - "그럼", "또", "그건" 등 후속 질문 지원
    - 세션 ID 자동 관리 (브라우저 localStorage)
```

#### 8.2.3 세션 관리 구현 (프론트엔드) ⭐ NEW

**JavaScript 세션 관리 코드**

```javascript
// 세션 ID 관리
class SessionManager {
    constructor() {
        this.sessionId = this.getOrCreateSessionId();
    }
    
    getOrCreateSessionId() {
        let sessionId = localStorage.getItem('chat_session_id');
        if (!sessionId) {
            sessionId = `session_${Date.now()}_${Math.random().toString(36).substring(7)}`;
            localStorage.setItem('chat_session_id', sessionId);
        }
        return sessionId;
    }
    
    resetSession() {
        localStorage.removeItem('chat_session_id');
        this.sessionId = this.getOrCreateSessionId();
    }
}

// 메시지 전송
async function sendMessage(question) {
    const sessionManager = new SessionManager();
    
    // n8n Webhook에 전송
    const response = await fetch('http://192.168.x.x:5678/webhook/chat', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
            'X-Session-ID': sessionManager.sessionId  // 세션 ID 헤더에 포함
        },
        body: JSON.stringify({
            question: question,
            session_id: sessionManager.sessionId
        })
    });
    
    const data = await response.json();
    
    // UI에 답변 표시
    displayMessage('user', question);
    displayMessage('assistant', data.answer, data.source_docs);
    
    return data;
}

// 새 대화 시작 버튼
function startNewConversation() {
    const sessionManager = new SessionManager();
    sessionManager.resetSession();
    
    // UI 초기화
    document.getElementById('chat-messages').innerHTML = '';
    
    // 환영 메시지 표시
    displayMessage('assistant', '새 대화를 시작합니다. 무엇을 도와드릴까요?');
}

// UI에 메시지 표시
function displayMessage(role, content, sources = []) {
    const messagesContainer = document.getElementById('chat-messages');
    const messageDiv = document.createElement('div');
    messageDiv.className = `message ${role}`;
    
    let html = `
        <div class="message-content">
            <span class="role-icon">${role === 'user' ? '👤' : '🤖'}</span>
            <div class="text">${content}</div>
        </div>
    `;
    
    // 출처 문서 표시 (AI 답변에만)
    if (role === 'assistant' && sources.length > 0) {
        html += `
            <div class="sources">
                📄 출처: ${sources.join(', ')}
            </div>
        `;
    }
    
    messageDiv.innerHTML = html;
    messagesContainer.appendChild(messageDiv);
    
    // 스크롤 하단으로 이동
    messagesContainer.scrollTop = messagesContainer.scrollHeight;
}
```

**HTML 구조**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>사내 AI 문서 검색 챗봇</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <div class="chat-container">
        <div class="chat-header">
            <h1>🤖 사내 AI 문서 검색 챗봇</h1>
            <button onclick="startNewConversation()" class="btn-new">
                🔄 새 대화
            </button>
        </div>
        
        <div id="chat-messages" class="chat-messages">
            <!-- 메시지가 여기에 동적으로 추가됨 -->
        </div>
        
        <div class="chat-input">
            <textarea 
                id="question-input" 
                placeholder="질문을 입력하세요..."
                onkeydown="handleKeyPress(event)"
            ></textarea>
            <button onclick="handleSubmit()" class="btn-send">
                전송 →
            </button>
        </div>
    </div>
    
    <script src="session-manager.js"></script>
</body>
</html>
```

---

## 9. 배포 및 운영 계획

### 9.1 배포 로드맵

#### Phase 1: 환경 구축 (1-2일)

**Day 1: 하드웨어 및 OS 설정**
```yaml
작업 항목:
  ✅ Windows Server 설치 및 업데이트
  ✅ Docker Desktop 설치
  ✅ 방화벽 규칙 설정
  ✅ 네트워크 구성 (고정 IP)
  ✅ 폴더 구조 생성 (C:\n8n-data\)

체크리스트:
  - [ ] Windows 최신 패치 적용
  - [ ] Docker 버전 확인 (v24.x)
  - [ ] 사내 IP 대역 확인
  - [ ] 포트 개방 확인 (5678, 6333, 11434)
```

**Day 2: 컨테이너 배포**
```yaml
작업 항목:
  ✅ Docker Compose 파일 작성
  ✅ n8n 컨테이너 실행
  ✅ Ollama 컨테이너 실행 및 모델 다운로드
  ✅ Qdrant 컨테이너 실행
  ✅ 컨테이너 간 네트워크 연결 확인

체크리스트:
  - [ ] n8n 웹 접속 확인 (http://192.168.x.x:5678)
  - [ ] Ollama 모델 다운로드 완료 (exaone3.5, nomic-embed-text)
  - [ ] Qdrant 대시보드 접속 (http://localhost:6333/dashboard)
  - [ ] Qdrant 컬렉션 생성 (documents)
```

#### Phase 2: 워크플로우 개발 (2-3일)

**Day 3-4: n8n 워크플로우 구축**
```yaml
작업 항목:
  ✅ 관리자 문서 업로드 워크플로우
  ✅ 사용자 챗봇 워크플로우
  ✅ 관리자 히스토리 워크플로우
  ✅ 오류 처리 및 로깅

체크리스트:
  - [ ] 각 워크플로우 JSON 임포트
  - [ ] 환경 변수 설정 (Ollama URL, Qdrant URL)
  - [ ] 테스트 문서로 전체 플로우 테스트
  - [ ] 에러 핸들링 확인
```

**Day 5: 웹 UI 개발**
```yaml
작업 항목:
  ✅ 관리자 페이지 HTML/CSS/JS
  ✅ 사용자 챗봇 페이지 HTML/CSS/JS
  ✅ n8n Webhook과 UI 연동
  ✅ 반응형 디자인 적용

체크리스트:
  - [ ] 파일 업로드 기능 테스트
  - [ ] 챗봇 질의응답 테스트
  - [ ] 히스토리 페이지 테스트
  - [ ] 크로스 브라우저 테스트
```

#### Phase 3: 테스트 (1-2일)

**Day 6: 기능 테스트**
```yaml
테스트 시나리오:
  ✅ 각 문서 형식 업로드 테스트
    - PDF (10페이지, 100페이지)
    - DOCX (이미지 포함)
    - Excel (다중 시트)
    - PPT (애니메이션 포함)
    - TXT, MD, CSV
  
  ✅ 질의응답 정확도 테스트
    - 단순 질문 (정책 조회)
    - 복잡한 질문 (비교, 계산)
    - 문서에 없는 정보 질문
  
  ✅ 성능 테스트
    - 응답 속도 측정
    - 동시 사용자 5명, 10명 테스트
    - 메모리/CPU 사용량 모니터링

체크리스트:
  - [ ] 모든 문서 형식 정상 처리
  - [ ] 답변 정확도 80% 이상
  - [ ] 평균 응답 시간 10초 이하
  - [ ] 시스템 안정성 확인
```

**Day 7: 보안 및 접근 테스트**
```yaml
테스트 시나리오:
  ✅ IP 접근 제어
    - 허용된 IP에서 접속
    - 외부 IP에서 접속 시도 (차단 확인)
  
  ✅ 데이터 보안
    - 히스토리 파일 권한 확인
    - 업로드 문서 권한 확인
  
  ✅ 다양한 PC에서 접속
    - Windows, Mac, Linux
    - Chrome, Edge, Firefox

체크리스트:
  - [ ] IP 제한 정상 작동
  - [ ] 파일 권한 적절히 설정
  - [ ] 모든 브라우저에서 정상 작동
```

#### Phase 4: 운영 준비 (1일)

**Day 8: 문서화 및 교육**
```yaml
작업 항목:
  ✅ 사용자 매뉴얼 작성
    - 관리자용 (문서 업로드 방법)
    - 사용자용 (챗봇 사용법)
  
  ✅ 운영 가이드 작성
    - 시스템 시작/중지 방법
    - 백업 절차
    - 트러블슈팅 가이드
  
  ✅ 내부 교육
    - 관리자 교육 (1시간)
    - 전체 직원 공지

체크리스트:
  - [ ] 매뉴얼 배포
  - [ ] 관리자 2명 이상 교육 완료
  - [ ] 공지 메일 발송
```

### 9.2 운영 절차

#### 9.2.1 일일 운영

```yaml
아침 (09:00):
  - 시스템 상태 확인 (Docker 컨테이너)
  - 로그 확인 (에러 발생 여부)
  - 디스크 사용량 확인
  - 세션 정리 스크립트 실행 확인 (자동 실행 검증) ⭐ NEW

점심 (12:00):
  - 질의응답 통계 확인 (사용량)
  - 활성 세션 수 모니터링 ⭐ NEW

저녁 (18:00):
  - 일일 백업 실행
  - 성능 지표 기록
```

**세션 정리 자동화** ⭐ NEW

세션 정리는 **작업 스케줄러**로 자동 실행되므로 별도 수동 작업 불필요:
- 실행 주기: 매시간
- 실행 스크립트: `C:\n8n-scripts\cleanup-sessions.ps1`
- 정리 기준: 마지막 접근 시간 1시간 초과
- 로그 확인: `C:\n8n-data\logs\session_cleanup.log`

#### 9.2.2 주간 운영

```yaml
매주 월요일:
  - 주간 사용 통계 리포트
  - 문서 업데이트 검토
  - 시스템 업데이트 확인

매주 금요일:
  - 주간 백업 검증
  - 디스크 정리 (오래된 로그)
```

#### 9.2.3 월간 운영

```yaml
매월 1일:
  - 월간 리포트 작성
  - 성능 최적화 검토
  - 사용자 피드백 수집 및 반영
```

### 9.3 백업 전략

#### 9.3.1 백업 대상

```yaml
중요도 상:
  - Qdrant 데이터 (/var/lib/qdrant)
  - 원본 문서 (C:\n8n-data\uploads)
  - 아카이브 문서 (C:\n8n-data\archives) ⭐ NEW
  - n8n 워크플로우 (JSON 파일)

중요도 중:
  - 히스토리 데이터 (C:\n8n-data\history)
  - 시스템 설정 (C:\n8n-data\config)

중요도 하:
  - 로그 파일 (선택적)
  - 세션 데이터 (백업 불필요, 1시간 TTL)
```

#### 9.3.2 백업 스케줄

```yaml
일일 백업:
  - 시간: 매일 새벽 2:00
  - 대상: Qdrant 데이터, 신규 문서
  - 방법: 증분 백업
  - 보관: 7일

주간 백업:
  - 시간: 매주 일요일 새벽 3:00
  - 대상: 전체 데이터
  - 방법: 전체 백업
  - 보관: 4주

월간 백업:
  - 시간: 매월 1일 새벽 4:00
  - 대상: 전체 시스템 (OS 포함)
  - 방법: 이미지 백업
  - 보관: 12개월
```

#### 9.3.3 백업 스크립트 (PowerShell)

```powershell
# daily-backup.ps1
$date = Get-Date -Format "yyyyMMdd"
$backupRoot = "D:\Backups\Daily"

# Qdrant 백업
docker exec qdrant qdrant-backup --output /snapshots/backup-$date
Copy-Item "C:\docker-data\qdrant\snapshots\backup-$date" "$backupRoot\qdrant\"

# 문서 백업
$uploadDir = "C:\n8n-data\uploads"
$yesterday = (Get-Date).AddDays(-1).ToString("yyyyMMdd")
Get-ChildItem $uploadDir | Where-Object {$_.LastWriteTime -gt $yesterday} | 
    Copy-Item -Destination "$backupRoot\uploads\" -Recurse

# 히스토리 백업
Copy-Item "C:\n8n-data\history" "$backupRoot\history\" -Recurse

Write-Host "✅ Daily backup completed: $date"
```

### 9.4 모니터링

#### 9.4.1 시스템 모니터링 지표

```yaml
성능 지표:
  - CPU 사용률 (평균, 최대)
  - RAM 사용량 (전체, 컨테이너별)
  - 디스크 I/O (읽기/쓰기)
  - 네트워크 트래픽

서비스 지표:
  - 질의응답 수 (일일, 주간)
  - 평균 응답 시간
  - 에러 발생 건수
  - 문서 업로드 수

사용자 지표:
  - 활성 사용자 수
  - 인기 질문 TOP 10
  - 사용 시간대 분포
```

#### 9.4.2 알림 설정

```yaml
경고 알림 (Warning):
  - CPU 사용률 > 80% (10분 이상)
  - RAM 사용량 > 90%
  - 디스크 여유 공간 < 50GB
  - 평균 응답 시간 > 15초

위험 알림 (Critical):
  - 컨테이너 다운
  - 디스크 여유 공간 < 20GB
  - 연속 에러 발생 (5회 이상)
  - 네트워크 연결 끊김

알림 방법:
  - 이메일
  - Slack (선택)
  - SMS (선택)
```

---

## 10. 예상 성능 지표

### 10.1 응답 시간 예측

#### 10.1.1 시나리오별 응답 시간

| 질문 유형 | 예상 시간 | 세부 항목 |
|----------|----------|----------|
| **단순 정보 조회** | 5-7초 | 질문 벡터화(0.5초) + 검색(0.3초) + LLM(4-6초) |
| **복잡한 비교/분석** | 7-10초 | 질문 벡터화(0.5초) + 검색(0.5초) + LLM(6-9초) |
| **문서에 없는 정보** | 4-6초 | 질문 벡터화(0.5초) + 검색(0.3초) + LLM(3-5초) |

#### 10.1.2 문서 처리 시간

| 작업 | 문서 크기 | 예상 시간 |
|------|----------|----------|
| **PDF 텍스트 추출** | 100페이지 | 10-15초 |
| **청크 분할** | 10,000단어 | 5초 |
| **벡터 임베딩** | 100 청크 | 20-30초 |
| **Qdrant 저장** | 100 청크 | 5초 |
| **전체 프로세스** | 100페이지 PDF | 약 60초 |

### 10.2 동시 처리 성능

#### 10.2.1 동시 사용자 시나리오

```yaml
시나리오 1: 저부하 (5명)
  - 대기 시간: 0-5초
  - 평균 응답 시간: 6초
  - 사용자 경험: ⭐⭐⭐⭐⭐ 매우 좋음

시나리오 2: 중부하 (10명)
  - 대기 시간: 5-15초
  - 평균 응답 시간: 12초
  - 사용자 경험: ⭐⭐⭐⭐ 좋음

시나리오 3: 고부하 (15명)
  - 대기 시간: 15-30초
  - 평균 응답 시간: 22초
  - 사용자 경험: ⭐⭐⭐ 보통 (피크 시간대 대기 필요)

시나리오 4: 과부하 (20명 이상)
  - 대기 시간: 30초 이상
  - 평균 응답 시간: 35초+
  - 사용자 경험: ⭐⭐ 개선 필요 (GPU 업그레이드 권장)
```

### 10.3 확장성 시뮬레이션

#### 10.3.1 문서 수 증가에 따른 성능

| 문서 수 | Qdrant 크기 | 검색 시간 | RAM 사용 |
|--------|------------|----------|---------|
| 100개 | 2GB | 0.3초 | 30GB |
| 500개 | 10GB | 0.5초 | 35GB |
| 1,000개 | 20GB | 0.8초 | 40GB |
| 5,000개 | 100GB | 1.5초 | 50GB |

💡 **권장**: 문서 1,000개 이상 시 SSD 용량 증설 및 RAM 업그레이드 (128GB) 고려

---

## 11. 비용 분석

### 11.1 초기 투자 비용

#### 11.1.1 하드웨어 비용 (시나리오 2 기준)

| 항목 | 수량 | 단가 | 합계 |
|------|------|------|------|
| CPU (AMD EPYC 7313P) | 1 | 800,000원 | 800,000원 |
| RAM (64GB DDR4 ECC) | 2 | 200,000원 | 400,000원 |
| SSD (500GB NVMe) | 1 | 90,000원 | 90,000원 |
| HDD (2TB SATA) | 1 | 100,000원 | 100,000원 |
| 메인보드 | 1 | 400,000원 | 400,000원 |
| 전원 (550W 80+ Gold) | 1 | 100,000원 | 100,000원 |
| 쿨링 시스템 | 1 | 70,000원 | 70,000원 |
| 케이스 | 1 | 200,000원 | 200,000원 |
| **하드웨어 소계** | - | - | **2,160,000원** |

#### 11.1.2 소프트웨어 비용

| 항목 | 라이선스 | 비용 |
|------|---------|------|
| Windows 10/11 Pro | 영구 | 200,000원 |
| Docker Desktop | 무료 | 0원 |
| n8n | 자체 호스팅 무료 | 0원 |
| Ollama | 오픈소스 | 0원 |
| Qdrant | 오픈소스 | 0원 |
| **소프트웨어 소계** | - | **200,000원** |

#### 11.1.3 개발 및 구축 비용

| 항목 | 예상 시간 | 시간당 비용 | 합계 |
|------|----------|------------|------|
| 시스템 설계 | 8시간 | - | - |
| 환경 구축 | 16시간 | - | - |
| 워크플로우 개발 | 24시간 | - | - |
| UI 개발 | 16시간 | - | - |
| 테스트 및 디버깅 | 16시간 | - | - |
| 문서화 및 교육 | 8시간 | - | - |
| **총 개발 시간** | **88시간** | - | - |

💡 **참고**: 내부 인력으로 개발 시 인건비는 별도, 외주 개발 시 약 500-800만원 추가 예상

#### 11.1.4 초기 투자 총계

```yaml
하드웨어: 2,160,000원
소프트웨어: 200,000원
기타 (케이블, 악세서리): 100,000원
------------------------
총 초기 투자: 약 2,460,000원
```

### 11.2 운영 비용 (월간)

#### 11.2.1 고정 비용

| 항목 | 월 비용 |
|------|--------|
| **전기료** (24시간 운영, 200W 평균) | 35,000원 |
| **네트워크/인터넷** (기존 사용 중) | 0원 |
| **백업 저장소** (클라우드 백업, 선택) | 10,000원 |
| **관리 인력** (시간당 1시간) | 0원 (기존 인력) |
| **월 운영 비용 합계** | **45,000원** |

#### 11.2.2 연간 운영 비용

```yaml
월 운영 비용: 45,000원
연간 운영 비용: 540,000원

추가 비용 (연간):
  - 하드웨어 보수 예비비: 200,000원
  - 소프트웨어 업데이트: 0원 (오픈소스)
  - 교육 및 유지보수: 0원 (내부 처리)
  
총 연간 비용: 약 740,000원
```

### 11.3 투자 대비 효과 (ROI)

#### 11.3.1 절감 효과

```yaml
직원 1명당 월 절감 시간:
  - 문서 검색 시간: 기존 30분/일 → 개선 5분/일
  - 절감: 25분/일 × 20일 = 500분/월 (약 8.3시간)

직원 50명 기준:
  - 절감 시간: 8.3시간 × 50명 = 415시간/월
  - 시간당 인건비 30,000원 기준
  - 월 절감 비용: 415시간 × 30,000원 = 12,450,000원

ROI 계산:
  - 초기 투자: 2,460,000원
  - 월 절감: 12,450,000원
  - 회수 기간: 0.2개월 (약 6일!)
```

💡 **결론**: 극도로 빠른 투자 회수! (직원 50명 기준)

---

## 12. 리스크 관리

### 12.1 기술적 리스크

#### 12.1.1 하드웨어 장애

| 리스크 | 발생 확률 | 영향 | 대응 방안 |
|--------|----------|------|----------|
| **서버 다운** | 낮음 | 높음 | - 예비 서버 준비 (선택)<br>- 정기 점검 (월 1회)<br>- 알림 시스템 구축 |
| **디스크 고장** | 중간 | 높음 | - RAID 구성 (선택)<br>- 일일 백업<br>- 클라우드 백업 (선택) |
| **전원 장애** | 낮음 | 중간 | - UPS 설치 (권장)<br>- 자동 재시작 스크립트 |

#### 12.1.2 소프트웨어 오류

| 리스크 | 발생 확률 | 영향 | 대응 방안 |
|--------|----------|------|----------|
| **워크플로우 오류** | 중간 | 중간 | - 버전 관리 (Git)<br>- 롤백 프로세스<br>- 테스트 환경 운영 |
| **모델 품질 저하** | 낮음 | 중간 | - 정기 평가 (월 1회)<br>- 모델 업데이트<br>- 피드백 수집 |
| **데이터 손실** | 낮음 | 높음 | - 다중 백업<br>- 백업 검증<br>- 재해 복구 계획 |

### 12.2 운영 리스크

#### 12.2.1 성능 저하

```yaml
원인:
  - 동시 사용자 증가
  - 문서 수 폭증
  - 시스템 리소스 부족

예방:
  - 정기 모니터링
  - 사용량 추세 분석
  - 사전 업그레이드 계획

대응:
  - 대기열 시스템 안내
  - GPU 추가 (성능 향상)
  - 서버 증설 (수평 확장)
```

#### 12.2.2 보안 침해

```yaml
원인:
  - 내부 네트워크 침입
  - 권한 오용
  - 소프트웨어 취약점

예방:
  - IP 화이트리스트 관리
  - 접근 로그 모니터링
  - 정기 보안 업데이트

대응:
  - 즉시 서비스 중단
  - 로그 분석
  - 보안 패치 적용
```

### 12.3 컨틴전시 플랜 (Contingency Plan)

#### 12.3.1 서버 장애 시나리오

```yaml
1단계: 장애 감지 (0-5분)
  - 모니터링 알림 수신
  - 관리자 긴급 연락

2단계: 초기 대응 (5-15분)
  - 서버 재시작 시도
  - 로그 확인
  - 사용자 공지 (이메일/슬랙)

3단계: 복구 작업 (15-60분)
  - 백업에서 데이터 복원
  - 컨테이너 재배포
  - 서비스 재개

4단계: 사후 처리 (1-3시간)
  - 원인 분석
  - 재발 방지 대책
  - 사용자 안내
```

#### 12.3.2 데이터 손실 시나리오

```yaml
1단계: 손실 확인 (0-10분)
  - 손실 범위 파악
  - 백업 상태 확인

2단계: 복구 준비 (10-30분)
  - 최신 백업 선정
  - 복구 시간 예측
  - 사용자 공지

3단계: 데이터 복구 (30-120분)
  - Qdrant 데이터 복원
  - 문서 파일 복원
  - 히스토리 복원

4단계: 검증 (1-2시간)
  - 데이터 무결성 검증
  - 테스트 질의응답
  - 정상화 확인
```

---

## 13. FAQ

### 13.1 기술 관련

**Q1. GPU 없이도 충분한가요?**
- A: 네, 시나리오 2 (CPU 전용)로 충분합니다. 응답 속도 5-8초면 실무에서 문제없습니다. 구글 검색과 비슷한 속도입니다.

**Q2. exaone3.5 모델은 어디서 다운로드하나요?**
- A: Ollama를 통해 `ollama pull exaone3.5` 명령으로 자동 다운로드됩니다.

**Q3. 문서 수가 1,000개를 넘으면 어떻게 하나요?**
- A: Qdrant는 수백만 개 벡터도 처리 가능합니다. 다만 디스크 용량(1TB 이상)과 RAM(128GB)을 증설하면 좋습니다.

**Q4. 다른 LLM 모델로 교체 가능한가요?**
- A: 네, Ollama는 100+ 모델을 지원합니다. `llama3.1`, `qwen2.5`, `mistral` 등으로 간단히 교체 가능합니다.

**Q5. 모바일에서도 사용할 수 있나요?**
- A: 네, 웹 기반이므로 사내 Wi-Fi에 연결된 스마트폰/태블릿에서도 접속 가능합니다.

### 13.2 운영 관련

**Q6. 관리자가 2명 이상일 수 있나요?**
- A: 네, 관리자 페이지는 사내 IP만 있으면 접속 가능하므로 여러 명이 관리할 수 있습니다.

**Q7. 문서를 삭제하면 어떻게 되나요?**
- A: Qdrant에서 해당 문서의 벡터를 삭제하는 워크플로우를 추가로 구현해야 합니다. (Phase 2 업데이트 예정)

**Q8. 전기료가 부담되는데 야간에 자동 종료할 수 있나요?**
- A: 네, Windows 작업 스케줄러로 야간(22:00-08:00) 자동 종료/시작이 가능합니다. 단, 24시간 운영을 권장합니다.

**Q9. 외부(집, 출장지)에서 접속할 수 있나요?**
- A: 현재 설계는 사내 IP만 허용합니다. VPN 구축 시 외부 접속도 가능합니다.

**Q10. 히스토리 데이터는 얼마나 보관하나요?**
- A: 기본적으로 무제한 보관입니다. 디스크 용량에 따라 정기 삭제(예: 1년 후) 정책을 수립할 수 있습니다.

**Q11. 세션 기반 대화가 무엇인가요?** ⭐ NEW
- A: 사용자가 "그럼", "또", "그건" 등으로 이전 대화를 참조하는 후속 질문을 할 수 있는 기능입니다. 세션은 1시간 동안 유지되며 자동으로 삭제됩니다.

**Q12. 브라우저를 닫으면 대화가 사라지나요?** ⭐ NEW
- A: 브라우저를 닫아도 1시간 이내에 다시 접속하면 이전 대화를 이어갈 수 있습니다. 1시간이 지나면 자동으로 삭제됩니다.

**Q13. 새로운 주제로 대화하고 싶으면 어떻게 하나요?** ⭐ NEW
- A: 챗봇 페이지의 "🔄 새 대화" 버튼을 클릭하면 새로운 세션이 시작됩니다.

### 13.3 비용 관련

**Q14. 클라우드(AWS, Azure)로 운영하면 어떨까요?**
- A: 가능하지만 월 비용이 50-100만원 발생합니다. 사내 서버가 훨씬 경제적입니다.

**Q15. Windows 대신 Linux를 사용하면 비용 절감이 되나요?**
- A: 네, Ubuntu Server(무료)를 사용하면 OS 라이선스 비용(20만원)을 절감할 수 있습니다.

**Q16. 향후 사용자가 100명으로 늘어나면 추가 비용은?**
- A: 하드웨어는 그대로 사용 가능하며, GPU(RTX 4060 Ti, 70만원)만 추가하면 충분합니다.

**Q17. 같은 파일을 다시 업로드하면 어떻게 되나요?** ⭐ NEW
- A: 자동으로 버전 번호가 부여됩니다. 기존 파일은 archives 폴더로 이동되며, 새 버전만 검색에 사용됩니다. 모든 이전 버전은 보존되어 언제든 복구할 수 있습니다.

**Q18. 이전 버전으로 되돌릴 수 있나요?** ⭐ NEW
- A: 네, 관리자 페이지에서 문서의 버전 히스토리를 확인하고 원하는 버전을 활성화할 수 있습니다. 현재 버전은 자동으로 아카이브로 이동합니다.

**Q19. 아카이브 폴더가 너무 커지면 어떻게 하나요?** ⭐ NEW
- A: 정책에 따라 정기적으로 오래된 버전을 삭제할 수 있습니다. 예를 들어, 1년 이상 된 아카이브를 삭제하거나 최근 3개 버전만 유지하는 등의 규칙을 설정할 수 있습니다.

---

## 14. 부록

### 14.1 용어 정리

| 용어 | 설명 |
|------|------|
| **RAG** | Retrieval-Augmented Generation. 문서 검색 + LLM 생성을 결합한 기술 |
| **임베딩** | 텍스트를 고차원 벡터로 변환하는 과정 (768차원) |
| **벡터 데이터베이스** | 벡터 간 유사도를 빠르게 검색하는 데이터베이스 (Qdrant) |
| **청크** | 긴 문서를 작은 단위로 나눈 조각 (512 토큰) |
| **LLM** | Large Language Model. 대규모 언어 모델 (exaone3.5) |
| **n8n** | 노코드/로우코드 워크플로우 자동화 플랫폼 |
| **Ollama** | 로컬에서 LLM을 실행하는 프레임워크 |
| **Cosine Similarity** | 두 벡터 간의 유사도를 측정하는 방법 |
| **세션 (Session)** | 사용자의 연속된 대화를 추적하기 위한 임시 데이터 (1시간 유지) |
| **멀티턴 대화** | 이전 대화 맥락을 유지하며 연속적으로 질문하는 방식 |
| **TTL** | Time To Live. 데이터의 유효 기간 (세션: 1시간) |
| **버전 관리 (Version Control)** | 동일 파일의 여러 버전을 추적하고 관리하는 시스템 |
| **아카이브 (Archive)** | 이전 버전 문서를 보관하는 저장소 |
| **활성 버전 (Active Version)** | 현재 검색에 사용되는 최신 버전 |

### 14.2 참고 자료

```yaml
공식 문서:
  - n8n: https://docs.n8n.io/
  - Ollama: https://ollama.ai/
  - Qdrant: https://qdrant.tech/documentation/

커뮤니티:
  - n8n 포럼: https://community.n8n.io/
  - Ollama GitHub: https://github.com/ollama/ollama
  - Qdrant Discord: https://qdrant.tech/discord/

학습 자료:
  - RAG 개념: https://research.ibm.com/blog/retrieval-augmented-generation-RAG
  - Vector Embeddings: https://www.pinecone.io/learn/vector-embeddings/
```

### 14.3 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
|------|------|----------|--------|
| 1.0 | 2024-01-15 | 초안 작성 | Claude AI |
| 1.1 | 2024-01-15 | 세션 기반 대화 관리 기능 추가 (멀티턴 지원) | Claude AI |
| 1.2 | 2024-01-15 | 문서 버전 관리 시스템 추가 | Claude AI |

---

## 📝 결론

본 프로젝트는 **약 250만원의 초기 투자**로 사내 직원들의 문서 검색 시간을 획기적으로 단축할 수 있는 **AI 기반 문서 검색 챗봇 시스템**입니다.

### 핵심 장점:
- ✅ **빠른 투자 회수**: 직원 50명 기준 약 6일
- ✅ **완전 자체 운영**: 외부 클라우드 불필요
- ✅ **확장 가능**: 문서 수, 사용자 수 증가에 대응
- ✅ **오픈소스**: 라이선스 비용 없음
- ✅ **사내 데이터 보안**: 내부망에서만 운영
- ✅ **자연스러운 대화**: 세션 기반 멀티턴 대화 지원
- ✅ **완전한 버전 관리**: 모든 문서 변경 이력 추적 및 복구 가능 ⭐ NEW

### 다음 단계:
1. 하드웨어 구매 승인
2. Docker Compose 파일 및 n8n 워크플로우 JSON 파일 제공
3. 설치 및 구축 (총 8일 소요)
4. 내부 테스트 및 피드백
5. 전사 오픈

**문의사항이 있으시면 언제든지 질문해주세요!** 🚀
