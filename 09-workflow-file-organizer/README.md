# 🗂️ AI 기반 로컬 파일 자동 정리 시스템

> n8n + OpenAI + Docker Compose로 구현한 지능형 파일 분류 및 자동 정리 워크플로우
> 

<img width="1580" height="518" alt="workflow" src="https://github.com/user-attachments/assets/59c2bc22-c236-476f-a112-595a4ae6b5f3" />

<img width="703" height="408" alt="documents" src="https://github.com/user-attachments/assets/a0a4a87b-dee2-4f7e-92c3-50e1d1eca7e3" />

<img width="1644" height="464" alt="google sheets" src="https://github.com/user-attachments/assets/626e49c5-01bf-4b9a-a6a5-71727497a93f" />

---

## 📑 목차

1. [문제 정의]
2. [솔루션 개요]
3. [핵심 차별점]
4. [기술 스택]
5. [시스템 구조]
6. [설치 및 설정]
7. [사용 방법]
8. [예상 비용]
9. [트러블슈팅]
10. [참고 문헌]

---

## 🎯 문제 정의

### 현재 문제점

- **다운로드 폴더의 무분별한 파일 누적**: 매일 수십 개의 파일이 다운로드되지만 정리되지 않음
- **수동 분류의 시간 낭비**: 파일 하나하나 열어보고 적절한 폴더를 찾아 이동하는 반복 작업
- **일관성 없는 폴더 구조**: 사람마다, 시기마다 다른 분류 기준으로 혼란 발생
- **중요 파일 누락**: 계약서, 영수증 등 중요 파일이 다운로드 폴더에 묻혀 놓침

### 목표 사용자

- **개인**: 매일 10개 이상의 파일을 다운로드하는 직장인, 학생
- **프리랜서**: 클라이언트별로 파일을 체계적으로 관리해야 하는 사람
- **연구자/개발자**: 논문, 코드, 문서 등 다양한 유형의 파일을 다루는 사람

---

## 💡 솔루션 개요

**AI가 파일명, 확장자, 기존 폴더 구조를 분석하여 지능적으로 최적의 위치를 제안하고 자동 이동하는 시스템**

### 작동 방식

```
📥 Downloads 폴더에 파일 추가
         ↓
🔍 매 5분마다 자동 스캔
         ↓
🤖 AI가 파일명 + 확장자 분석
         ↓
📊 기존 폴더 구조 학습
         ↓
🎯 최적 위치 결정
         ↓
📂 자동으로 폴더 생성 및 이동
         ↓
📝 Google Sheets에 로그 기록
```

---

## ⭐ 핵심 차별점

### 일반 파일 정리 도구 vs 본 시스템

| 기능 | 일반 도구 | **본 시스템** |
| --- | --- | --- |
| **분류 기준** | 확장자만 | 파일명 + 확장자 + 의미 분석 |
| **폴더 구조** | 고정된 규칙 | 기존 구조 학습 + 자동 적응 |
| **새 폴더 생성** | 수동 | AI가 필요 시 자동 생성 |
| **분류 정확도** | 60~70% | 85~95% (AI 기반) |
| **로깅** | 없음 | 모든 이동 기록 추적 |
| **한글 지원** | 제한적 | 완벽 지원 |
| **비용** | 유료 ($10~30/월) | 오픈소스 (API 사용료만) |

### 실제 분류 예시

```
"2024_계약서_최종본.pdf" → Documents/계약서/
"meeting_notes_20241124.txt" → Documents/회의록/
"invoice_202411.xlsx" → Documents/재무/영수증/
"screenshot_bug.png" → Documents/스크린샷/
"machine_learning_paper.pdf" → Documents/논문/

```

---

## 🛠️ 기술 스택

### 핵심 기술

