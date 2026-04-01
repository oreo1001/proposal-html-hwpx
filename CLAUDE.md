# 제안서 스튜디오 — 프로젝트 컨텍스트

## 목적
한글(HWP/HWPX) 제안서 작성 워크플로우를 자동화한다.  
참고자료(PDF·PPTX·XLSX·DOCX·HWPX) → Claude 제안서 생성(마크다운) → HWP 붙여넣기용 plain text 변환.

## 핵심 워크플로우
```
references/  →  [proposal_studio.html]  →  마크다운 생성  →  형식 변환  →  HWP 붙여넣기
(참고자료)        Claude API 호출           (LLM 작성)        (LLM/규칙)    plain text 복사
                                                                           + 표 → outputs/ Word
```

## 폴더 구조
```
proposal-md-hwpx/
├── references/          참고자료 (PDF, PPTX, XLSX, DOCX, HWPX) — 제안서 내용 소스
├── templates/           형식 템플릿 HWPX 파일 — 계층 구조 스키마 추출용
├── outputs/             생성된 출력물 (표 Word 파일 등)
├── proposal_studio.html 메인 통합 도구 ← 주로 사용
├── claude_md_converter.html  기존 변환기 (규칙 기반만)
├── claude_md_analyzer.html   기존 분석기 (규칙 기반만)
└── CLAUDE.md            이 파일
```

## 주요 도구

### proposal_studio.html (메인)
탭 구조:
- **① 참고자료 & 설정**: 파일 업로드 → 텍스트 추출 → 질문 입력 → Claude API로 제안서 생성
- **② 마크다운 편집**: 생성된 마크다운 확인/수정
- **③ 변환 출력**: HWP 붙여넣기용 plain text + 표 Word 다운로드

### 형식 스키마 (핵심 개념)
한글 제안서의 계층 구조는 XML 스타일이 아닌 **텍스트 기호**로 인코딩됨:
```
바. 대분류
 ㅇ 중분류
  - 설명
    1) 번호항목
    Ⅰ. 소제목
     * 세부내용
```
HWPX 파일에서 이 패턴을 추출 → LLM이 분석 → 형식 스키마 자동 생성 가능.

### LLM 변환 vs 규칙 기반
- **LLM 변환** (권장): Claude API가 마크다운 depth 분석 후 지정 형식에 맞게 변환. 유연.
- **규칙 기반**: `바. ㅇ - 1) Ⅰ. *` 6단계 고정 규칙. API 없이 즉시 변환.

## 파일 처리 방식
| 형식 | 추출 방법 |
|------|-----------|
| PPTX | JSZip → `ppt/slides/slide*.xml` XML 파싱 |
| DOCX | JSZip → `word/document.xml` XML 파싱 |
| HWPX | JSZip → `Contents/section0.xml` XML 파싱 |
| XLSX | JSZip → `xl/sharedStrings.xml` + `xl/worksheets/*.xml` |
| PDF  | PDF.js (CDN 동적 로드) |
| HWP  | 바이너리 형식 — 지원 불가, HWPX 변환 필요 |

## 표 처리
표는 HWP에 plain text로 붙여넣기 불가 → `★ [표 삽입 — Word 파일 참조]` 플레이스홀더로 대체.

### 표 자동 판단 규칙
제안서 작성 시 참고자료를 분석하여 **표가 유효한지 자동 판단**한다:
- 비교 항목 3개 이상 → 비교표
- 기능/구성요소 나열 → 구성표
- 수치 지표 2개 이상 → 기대효과표
- 단계/절차 나열 → 단계표

표가 필요하다고 판단되면:
1. 본문에 `  ★ [표 삽입 — outputs/표_YYYY-MM-DD.docx 참조]` 플레이스홀더 삽입
2. `outputs/표_YYYY-MM-DD.docx` 를 python-docx로 **자동 생성**

## Claude API 호출 패턴
```javascript
// 모든 API 호출은 callClaude(systemPrompt, userContent) 함수 사용
// header: 'anthropic-dangerous-direct-browser-access': 'true'  (브라우저 직접 호출)
// API key: localStorage에 저장 ('_studio_apikey')
```

## 코드 수정 시 주의사항
- `ruleConvert()` 함수는 기존 `claude_md_converter.html`과 동일 로직 유지 (하위호환)
- LLM 변환 실패 시 자동으로 규칙 기반으로 폴백
- 표 Word 생성은 `docx` CDN 라이브러리 사용 (`window.docx`)
