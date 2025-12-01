#n8n #자동화 #노코드 #OCR #문서변환 #upstage 
## OCR 문서변환 워크플로우
___
### 📌 1. 프로젝트 개요
   사내에서 영수증, 계약서, 참고 자료 등의 이미지를 업로드하게 되면, 해당 이미지에서 텍스트를 추출하고 수동으로 내용을 요약하여 공유하는 과정이 반복될 수 있습니다. 이 수동 작업은 매번 시간을 소요하며, 휴먼 에러(오탈자, 누락) 발생 가능성이 높습니다. 이에 이미지 데이터를 텍스트로 변환, 추출하여 실시간 알림을 발송하는 워크플로우를 구상하였습니다.   

   ### 문제정의 (What)
   이미지 형태의 서류 (계약서, 처방전, 영수증, 기타 문서 이미지)의 텍스트 데이터를 추출합니다.  

   ### 목표 (Why)
   **문제점**: 사람이 직접 눈으로 문서를 확인하고 손으로 타이핑했던 단순반복 작업은 시간적, 육체적 비효율 및 불필요한 비용을 초래하며 정확도가 낮습니다. 
   **목표**: #OCR #VLM 기술을 활용하여 이미지 문서의 단순반복 데이터 입력/공유 작업을 자동화하고, 인적 오류를 제거하는 것을 목표로 합니다.

   ### 방법 (How) 
   - 데이터 수집 : Google Drive에 분석할 이미지 업로드 
   - 데이터 분석**: 파일을 다운로드하여 Upstage OCR을 통해 image 데이터를 정돈된 text 데이터(Slack Block Kit JSON)로 변환 
   - 실시간 알림: 슬랙 채널에 정돈된 텍스트 파일이 메시지로 즉시 발송

   ### 결과 (Result) 
   
   
   
   
   - 정량적 (Quantitative): OCR 분석 및 결과 공유에 드는 수동 작업 시간 (예: 건당 5분)을 0으로 단축. 데이터 즉시 접근성 확보 및 후속 처리 시간 절감. 
   - 정성적 (Qualitative): 이미지 문서 수동 검토 및 텍스트 복사-붙여넣기 과정에서 발생하는 인적 실수 감소 및 데이터 정확도 향상 핵심 정보의 사내 실시간 공유를 통한 협업 속도 개선. 


   ### 유사케이스
   - 영수증 처리: 회계 팀에서 수백 장의 영수증 이미지를 수동으로 입력해야 하는 경우. 
   - 계약서 관리: 임대차계약서, 차량구매 계약서 등 현장 계약서를 전산으로 관리해야 하는 경우
   - 고객 피드백 수집: 고객이 수기로 작성한 설문지나 피드백 이미지 파일을 시스템에 입력해야 하는 경우. 
   - 연구 문서 관리: 논문, 책 표지 등에서 제목, 저자, 핵심 키워드를 추출하여 DB에 정리해야 하는 경우. 

### 2. 예상 사용자/ 부서
|예상 사용자| 부서|
|--------|--------|
|핵심 사용자|	문서 관리가 필요한 팀의 PM/팀 리더|
|운영| 부서	업로드된 문서를 기반으로 후속 조치를 취해야 하는 사무지원/관리 부서|
|기타 부서|	디자인 시안, 참고 서적 이미지 등을 공유하고 즉시 피드백을 받아야 하는 디자인/개발팀|


### 3. 기술 검토 
   - 주요 사용 노드 (n8n):
   <details>
  <summary>Google Drive 트리거</summary>

- On changes involving a specific folder
- 지정된 Google Drive 폴더에 파일 생성 시 워크플로우를 시작합니다.
    </details>

   <details>
  <summary>Google Drive (Download)</summary>

- Trigger에서 받은 파일 ID를 사용하여 실제 이미지 파일(바이너리 데이터)을 다운로드합니다.
   </details>

    <details>
  <summary>Edit Fields</summary>

- 다운로드된 바이너리 데이터의 필드 이름을 OCR 노드가 인식하도록 정리하고, 채널 정보 등을 설정합니다.
    </details>

    <details>
  <summary>Upstage Document OCR</summary>

- 다운로드된 이미지 파일을 분석하여 텍스트를 추출하고 Slack Block Kit 형식의 JSON을 출력합니다.
    </details>

    <details>
  <summary>Slack</summary>

- OCR 분석 결과를 Slack 채널에 보기 좋은 메시지(Blocks) 형태로 최종 전송합니다. 
    </details>
        
### 연동 API 
- Google Drive API(파일 업로드 되면 n8n이 즉시 감지하고 워크플로우 실행)
- Upstage Document OCR API (이미지 OCR 및 구조화된 JSON 데이터 추출)
- Slack API (지정한 채널에 메시지 발송)

### 데이터베이스
- 향후 확장 시 Google Sheets 또는 Airtabl, PostgresSQL 연동 가능

```
Drive Trigger (메타데이터 수신) 
→ Drive Download (바이너리 데이터 확보) 
→ Edit Fields (바이너리 필드 이름 정돈) 
→ Upstage OCR (분석 및 Slack Block JSON 출력) 
→ Slack (메시지 전송)
```

### 4. 사용방법 
- 구현 시 준비사항
    - Google Drive Credential 설정: Google Cloud Console에서 OAuth 2.0 자격 증명 생성 및 권한 범위(Scope) 설정 (/auth/drive 포함).
    - Slack Credential 설정: Slack App 생성 및 chat:write, files:read 등 필요한 권한 부여 후 n8n에 연동.
    - Upstage API Key 설정: Upstage OCR 서비스 API Key 발급 후 n8n에 Credential 연동.  

- 관리자 (최초 1회 설정) 
    - Drive 감시 폴더 설정: When Image Uploaded to Drive 노드에 감시할 Google Drive 폴더 ID를 정확하게 입력.
    - Slack 채널 ID 입력: Workflow Configuration 노드 또는 Slack 노드에 결과를 보낼 Slack 채널 ID/이름을 고정값으로 입력.
    - 워크플로우 활성화: 테스트 완료 후 워크플로우를 Active 상태로 전환.

- 사용자 (임직원)
    - 파일 업로드: 설정된 Google Drive 감시 폴더에 이미지 파일(JPG, PNG 등)을 업로드합니다. (사용자는 별도의 조작이 필요 없습니다.)
    - 결과 확인: 업로드 후 수 초 내에 Slack 채널에 OCR 분석 결과 메시지를 확인합니다.


### 참조문서
   - [n8n 공식홈페이지](https://n8n.io/)
   - [n8n 공식 문서](https://docs.n8n.io/integrations/)
   - [n8n 한글 가이드북 slack api 챕터](https://wikidocs.net/290920)
   - [업스테이지 콘솔](https://console.upstage.ai/docs/getting-started)
   - [구글 api](https://cloud.google.com/cloud-console?hl=ko)
   - [슬랙 api](https://api.slack.com) 