- **n8n (Self-hosted)**: 워크플로우 자동화 플랫폼
- **Docker Compose**: 컨테이너 관리 및 자동 시작
- **OpenAI GPT-4o-mini**: 파일 분류 AI 엔진
- **Google Sheets API**: 이동 기록 로깅
- **Node.js fs 모듈**: 파일 시스템 작업

### 필수 준비물

- Docker Desktop (Windows/Mac)
- OpenAI API Key
- Google Cloud Console 프로젝트 (Sheets API 활성화)

---

## 📐 시스템 구조

### 워크플로우 다이어그램

```
┌─────────────────────────────────────────────────────────────┐
│                   🗂️ 파일 자동 정리 시스템                      │
└─────────────────────────────────────────────────────────────┘

┌──────────────┐
│ 1. Schedule  │  ⏰ 매 5분마다 실행
│   Trigger    │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 2. 폴더 스캔  │  📂 Downloads 폴더의 모든 파일 목록 가져오기
│   (Code)     │      fs.readdirSync() 사용
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 3. 파일 정보  │  🔍 각 파일의 이름, 확장자, 타입 분석
│    추출       │      예: test.pdf → {filename, extension, fileType}
│   (Code)     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 4. 기존 폴더  │  📊 Documents 하위에 이미 존재하는 폴더 목록 조회
│    목록 조회  │      AI에게 참고 자료로 전달
│   (Code)     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 5. OpenAI    │  🤖 AI 분석:
│    파일 분석  │      - 파일명에서 의미 추출
│              │      - 기존 폴더 중 적합한 곳 선택
│              │      - 새 폴더 필요 시 제안
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 6. JSON 파싱 │  📋 AI 응답을 구조화된 데이터로 변환
│    및 통합    │      {category, target_folder, confidence, ...}
│   (Code)     │
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 7. 파일 이동  │  🚚 실제 파일 이동 수행:
│   (Code)     │      1. 목적지 폴더 자동 생성
│              │      2. 파일 복사
│              │      3. 원본 삭제
└──────┬───────┘
       │
       ▼
┌──────────────┐
│ 8. Google    │  📝 모든 이동 기록을 스프레드시트에 저장
│    Sheets    │      (timestamp, filename, category, ...)
│    로그       │
└──────────────┘

```

---

## 🚀 설치 및 설정

### Step 1: 폴더 구조 생성

**Windows 탐색기에서 아래 폴더들을 생성하세요:**

```
C:\Users\YOUR_USERNAME\Desktop\
├── N8N\                    ← Docker Compose 설정 파일 위치
├── test\
│   ├── Downloads\          ← 파일이 들어올 폴더
│   └── Documents\          ← 정리된 파일이 저장될 폴더
└── n8n_data\               ← n8n 데이터 저장 (자동 생성됨)

```

---

### Step 2: Docker Compose 설정

### 2-1. `docker-compose.yml` 파일 생성

**위치:** `C:\Users\YOUR_USERNAME\Desktop\N8N\docker-compose.yml`

**내용:**

```yaml
version: '3.8'

services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: unless-stopped
    ports:
      - "5678:5678"
    environment:
      - NODE_FUNCTION_ALLOW_BUILTIN=fs,path
      - NODE_FUNCTION_ALLOW_EXTERNAL=
    volumes:
      - C:/Users/YOUR_USERNAME/Desktop/test:/data/test
      - C:/Users/YOUR_USERNAME/Desktop/n8n_data:/home/node/.n8n

```

⚠️ **중요:** `YOUR_USERNAME`을 실제 Windows 사용자 이름으로 변경하세요!

### 2-2. n8n 실행

**PowerShell 열기:**

```powershell
# N8N 폴더로 이동
cd C:\Users\YOUR_USERNAME\Desktop\N8N

# Docker Compose로 n8n 시작
docker-compose up -d

```

**예상 출력:**

```
[+] Running 2/2
 ✔ Network n8n_default  Created
 ✔ Container n8n        Started

```

### 2-3. 접속 확인

브라우저에서 `http://localhost:5678` 접속

---

