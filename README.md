# GMP MVP 하네스 템플릿 v2 (골든 스캐폴드, Codex 전용, FDS 작성 모드 포함)

제약업계 비개발자 교육생이 URS(.docx) 하나로 **60분 안에** Next.js + Google Sheets + Vercel 기반 GMP MVP 웹앱을 멀티에이전트 병렬로 생성하고,
이어서 개발 서버 QA, 수정 요청, 문서 초안, 커밋 준비까지 진행하는 바이브코딩 교육용 템플릿이다.
이 폴더는 **Codex CLI 전용** 판이다(Claude Code와 Antigravity CLI 조는 `05 Harness`). Codex는 `AGENTS.md`를 자동으로 읽고, `agents/`의 정의 파일 전문을 서브에이전트 프롬프트에 넣어
분석 3개, 구현 4개를 병렬로 돌린다(서브에이전트를 쓸 수 없으면 순차 처리). 원샷 빌드는 한 턴에 끝내며, 푸시와 배포는 AI가 아니라 사람이 한다.

이 하네스 하나로 3종 시스템(장비사용기록서 전자로그북, 세척밸리데이션 관리, 실험실 재고관리)을 만든다. 시스템, 역할, 계정의 차이는
URS(KPBMA-EDU-001-URS 양식의 Section 2, 6, 7.1)가 정하고, 하네스는 특정 시스템을 가정하지 않는다.
화면은 `design.md`(한국제약바이오협회 CI와 교육 포털 디자인)를 따른다.

> 이 README는 템플릿 설명용이다. 원샷 빌드가 끝나면 builder-d4가 생성한 앱의 README로 교체된다.

## 구성

| 경로 | 내용 |
|---|---|
| `AGENTS.md` | 하네스 마스터 프롬프트 (Codex가 자동으로 읽음). 원샷 빌드 STEP 0~4(60분), 수정 요청 모드, 커밋 준비, 언어와 보고 형식, Codex 서브에이전트 규칙 |
| `CLAUDE.md` | `AGENTS.md`를 가리키는 안내문 |
| `design.md` | 디자인 가이드: 색과 글꼴 토큰, 화면 골격, 부품 규격, 로그인 화면, 출력물, CI 사용 규칙 |
| `docs/FDS_GUIDE.md`, `docs/FDS_TEMPLATE.md`, `scripts/build-fds.mjs` | FDS 작성 모드: 구현 완료 후 AI가 현재 상태를 고정 목차(FDS 양식 1 ~ 6장)로 쓰고 `npm run fds`가 검증(FS/DS ↔ URS 연결)과 Word 문서(.docx) 생성 |
| `docs/DRAFTS.md` | 문서 초안 지원(DQ, IOQ, RTM) 지침 |
| `agents/` | 서브에이전트 7종 정의 (analyzer-a/b/c, builder-d1~d4). Codex가 서브에이전트 프롬프트에 전문을 넣어 쓴다 |
| `.gitattributes`, `.editorconfig` | LF 줄바꿈 고정 (Codex apply_patch 실패 방지) |
| `app/globals.css` | 디자인 토큰(@theme), 로그인 배경, 인쇄 스타일 |
| `app/layout.tsx`, `components/SideNav.tsx` | 화면 골격: 네이비 상단바(CI, 시스템명, 역할 배지), 사이드 메뉴, 풋터 고지 |
| `components/ui.tsx` | 공용 부품: PageTitle, Card, Kpi, Button, Field, 입력, Table, StatusBadge, NoticeBox, Modal, Banner |
| `lib/brand.ts` | 시스템 제목(CSV실습과정 N조 시스템명), CI 경로, 풋터 문구, 초기 비밀번호 상수 |
| `lib/sheets.ts` | Google Sheets 접근 규격 코드(getRows, appendRow, updateRowById, ensureTab). **수정 금지** |
| `docs/SHEETS_SPEC.md`, `scripts/check-sheets.mjs` | 규격 코드의 사용 규칙과 원문, 무결성 확인(`npm run check:sheets`)과 복원(`-- --restore`) |
| `scripts/check-commit.mjs`, `.gitignore` | 커밋 전 점검(`npm run check:commit`): 비밀 파일, 빌드 산출물, 과대 파일 차단. 산출물 md 는 함께 커밋 |
| `lib/auth.ts` `lib/audit.ts` `lib/kst.ts` | 세션, 감사추적, KST 표기 헬퍼 |
| `app/login` + `/api/login` `/api/logout` `/api/seed` | 계정 선택 로그인 화면과 시드 스켈레톤 |
| `public/kpbma-ci.png`, `public/kpbma-ci-white.png`, `app/icon.png` | 한국제약바이오협회 CI(파랑, 흰색)와 탭 아이콘 |
| `scripts/convert-urs.mjs` | URS .docx → `docs/urs/*.md` 변환기 (표 보존, 조항 ID 카운트) |
| `scripts/new-project.mjs` | 템플릿을 새 프로젝트 폴더로 복제 (`npm run new -- <이름>`) |
| `docs/urs/` | 조별 URS를 넣는 곳 (.docx 또는 변환된 .md) |

