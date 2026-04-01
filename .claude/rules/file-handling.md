---
description: 이 프로젝트에서 파일을 읽고 쓸 때 적용되는 규칙
globs: ["references/**", "templates/**", "outputs/**"]
---

# 파일 처리 규칙

## references/ 폴더
- 읽기 전용 참고자료. 절대 수정하지 않는다.
- 지원 형식: PDF, PPTX, XLSX, DOCX, HWPX
- HWP 바이너리는 파싱 불가 — HWPX 변환 요청

## templates/ 폴더
- 형식 스키마 추출용 HWPX 파일
- 내용이 아닌 **계층 구조 기호**(ㅇ, -, 1), * 등)만 분석
- 읽기 전용

## outputs/ 폴더
- 생성된 Word 파일(.docx) 저장 위치
- 파일명 규칙: `표_YYYY-MM-DD.docx`
- Python으로 직접 생성할 경우 python-docx 사용

## HWPX 파싱 방법 (Python)
```python
import zipfile, re

with zipfile.ZipFile('파일.hwpx') as z:
    # 텍스트 추출
    xml = z.read('Contents/section0.xml').decode('utf-8')
    paragraphs = re.findall(r'<hp:p\b[^>]*>(.*?)</hp:p>', xml, re.DOTALL)
    texts = [re.sub(r'<[^>]+>', '', p).strip() for p in paragraphs]
    texts = [t for t in texts if t]
```

## 형식 스키마 인식 기호
| 기호 | 계층 | 설명 |
|------|------|------|
| `바.` `가.` 등 | 1단계 | 한글 자모 + 점, 최상위 |
| `ㅇ ` | 2단계 | 중분류 |
| `- ` | 3단계 | 설명/항목 |
| `1) ` | 4단계 | 번호 항목 |
| `Ⅰ. ` | 5단계 | 로마자 소제목 |
| `* ` | 6단계 | 세부 내용 |
