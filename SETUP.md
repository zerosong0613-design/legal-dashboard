# Compliance RiskOps — 연결 설정 가이드

> `index_sharepoint.html` 파일 상단 `CONFIG` 블록만 수정하면 연동이 완료됩니다.

---

## 현재 상태 확인

파일을 브라우저에서 열면 우하단에 **🔌 오프라인 모드** 배너가 표시됩니다.  
샘플 데이터로 모든 기능이 동작하며, 아래 단계를 완료하면 실데이터로 전환됩니다.

---

## 1단계 — IT팀 SharePoint 구성 요청

`SharePoint_구성요청서.docx` 를 IT인프라팀에 제출합니다.

구성 완료 후 IT팀으로부터 받아야 할 정보:

```
SharePoint 사이트 URL: https://YOUR_TENANT.sharepoint.com/sites/ComplianceRiskOps
```

---

## 2단계 — Power Automate Flow 생성 (직접 수행)

`PowerAutomate_Flow_설계서.docx` 를 보고 Flow 2개를 직접 만듭니다.  
나머지 4개(F-02~F-05)는 SP 이벤트/예약 트리거라 URL이 필요 없습니다.

### F-01 생성 방법 (제출하기 연동)

1. Power Automate → **만들기** → **인스턴트 클라우드 Flow**
2. 트리거: **"HTTP 요청을 받은 경우"** 선택
3. JSON 스키마에 아래 붙여넣기:

```json
{
  "type": "object",
  "properties": {
    "riskId":      { "type": "string" },
    "riskTitle":   { "type": "string" },
    "team":        { "type": "string" },
    "year":        { "type": "integer" },
    "month":       { "type": "integer" },
    "status":      { "type": "string" },
    "content":     { "type": "string" },
    "specialNote": { "type": "string" },
    "submittedBy": { "type": "string" }
  }
}
```

4. **SharePoint – 항목 가져오기** 액션 추가  
   - 사이트 주소: `{SP 사이트 URL}`  
   - 목록 이름: `MonthlyReport`  
   - 필터 쿼리: `RiskId eq '@{triggerBody()?['riskId']}' and Year eq @{triggerBody()?['year']} and Month eq @{triggerBody()?['month']}`

5. **조건** 추가 → 결과 있음 / 없음 분기  
   - 있음(예): **SharePoint – 항목 업데이트** (PATCH)  
   - 없음(아니요): **SharePoint – 항목 만들기** (POST)

6. 두 분기 모두 `SubmitStatus` 필드를 `"제출완료"` 로 설정

7. **Outlook – 이메일 보내기 (V2)** 액션 추가  
   - 수신자: 컴플라이언스팀 이메일  
   - 제목: `[RiskOps] 이행현황 제출 알림 – @{triggerBody()?['team']} / @{triggerBody()?['month']}월`

8. **HTTP 응답** 액션 추가 → 상태 코드 `200`, 본문 `{"success": true}`

9. **저장** → HTTP POST URL 복사 → `CONFIG.FLOW_SUBMIT_URL` 에 붙여넣기

### F-06 생성 방법 (임시저장 연동)

F-01과 동일한 구조로 생성합니다. 단:
- **Outlook 이메일 액션 없음** (임시저장은 알림 불필요)
- `SubmitStatus` 필드를 `"임시저장"` 으로 설정
- 저장 후 URL을 `CONFIG.FLOW_SAVE_URL` 에 입력

### F-07 생성 방법 (이의신청 알림 — v2.0 신설)

팀 담당자가 평가 결과에 이의를 제기할 때 컴플라이언스팀 관리자에게 알림 이메일을 발송하고,
MonthlyReport에 이의신청 이력을 기록하는 전용 Flow입니다.

1. Power Automate → **만들기** → **인스턴트 클라우드 Flow**
2. 트리거: **"HTTP 요청을 받은 경우"** 선택
3. JSON 스키마에 아래 붙여넣기:

```json
{
  "type": "object",
  "properties": {
    "type":         { "type": "string" },
    "riskId":       { "type": "string" },
    "riskTitle":    { "type": "string" },
    "team":         { "type": "string" },
    "year":         { "type": "integer" },
    "month":        { "type": "integer" },
    "reason":       { "type": "string" },
    "submittedBy":  { "type": "string" }
  }
}
```

4. **SharePoint – 항목 수정** 액션 추가  
   - 사이트 주소: `{SP 사이트 URL}`  
   - 목록 이름: `MonthlyReport`  
   - 필터 쿼리로 해당 riskId+year+month 항목 찾은 후 PATCH  
   - 설정 필드: `RevisionRequest` = `"예"`, `RevisionNote` = `@{triggerBody()?['reason']}`

5. **Outlook – 이메일 보내기 (V2)** 액션 추가  
   - 수신자: 컴플라이언스팀 관리자 이메일  
   - 제목: `[RiskOps] ⚠ 이의신청 접수 – @{triggerBody()?['team']} / @{triggerBody()?['month']}월`  
   - 본문: 팀, 리스크명, 이의신청 사유 포함