## 빠른 시작

```bash
npm install
cp .env.example .env.local   # 3종 값 입력 (서비스 계정 이메일, 키, 시트 ID)
npm run dev
```

브라우저: `http://localhost:3000/api/seed` 호출(USERS, AUDIT 탭과 계정 생성) → `/login`에서 계정을 고르면 비밀번호 `1234`가 자동 입력된다.

새 프로젝트를 만들 때(교육 후 자기 환경): 템플릿 폴더에서 `npm run new -- my-project` → `cd ../my-project` → `npm install`.

## 실습 흐름

1. URS `.docx`를 `docs/urs/`에 넣는다.
2. Codex(전체 자동 모드)에 입력: **"하네스 절차대로 URS MVP 빌드를 시작해."** 턴이 중간에 끝나면 `계속`을 입력한다.
3. 60분 후 산출물: 동작 앱 + `IMPLEMENTED.md`(RTM 입력, 미구현 사유) + `PLAN.md` + `DECISIONS.md` + `CHANGELOG.md`. 완료 보고는 한국어 보고용 문체.
4. 개발 서버에서 QA → `수정 요청: …` 으로 고치기 (반복, 1일차) → `FDS를 작성해줘` (2일차, Word 문서 생성) → DQ Matrix → 비밀번호 변경 → `커밋해` (푸시는 사람이 터미널에서 `git push`, Vercel 자동 배포)
   → `IOQ 시나리오 초안을 만들어줘` → IOQ 수행 (배포 URL) → `RTM 입력을 정리해줘` → VSR.

## 계정

계정 ID, 이름, 역할 코드는 URS §6.1과 §7.6을 그대로 따른다(템플릿의 admin, user, reviewer는 URS를 읽기 전의 자리표시다).
첫 빌드에서는 모든 계정의 비밀번호가 `1234`로 통일된다(QA 편의, `lib/brand.ts`의 INITIAL_PASSWORD). 배포 전에 비밀번호 변경 화면에서 바꾼다.
로그인 화면은 계정 선택 드롭다운, 아이디, 비밀번호 순서다. 드롭다운에서 계정을 고르면 아이디가 채워지고, 비밀번호가 초기값인 계정은 1234도 함께 채워진다(바꾼 계정은 직접 입력).
시스템 제목은 "CSV실습과정 [조번호]조 [시스템명]" 형식으로 고정되며 `lib/brand.ts`의 TEAM_NO와 SYSTEM_NAME만 URS대로 바뀐다.

## 문서 작성 규칙

하네스가 만드는 산출물과 보고는 한국어 보고용 문체("~하였습니다", "~입니다")로, 진행 중 메시지는 영어로 나온다.
방점(·)과 대시(—)를 쓰지 않고 콤마로 나열하며, "데이터 완전성", "규격서" 용어를 쓴다(AGENTS.md 2절).
