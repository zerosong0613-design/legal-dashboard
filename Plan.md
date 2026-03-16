# Plan.md — 법무팀 사건 관리 시스템 개발 계획

> 최종 업데이트: 2026-03-16

---

## 1. 완료된 작업

### v1.0 — 초기 시스템 (`589d059`)
- 단일 파일 HTML 대시보드 (index.html ~5000줄)
- 5개 화면: 대시보드, 사건 목록, 아카이브, 보고서, 법무 비용
- 사건 CRUD, 분류(LIT/ADV/INT/REG/ETC), 상태 관리(접수→완료)
- 기일 D-day, 담당자 현황, 분류별 탭
- 사건 상세: 스텝바, 기본정보, 관계자, 기일, 파일, 메모, 타임라인
- 보고서: 월간/연간, Chart.js 차트, CSV/PPTX 내보내기
- 법무 비용: 정기비용 + 사건별 비용 관리
- PIN 간이 인증 (매니저/팀장/임원)
- 대외비 사건 접근 제어
- 모바일 반응형 + 하단 탭 네비게이션
- localStorage 기반 데이터 저장
- AI 연동 포인트 (Anthropic API — 요약, 파일 에이전트, 글로벌 챗봇)

### v1.1 — UI/UX 현대화 (`589d059`)
- **디자인 토큰**: border-radius 12px, 다중 레이어 그림자, font-smoothing
- **사이드바**: 흰색 배경, active 상태 navy 배경+흰색 텍스트 (Linear/Notion 스타일)
- **요약 카드**: 좌측 컬러 보더 (navy/gold), 숫자 36px 확대
- **분류 탭**: 밑줄 → pill 스타일 (회색 배경 + 흰색 활성 탭)
- **테이블**: 행 간격 확대, 헤더 uppercase, 블루 틴트 호버
- **버튼**: 높이 36px, box-shadow, hover lift 효과
- **모달**: 밝은 헤더, 16px radius, backdrop-filter blur
- **상세 페이지**: 헤더 그라데이션, 스텝바 ring shadow
- **태그**: 패딩/radius 확대, 굵기 강화
- **패널 타이틀**: Georgia → 시스템 폰트, 14px 700
- **마이크로인터랙션**: 입력 포커스 ring, D-day hover 슬라이드, 토스트 그림자

### v1.2 — JSONBin.io 클라우드 데이터 연동 (`583164d` ~ `e8b70de`)
- JSONBIN 객체: init/load/push/createBin — 전체 CRUD
- save() 함수에 1.5초 debounce 클라우드 push 추가
- 페이지 로드 시 localStorage 먼저 렌더 → JSONBin 비동기 갱신
- Bin ID 없으면 자동 생성 후 localStorage에 저장
- 사이드바 데이터 패널에 API Key / Bin ID 입력 UI
- Bin ID 📋 복사 버튼 (다른 기기 연결용)
- JSONBin 설정 UI 레이아웃 모바일/PC 잘림 수정

### v1.3 — 모바일 사이드바 개선 (`35068f8` ~ `a8b62ea`)
- 사이드바: display:none → fixed 오버레이 슬라이드 (280px, 반투명 배경)
- 하단 탭에 설정(☰) 버튼 추가, 더보기→아카이브(📦)로 변경
- 오버레이 배경 클릭 시 사이드바 닫기
- mini 모드 footer 텍스트/알림 잘림 수정
- nav-item, 로고, footer 텍스트 줄바꿈 방지 (white-space:nowrap)

---

## 2. 향후 개선 계획

### 🔴 P0 — 핵심 (데이터 안정성·보안)

| 항목 | 설명 |
|------|------|
| **데이터 충돌 해결** | 다중 기기 동시 편집 시 JSONBin 데이터 충돌 — updatedAt 타임스탬프 비교 후 병합/경고 로직 필요 |
| **API Key 보안** | JSONBin API Key가 브라우저 localStorage에 평문 저장됨 — Vercel API Route 프록시 또는 환경변수 방식 전환 필요 |
| **자동 백업** | JSONBin 장애 대비 주기적 localStorage 스냅샷 또는 자동 JSON 백업 |
| **인증 강화** | 현재 PIN 간이 인증은 보안 취약 — SSO/OAuth 연동 또는 최소한 비밀번호 해시 저장 검토 |

### 🟠 P1 — 중요 (기능 완성도)

| 항목 | 설명 |
|------|------|
| **SharePoint 실연동** | SETUP.md 기반 SP REST API 연동 구현 — CONFIG.OFFLINE_MODE=false 시 SP Lists 읽기/쓰기 |
| **Power Automate Flow 연결** | F-01(제출), F-06(임시저장), F-07(이의신청) HTTP 트리거 실연동 |
| **AI 기능 활성화** | Anthropic API Key 연동 → 사건 요약, 파일 에이전트, 글로벌 챗봇 실동작 확인 |
| **오프라인→온라인 동기화** | 네트워크 끊김 시 localStorage에 쌓고, 복구 시 자동 push하는 큐 메커니즘 |
| **사건 검색 고도화** | 현재 키워드 검색 → 필터 조합(분류+상태+담당자+기간) 동시 적용, URL 파라미터 연동 |

### 🟡 P2 — 개선 (UX·편의성)

| 항목 | 설명 |
|------|------|
| **다크 모드** | CSS 변수 기반으로 다크 테마 추가 (prefers-color-scheme 자동 감지) |
| **드래그&드롭 칸반** | 사건 목록 칸반 뷰에서 상태 변경을 드래그로 처리 |
| **알림 Push** | 기일 D-3 알림을 브라우저 Notification API 또는 이메일로 발송 |
| **대시보드 커스터마이징** | 위젯 순서/표시 여부를 사용자별로 설정 |
| **첨부파일 실제 업로드** | 현재 파일명/URL만 저장 → SP Document Library 또는 클라우드 스토리지 연동 |
| **보고서 PDF 내보내기** | 현재 인쇄만 가능 → html2pdf.js 등으로 직접 PDF 생성 |
| **다국어 지원** | 현재 한국어 하드코딩 → i18n 구조로 영문 지원 가능하도록 |

### 🟢 P3 — 장기 (아키텍처)

| 항목 | 설명 |
|------|------|
| **파일 분리** | 단일 HTML → CSS/JS 분리, 모듈화 (ES Modules) |
| **프레임워크 전환** | 규모 확장 시 React/Vue + Vite로 마이그레이션 검토 |
| **백엔드 도입** | JSONBin → Supabase/Firebase 등 실제 DB로 전환, 인증/권한 서버 사이드 처리 |
| **CI/CD** | GitHub Actions → Vercel 자동 배포, PR 미리보기 환경 |
| **테스트** | 주요 함수(save, dday, genNum 등) 단위 테스트 추가 |

---

## 3. 기술 스택 현황

| 구분 | 현재 | 목표 |
|------|------|------|
| **프론트엔드** | 단일 HTML (Vanilla JS) | 유지 또는 React/Vue |
| **데이터 저장** | localStorage + JSONBin.io | SharePoint Lists 또는 Supabase |
| **차트** | Chart.js 4.4.0 (CDN) | 유지 |
| **내보내기** | PptxGenJS (CDN) | + html2pdf.js |
| **AI** | Anthropic API (클라이언트 직접 호출) | Vercel API Route 프록시 |
| **배포** | 로컬 파일 / Vercel | Vercel (GitHub 연동) |
| **인증** | PIN 간이 (localStorage) | SSO/OAuth |