6. **HTTP 응답** 액션 추가 → 상태 코드 `200`, 본문 `{"success": true}`

7. **저장** → HTTP POST URL 복사 → `CONFIG.FLOW_REVISION_URL` 에 붙여넣기

> **F-03과의 차이:** F-07은 팀→관리자 방향(이의신청), F-03은 관리자→팀 방향(수정 요청). 두 Flow는 독립적으로 동작합니다.

---

## 3단계 — index_sharepoint.html CONFIG 수정

파일 상단(약 10번째 줄)의 `CONFIG` 블록을 수정합니다:

```javascript
const CONFIG = {
  SP_SITE_URL:      'https://YOUR_TENANT.sharepoint.com/sites/ComplianceRiskOps',
  //                 ↑ IT팀에서 받은 SharePoint 사이트 URL

  LIST_RISK_MASTER:   'RiskMaster',      // ← 변경 불필요 (List명이 다르면 수정)
  LIST_MONTHLY:       'MonthlyReport',   // ← 변경 불필요
  LIST_EVALUATION:    'Evaluation',      // ← 변경 불필요

  FLOW_SUBMIT_URL:  'https://prod-**.logic.azure.com/...',
  //                 ↑ F-01 HTTP 트리거 URL (Power Automate에서 복사)

  FLOW_SAVE_URL:    'https://prod-**.logic.azure.com/...',
  //                 ↑ F-06 HTTP 트리거 URL

  FLOW_REVISION_URL: 'https://prod-**.logic.azure.com/...',
  //                 ↑ F-07 이의신청 HTTP 트리거 URL (v2.0 신설)

  OFFLINE_MODE: false,   // ← true → false 로 변경하면 SP 연동 활성화
};
```

> **⚠️ 주의:** Flow URL은 절대 공개 저장소(GitHub public 등)에 커밋하지 마세요.

---

## 4단계 — AI 요약 기능 활성화 (선택)

보고서 총평 자동생성 / 이행내용 AI 요약 / 평가 의견 자동생성 기능을 쓰려면 Anthropic API Key가 필요합니다.

```javascript
CONFIG.ANTHROPIC_API_KEY = 'sk-ant-api03-...';
```

> **운영 권고:** API Key는 브라우저에 노출되므로, 운영 환경에서는 Vercel API Route(`/api/proxy.js`)를 거치는 방식을 사용하세요. 기존 `index_legal-deal.html` 프로젝트의 프록시 패턴을 그대로 재사용할 수 있습니다.

---

## 5단계 — Vercel 배포

```bash
# GitHub 저장소에 파일 업로드 후 Vercel 연동
# 또는 Vercel CLI 사용:

npm install -g vercel
vercel --prod
```

배포 후 사이트 URL을 SharePoint CORS 허용 목록에 추가하도록 IT팀에 요청합니다.

---

## 체크리스트

```
□ IT팀에 SharePoint_구성요청서.docx 제출
□ SharePoint 사이트 URL 수신
□ Power Automate F-01 생성 및 URL 복사
□ Power Automate F-06 생성 및 URL 복사
□ Power Automate F-07 생성 및 URL 복사 (이의신청 알림 — v2.0 신설)
□ Power Automate F-02 생성 (Evaluation List 트리거)
□ Power Automate F-03 생성 (RevisionRequested 트리거)
□ Power Automate F-04 생성 (매월 26일 예약)
□ Power Automate F-05 생성 (매월 30일 예약)
□ CONFIG.SP_SITE_URL 입력
□ CONFIG.FLOW_SUBMIT_URL 입력
□ CONFIG.FLOW_SAVE_URL 입력
□ CONFIG.FLOW_REVISION_URL 입력 (v2.0 신설)
□ CONFIG.OFFLINE_MODE = false 변경
□ (선택) CONFIG.ANTHROPIC_API_KEY 입력
□ Vercel 배포 후 URL 확인
□ IT팀에 CORS 허용 URL 추가 요청
□ 팀 담당자 1~2명 파일럿 테스트
```

---

## 자주 묻는 문제

**Q. 화면이 열리는데 데이터가 안 나와요**  
A. 브라우저 콘솔(F12)에서 오류 확인. `401 Unauthorized` → SharePoint 로그인 상태 확인. `CORS 오류` → IT팀에 CORS 설정 요청.

**Q. Flow URL을 입력했는데 제출이 안 돼요**  
A. Power Automate Flow가 **켜짐** 상태인지 확인. URL이 만료되지 않았는지 확인 (Flow 재저장 시 URL 재발급).

**Q. AI 요약 버튼이 에러가 나요**  
A. `CONFIG.ANTHROPIC_API_KEY` 에 유효한 API Key가 입력되어 있는지 확인. claude.ai에서 발급한 키가 아닌 Anthropic Console(console.anthropic.com)에서 발급한 API Key를 사용해야 합니다.

**Q. 새로 고침하면 입력한 데이터가 사라져요**  
A. `OFFLINE_MODE: true` 상태에서는 정상입니다. SP 연동 완료 후 `false` 로 변경하면 해결됩니다.

---

*작성: 컴플라이언스팀 (법무) | 2026-03-15*
