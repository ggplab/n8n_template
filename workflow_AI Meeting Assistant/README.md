# AI Meeting Assistant: 회의 요약봇



> **AI 회의록 구조화 전문 에이전트**
실시간 음성 기록을 기반으로 한 똑똑한 회의 구조 분석과 담당자별 행동 계획 자동 추출
> 
- n8n 화면: ![AI 회의 요약 워크플로우 다이어그램](https://github.com/ggplab/n8n_template/blob/main/workflow_AI%20Meeting%20Assistant/screenshot/AI%20Meeting%20Assistant_Workflow.png)



- 실제 결과물 화면: <div align="center">
  <img src="https://github.com/ggplab/n8n_template/blob/main/workflow_AI%20Meeting%20Assistant/screenshot/AI%20Meeting%20Assistant_Result_Slack.png?raw=true" alt="Slack 요약 결과 화면" width="400px"/>
  
  <img src="https://github.com/ggplab/n8n_template/blob/main/workflow_AI%20Meeting%20Assistant/screenshot/AI%20Meeting%20Assistant_Result_Obsidian.png?raw=true" alt="Obsidian 요약 결과 화면" width="400px"/>
</div>

## 📋 1. 목차



- 문제 정의
- 예상 사용자/부서
- 사용 방법
- 사용 비용
- 참고 문헌

## 🎯 2. 문제 정의



### 비효율적인 반복 작업과 핵심 정보 손실

회의가 끝난 후, 녹음된 음성 파일을 다시 듣고, 핵심 내용을 정리하며, 담당자별 할 일을 분리하여 협업 툴에 분배하는 일련의 과정은 **비효율적이며 반복적인 수동 데이터 변환 작업**입니다. 이 과정에 막대한 시간이 소요될 뿐만 아니라, 다음과 같은 문제들이 발생하여 프로젝트 진행에 병목 현상을 일으킵니다.

- **정보 손실:** 수동 타이핑 중 **핵심 결정 사항**이 누락되거나, 오기되어 기록의 신뢰성이 저하됩니다.
- **행동 불명확:** **'누가', '무엇을', '언제까지'** 해야 하는지가 불분명하게 전달되어, 팀원들의 **행동 항목(Action Items)** 이행이 지연됩니다.

### 달성 가치

반복적이고 비효율적인 작업을 제거함으로써 구성원들은 핵심 업무에 더 집중할 수 있으며, 회의 종료 직후 음성 데이터를 정확하게 분석합니다. **발언자별 핵심 발언, 마감 기한이 명확한 액션 아이템, 시간대별 타임라인**을 자동으로 구조화하고, Obsidian 및 Slack에 최적화된 포맷으로 즉시 전달하여 팀의 효율성과 행동력을 극대화합니다.

## 📌 3. 예상 사용자/부서



> 본 워크플로우는 정기적/비정기적 회의를 진행하고 회의록을 관리해야 하는 모든 사용자에게 유용합니다.
> 
- **PM (프로젝트 관리자)**: 프로젝트 진행 상황과 결정 사항을 명확히 트래킹
- **마케터 / 기획자**: 아이데이션(idea+action) 회의나 기획 회의의 핵심 내용을 빠르게 정리
- **팀 리더 및 임원**: 주요 회의의 결정 사항을 신속하게 파악
- **모든 사무직 종사자**: 단순 회의록 작성 업무에서 해방

## 💬 4. 사용 방법


<details>
  <summary>필요한 API 연결</summary>
  <ul>
    <li>Google Drive API 발급 및 Credential 연결</li>
    <li>CloudConvert API 발급 및 Credential 연결</li>
    <li>OPENAI API 발급 및 Credential 연결</li>
    <li>Gemini API 발급 및 Credential 연결</li>
    <li>Slack API 발급 및 Credential 연결</li>
    <li>Slack Bot 발급 및 Credential 연결</li>
</details>



<details>
  <summary>워크플로우 상세 과정</summary>
  <ul>
    <li><strong>Google Drive Trigger & Download</strong>: Google Drive에 새로운 녹음 파일이 업로드되는 것을 감지하고, 해당 <strong>대용량 파일</strong>을 n8n의 메모리로 다운로드합니다.</li>
    <li><strong>CloudConvert</strong>: <strong>OpenAI Whisper의 25MB 제한</strong>을 우회하기 위해, 다운로드된 오디오 파일의 음질을 <strong>손실 없이 압축</strong>하여 파일 크기를 줄입니다.</li>
    <li><strong>Transcribe a recording (OpenAI Whisper)</strong>: 압축된 오디오를 듣고, 내용의 <strong>원시 텍스트(Raw Transcript)</strong>로 변환합니다.</li>
    <li><strong>AI Agent 1 (Correction)</strong>: <strong>Proofreader</strong> 역할로, Whisper가 생성한 원시 텍스트의 맞춤법, 문법, 그리고 <strong>팀원 이름 등의 고유명사 오류를 수정</strong>하여 텍스트를 정제합니다.</li>
    <li><strong>Convert to File 2 & Upload file 2</strong>: (백업 경로) 정제된 원본 텍스트를 <code>.txt</code> 파일로 만들어 <strong>Google Drive에 안전하게 백업</strong>합니다.</li>
    <li><strong>AI Agent 2 (Summarization)</strong>: <strong>구조 분석가</strong> 역할로, 정제된 텍스트를 기반으로 <strong>Executive Summary, 담당자별 액션 아이템, 타임라인 표</strong> 등 최종 아카이빙 포맷(Markdown)으로 요약하고 구조화합니다.</li>
    <li><strong>Convert to File 3 & Upload file 3</strong>: 최종적으로 구조화된 요약본을 <code>.md</code> 파일로 변환하여 <strong>Obsidian 또는 Google Drive에 영구 보관/아카이빙</strong>합니다.</li>
    <li><strong>AI Agent 3 (Slack Converter)</strong>: <strong>포맷 변환기</strong> 역할로, 옵시디언용으로 만든 복잡한 마크다운(표, #헤더)을 <strong>Slack에서 깨지지 않고 보기 쉬운 리스트 형식(mrkdwn)</strong>으로 변환합니다.</li>
    <li><strong>Send a message</strong>: 최종적으로 가독성이 최적화된 <strong>액션 아이템 및 요약 메시지</strong>를 팀 협업 채널인 Slack으로 즉시 발송합니다.</li>
  </ul>
</details>


## 💵 5. 사용 비용



|  | **Cost** |
| --- | --- |
| **OPENAI API**  | $0.13 |
| **Gemini API** | ₩1,700 |
| **Slack API** | free |
| **CloudConvert API** | free for 10 times per day |

## 🗃️ 6. 참고 문헌



https://n8n.io/workflows/6139-transcribe-and-summarize-audio-with-whisper-and-gpt-from-google-drive-to-notion/