### Step 3: n8n 워크플로우 Import

1. n8n 웹 인터페이스 접속
2. 좌측 메뉴 → **Workflows** → **Import from File**
3. `file_organizer_v2_commented.json` 파일 선택
4. Import 완료

---

### Step 4: Credential 연결

### 4-1. OpenAI API 설정

1. [OpenAI Platform](https://platform.openai.com/api-keys)에서 API Key 발급
2. n8n에서 `5. OpenAI 파일 분석` 노드 클릭
3. **Credential** → **Create New**
4. API Key 입력 후 저장

### 4-2. Google Sheets API 설정

**Google Cloud Console 설정:**

1. [Google Cloud Console](https://console.cloud.google.com/) 접속
2. 새 프로젝트 생성
3. **APIs & Services** → **Enable APIs and Services**
4. **Google Sheets API** 검색 후 활성화
5. **Credentials** → **Create Credentials** → **OAuth 2.0 Client ID**
    - Application type: **Web application**
    - Authorized redirect URIs: `http://localhost:5678/rest/oauth2-credential/callback`
6. Client ID와 Client Secret 복사

**n8n에서 연결:**

1. `8. Google Sheets 로그` 노드 클릭
2. **Credential** → **Create New**
3. OAuth2 정보 입력 후 인증
4. 새 Google Sheets 생성
5. Sheet ID를 복사하여 노드에 입력

**Google Sheets 구조 (첫 번째 행):**

| timestamp | original_path | filename | extension | file_type | category | target_folder | new_folder_created | confidence | reasoning | moved | action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

---

### Step 5: 경로 설정 (중요!)

**3곳의 경로를 일치시켜야 합니다:**

### ① Docker Compose 파일 (`docker-compose.yml`)

```yaml
volumes:
  - C:/Users/YOUR_USERNAME/Desktop/test:/data/test

```

### ② 노드 2번 (폴더 스캔)

```jsx
const downloadsPath = '/data/test/Downloads';

```

### ③ 노드 7번 (파일 이동)

```jsx
const baseDir = '/data/test/Documents';

```

**경로 변경 예시:**

만약 D 드라이브를 사용한다면:

- Docker Compose: `D:/MyFiles:/data/myfiles`
- 노드 2: `'/data/myfiles/Downloads'`
- 노드 7: `'/data/myfiles/Documents'`

---

### Step 6: 저장 및 활성화

1. 워크플로우 우측 상단 **Save** 클릭
2. **Active** 토글 켜기 ✅
3. 자동 실행 시작!

---

### Step 7: 재부팅 테스트

### 자동 시작 확인:

1. **컴퓨터 재부팅**
2. **Docker Desktop 실행** (또는 자동 실행 설정)
3. **브라우저에서 `http://localhost:5678` 접속**
4. **n8n이 자동으로 실행 중이면 성공!** ✨

### Docker Desktop 자동 시작 설정:

1. Docker Desktop 설정 열기
2. **General** 탭
3. ✅ **Start Docker Desktop when you log in** 체크

---

## 📖 사용 방법

### 기본 사용

1. **Downloads 폴더에 파일 추가**
    - 브라우저에서 파일 다운로드
    - 또는 파일 복사/이동
2. **자동 정리 대기**
    - 매 5분마다 자동으로 스캔 및 정리
    - 또는 n8n에서 수동 실행 (Execute Workflow)
3. **결과 확인**
    - Documents 폴더에서 정리된 파일 확인
    - Google Sheets에서 이동 기록 확인
        
        ---
        
        ### Docker Compose 관리 명령어
        
        ```powershell
        # N8N 폴더로 이동 (항상 먼저!)
        cd C:\Users\YOUR_USERNAME\Desktop\N8N
        
        # 시작
        docker-compose up -d
        
        # 중지
        docker-compose down
        
        # 재시작
        docker-compose restart
        
        # 로그 보기 (문제 발생 시)
        docker-compose logs -f n8n
        
        # 상태 확인
        docker-compose ps
        
        ```
        
        ---
        
        ### Docker Desktop에서 관리
        
        **더 쉬운 방법:**
        
        1. **Docker Desktop** 실행
        2. **Containers** 탭 클릭
        3. **n8n** 찾기
        4. **재생 버튼(▶️) / 중지 버튼(⏸️) / 재시작 버튼(🔄)** 클릭!
        
        ---
        
        ### AI 분류 로직
        
        ### 분류 우선순위
        
        1. **파일명 키워드 인식**
            
            ```
            "invoice" → 재무/영수증
            "contract" → 계약서
            "resume" → 취업/이력서
            "screenshot" → 스크린샷
            
            ```
            
        2. **확장자 기반 타입**
            
            ```
            .pdf, .docx → 문서
            .jpg, .png → 이미지
            .mp4, .mov → 비디오
            .zip, .rar → 압축파일
            
            ```
            
        3. **기존 폴더 구조 우선**
            - 이미 존재하는 폴더 중 적합한 곳 선택
            - 새 폴더는 필요할 때만 생성
        
        ### 분류 예시
        
        ```
        입력: "2024_세금계산서_11월.pdf"
        
        AI 분석:
        {
          "category": "재무",
          "subcategory": "세금계산서",
          "target_folder": "재무/세금계산서",
          "create_new_folder": true,
          "confidence": 0.92,
          "reasoning": "파일명에 '세금계산서' 키워드 포함, PDF 문서"
        }
        
        결과: Documents/재무/세금계산서/ 폴더 생성 후 이동
        
        ```
        
        ---
        
        ### 실행 빈도 조정
        
        **1번 노드 (Schedule Trigger) 설정 변경:**
        
        ```jsx
        // 현재: 매 5분
        "minutesInterval": 5
        
        // 변경 예시:
        // 매 10분
        "minutesInterval": 10
        
        // 매 1시간
        "field": "hours",
        "hoursInterval": 1
        
        // 매일 오전 9시 (Cron 표현식)
        "field": "cronExpression",
        "cronExpression": "0 9 * * *"
        
        ```
        

---

## 💰 예상 비용

### OpenAI API 비용 (GPT-4o-mini)

| 사용량 | Input Tokens | Output Tokens | 월 비용 |
| --- | --- | --- | --- |
| 하루 10개 파일 | ~500 tokens/call | ~150 tokens/call | **$0.30** |
| 하루 30개 파일 | ~500 tokens/call | ~150 tokens/call | **$0.90** |
| 하루 100개 파일 | ~500 tokens/call | ~150 tokens/call | **$3.00** |

**계산 근거:**

- Input: $0.150 / 1M tokens
- Output: $0.600 / 1M tokens
- 평균 1회 호출: ~500 input + ~150 output tokens

### 무료 구성 요소

- ✅ n8n Self-hosted (오픈소스)
- ✅ Docker Compose (무료)
- ✅ Google Sheets API (무료)

### 비용 절감 팁

1. **빈도 조정**: 5분 → 10분으로 변경 (50% 절감)
2. **프롬프트 최적화**: 불필요한 설명 제거
3. **필터링**: 이미 정리된 파일 제외

---

## 🔧 트러블슈팅

### ❌ "Module 'fs' is disallowed"

**원인:** n8n에서 fs 모듈이 차단됨

**해결:**

docker-compose.yml에 환경 변수가 있는지 확인:

```yaml
environment:
  - NODE_FUNCTION_ALLOW_BUILTIN=fs,path

```

있다면 컨테이너 재시작:

```powershell
docker-compose restart

```

---

### ❌ "ENOENT: no such file or directory"

**원인:** Docker 볼륨 마운트 경로 문제

**해결:**

1. `docker-compose.yml` 확인:
    
    ```yaml
    volumes:
      - C:/Users/YOUR_USERNAME/Desktop/test:/data/test
    
    ```
    
2. 실제 폴더가 존재하는지 확인
3. 경로가 정확한지 확인
4. 컨테이너 재시작:
    
    ```powershell
    docker-compose down
    docker-compose up -d
    
    ```
    

---

### ❌ "Could not get parameter" (Google Sheets)

**원인:** Google Sheets 노드 설정 오류

**해결:**

1. Credential 재연결
2. Sheet ID 확인
3. Sheet 이름 확인 (`file-log`)
4. Values to Send 매핑 재설정

---

### ❌ 파일이 이동 안 됨 (moved: false)

**원인:** 파일 경로 또는 권한 문제

**체크리스트:**

- [ ]  원본 파일이 실제로 존재하는가?
- [ ]  Docker 볼륨 마운트가 올바른가?
- [ ]  노드 2, 7번의 경로가 `/data/test/...` 형식인가?
- [ ]  파일이 다른 프로그램에서 열려있는가?

**해결:**

```jsx
// 7번 노드에서 에러 로그 확인
console.log('원본 경로:', sourcePath);
console.log('목적지 경로:', targetPath);

```

Docker Compose 로그 확인:

```powershell
docker-compose logs -f n8n

```

---

### ❌ 재부팅 후 n8n이 자동 시작 안 됨

**원인:** Docker Desktop이 자동 시작되지 않음

**해결:**

1. **Docker Desktop 자동 시작 설정:**
    - Docker Desktop 설정
    - General 탭
    - ✅ "Start Docker Desktop when you log in" 체크
2. **restart 정책 확인:**
    
    ```yaml
    restart: unless-stopped  # 이게 있어야 함
    
    ```
    
3. **수동 시작:**
    
    ```powershell
    cd C:\Users\YOUR_USERNAME\Desktop\N8N
    docker-compose start
    
    ```
    

---

### ⚠️ Windows 경로 주의사항

**Docker Compose에서 Windows 경로:**

```yaml
# ✅ 올바른 형식
volumes:
  - C:/Users/com/Desktop/test:/data/test

# ❌ 잘못된 형식
volumes:
  - C:\Users\com\Desktop\test:/data/test  # 역슬래시 X

```

**n8n 코드 노드에서는 Docker 경로 사용:**

```jsx
// ✅ Docker 내부 경로
const downloadsPath = '/data/test/Downloads';

// ❌ Windows 경로 (작동 안 함)
const downloadsPath = 'C:/Users/com/Desktop/test/Downloads';

```

---

## 📚 참고 문헌

### 공식 문서

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Code Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.code/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [OpenAI API Reference](https://platform.openai.com/docs/api-reference)
- [Google Sheets API](https://developers.google.com/sheets/api)

### 관련 프로젝트

- [n8n Community Workflows](https://n8n.io/workflows/)
- [Hazel (Mac 파일 정리 도구)](https://www.noodlesoft.com/)
- [DropIt (Windows 파일 정리 도구)](http://www.dropitproject.com/)

### 레퍼런스 워크플로우

- [n8n File Organization Template](https://n8n.io/workflows/)

---

## 📸 실행 결과 예시

### 자동 생성된 폴더 구조

```
Documents/
├── 강의/
│   └── 교안/
├── 기타/
├── 기획안/
├── 면접/
├── 문서/
├── 아이디어/
├── 클 개발 가이드/
├── 정리/
├── 취업/
└── 코드북/

```

### Google Sheets 로그 예시

| timestamp | filename | category | target_folder | confidence | moved |
| --- | --- | --- | --- | --- | --- |
| 2025-11-24T07:44:30Z | contract.pdf | 계약서 | 계약서 | 0.95 | true |
| 2025-11-24T07:44:35Z | invoice.xlsx | 재무 | 재무/영수증 | 0.92 | true |

---

**이제 여러분의 Downloads 폴더는 항상 깔끔하게 유지됩니다! 🎊**
